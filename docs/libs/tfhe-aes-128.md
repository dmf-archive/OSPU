# TFHE AES-128

## AES Basics

Advanced Encryption Standard (AES) is a symmetric encryption algorithm that is widely used for secure data encryption. In this section, we introduce the main computation operations of AES, and the compute patterns of each operation. The full details of AES can be found in the Ref \[[1](https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.197.pdf)\].

AES takes a 128-bit plaintext and a secret key as input, and outputs a 128-bit ciphertext. The secret key is an integer of 128 bits, 192 bits, or 256 bits. For simplicity, we only introduce the AES with 128-bit key (AES-128), and the main computation operations are similar for AES-192 and AES-256.

**Layout.** In AES, the 128-bit (i.e. 16 bytes) plaintext is regarded as a 4x4 matrix of 8-bit integers. The layout is column-major order, i.e. the first 4 bytes are the first column, the second 4 bytes are the second column, and so on.

**Key expansion.** Before compuation, the 128-bit input key needs to be expanded into 11 round keys and each round key is 128-bit. The key expansion consists of bitwise `xor` operations and `SubBytes` operations which will be explained below.

**Main operations.** AES contains several repeated rounds of computation, and each round consists of 4 different operations:

1. `AddRoundKey`: Adds the round key to the plaintext. The addition is performed in Galois Field ($2^8$)(See Ref \[[2](https://en.wikipedia.org/wiki/Finite_field)\]), i.e. each bit of the round key is xor'ed with the corresponding bit of the plaintext. In short, `AddRoundKey` performs a series of bitwise `xor` operations.
2. `SubBytes`: Replaces each byte in the plaintext with a different byte according to a lookup table (as known as _S-box_), where plaintext's bytes are treated as indices of the lookup table.
3. `ShiftRows`: Circularly shifts each row of the plaintext to the left by a different amount. The first row remains unchanged, the second row is shifted by 1, the third row is shifted by 2, and the fourth row is shifted by 3. `ShiftRows` only changes the layout of each byte, it does not change the value of each byte.
4. `MixColumns`: Multiplies the plaintext matrix (4x4) by a 4x4 constant matrix. The elementwise multiplication and addition are also performed in Galois Field ($2^8$), which are bitwise `xor` operations in the implementation.

## FHE AES Design

To perform AES operations homomorphically, the core challenge is to implement the `SubBytes` operation. In TFHE, implementing the 8-bit to 8-bit lookup table incurrs a non-trivial overheads, as each 8-bit integer needs to be represented as several ciphertexts (see Ref \[[3](https://docs.zama.ai/tfhe-rs/references/fine-grained-apis/integer/operations)\]).

To address this challenge, instead of indexing the _S-box_ table, we choose to implement the `SubBytes` operation by computing the _S-box_ table on the fly. Specifically, we use the Boyar-Peralta method (see Ref \[[4](https://eprint.iacr.org/2009/191.pdf)\]) to compute the _S-box_, which can compute the _S-box_ table using 32 bitwise `and` and 83 bitwise `xor` operations. An example implementation of Boyar-Peralta method can be found at Ref \[[5](https://www.bearssl.org/gitweb/?p=BearSSL;a=blob;f=src/symcipher/aes_ct.c;h=66776d9e206c92cbbf3fa799c651a3d3652bd75a;hb=HEAD#l29)\]:

```c++
// x0,x1,..,x7 is the bits of the input byte, and x0 is the most significant bit
/*
 * Top linear transformation.
 */
y14 = x3 ^ x5;
y13 = x0 ^ x6;
y9 = x0 ^ x3;
y8 = x0 ^ x5;
t0 = x1 ^ x2;
y1 = t0 ^ x7;
y4 = y1 ^ x3;
y12 = y13 ^ y14;
y2 = y1 ^ x0;
y5 = y1 ^ x6;
y3 = y5 ^ y8;
t1 = x4 ^ y12;
y15 = t1 ^ x5;
y20 = t1 ^ x1;
y6 = y15 ^ x7;
y10 = y15 ^ t0;
y11 = y20 ^ y9;
y7 = x7 ^ y11;
y17 = y10 ^ y11;
y19 = y10 ^ y8;
y16 = t0 ^ y11;
y21 = y13 ^ y16;
y18 = x0 ^ y16;

/*
 * Non-linear section.
 */
t2 = y12 & y15;
t3 = y3 & y6;
t4 = t3 ^ t2;
t5 = y4 & x7;
t6 = t5 ^ t2;
t7 = y13 & y16;
t8 = y5 & y1;
t9 = t8 ^ t7;
t10 = y2 & y7;
t11 = t10 ^ t7;
t12 = y9 & y11;
t13 = y14 & y17;
t14 = t13 ^ t12;
t15 = y8 & y10;
t16 = t15 ^ t12;
t17 = t4 ^ t14;
t18 = t6 ^ t16;
t19 = t9 ^ t14;
t20 = t11 ^ t16;
t21 = t17 ^ y20;
t22 = t18 ^ y19;
t23 = t19 ^ y21;
t24 = t20 ^ y18;

t25 = t21 ^ t22;
t26 = t21 & t23;
t27 = t24 ^ t26;
t28 = t25 & t27;
t29 = t28 ^ t22;
t30 = t23 ^ t24;
t31 = t22 ^ t26;
t32 = t31 & t30;
t33 = t32 ^ t24;
t34 = t23 ^ t33;
t35 = t27 ^ t33;
t36 = t24 & t35;
t37 = t36 ^ t34;
t38 = t27 ^ t36;
t39 = t29 & t38;
t40 = t25 ^ t39;

t41 = t40 ^ t37;
t42 = t29 ^ t33;
t43 = t29 ^ t40;
t44 = t33 ^ t37;
t45 = t42 ^ t41;
z0 = t44 & y15;
z1 = t37 & y6;
z2 = t33 & x7;
z3 = t43 & y16;
z4 = t40 & y1;
z5 = t29 & y7;
z6 = t42 & y11;
z7 = t45 & y17;
z8 = t41 & y10;
z9 = t44 & y12;
z10 = t37 & y3;
z11 = t33 & y4;
z12 = t43 & y13;
z13 = t40 & y5;
z14 = t29 & y2;
z15 = t42 & y9;
z16 = t45 & y14;
z17 = t41 & y8;

/*
 * Bottom linear transformation.
 */
t46 = z15 ^ z16;
t47 = z10 ^ z11;
t48 = z5 ^ z13;
t49 = z9 ^ z10;
t50 = z2 ^ z12;
t51 = z2 ^ z5;
t52 = z7 ^ z8;
t53 = z0 ^ z3;
t54 = z6 ^ z7;
t55 = z16 ^ z17;
t56 = z12 ^ t48;
t57 = t50 ^ t53;
t58 = z4 ^ t46;
t59 = z3 ^ t54;
t60 = t46 ^ t57;
t61 = z14 ^ t57;
t62 = t52 ^ t58;
t63 = t49 ^ t58;
t64 = z4 ^ t59;
t65 = t61 ^ t62;
t66 = z1 ^ t63;
s0 = t59 ^ t63;
s6 = t56 ^ ~t62;
s7 = t48 ^ ~t60;
t67 = t64 ^ t65;
s3 = t53 ^ t66;
s4 = t51 ^ t66;
s5 = t47 ^ t65;
s1 = t64 ^ ~s3;
s2 = t55 ^ ~t67;

// s0,s1,..,s7 is the bits of the output byte, and s0 is the most significant bit
```

Excepting the `SubBytes` operation, other operations, such as `AddRoundKey`, `ShiftRows`, and `MixColumns`, can be implemented as bitwise `xor` operations. The key expansion can also be implemented using bitwise `xor` operations and `SubBytes` operations. In summary, we can implement a fully homomorphic version of the AES algorithm using only bitwise `xor` and bitwise `and` operations.

## Implementation Details

We implement the AES algorithm using the latest [TFHE-rs v0.11.2](https://github.com/zama-ai/tfhe-rs/releases/tag/tfhe-rs-0.11.2) and existing parameter sets. Our goal is to provide a high-throughput FHE AES implementation, which means performing AES encryption on more inputs per unit of time.

**Data representation.** We represent each bit of the input and the key as a ciphertext. The parameter set of ciphertext can both affect the overheads of bitwise operations and the number of bootstraps. To achieve good performance, we choose the shortint with 1 message bit and 2 carry bits (`V0_11_PARAM_MESSAGE_1_CARRY_2_KS_PBS_GAUSSIAN_2M64`) as the ciphertext type.

**Bitwise operations.** The bitwise `xor` operation can be implemented as a simple addition of two ciphertexts.
The bitwise `and` operation is a bit more complex. We implement it using PBS. Specifically, we first merge the two ciphertexts into one ciphertext by shifting one ciphertext's data into its carry bit, and adding the two ciphertext. After merge, we perform a PBS to compute the `and` result. Here is an example implementation of the `and` function:

```rust
fn and(
    l: &Ciphertext,
    r: &Ciphertext,
    sks: &ServerKey,
) -> Ciphertext {
    // shift l's message bit into the highest carry bit
    let t = sks.unchecked_scalar_mul(l, 4);
    // merge l and r into one ciphertext, to ensure correctness, the highest carry bit of r should be zero
    let t = sks.unchecked_add(&t, r);
    // generate lookup table for the `and` operation
    let and_lut = sks.generate_lookup_table(|x: u64| (x >> 2) & (x & 1));
    // apply the lookup table to get the `and` result
    return sks.apply_lookup_table(&t, &and_lut);
}
```

**Bootstrap.** In the implementation, we mannually insert bootstrap operations to reset noise when needed. In each round of AES, we insert bootstrap operations for the entire 128-bit input before each AES computation operation. The op sequence in each round looks like: `Bootstrap`, `SubBytes`, `ShiftRows`, `Bootstrap`, `MixColumns`, `Bootstrap`, `AddRoundKey`.
There is no bootstrap operation within `MixColumns` and `AddRoundKey` because of the parameter set we choose. However, bootstrap operations are needed within `SubBytes` due to the large number of `xor` and `and` operations. In `SubBytes`, we manually insert bootstraps where necessary to ensure the correctness.

**Parallelism.** The key to achieving high-throughput AES is to fully utilize hardware resources and improve CPU utilization. To achieve this, we exploit both internal parallelism within AES and external parallelism outside AES. The internal parallism is achieved by parallelizing each operation in AES. For example, the `SubBytes` operations dominate the overall execution time, and since it is a per-byte operation, computation can be performed on all the 16 bytes in parallel. We use the [rayon](https://docs.rs/rayon/latest/rayon/) library to implement the parallel computation, and below is an example of how to parallelize the `SubBytes` operation:

```rust
// input is a vector of 128 ciphertexts, each ciphertext corresponding to a bit
input.par_chunks_mut(8).for_each(|chunk| {
    // a chunk contains 8 ciphertexts corresponding to a byte
    fhe_sbox(sks, chunk);
});
```

The external parallelism is achieved by parallelizing the AES encryption of multiple inputs. For example, in the counter-mode AES (see Ref \[[6](<https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_(CTR)>)\]), each input can be encrypted independently. To improve the overall throughput, we pre-allocate several thread pools where each thread pool contains 16 threads and each thread pool is used to encrypt one input at a time. The number of thread pools is determined by the number of CPU cores satisfying that the total number of threads of all thread pools is equal to the number of CPU cores. We bind all the threads to different CPU cores to increase cache locality and reduce the system overheads associated with thread management (e.g. thread migration). Below is an example of how to create the thread pools and bind the threads to different CPU cores:

```rust
let mut thread_pools = Vec::new();
let cpus = num_cpus::get();

const THREAD_GROUP_SIZE: usize = 16;
for i in 0..cpus / THREAD_GROUP_SIZE {
    let pool = ThreadPoolBuilder::new()
        .spawn_handler(move |thread| {
            std::thread::spawn(move || {
                let group_offset = i * THREAD_GROUP_SIZE;
                let cores = vec![thread.index() + group_offset];
                set_thread_affinity(&cores).expect("Failed to set thread affinity");
                thread.run()
            });
            Ok(())
        })
        .num_threads(THREAD_GROUP_SIZE)
        .build()
        .unwrap();
    thread_pools.push(pool);
}
```

## Usage

In our implementation, we mainly expose two apis: `fhe_key_expansion` and `fhe_aes_encrypt`. To use these apis, one should first encrpyte each bit of the input and the key into a ciphertext, and then use `fhe_key_expansion` to perform the AES key expansion, and finally use `fhe_aes_encrypt` to perform the AES encryption. Below is an example of how to use these two apis to do AES encryption with FHE-encrypted input and FHE-encrypted key:

```rust
use tfhe::shortint::parameters::classic::gaussian::p_fail_2_minus_64::ks_pbs::*;
use tfhe::shortint::prelude::*;

pub fn fhe_decrypt_data(cks: &ClientKey, encrypted_bits: &Vec<Ciphertext>) -> u128 {
    let mut decrypted_bits = Vec::new();
    for encrypted_bit in encrypted_bits {
        decrypted_bits.push(cks.decrypt(encrypted_bit));
    }
    let mut decrypted_data = 0u128;
    for i in 0..128 {
        decrypted_data |= (decrypted_bits[i] as u128) << (127 - i);
    }
    decrypted_data
}

pub fn fhe_encrypt_data(cks: &ClientKey, data: u128) -> Vec<Ciphertext> {
    let mut encrypted_bits = Vec::new();
    for i in (0..128).rev() {
        let bit = (data >> i) & 1 == 1;
        let encrypted_bit = cks.encrypt(bit as u64);
        encrypted_bits.push(encrypted_bit);
    }
    encrypted_bits
}

mod fhe_aes;
fn main() {
    // test vector from https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38a.pdf
    let key = 0x2b7e151628aed2a6abf7158809cf4f3cu128;
    let input = 0xf0f1f2f3f4f5f6f7f8f9fafbfcfdfeffu128;
    let expect = "ec8cdf7398607cb0f2d21675ea9ea1e4";

    // Step 1: generate keys and encrypt the key and input
    let (cks, sks) = gen_keys(V0_11_PARAM_MESSAGE_1_CARRY_2_KS_PBS_GAUSSIAN_2M64);
    let mut encrypted_key = fhe_encrypt_data(&cks, key);
    let mut encrypted_input = fhe_encrypt_data(&cks, input);

    // Step 2: perform AES key expansion
    let encrypted_round_keys = fhe_aes::keyexp::fhe_key_expansion(&sks, &mut encrypted_key);

    // Step 3: perform AES encryption
    fhe_aes::aes::fhe_aes_encrypt(&sks, &mut encrypted_input, &encrypted_round_keys);

    // Verify the result
    let aes_result =
        hex::encode(fhe_decrypt_data(&cks, &encrypted_input).to_be_bytes());
    println!("Expected result: {expect}");
    println!("FHE AES result : {aes_result}");
    assert_eq!(aes_result, expect);

    println!("AES test passed!");
}
```

We also provide three test cases in the `src` directory: `simple_aes_test.rs`, `test_vector_test.rs` and `random_test.rs`. Readers can refer to these test cases to verify the correctness of the implementation with [standard test vectors](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38a.pdf) and randomly generated test cases. Here is an example of how to run these tests:

```bash
# run the aes test with standard test vectors
cargo run --release --bin test_vector_test
# run the aes test with randomly generated test cases
cargo run --release --bin random_test
```

## Benchmarking

As described in [Implement a Fully Homomorphic Version of the AES-128 Cryptosystem using TFHE-rs](https://github.com/zama-ai/bounty-program/issues/135),
we provide a benchmarking tool in `src/main.rs` that implements the counter-mode AES. It takes 3 inputs: `--number-of-outputs`, `--iv` and `--key`. When running, it can generate the requested number of AES values **homomorphically** (both the counter increment and the AES encryption are computed homomorphically), and verify the correctness by comparing the result with a cleartext implementation.

Here is an example of how to run the benchmarking tool:

```bash
# prepare the rust environment
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs  > sh.rustup.rs
bash sh.rustup.rs -y --default-toolchain nightly-2024-11-29
. "$HOME/.cargo/env"

# build the project
cargo build --release

# run the benchmark, takes about ~8 minutes on a vm with 192 vcpus of 4th gen AMD EPYC processors
cargo run --release -- --key 0x2b7e151628aed2a6abf7158809cf4f3c --iv 0xf0f1f2f3f4f5f6f7f8f9fafbfcfdfeff --number-of-outputs 192
```

To get a reasonable benchmarking result, the `--number-of-outputs` should be a multiple of the number of thread pools (`number_of_cpus / 16`, e.g., 12 for a machine with 192 cpus).

## References

1. AES specification. <https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197-upd1.pdf>
2. Finite field. <https://en.wikipedia.org/wiki/Finite_field>
3. TFHE integer representation. <https://docs.zama.ai/tfhe-rs/references/fine-grained-apis/integer/operations>
4. Boyar-Peralta method. <https://eprint.iacr.org/2009/191.pdf>
5. An implementation of Boyar-Peralta method. <https://www.bearssl.org/gitweb/?p=BearSSL;a=blob;f=src/symcipher/aes_ct.c;h=66776d9e206c92cbbf3fa799c651a3d3652bd75a;hb=HEAD#l29>
6. Counter-mode AES. <https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_(CTR)>
