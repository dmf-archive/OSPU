TFHE-rs: A (Practical) Handbook

## First Edition

### Mathieu Ballandras, Mayeul de Bellabre, Loris Bergerat, Charlotte Bonte, Carl

### Bootland, Benjamin R. Curtis, Jad Khatib, Jakub Klemsa, Arthur Meyre

### Thomas Montaigu, Jean-Baptiste Orfila*, Nicolas Sarlin, Samuel Tap, David

### Testé

### Zama

### <firstname.lastname@zama.ai>

### *<jb.orfila@zama.ai>

### February 27, 2025

```
Abstract
This document presents TFHE-rs, Zama’s Rust library implementing an enhanced variant of
TFHE. TFHE is a well-known fully homomorphic encryption (FHE) scheme that enables fast
bootstrapping for small messages. TFHE-rs introduces advanced encoding techniques to support
larger message sizes and employs ciphertext compression to mitigate expansion overhead. To
provide a complete foundation, we first recall the necessary cryptographic concepts required to
understand TFHE. We then present a comprehensive overview of TFHE-rs, detailing part of its
core algorithms and the methods used to construct homomorphic integer ciphertexts—capable of
encrypting, for example, a u64—thus significantly expanding the range of possible message spaces.
The document systematically presents the operations available on these homomorphic integer
types, integrating references to the TFHE-rs API. Additionally, we offer extensive benchmarking
results that justify the design choices made in the library, evaluating the latency of all integer
operations introduced in this work. Finally, we present recommended parameter sets that ensure
secure and efficient execution of these algorithms across a range of failure probabilities.
```

## CONTENTS CONTENTS

- 1 Introduction Contents
  - 1.1 Document Goals
  - 1.2 Document Structure
- 2 The TFHE scheme
  - 2.1 Hard Lattice Problems: the LWE and GLWE Assumptions
  - 2.2 The Morphology of FHE Ciphertexts.
    - 2.2.1 LWE, RLWE & GLWE Ciphertexts
    - 2.2.2 Lev, RLev & GLev Ciphertexts.
    - 2.2.3 GSW, RGSW & GGSW Ciphertexts.
  - 2.3 Encoding Booleans and Small Integers
    - 2.3.1 Carry Space.
    - 2.3.2 Boolean Encoding
    - 2.3.3 Degree of Fullness
  - 2.4 Simple Operations
    - 2.4.1 Linear Operations
    - 2.4.2 Sample Extract.
    - 2.4.3 Public Key Encryption.
  - 2.5 Key Switching
    - 2.5.1 Decomposition
    - 2.5.2 LWE and GLWE Key Switch
    - 2.5.3 Packing Key Switch
  - 2.6 Building Blocks of the PBS
    - 2.6.1 Modulus Switch
    - 2.6.2 External Product.
    - 2.6.3 CMUX
    - 2.6.4 Blind Rotate
  - 2.7 PBS & Its Variants.
    - 2.7.1 Programmable Bootstrap (PBS)
    - 2.7.2 Multi-Bit Blind Rotate
    - 2.7.3 ManyLUT PBS.
    - 2.7.4 Alternative Approaches to the Blind Rotate.
    - 2.7.5 Atomic Patterns
- 3 Arithmetic over Large Homomorphic Integers
  - 3.1 Radix Encoding of Large Homomorphic Integers
  - 3.2 Notations and Assumptions
    - 3.2.1 Example: Addition.
    - 3.2.2 Input/Output Assumptions
    - 3.2.3 Toolbox
  - 3.3 Bitwise Operations
    - 3.3.1 Ciphertext-Ciphertext Addition.
    - 3.3.2 Plaintext-Ciphertext Addition.
  - 3.4 Equality.
    - 3.4.1 Ciphertext-Ciphertext Equality.
    - 3.4.2 Plaintext-Ciphertext Equality.
  - 3.5 Carry Propagation
    - 3.5.1 Step 1: Groups and Block States.
    - 3.5.2 Step 2: Propagation States and Group Carries CONTENTS CONTENTS
    - 3.5.3 Step 3: Resolving the Group Carries
    - 3.5.4 Step 4: Final Step
    - 3.5.5 Full-Fledged Carry Propagation.
    - 3.5.6 Complexity of Carry Propagation.
  - 3.6 Addition.
    - 3.6.1 Ciphertext-Ciphertext Addition.
    - 3.6.2 Plaintext-Ciphertext Addition.
  - 3.7 Negation.
  - 3.8 Subtraction
    - 3.8.1 Plaintext-Ciphertext Subtraction.
  - 3.9 Block Shifts and Rotations
    - 3.9.1 Plaintext-Ciphertext Block Shift & Rotation
    - 3.9.2 Ciphertext-Ciphertext Block Shift & Rotation
  - 3.10 Bit Shifts and Rotations.
    - 3.10.1 Plaintext-Ciphertext Bit Shift & Rotation.
    - 3.10.2 Ciphertext-Ciphertext Bit Shift & Rotation.
  - 3.11 Absolute Value
  - 3.12 Sum
  - 3.13 Multiplication.
    - 3.13.1 Ciphertext-Ciphertext Multiplication.
    - 3.13.2 Plaintext-Ciphertext Multiplication
  - 3.14 Detecting Overflows
    - 3.14.1 Addition.
    - 3.14.2 Subtraction
    - 3.14.3 Multiplication.
  - 3.15 Conditionals.
  - 3.16 Comparisons
  - 3.17 Division and Modulo.
    - 3.17.1 Ciphertext-Ciphertext Division & Modulo.
    - 3.17.2 Plaintext-Ciphertext Division & Modulo.
- 4 LWE Ciphertext Compression
  - 4.1 Input Ciphertext Compression
  - 4.2 Intermediate Ciphertext Compression
- 5 Benchmarks and Parameters
  - 5.1 Bootstrapping Related Benchmarks.
    - 5.1.1 Classical Bootstrapping Performance.
    - 5.1.2 MultiBit Bootstrapping Performance.
  - 5.2 Integer Benchmarks
    - 5.2.1 Choosing the Best Precision and Bootstrapping Algorithm.
    - 5.2.2 Plaintext-Ciphertext Operation Benchmarks
    - 5.2.3 Ciphertext-Ciphertext Operation Benchmark
  - 5.3 Compression and Decompression Benchmarks
  - 5.4 Parameters
- 6 Find Out More

## 1 Introduction Contents

## 1 Introduction

Fully homomorphic encryption (FHE) enables arbitrary computations on encrypted data without re-
quiring decryption. Because the data remains encrypted during computation, a data owner can securely
outsource processing to a third party without concern that it will be misused, shared with unautho-
rized parties, or exposed in a security breach. The first encryption scheme that could be considered
_fully homomorphic_ was introduced by Gentry [Gen09]. To achieve this, Gentry introduced the notion
of _bootstrapping_. Bootstrapping is an operation that, informally, refers to homomorphically evaluating
the decryption circuit to remove one encryption layer while simultaneously adding another.
This process is necessary because each ciphertext contains some noise, which increases with each
computation. If left unchecked, the noise eventually grows too large, resulting in incorrect decryp-
tion. Bootstrapping replaces a highly noisy encryption layer with a less noisy one, allowing further
computations while preventing excessive noise accumulation. All practical FHE schemes follow this
same blueprint: ciphertexts inherently contain noise, and bootstrapping is used during computation
to allow more operations to be realized. This applies to the TFHE scheme as well [CGGI16a].
TFHE is a fully homomorphic encryption (FHE) scheme based on variants of the Learning with
Errors (LWE) problem. Without going into details, an LWE-based ciphertext consists of a vector of
uniformly chosen modular integers (the mask) and an additional integer (the body), which combines
the secret key, the message, the mask, and some noise. Using these ciphertexts, TFHE supports the
evaluation of small lookup tables via the so-called programmable bootstrapping (PBS) operation. The
efficiency of PBS is one of the defining features of TFHE, enabling efficient computation of functions
over ciphertexts. However, TFHE has notable drawbacks, chief among them being its limited support
for small message sizes (typically 1 to 8 bits) and its restriction to operating on a single message at a
time. As a result, a single PBS operation alone offers limited utility. Another drawback is that TFHE
ciphertexts have a significantly larger expansion factor—the ratio between ciphertext size and plaintext
size—compared to other FHE schemes that can pack and process multiple data values simultaneously.
TFHE-rs is a Rust-based library developed by Zama that implements its variant of the TFHE
scheme. Zama’s TFHE variant is designed to address these drawbacks. Specifically, it supports
encrypted integer arithmetic for _large integers_ , such asu16, u32, u64. At a high level, this is
achieved by grouping individual LWE ciphertexts, each encrypting a small number of bits, and using
an encoding that preserves the structure of the large integer. Using this approach as a building block,
the library provides various algorithms for performing full integer arithmetic (and more) on encrypted
data. A graphical representation of this concept is shown in Figs. 1 and 2.

Figure 1:Representation of an LWE ciphertext used in the TFHE scheme. Here, the yellow blocks
represent the random mask terms moduloqused in encryption (denotedaiin future sections), and the
grey block represents the body term which contains the encrypted message bits (denotedbin future
sections). The number of yellow blocks (denotednin future sections), and the chosen modulusqare
directly related to performance and security. Typically, these ciphertexts encrypt a small integer, say
1-8 bits, supporting up tou8and enable encrypted arithmetic over small values.

##### 1 INTRODUCTION

Figure 2:Representation of an integer ciphertext as constructed in the TFHE-rs library from a number
of LWE ciphertexts. In order to construct these ciphertexts, we need several LWE ciphertexts as
presented in Fig. 1. The number of LWE ciphertexts required to build an integer is directly related
to performance. These integer ciphertexts can encrypt integers of size up tou256on which we can
perform full integer arithmetic.

One of the main goals of TFHE-rs is to abstract the complexity of cryptography, ensuring that
users do not need to worry about parameters and thus the security, correctness, or robustness, as
these aspects are handled internally by the library. As a result, TFHE-rs abstracts the complexity
of interacting with ciphertext types, providing a seamless user experience. Specifically, users interact
directly with encrypted values, such asu64, and can perform various encrypted operations (e.g.,
addition or comparison) without handling the individual ’chunks’ composing the integer ciphertext.
In Listing 1 , the twou8values,clear_1andclear_2, are generated, encrypted, and then
subjected to a variety of operations, all of which are described in later sections of this document. Each
ciphertext consists of four LWE ciphertexts (blocks), but this complexity is entirely hidden from the
user. The interaction between the four LWE ciphertexts formingct_1and the four LWE ciphertexts
formingct_2is handled entirely by the TFHE-rs API.

```
use tfhe::prelude::*;
use tfhe::{generate_keys, set_server_key, ConfigBuilder, FheUint64};
fn main() {
let config = ConfigBuilder::default().build();
// Evaluation keys generated on the client-side
let (client_key, server_key) = generate_keys(config);
set_server_key(server_key);
// Plaintext values generated on the client-side
let clear_1 = 35 u64 ;
let clear_2 = 7 u64 ;
// Client-side Encryption
let ct_1 = FheUint64::encrypt(clear_1, &client_key);
let ct_2 = FheUint64::encrypt(clear_2, &client_key);
// Take a reference to avoid moving data when doing the computation
let ct_1 = &ct_1;
let ct_2 = &ct_2;
// Server-side computation using Rust's built-in operators
let add = ct_1 + ct_2;
let mul = ct_1 * ct_2;
let shl = ct_1 << ct_2;
// Client-side decryption and verification of proper execution
,! (identical for other operations)
let decrypted_add: u64 = add.decrypt(&client_key);
}
Listing 1: Example of the TFHE-rs high-level API
```

1.1 Document Goals 1 INTRODUCTION

The individual LWE ciphertexts in the above example have a large expansion factor (in the tens
of thousands), meaning that homomorphic integer types are significantly larger than their plaintext
equivalents. Therefore, in applications requiring the storage of many such integers, a compression
algorithm for these large LWE ciphertexts is beneficial. The TFHE-rs library includes a compression
algorithm that can pack multiple homomorphic integers (each consisting of several LWE ciphertexts)
into a _single_ GLWE ciphertext, significantly reducing the ciphertext expansion factor.

Figure 3:Representation of several integer ciphertexts being compressed into a single GLWE ciphertext

### 1.1 Document Goals

The goal of this document is to give a thorough overview of the TFHE-rs library and the algorithms
contained within it. In particular, we outline the suite of algorithms available in the TFHE-rs library
for both small message sizes (as represented in Fig. 1 ) and large message sizes (as represented in Fig. 2 ).
Furthermore, this document is meant for developers and researchers using the TFHE-rs library to:

```
1.Understand the algorithms used in the TFHE scheme.
```

```
2.Introduce the integer arithmetic implemented in the TFHE-rs library.
```

```
3.Provide all the necessary details to build a new backend for the TFHE-rs library (for example,
for specific hardware) and then benchmark it against other backends.
```

We focus in particular on describing operations, and the resulting algorithms that are implemented.
The document does not cover correctness or noise arguments but instead points to existing arguments
in the literature where relevant.

### 1.2 Document Structure

In Section 2 , we outline notation, the underlying hard problems which guarantee security, and the
low-level operations which make up the core of the TFHE cryptosystem. Namely, we cover basic
ciphertext structures, encoding, encryption, decryption, and decoding algorithms, and the various
operations supported in the core TFHE scheme.
In Section 3 , we first introduce an encoding for large integers that precedes the TFHE encoding and
encryption steps, resulting in something referred to as the _homomorphic integer_. Next, we outline all of

1.2 Document Structure 1 INTRODUCTION

the supported algorithms over these homomorphic integer types, including basic arithmetic operations,
bit-wise (logical) operations, comparisons, etc.
In Section 4 , we look at a notion of ciphertext compression for the TFHE scheme, outlining the
compression and decompression algorithms.
In Section 5 , we cover a variety of benchmarks for the operations supported in TFHE-rs. In
particular, we consider the latency for a variety of homomorphic integer sizes.

## 2 The TFHE scheme

## 2 The TFHE scheme

**Overview of TFHE.** The TFHE fully homomorphic encryption scheme as implemented in the
TFHE-rs library is a variant of the one introduced in [CGGI16b,CGGI17,CGGI20]. As previously
mentioned, TFHE differentiates itself from the other (G)LWE-based FHE schemes because it supports
a _very efficient bootstrapping_ technique which we call the _programmable bootstrap_ (PBS) operation.
This operation is a bootstrap in that it resets the amount of noise in a ciphertext to a fixed level but it
is also _programmable_ , which means that at the same time, it can apply a function (via a lookup table
evaluation) to the underlying data. In this sense, it is also sometimes called a _functional_ bootstrapping.
Depending on how the data is encoded in the ciphertext, one may either evaluate any function (which
limits the usable message space to half) or be restricted to a so-called negacyclic function.
Unlike many other FHE schemes that use addition and multiplication as the fundamental opera-
tions from which to build more complicated functionality, there is no native multiplication in TFHE.
However, several methods using the PBS allow the multiplication of two TFHE ciphertexts. It is the
PBS operation that enables TFHE to go from being an additively homomorphic scheme to a fully
homomorphic one.
TFHE was not the first FHE scheme to operate this way; it was originally proposed as an improve-
ment of _FHEW_ [DM15], a _GSW_ [GSW13]-based scheme featuring fast bootstrapping for homomorphic
boolean gate evaluation. Apart from improving the bootstrapping of FHEW, TFHE also introduced
new techniques in order to support more functionalities and to improve the homomorphic evaluation of
complex circuits. The efficiency of TFHE comes in part from the choice of a small ciphertext modulus
which allows to use CPU native types to represent a ciphertext both in the standard domain and in
the Fourier domain.

**Notation.** Letqbe a positive integer, which will denote the ciphertext modulus. The ring of integers
moduloq,Z/qZ, is denoted byZq, andRq,Nrepresents the polynomial quotient ringZq[X]/(XN+1),
whereNis a power of two. Elements ofZqare given by lowercase letters, such asa, and elements of
the ringRq,Nare given by uppercase letters, such asA. The coefficients of a polynomialAare given
by(a 0 ;a 1 ;;aN− 1 ). Vectors are represented by bold letters: a vector of elements inZqis given by,
e.g.,aand a vector of elements inRq,Nis given by, e.g.,A. Roundingxup, down, and to the closest
integer is given bydxe,bxc, andbxe. Reducingxmoduloqis given byxmodq, and we extend this
operation element-wise for vectors and coefficient-wise for polynomials. The dot product of two vectors
is given byha;bi=

##### P

iaibiand, similarly to reduction moduloq, this notation extends to vectors of
polynomials. We denote the uniform distribution over a setSbyU(S), and a Gaussian distribution
with meanμand standard deviationasDμ,σ, in the case thatμ= 0we simply writeDσ.
We use the notationa A()forabeing the result of some deterministic algorithmAwhich can
take any number of arguments. Similarly, we use the notationa - A()to denoteaas the output of
a probabilistic algorithmA, which again can take a number of arguments.
In this document, we use different notation than the original TFHE papers [CGGI16b,CGGI17,
CGGI20]. In those papers, the message and ciphertext spaces are expressed by using the real torus
T=R/Z. In the TFHE library [CGGI16c],Tis typically implemented by using native arithmetic
modulo 232 or 264 , which means that they work onZq(withq= 2^32 orq= 2^64 ). This is why we
prefer to useZqinstead ofT, as already adopted in [CJL+ 20 ]. It is made possible because there is an
isomorphism betweenZqand^1 qZ/Zas explained in [BGGJ20, Section 1].

**Outline.** We begin by introducing the hard lattice problems on which TFHE is based, explaining why
low-level ciphertexts take their specific form. Next, we describe these low-level ciphertexts and how
they can be grouped to construct other basic ciphertext types used in the TFHE scheme, along with
a simple method for encoding small integers. We then present basic operations that can be performed

2.1 Hard Lattice Problems: the LWE and GLWE Assumptions 2 THE TFHE SCHEME

on these ciphertexts. Following this, we introduce decomposition and explore several variants of
the key-switching operation. Finally, we cover all the building blocks of the original Programmable
Bootstrapping (PBS) and discuss some of its variants.

### 2.1 Hard Lattice Problems: the LWE and GLWE Assumptions

The security of the TFHE scheme is based on a family of hard problems, namely theLearning With
Errors(LWE) problem of Regev [Reg05,Reg09] and its generalized variants [SSTX09,LPR10,BGV12,
LS15]. These problems are the most commonly used foundations for building FHE schemes. However,
other problems have also been explored [vDGHV10,LATV12,BIP+22a], such as the NTRU problem.
Informally, the LWE problem can be thought of as solving a system of noisy linear equations. The
problem comes in two flavours: a search problem, and a decision problem. The two flavours have been
shown to be equivalent in terms of security [ACPS09,MM11,MP12].

**Definition 1 (Learning With Errors (LWE))** _Let_ n 2 N _be a positive integer, which we call the
LWE dimension. Let_ q 2 N _be a positive integer, which we call the ciphertext modulus. Let_ s=
(s 0 ;;sn− 1 ) 2 Znq _be a vector, which we call the secret, where for each_ 0 i < n _,_ si _is sampled from
some distribution_ D_. We call_ D _the secret distribution. Further, let_  _be another distribution which we
call the error distribution. We define a sample from the learning with errors distribution_ LWEq,n,χ,D
_to be of the form_ (a;b=

```
Pn− 1
i=0aisi+e)^2 Z
```

```
n+
q , where a= (a^0 ;:::;an−^1 )^ - U(Zq)
```

```
n , meaning that
```

_each_ ai _is sampled uniformly and independently from_ Zq _, and the error or noise_ e 2 Zq _is sampled
from_ _.
The_ decisionalLWEq,n,χ,D _problem [Reg05] consists in distinguishing independent samples from_
LWEq,n,χ,D _from the same amount of samples from the uniform distribution_ U(Zq)n+_.
The_ search problem _consists of finding the secret_ s _given an arbitrary polynomial number of samples
from the learning with error distribution_ LWEq,n,χ,D_._

The hardness of both of these problems depends on the parameters(q;n)and on the noise distri-
butionand the secret key distributionD. The hardness of anLWEinstance is described by a value
which is called the _security level_. For an attacker to break anLWEinstance with a security level,
they should perform at least 2 λoperations. More precisely, breaking an LWE instance with a security
levelshould take as many (or more) computational resources than those required to break a block
cipher with a-bit key^1. In practice,= 128is the default value to guarantee long-term security of
a particular instance.
While the LWE problem is a strong candidate for a hard problem, it results in a significant expansion
factor between the unencrypted and encrypted data. When multiple values are to be encrypted, one
can decrease this expansion factor by using a generalized version of the LWE problem which uses
more complex mathematical structures than just integers. Below, we define theGeneral Learning
With Errors(GLWE) problem from [BGV12,LS15] (also known as the Module Learning With Errors
problem) on the ringRq,Ndefined asRq,N=Zq[X]/
XN+ 1
withNa power of two. Elements of
this ring can be written as polynomials of at most degreeN 1 having integer coefficients. Addition
and multiplication follow standard polynomial operations moduloq. However, after multiplication, the
result is reduced to a degree at mostN 1 by taking the remainder modulo the polynomialXN+ 1.
Using the relationXN= 1 , any term with an exponent greater thanNcan be reduced accordingly.
See also Remark 14.

**Definition 2 (General Learning With Errors (GLWE))** _Let_ N 2 N _be a power of two, which
we call the polynomial size. Let_ k 2 N _be a positive integer, which we call the GLWE dimension. Let_

(^1) <https://csrc.nist.gov/projects/post-quantum-cryptography/post-quantum-cryptography-standardization/>
evaluation-criteria/security-(evaluation-criteria)

2.2 The Morphology of FHE Ciphertexts 2 THE TFHE SCHEME

q 2 N _be a positive integer, which we call the ciphertext modulus. Let_ S= (S 0 ;;Sk− 1 ) 2 Rkq,N

_be a vector, which we call the secret, where for each_ 0 i < k _,_ Si=

##### PN− 1

```
j=0 si,jX
```

```
j is sampled from
```

_some distribution_ D _which we call the secret distribution, and let_  _be an error (noise) distribution.
We define a sample from the_ general learning with errors distributionGLWEq,N,k,χ,D _to be of the form_

(A;B=

```
Pk− 1
i=0AiSi+E)^2 R
```

```
k+
q,N where A= (A^0 ;:::;Ak−^1 )^ - U(Rq,N)
```

```
k , meaning that all the
```

_coefficients of_ Ai _are sampled uniformly and independently from_ Zq _, and the error (noise) polynomial_
E 2 Rq,N _is such that all the coefficients are sampled from_ _.
The decisional_ GLWEq,N,k,χ,D _problem [LS15,BGV12] consists in distinguishing_ m _independent
samples from_ U(Rq,N)k+1 _from the same amount of samples from_ GLWEq,N,k,χ,D _, where_ S 2 Rkq,N
_follows the distribution_ D_.
The search problem consists in finding_ S _given an arbitrary polynomial number of samples from the
learning with error distribution_ GLWEq,N,k,χ,D_._

If we chooseN= 1andk=n, the GLWE distributionGLWEq,N,k,χ,Dis the same as the LWE
distributionLWEq,n,χ,D. If we chooseN > 1 andk= 1, the GLWE distribution is called the RLWE
distribution [SSTX09,LPR10]. The GLWE problem can be seen as a generalization of both the LWE
and the RLWE problems.
In general, the secret key distributionDis such that the polynomial coefficients are usually either
sampled from a uniform binary distribution, a uniform ternary distribution, or a Gaussian distribution
([BJRW23,ACPS09]).

**Remark 1 (Alternative hardness assumptions)** _Some recent works adapt TFHE algorithms to
operate on ciphertexts whose security is based on different hard problems. For example, the authors
of [BIP_ + _22b,XZD_ + _23 ] adapt TFHE’s PBS algorithm to work on NTRU ciphertexts rather than GLWE
ciphertexts_

### 2.2 The Morphology of FHE Ciphertexts

In this section, we describe several kinds of low-level ciphertexts. The security of these ciphertexts
relies on the hardness of the problems introduced in Definitions 1 and 2.
First, we recall the LWE and the GLWE ciphertexts which are the most common low-level type of
ciphertexts. GLev and GGSW ciphertexts are then defined as special collections of GLWE ciphertexts.

#### 2.2.1 LWE, RLWE & GLWE Ciphertexts

The most common type of ciphertexts in TFHE as described in [CGGI20] is the LWE ciphertext.
The security of an LWE ciphertext relies on the LWE problem (Definition 1 ). Intuitively, an LWE
ciphertext can be viewed as a sample from an LWE distribution with an encoded message added to its
last component. We will come back to what is meant by an encoded message in Section2.3, for now,
what is important is that it is just some element ofZq.

**Definition 3 (LWE Ciphertext)** _Let_ q 2 N _be a positive integer, which we call the ciphertext mod-
ulus. Let_ n 2 N _be a positive integer, which we call the LWE dimension. Given an encoded message_
me 2 Zq _and a secret key_ s= (s 0 ;;sn− 1 ) 2 Znq _, an_ LWE _ciphertext of_ me _under the secret key_ s _is
defined as the tuple:_

```
ct=
a 0 ;;an− 1 ;b=
```

```
nX− 1
```

```
i=
```

```
aisi+me+e
```

##### 

```
2 LWEs(me)Znq+1; (1)
```

_where each_ ai _can be viewed as being a sample from the uniform distribution over_ Zq _and_ e _is a noise
(error) in_ Zq _which can be viewed as having been sampled from an error distribution_ _. The parameter_
n 2 N _represents the number of elements in the_ LWE _secret key._

2.2 The Morphology of FHE Ciphertexts 2 THE TFHE SCHEME

Where the usage of “can be viewed as being a sample from” is to capture not just freshly encrypted
ciphertexts (for which theaiandeare actually sampled from the respective distributions), but also
ciphertexts that have been processed in some way. For the elementsaiit is important for security
that they look uniformly random. For the error terme, the distributionwill change depending
on where the ciphertext has come from and what operations it has performed on it. It is important
to track theoretically this distribution while doing computations for correctness. In TFHE, low-level
ciphertexts do not need to be explicitly tagged with this information, provided computations follow
predefined patterns, known as atomic patterns, which are outlined in Section2.7.5. These patterns
ensure correct computation up to a given failure probability associated with the parameter set.
As mentioned earlier, inside many TFHE algorithms, we use another type of ciphertext, this time
using a sample from a GLWE distribution to blind an encoded message. We naturally call a ciphertext
of this form a GLWE ciphertext, and it can be seen as a generalization of an LWE ciphertext. The
security of a GLWE ciphertext relies on the hardness of the GLWE problem introduced in Definition 2.
In Definition 4 , we again useRq,N=Zq[X]/
XN+ 1
withNa power of two.

**Definition 4 (GLWE Ciphertext)** _Given an encoded message_ Mf 2 Rq,N _and a secret key_ S=

(S 0 ;;Sk− 1 ) 2 Rkq,N _, a_ GLWE _ciphertext of_ Mf _under the secret key_ S _is a tuple:_

##### CT=

```
A 0 ;;Ak− 1 ;B=
```

```
kX− 1
```

```
i=
```

```
AiSi+Mf+E
```

##### 

```
2 GLWES(Mf)Rkq,N+
```

_where each_ Ai _is a polynomial in_ Rq,N _with coefficients which are sampled from the uniform distribution
over_ Zq _, and_ E _is a noise (error) polynomial in_ Rq,N _, with coefficients which are sampled from an
error distribution_  _(see also Remark 2 ). The parameter_ k 2 N _represents the number of polynomials
in the_ GLWE _secret key and is called the GLWE dimension. We also call_ N _the polynomial size of_ CT
_and_ q _its ciphertext modulus._

**Remark 2** _In the above definition, we assumed that the noise polynomial has coefficients which all
follow the same distribution. Occasionally, we will have ciphertexts for which this is not the case, then
we would need to specify different distributions_ i _for each coefficient of_ E_._

**Remark 3 (Independence of Noise)** _Note that in the definition of a ciphertext, we do not require
the coefficients of the noise polynomial to be independent. Ideally, they would have this property, but
this is not always the case._

If we setN= 1andk=nin Definition 4 , we obtain the same type of ciphertext as in Definition 3.
In the special case whereN= 1, the security of the encryption relies on the hardness of the LWE
problem (Definition 1 ) and not on the GLWE problem (Definition 2 ). By convention, throughout this
paper, we will write an LWE ciphertext, an LWE secret key, and an integer message in lowercase, e.g.
ct,s, andm. On the contrary, we use upper case for a GLWE ciphertext, a GLWE secret key, and a
polynomial message whenN > 1 , e.g.CT,S, andM.
Another special case of Definition 4 is when we setk= 1andN > 1. This ciphertext is called an
_RLWE ciphertext_.

**Remark 4 (Mask and Body of a Ciphertext)** _In Definition 3 , respectively Definition 4 , we call
the elements_ (a 0 ;;an− 1 ) _, respectively_ (A 0 ;;Ak− 1 ) _, the ciphertext mask while we call the remain-
ing element,_ b _, respectively_ B _, the ciphertext body._

2.2 The Morphology of FHE Ciphertexts 2 THE TFHE SCHEME

**Remark 5 (Fresh Ciphertext Compression)** _Freshly encrypted (G)LWE ciphertexts can be com-
pressed via the usage of a cryptographically secure pseudo-random number generator (CSPRNG), de-
noted by , to derive the random mask terms_ a 0 ;:::;an− 1 _via evaluation of on a seed_ 2 f 0 ; 1 gλ _,
e.g._
(a 1 ;;an− 1 ) ():
_This means that a fresh LWE ciphertext_ (a 1 ;;an− 1 ;b) _can be reduced to_ (;b)_. The same tech-
nique can be applied to GLWE ciphertexts to generate the random mask terms_ (A 1 ;;Ak− 1 )_._

**Remark 6 (Secret Key and Error Distributions)** _In TFHE, we typically use a secret key distri-
bution that samples coordinates/coefficients independently at random from a binary distribution. Other
distributions such as ternary and even discrete Gaussian distributions can also be considered though
typically only in the GLWE setting due to how bootstrapping is done in TFHE.
For the error (or noise) distribution, the classical example is to use a discrete Gaussian distribu-
tion (coefficient-wise if GLWE) which has some relatively small variance. Furthermore, due to the
central limit theorem, after computation, it is often the case that the noise inside a ciphertext has a
distribution that is well modeled by a (discrete) Gaussian distribution, regardless of what the initial
noise distribution was. For simplicity, the discreteness of the Gaussian is usually ignored and we treat
them as continuous. This holds as long as the standard deviation of the discrete Gaussian distribution
is large enough.
More recently, a second type of error distribution, which we call a TUniform distribution has been
considered for encrypting fresh ciphertexts (i.e., ciphertexts which are the output of the encryption
algorithm). This distribution is close to the uniform distribution on some interval_ [B+ 1;B] _of
length_ 2 B _, except that we add_ B _to its support and set the probability of sampling both_ B _and_ B _to
be_ 1/4B _, ensuring the mean of this distribution is zero. This distribution has the advantage of being
much easier to sample from which is crucial in the threshold encryption setting where the noise has to
be generated in a secret-shared manner._

**Remark 7 (Private and Public Key Encryption)** _There are two possible ways to encrypt a ci-
phertext, depending on whether the system has been set up to be a private or public key scheme. In a
private key scheme, only the holder of the secret key can encrypt an encoded message while in a pub-
lic key scheme, anyone who knows the public key can encrypt an encoded message but only someone
knowing the private key can decrypt. The typical way of going from a private key FHE scheme to a
public one is for a number of encryptions of zero to be published as the public key. Then, to encrypt an
encoded message using the public key one can take the sum of a random subset of the encryptions of
zero and then add the encoded message to the body of the resulting ciphertext (plus some extra noise).
For more details see Section2.4.3._

We present the (private key) encryption algorithm in Algorithm 1 for encrypting GLWE ciphertexts,
the case of encrypting LWE ciphertexts can be seen as havingN= 1asRq, 1 =Zq.
In the next definition, we show how to decrypt a GLWE ciphertext (and therefore also as a special
case, an LWE ciphertext). We point out that after decryption we get an encoded message, thus there
is typically a decoding step required to remove the noise/error before obtaining the data that was
encrypted in the ciphertext.

**Definition 5 (GLWE Decryption)** _Given a GLWE secret key_ S= (S 0 ;;Sk− 1 ) 2 Rkq,N _and a_

_GLWE ciphertext_ CT _of_ Mf _under the secret key_ S _as defined in Definition 4 , the decryption of_ CT _is
defined as:_

```
M=B
```

```
kX− 1
```

```
i=
```

```
AiSi
```

```
=Mf+E 2 Rq,N
```

##### (2)

2.2 The Morphology of FHE Ciphertexts 2 THE TFHE SCHEME

```
Algorithm 1: CT Encrypt
```

##### 

```
M;fS;σ
```

##### 

```
Context:
```

```
(
σ: a noise distribution with standard deviationwhich depends onkN
U
```

```

Rq,N
```

```

: the uniform distribution overRq,N
```

```
Input:
```

```
(
S= (S 0 ; : : : ; Sk− 1 ) :the GLWE secret key
Mf 2 Rq,N:an encoded message
Output: CT 2 GLWES
```

```

Mf
```

```

```

```
1 for 0 jN 1 do
2 ej - σ
3 E
PN− 1
j=0ejX
j
4 for 0 ik 1 do
5 Ai - U

Rq,N

```

```
6 B
Pk− 1
i=0AiSi+E+Mf
7 CT (A 0 ;; Ak− 1 ; B)
8 return CT
```

_The decryption of a GLWE ciphertext is the modular addition between the encoded message_ Mf _and
the noise polynomial_ E_. In [CGGI20],_ M _is called the_ phase _of_ CT _and is denoted_ (CT;S)_._

```
An algorithmic description of decryption is given in Algorithm 2.
```

```
Algorithm 2: M Decrypt(CT;S)
```

```
Input:
```

```
(
S= (S 0 ; : : : ; Sk− 1 ) :the GLWE secret key
CT= (A 0 ; : : : ; Ak− 1 ; B) 2 GLWES
```

```

Mf
```

```

: the GLWE ciphertext
Output: M 2 Rq,N:a noisy version ofM;f called the phase ofCT
1 M B
Pk− 1
i=0AiSi
2 return M
```

One can view the phase as simply another encoding of the underlying data that is being encrypted.
It may or may not be possible to extract this data from the new encoding because of the noise term
E. This noise is in the lower bits of the encoded message so intuitively any encoding will want to place
the actual message in the higher order bits. Choosing suitable parameters allows to be able to decode
this noisy encoding with very high probability. More details on the encodings used in TFHE-rs are
given in Sections2.3and3.1.
We will sometimes use ciphertexts that are trivially encrypted. The next definition explains what
a trivial encryption is but intuitively it is where all randomness is set to zero.

**Definition 6 (Trivial Encryption)** _Given an encoded message_ Mf 2 Rq,N _and a_ GLWE _dimension_

k _, a_ trivialGLWE _encryption of_ Mf _is defined as the tuple:_

```
CT(Triv)=
```

##### 

```
0 ;; 0 ;B=Mf
```

##### 

```
2 GLWE(k,NTriv)(Mf)Rkq,N+1;
```

_where_ GLWE(k,NTriv)(Mf) =

```
n
0 ;; 0 ;Mf
```

```
o
GLWES(Mf) for any S 2 Rkq,N. Informally, CT(Triv) can
```

_be seen as a GLWE encryption of_ Mf _under any secret key_ S 2 Rkq,N_.
Of course, as the mask polynomials and the noise polynomial are set to zero, these ‘ciphertexts’ are
not secure as there is no encryption. They are mainly used to simplify the notation in some algorithms
referring to an input that can be either a ciphertext or a plaintext._

2.2 The Morphology of FHE Ciphertexts 2 THE TFHE SCHEME

#### 2.2.2 Lev, RLev & GLev Ciphertexts

Using Definition 4 , we can build more complex types of ciphertexts that are required to define the
public material used in some FHE algorithms. For instance, the keyswitching key in Algorithm 10
can be defined with the help ofGLevciphertexts. Informally, aGLevciphertext is just a collection
of related GLWE ciphertexts that come with two additional parameters: a decomposition base and a
decomposition level. Note that we do not encrypt an encoded message here but a plaintext directly, as
such these will only be of practical use if this plaintext is small compared to the ciphertext modulusq.

**Definition 7 (GLev Ciphertext [CLOT21])** _Given a ciphertext modulus_ q _, a decomposition base_
B 2 N _and a decomposition level_ ` 2 N _such that_ Bℓ _divides_ q _, a_ GLevciphertext _of a plaintext_
M 2 Rq,N _under a GLWE secret key_ S 2 Rkq,N _is defined as the following tuple of GLWE ciphertexts:_

```
CT= (CT 0 ;;CTℓ− 1 ) 2 GLevBS,ℓ(M)R
ℓ×(k+1)
q,N (3)
```

_such that for each_ 0 j < ` _,_

```
CTj 2 GLWES
```

```
 q
Bj+
```

##### M

##### 

```
Rkq,N+1:
```

AGLevciphertext withN= 1is called aLev _ciphertext_ and in this case we typically writenin
place ofkfor the size of theLWEsecret key. AGLevciphertext withk= 1andN > 1 is called aRLev
_ciphertext_.

**Remark 8** _One can define a_ GLev _ciphertext for a ciphertext modulus_ q _for which_ Bℓ _does not divide_
q _in a number of ways, for example by rounding each_ Bqj+1M _before encrypting it. Such GLev
ciphertexts behave in much the same way but with some extra noise terms appearing in the analysis.
To keep things simple we will stick to the case for which_ Bℓ _divides_ q _in this document._

#### 2.2.3 GSW, RGSW & GGSW Ciphertexts

Using theGLevciphertexts introduced in Definition 7 , we can define another type of ciphertext, the
GGSWciphertext, that can also be used to describe the public material of some FHE algorithms, in
particular the bootstrapping key (Definition 14 ). Informally, aGGSWciphertext is a collection ofGLev
ciphertexts that use the secret key elements to define theGLevplaintexts. These ciphertexts were first
introduced in [GSW13].

**Definition 8 (GGSW Ciphertexts [GSW13,CLOT21])** _Given a decomposition base_ B 2 N _and
a decomposition level_ ` 2 N _, a_ GGSWciphertext _of a plaintext_ M 2 Rq,N _under a GLWE secret key_
S= (S 0 ;;Sk− 1 ) 2 Rkq,N _is defined as follows:_

##### CT=

#####

```
CT 0 ;:::;CTk
```

##### 

```
2 GGSWBS,ℓ(M)R(q,Nk+1)×ℓ×(k+1) (4)
```

_such that for each_ 0 ik _,_

```
CTi 2 GLevBS,ℓ(SiM)R
ℓ×(k+1)
q,N :
```

with the convention thatSk= 1 by definition.
AGGSWciphertext withN= 1is called aGSW _ciphertext_ , and aGGSWciphertext withk= 1
andN > 1 is called aRGSW _ciphertext_.

2.3 Encoding Booleans and Small Integers 2 THE TFHE SCHEME

### 2.3 Encoding Booleans and Small Integers

In [CGGI20], the authors extensively explained how to use TFHE with Boolean messages. In this
section, we explain how to generalize the encoding to support small integers.
Before applying the encryption procedure in Definitions 3 and 4 , we first need to encode the
message, as explained in the following definition. In TFHE, inputs are most often LWE ciphertexts
for which the same blueprint can be used to encode these input messages in the special case where
R=Z.

**Definition 9 (GLWE Encode & Decode)** _Let_ q 2 N _be the ciphertext modulus. Let further_ p 2 N
_be a_ message modulus _and_  2 N _the number of_ bits of padding^2 _, which satisfy_ 2 πpq _; here_ 2 πp
_is called the_ plaintext modulus_. Let_ M 2 Rp,N _be a message. We define the encoding of_ M _as:_

```
Mf=Encode(M; 2 πp;q) =b∆Me2Rq,N (5)
```

_with_ ∆ = 2 πq·p 2 Q _referred to as the_ scaling factor _(see a visual example in Fig. 4 for_ N= 1 _). To be_

_precise, we lift_ M _to a polynomial with coefficients in_ [0;p) _, multiply by_ ∆ _, then round each coefficient
to an integer and finally reduce modulo_ q_. Note that the final modular reduction will only change_ q _to
zero and can only occur when_ = 0_. When_  > 0 _, we may also sometimes want to allow_ M 2 R 2 π·p,N
_to set explicitly the padding bits, but the encoding procedure remains the same.
To decode_ Mf 2 Rq,N _, we compute the following function:_

```
M=Decode
```

##### 

```
M;f 2 πp;q
```

##### 

##### =

##### $

```
Mf
∆
```

##### '

```
2 R 2 π·p,N: (6)
```

_To be precise, we lift_ Mf _so that it has integer coefficients in_ [0;q) _, then divide each coefficient by_
∆ _and round it to the nearest integer and reduce modulo_ 2 πp _if necessary.
In practice, we have seen after decryption that_ Mf _contains a_ small _error term_ E=

##### PN− 1

```
i=0 eiX
```

```
i 2
```

R _, such that we can write_ Mf= ∆M+E 2 Rq,N_. This error term is the noise in the GLWE ciphertext
and thus grows with homomorphic computations. The decoding algorithm fails if and only if there is
at least one_ i _, with_ 0 i < N _such that_ jeij ∆ 2_. Letting_ M _be the true underlying message that is
the result of performing the computations on unencrypted data, we can write this probability as_

```
Pr
```

(^) N− 1
[
i=
jeij

##### ∆

##### 2

##### 

```
=Pr
```

##### 

```
Decode
```

##### 

```
M;f 2 πp;q
```

##### 

##### 6 =M

##### 

##### : (7)

```
? p e
MSB LSB
```

Figure 4: Plaintext binary representation with = 2bits of padding (gray), message modulus
p= 16 = 2^4 (yellow), such that the plaintext modulus satisfies 2 πpq= 2^32 , and the errore(red),
the white part is empty. The most significant bits (MSB) are on the left and the least significant bits
(LSB) are on the right.

**Remark 9 (Why use bits of padding?)** _Definition 9 provides the option of using bits of padding
in an encoding. At first glance, this may appear like we are wasting space which could be used to_

(^2) For simplicity we use a power of 2 for the padding, but this is not a necessary condition.

2.3 Encoding Booleans and Small Integers 2 THE TFHE SCHEME

_accommodate larger messages by increasing the message modulus instead. The main reason for this is
the PBS operation, which, without a padding bit, can only be used with negacyclic functions. These
functions exhibit a form of two-fold symmetry_^3 _, which is quite restrictive. Having at least one bit
of padding (_  1 _) removes this restriction and means that we can apply any function that can be
represented using a lookup table during the PBS operation as long as the value of the padding bit is
known. Typically, we ensure that the padding bits remain zero throughout. For most applications,_
= 1 _is the most suitable choice, however, we will see that encodings using_ = 0 _and even_  > 1
_have their place in TFHE._

#### 2.3.1 Carry Space

The encoding introduced in Definition 9 can be modified in order to include an extra _carry space_ on
the left of the message space – essentially splitting the original message space into a carry subspace
and a new, reduced message subspace. We denote bycarrythe carry modulus and bymsgthe new
message modulus (i.e.,carrymsg=p); cf. Fig. 5.

```
? p
```

```
carrymsg
```

```
e
MSB LSB
```

Figure 5: Plaintext binary representation with one bit of padding (gray), carry moduluscarry= 8 = 2^3
(orange), the new, reduced message modulusmsg= 8 = 2^3 (yellow), so the carry-message modulus is
p= 64 = 23+3(orange+yellow).

As previously mentioned, for the correctness of the PBS, we need the padding bit(s) to be equal
to zero (at least to be known). So if we perform addition(s), we are at risk of overwriting the padding
bit and thus failing the following bootstrapping. To mitigate this risk, the carry space, originally
initialized to zero, is introduced as a dedicated space to absorb the additional bits after performing
linear operations, i.e., addition or multiplication by a known integer.
Once the carry space is potentially full, we need to perform a PBS which ensures that the message is
reduced modulomsg(i.e., the carry space is cleaned), which in turn provides space for future additions.
We remark that this carry-message encoding is one of the cornerstones of the arithmetic operations
over large integers that will be introduced later in Section 3.
To give an example, let us assume that we havemsg= 2^2 with no carry space (i.e., for now, we
assumecarry= 2^0 ), and two messagesm 0 = 3andm 1 = 2. Both can be written on 2 bits asm 0 = 11 2
andm 1 = 10 2. However, the sum ofm 0 andm 1 can not be written on 2 bits sincem 0 +m 1 = 101 2 = 5,
which motivates the need for a non-trivial carry space.
Another advantage of the carry subspace is that we can delay having to perform a bootstrapping
when computing linear operations. This is useful as the PBS operation is much more costly to perform
than linear operations.

#### 2.3.2 Boolean Encoding

In TFHE-rs, Boolean values, represented by 1 fortrueand 0 forfalse, are encoded within the same
carry-message space as small integers (i.e., using the same parameters in practice) with only a simple
change: the message part is decreased to a single bit while expanding the carry part to maintain the

(^3) See Remark 24 for a precise definition.

2.4 Simple Operations 2 THE TFHE SCHEME

overall carry-message size. This means that we have:

```
msg(Bool)= 2;
```

```
carry(Bool)=
```

```
msgcarry
msg(Bool)
```

##### =

```
p
2
```

##### 

The same rule holds for the carry initialization, i.e., the initial value is zero.

**Remark 10 (Boolean Encoding in [CGGI20])** _In [CGGI20], TFHE is described with a different
Boolean encoding which has_ msg= 2 _,_ carry= 2 _, and_ = 0_. Thanks to the carry bit, linear combinations
between two ciphertexts can be performed before applying a PBS. In this manner, the encoding supports
every commutative logical gate with two inputs by choosing the appropriate function to apply during
the PBS. Furthermore, the functions evaluated during the PBS can be chosen to be negacyclic (while
adding a public constant afterwards to get back to the proper encoding) so we do not require a bit of
padding._

#### 2.3.3 Degree of Fullness

In order to keep track of the worst-case message in a ciphertext – i.e., check if there is still room to
perform more linear operations – we implement a piece of metadata that we call the _degree of fullness_.

**Definition 10 (Degree of Fullness)** _Let_ ct _be an_ LWE _ciphertext with_  _bits of padding and message
(or carry-message) modulus_ p_. Let_ ct _encrypt a (properly encoded) message_ m _,_ 0 m < p _,
where_  _is a publicly known upper bound on_ m_. We refer to_  _as the_ degree of fullness _, denoted_
deg(ct) 2 [0;p1]_._

**Remark 11** _Typically, with the carry-message encoding,_ msg 1 _is used as the initial degree of full-
ness of a fresh ciphertext, unless we have other information. It must be ensured that when entering
bootstrapping, the degree of fullness is in the prescribed interval_ [0;p1]

In summary, _the carry subspace acts as a buffer_ to contain the carry information derived from
linear operations, and _the degree of fullness acts as a measure_ that indicates when this buffer cannot
support any more linear operations. Once this limit is reached, the carry subspace needs to be emptied
by applying a PBS.

**Remark 12 (Noise Growth)** _Generally with FHE, one has to monitor the noise growth in order
to guarantee the correctness of successive operations. However, with the concept of a carry buffer, we
can choose parameters such that the noise is always under a certain level if the degree of fullness has
not reached the maximal value allowed. When we approach this value for the degree, a bootstrapping
operation is performed and the noise is reduced at the same time._

In the rest of this section, we will abstract away the encoding as this allows us to define TFHE
operations intuitively as performing some corresponding operation on the encoded message without
having to specify exactly which encoding is being used each time. Whether the resulting encoding
is sufficient to be able to recover the correct data using some decoding procedure is not discussed
here and is much more technically involved. Sufficed to say, correct parameter choices are the key to
functional correctness.

### 2.4 Simple Operations

In this section, we recall the simplest operations that can be performed over ciphertexts, namely
addition and multiplication by a public polynomial (scalar multiplication). Then we introduce the
sample extraction procedure which converts a GLWE ciphertext into an LWE ciphertext and show
how this can be used to construct an efficient public key encryption scheme.

2.4 Simple Operations 2 THE TFHE SCHEME

#### 2.4.1 Linear Operations

The first and simplest operation that can be done over ciphertexts is addition. Informally, the addition
of two ciphertexts is done by performing coordinate-wise addition moduloq. The resulting noise will
be the sum of the two noises. Formally, Algorithm 3 defines how to add two ciphertexts that are
encrypted under the same secret key.

```
Algorithm 3: CT 3 Add(CT 1 ;CT 2 )
```

```
Context:
```

```
(
S 2 Rkq,N:the input GLWE secret key
CTi=

Ai, 0 ;; Ai,k− 1 ; Bi

2 Rkq,N+1fori2f 1 ; 2 g
```

```
Input:
```

```
8
<
:
```

```
CT 12 GLWES
```

```

gM 1
```

```

;withgM 12 Rq,N
CT 22 GLWES
```

```

gM 2
```

```

;withgM 22 Rq,N
Output: CT 32 GLWES
```

```

gM 1 +gM 2

```

```
1 for 0 jk 1 do
2 A 3 ,j A 1 ,j+A 2 ,jmodq
3 B 3 B 1 +B 2 modq
4 CT 3

A 3 , 0 ;; A 3 ,k− 1 ; B 3

5 return CT 3
```

**Theorem 1 (GLWE addition)** _Let_ S= (S 0 ;;Sk− 1 ) _be a GLWE secret key. Suppose that_ CT 1 =
(A 1 , 0 ;;A 1 ,k− 1 ;B 1 ) _and_ CT 2 = (A 2 , 0 ;;A 2 ,k− 1 ;B 2 ) _are GLWE encryptions of encoded messages_

Mf 1 _and_ Mf 2 _, respectively, encrypted under the GLWE secret key_ S _and whose noise polynomials are_ E 1
_and_ E 2 _, respectively.
The addition of_ CT 1 _and_ CT 2 _defined in Algorithm 3 gives a GLWE ciphertext_ CT 3 _which is an
encryption of the encoded message_ Mf 1 +Mf 2 _under_ S_. Furthermore, the noise in_ CT 3 _is equal to_
E 1 +E 2_.
If_ E 1 _and_ E 2 _can be viewed as having been sampled from two centered Gaussian distributions_
σ 1 =N

#####

##### 0 ;^21

##### 

```
and σ 2 =N
```

#####

##### 0 ; 22

##### 

_where_ ( 1 ; 2 ) 2 R^2 > 0 _are the two standard deviations, then under
the assumption that_ σ 1 _and_ σ 2 _are statistically independent,_ E 1 +E 2 _follows a centered Gaussian
distribution_ pσ 2
1 +σ 22

##### =N

#####

##### 0 ;^21 + 22

##### 

##### 

_The algoritmic complexity of Algorithm 3 is_ CAdd= (k+ 1)NC(add) _, where_ C(add) _denotes the
complexity of adding two elements from_ Zq_._

Algorithm 3 shows how to add two GLWE ciphertexts, it is easy to generalize this to adding many
ciphertexts together. We will also write the addition of ciphertexts using the usualΣ-notation, thus
the sum of the GLWE ciphertextsCTiis written

##### P

iCTi.
With the help of Theorem 1 , we can observe that after the addition, the variance of the noise is
larger than the noise variances of the input ciphertexts. As previously mentioned, this is not only
valid for the addition operation, most operations performed over ciphertexts will increase the variance
of the noise. The exceptions being the rotation operation we will see next and of course, the various
types of bootstrapping operations we will see later.

**Remark 13 (Addition of a public constant)** _By having one of the input ciphertexts, say_ CT 2 _,
in Algorithm 3 be trivial encryption of a public constant that has been suitably encoded, we can add a
public constant to a GLWE ciphertext without increasing the noise._

Before explaining how to multiply a GLWE ciphertext with a polynomial, let us see what happens
when we multiply a GLWE ciphertext with a power ofX. We call this operation a _rotation_.

2.4 Simple Operations 2 THE TFHE SCHEME

```
Algorithm 4: CTout Rotate(CTin;!)
```

```
Context:
```

```
(
S 2 Rkq,N:the input GLWE secret key
CTin= (A 0 ;; Ak− 1 ; B) 2 Rkq,N+
```

```
Input:
```

```
(
CTin 2 GLWES
```

```

Mf
```

```

;withMf 2 Rq,N
!:an integer
Output: CTout 2 GLWES
```

```

XωMf
```

```

```

```
1 for 0 ik 1 do
2 A′i XωAi
3 B′ XωB
4 CTout
```

```

A′ 0 ;; A′k− 1 ; B′
```

```

```

```
5 return CTout
```

**Theorem 2 (Multiplication between a GLWE Ciphertext and a Power of** X **)** _Suppose that_
S= (S 0 ;;Sk− 1 ) _is a GLWE secret key. Let_ CTin _be a GLWE ciphertext encrypting the en-
coded message_ Mf _under the GLWE secret key_ S _with error polynomial_ E _whose coefficients can be
viewed as having been sampled from a symmetric error distribution_ _. Here symmetric means that_
P(x - ) =P(x - )_. Also, let_! _be an integer.
The rotation of_ CTin _by_! _as defined in Algorithm 4 gives a GLWE ciphertext_ CTout _which is an
encryption under_ S _of_ XωMf_. The noise in_ CTout _is equal to_ XωE_. Further, each coefficient of the
noise in the GLWE ciphertext_ CTout _also follows the noise distribution_ _._

Note that the rotation ofCTindepends only on the value of!modulo 2 NsinceX^2 N= 1inRq,N.
We remark that both the discrete Gaussian distributions which we use (i.e., those with zero mean),
and the TUniform distribution are symmetric.
As we can add GLWE ciphertexts together (Theorem 1 ) and multiply them by powers ofX(The-
orem 2 ), we can combine them to perform the product between a GLWE ciphertext and a polynomial
with integer coefficients. We call this operation scalar multiplication, with the integer polynomial
being the scalar.

```
Algorithm 5: CTout ScalarMult(CTin;D)
```

```
Context:
```

```
(
S 2 Rkq,N:the input GLWE secret key
CTin= (A 0 ;; Ak− 1 ; B) 2 Rkq,N+
```

```
Input:
```

```
(
CTin 2 GLWES
```

```

Mf
```

```

;withMf 2 Rq,N
D 2 Z[X]
Output: CTout 2 GLWES
```

```

DMf
```

```

```

```
1 for 0 ik 1 do
2 A′i DAi
3 B′ DB
4 CTout
```

```

A′ 0 ;; A′k− 1 ; B′
```

```

```

```
5 return CTout
```

**Theorem 3 (Multiplication between a GLWE Ciphertext and a Polynomial)** _Suppose that_
S= (S 0 ;;Sk− 1 ) _is a GLWE secret key. Let_ CTin _be a GLWE ciphertext encrypting the encoded_

_message_ Mf _under the GLWE secret key_ S _with noise polynomial_ E_. Let_ D 2 Z[X] _be such that_

2.4 Simple Operations 2 THE TFHE SCHEME

##### D=

##### PN− 1

```
i=0 diX
i. The scalar multiplication of CTin with D as defined by Algorithm 5 gives a GLWE
```

_ciphertext_ CTout _which is an encryption of_ DMf _under_ S_. The noise in_ CTout _is equal to_ DE_.
Furthermore, if the coefficients of_ E _can be viewed as independently sampled from a centered Gaus-
sian distribution_ σ _, then each coefficient of the noise in_ CTout _follows a centered Gaussian distribution_
ν·σ _where_  _is the 2-norm of_ D _defined via_ ^2 =

##### PN− 1

```
i=0 d
```

```
2
i.
```

**Remark 14 (Fast Polynomial Multiplication)** _Naïvely, polynomial multiplication (Lines 2 and 3
of Algorithm 5 ) needs_ O(N^2 ) _operations. In the TFHE-rs library, polynomial multiplication is imple-
mented with the use of_ Fast Fourier Transform _(FFT) which reduces the complexity to_ O(NlogN)_.
However, it comes with a particular caveat: due to the limited precision of floating point types, a new
source of error needs to be accounted for.
Besides FFT-backed fast polynomial multiplication, there exist other methods, in particular, one
backed by the_ Number Theoretic Transform _(NTT), possibly combined with the_ Chinese Remainder
Theorem _(CRT). This one however requires_ q _of a special form: namely a product of more than one
prime with some additional properties. Hence, additional error terms due to rounding would be present._

Note that scalar multiplication is only practical when the scalar is “small” since the noise grows
quickly with the 2-norm of the scalar. We will see later how we can get around this requirement that
scalars must be small using the idea of decomposition.

**Remark 15 (Linear Operations on GLev and GGSW Ciphertexts)** _Since both GLev and GGSW
ciphertexts can be seen as collections of GLWE ciphertexts, Theorems 1 and 3 can be trivially extended
to GLev and GGSW ciphertexts in a component-wise manner._

Using Theorem 1 and Theorem 3 , we can define the TFHE dot product. We present it for GLWE
ciphertexts for genericity however in the TFHE-rs library this is mainly used for LWE ciphertexts.

```
Algorithm 6: CTout DotProduct(fCTigi;f!igi)
Context:
```

```
n
S= (S 0 ; S 1 ;; Sk− 1 ) 2 Rkq,N:the input GLWE secret key
```

```
Input:
```

```
(
CTi 2 GLWES
```

```
PN− 1
j=0 mgi,jX
j

;withmgi,j 2 Zq;for 0 i < ; 0 j < N
!i 2 Z[X]for 0 i <
Output: CTout 2 LWEs
```

```

M]out
```

```

;withM]out=
Pα− 1
i=
```

```
PN− 1
j=0 mgi,jXj!i
1 CTout
Pα− 1
i=0ScalarMult(CTi; !i)
2 return CTout
```

**Theorem 4 (Homomorphic Dot Product)** _Let_ cti 2 LWEs(mfi)Znq+1 _for_ 0 i  1 _be a
list of LWE ciphertexts encrypted under an LWE secret key_ s= (s 0 ;;sn− 1 ) _with noises_ ei _that
can be viewed as having been sampled independently from a centered Gaussian distribution_ σ_. Let_
f!ig 0 ≤i≤α− 12 Zα _be arbitrary integers which we call weights.
A homomorphic dot product is a dot product between a vector of LWE ciphertexts and a vector of
integers and can be computed using Algorithm 6. It results in an LWE ciphertext_ ctout _encrypting the
message_

Pα− 1
i=0mfi!i _under the secret key_ s_. The noise in the output ciphertext follows the distribution_
ν·σ=N

#####

##### 0 ;^2 ^2

##### 

```
with ^2 =
```

```
Pα− 1
i=0!
```

2
i _, the squared 2-norm of the vector of weights.
Thus, given a dot product between a vector of ciphertexts, whose noises are independently sampled
from the same (Gaussian) distribution, and a vector of integers, the output noise of the dot product
can be fully characterized by its 2-norm_ _._

2.4 Simple Operations 2 THE TFHE SCHEME

We will also use the notationhx;yito denote the dot product between a vectorxof ciphertexts and
a vectoryof scalars. As seen previously, these ciphertexts can be GLWE ciphertexts not just LWE
ciphertexts and we will soon see examples of this where the vector of GLWE ciphertexts is a GLev
ciphertext.

**Remark 16 (Subtraction)** _We note that subtraction is a special case of the dot product with_ = 2 _,_
! 0 = 1 _and_! 1 = 1_. We will write_ CT 1 CT 2 _for the subtraction of_ CT 2 _from_ CT 1_._

#### 2.4.2 Sample Extract

```
Algorithm 7: ct SE(CT;)
```

```
Context:
```

```
8
>>>
>>>
<
>>>
>>>
:
```

```
s= (s 0 ;; skN− 1 ) : the output LWE secret key
S= (S 0 ; : : : ; Sk− 1 ) : the input GLWE secret key
whereSi=
PN− 1
j=0 si·N+jXj;for^0 ik^1 ;
Mf=PNi=0−^1 mfiXi:an encoded polynomial message
B=PNj=0−^1 bjXjandAi=PNj=0−^1 ai,jXjfor 0 ik 1
```

```
Input:
```

```
(
CT= (A 0 ;; Ak− 1 ; B) 2 GLWES
```

```

Mf
```

```

```

```
2f 0 ;; N 1 g
Output: ct 2 LWEs(gmα)
1 b′ bα
2 for 0 ik 1 do
3 for 0 j do
4 a′i·N+j ai,α−j
5 for + 1j < N do
6 a′i·N+j ai,N+α−j
```

```
7 ct (a′ 0 ;; a′kN− 1 ; b′)
8 return ct
```

```
Sample extract is an operation taking as input a GLWE ciphertext,CT 2 GLWES
```

##### P

```
N− 1
i=0 mfiX
```

```
i
```

##### 

##### 

and outputting an LWE ciphertext,ct 2 LWE ̄s(gmα), where 0  N 1 is given as a further input
(the default being = 0). The output ciphertextctwill be encrypted under a flattened representation
of the input GLWE keyS. We explain in Definition 11 what a flattened GLWE secret key is.

**Definition 11 (Flattened Representation of a GLWE Secret Key)** _A GLWE secret key_

##### S=

##### 0

##### @S 0 =

##### NX− 1

```
j=0
```

```
s 0 ,jXj;;Sk− 1 =
```

##### NX− 1

```
j=0
```

```
sk− 1 ,jXj
```

##### 1

```
A 2 Rkq,N
```

_can be_ flattened _into an LWE secret key_ ̄s= ( ̄s 0 ;; ̄skN− 1 ) 2 ZkNq _by defining_ s ̄iN+j :=si,j _, for_
0 i < k _and_ 0 j < N_._

A version of the sample extract for RLWE ciphertexts was introduced in [CGGI20, Section 4.2]
and can be easily extended to GLWE ciphertexts as is done in Algorithm 7. Informally, the sample
extract consists in simply rearranging, possibly with negation, some of the coefficients of the GLWE
input ciphertext to build the output LWE ciphertext encrypting one of the coefficients of the input
encoded polynomial message.

2.4 Simple Operations 2 THE TFHE SCHEME

**Theorem 5 (Sample Extract)** _Let_ 0  N 1 _be an index. Let_ CT 2 GLWES

##### P

```
N− 1
i=0 mfiX
```

```
i
```

##### 

_be a GLWE ciphertext with noise polynomial_ E =

##### PN− 1

```
i=0 eiX
```

```
i where ei can be viewed as having
```

_been sampled from_ σ_. After the sample extract operation, Algorithm 7 , we obtain a ciphertext_ ct 2
LWE ̄s(gmα) _encrypted with_ ̄s _, the flattened representation of_ S _(Definition 11 ) and the noise in_ ct _is_
eα_. As such the noise in the output LWE can also be viewed as being sampled from_ σ _, hence sample
extract does not add any noise.
The complexity of the sample extract is:_

```
CSE= (k+ 1)NC(copy) (8)
```

_where_ C(copy) _denotes the complexity of performing a copy operation on an element in_ Zq_. In practice,
most of the time we can neglect the cost of the sample extract as it is very fast compared to the other
FHE algorithms._

#### 2.4.3 Public Key Encryption

As mentioned previously, there are two types of encryption schemes we can consider: a public key
scheme and a private key scheme. One can typically go from a private key scheme to a public one
by making public a number of encryptions of zero. Here we explain a way to construct a public key
scheme for TFHE which has a smaller public key size, based on the work in [Joy24].
The public key consists of a single RLWE sample,PK= (A;B) 2 RLWES(0), so thatB=AS+E
for some error termE 2 Rq,Nand secret keyS 2 Rq,N.
Then to encrypt an encoded messageme 2 Zqusing this public key one first samples an element
R 2 Rq,N with binary coefficients, two small noise termsE′ 2 Rq,N ande 2 Zqand computes
A′=AR+E′,B′=BR+e+me(note we add elements ofZqto elements ofRq,Nby treating
them as constant polynomials). Finally, one applies the sample extraction procedure to the RLWE
ciphertext(A′;B′)(with = 0) to get an LWE ciphertext inZNq+1encryptingmeunder the secret key
sthat is the flattening ofS. A detailed algorithm is given in Algorithm 8.
One can then perform an LWE keyswitch if one wishes to use an LWE dimension that is notN
(which must be a power of two).

```
Algorithm 8: ct Encrypt(m;e PK;σ)
```

```
Context:
```

```
8
><
>:
```

```
σ:a noise distribution with standard deviationwhich depends onN
U

R 2 ,N

:the uniform distribution overR 2 ,N;the samples from which we consider inRq,N
s:the flatten GLWE secret keyS
```

```
Input:
```

```
(
PK= (A; B) 2 RLWES(0) :the public key
me 2 Zq: an encoded message
Output: ct 2 LWEs(me)ZNq+1
1 for 0 jN do
2 ej - σ;
3 E′
PN− 1
j=0ejXj;
4 R - U

R 2 ,N

;
5 A′ AR+E′;
6 B′ BR+eN+me;
7 ct SE((A′; B′);0);
8 return ct
```

**Remark 17 (Zero-Knowledge Proofs)** _When using a public key encryption scheme, anyone can
encrypt data and send this ciphertext to be computed on homomorphically. This opens up the possibility_

2.5 Key Switching 2 THE TFHE SCHEME

_of malicious users sending malformed ciphertexts which could result in information leakage during
computation or even degradation of security if not managed appropriately.
To this end, the use of zero-knowledge proofs is employed to give some guarantee that a given ci-
phertext is not malformed. Put simply, a zero-knowledge proof allows one entity to prove to another
that it knows some data without leaking any information about what value the data actually takes. Ide-
ally, in our situation, any ciphertext encrypted using the public key would come with a zero-knowledge
proof that the sender knows both the message and the randomness used in generating the ciphertext. In
practice, proving only that the randomness generated is smaller than some bound using some norm is
enough (the randomness is directly linked to the size of the noise in the output ciphertext). TFHE-rs
then provides the ability to verify these zero-knowledge proofs. Details on this can be found in [Lib24]._

### 2.5 Key Switching

We saw with Theorem 1 that we can add ciphertexts encrypted under the same GLWE keys. However,
we cannot do the same for ciphertexts encrypted under different keys, even if they have the same
dimension. In this subsection, we introduce the concept of a key switch which can solve this issue and
more.
A key switch allows to homomorphically change the key a ciphertext is encrypted under. Given a

ciphertextCTin 2 GLWESin

##### 

```
Mf
```

##### 

```
we can use a key switch to obtain a ciphertextCTout 2 GLWESout
```

##### 

```
Mf
```

##### 

Soutcan have different parameters thanSin, for instance the polynomial sizeNand the GLWE di-
mensionkcould be different. Several algorithms have been presented in the literature that allow to
perform a key switch. Many of them offer some further functionality than simply changing the secret
key. We will describe the main versions here. Every version needs some public material to perform
the key switch, namely the keyswitching key. Intuitively, the keyswitching key is composed of the key
Sinencrypted underSout.

#### 2.5.1 Decomposition

First, we introduce the radix decomposition: this algorithm is used as a sub-routine in every key switch
algorithm and in other FHE algorithms as well.

**Balanced Radix Decomposition.** Several ways exist to perform a radix decomposition. The
classical radix decomposition was described in [CGGI20]. Here we use a balanced decomposition
algorithm inspired by Joye [Joy21] as it ensures that the distribution of the digits of the decomposition
has zero mean. We also assume here thatBℓdividesq. This assumption can be removed in a number
of ways but leads to a more complicated calculation of the resulting noise.

**Definition 12** _Let_ B 2 N _be a decomposition base and_ ` 2 N _be a decomposition level such that_ Bℓ
_divides_ q_. A decomposition of_ x 2 Zq _with respect to_ B _and_ ` _is a vector_ (x 1 ;;xℓ) 2 Zℓ _such that_

```
D
(x 1 ;;xℓ);
```

```
q
B
```

##### 

```
q
Bℓ
```

##### E

##### =

##### Xℓ

```
i=1
```

```
xi
q
Bi
```

##### =

##### 

```
x
```

##### Bℓ

```
q
```

##### 

##### 

```
q
Bℓ
```

```
2 Zq: (9)
```

_We refer to_ (x 1 ;;xℓ) _as the decomposition vector of_ x _or simply the decomposition of_ x_.
In [CGGI20],_ (x 1 ;;xℓ) _are defined as the unique integers satisfying Equation_ ( 9 ) _with_ B 2 
xi<B 2_.
In [Joy21], Joye introduces a balanced decomposition algorithm when_ q=Bℓ _such that_ B 2 
xiB 2 _, which is better for the noise propagation as it gives a centered distribution for the_ xi_. We
note that their algorithm is not deterministic for some inputs. We present a modified version of this_

2.5 Key Switching 2 THE TFHE SCHEME

_in Algorithm 9 which is deterministic and still has the balanced property we want when_ Bℓ< q_. We
denote this balanced decomposition of_ x _by_ BalDec(B,ℓ)(x)_._

```
Algorithm 9: (x 1 ;;xℓ) BalDec(B,ℓ)(x)
Context: B; `: decomposition parameters such that 2 Bℓdividesq
Input: x: an integer inZq
Output: (x 1 ;; xℓ) 2
```

```
h
B 2 ;B 2
```

```
iℓ
, such that Equation ( 9 ) holds and the expectation of each digit is zero on
uniformly random inputs
1 y
```

```
j
x^2 Bqℓ
```

```
k
```

```
2 [y] 2
3 z
```

```
j
y+1
2
```

```
k
```

```
4 if z >B
ℓ
2 or
```

```

z=B
ℓ
2 and^ = 1
```

```

then
5 z zBℓ
6 for 0 i` 1 (in ascending order) do
7 zi zmodB // So 0 zi<B
8 z (zzi)/B
9 if zi>B 2 or
```

```

zi=B 2 and(zmodB)B 2
```

```

then
10 zi ziB
11 z z+ 1
12 xℓ−i zi
13 return (x 1 ;; xℓ)
```

**Remark 18 (TFHE-rs specific implementation of** BalDec **)** _Algorithm 9 as implemented in the
TFHE-rs library uses a specific world size, which we denote by_ BITS _, and elements of_ Zq _are stored
in the top_ log 2 (q) _bits of each word, hence the bottom_ BITSlog 2 (q) _will be zero. This means that_ q
_in Line 1 will be replaced by_ 2 BITS_. Furthermore, Line 5 is performed modulo_ 2 BITS _which can affect
the value of_ zℓ− 1 _if it is initally set to_ B/2_. In particular, this means that the balanced property is not
preserved for all parameter choices and we have the additional restriction that_ Bℓ+1 _must divide_ 2 BITS
_in order to have a correctly balanced decomposition._

Decomposition can naturally be generalized to work for a wider range of input types. When
applying the decomposition on a vector of integers, the result is a vector of decomposition vectors
of integers. It is also possible to decompose an integer polynomialP 2 Rqwith this algorithm in a
componentwise manner. The result will satisfy

```
D
BalDec(B,ℓ)(P);
```

```
q
B
```

##### 

```
q
Bℓ
```

##### E

##### =

##### 

##### P

##### Bℓ

```
q
```

##### 

##### 

```
q
Bℓ
```

```
2 Rq,N
```

withBalDec(B,ℓ)(P)being a vector of polynomials. Applying this kind of decomposition on a vector
of polynomials, we get a vector of vectors of polynomials, and so on.
With decomposition now defined, we can see the utility of the GLev ciphertexts at work.

#### 2.5.2 LWE and GLWE Key Switch

Using Theorem 1 and Definition 12 , Algorithm 10 defines the LWE key switch as was introduced
in [CGGI17, Algorithm 1]. The output ciphertext is a GLWE ciphertext which, of course, can also be
an LWE ciphertext whenN= 1, which is the typical case.

2.5 Key Switching 2 THE TFHE SCHEME

```
Algorithm 10: CT KS(ct;KSK)
```

```
Context:
```

```
(
s= (s 0 ;; sn− 1 ) :the input LWE secret key
S= (S 0 ; : : : ; Sk− 1 ) :the output GLWE secret key
```

```
Input:
```

```
8
<
:
```

```
ct= (a 0 ;; an− 1 ; b) 2 LWEs(me)
KSK=
```

```
n
CTi 2 GLevBS,ℓ(si)
```

```
o
0 ≤i≤n− 1
: keyswitching key fromstoS
Output: CT 2 GLWES(me)
1 CT (0;; 0 ; b)
Pn− 1
i=0
```

```
D
CTi;BalDec(B,ℓ)(ai)
```

```
E
```

```
2 return CT
```

**Theorem 6 (LWE Key Switch)** _Let_ ct 2 LWEs(me)Znq+1 _be an LWE ciphertext encrypting_ me 2
Zq _, under the LWE secret key_ s= (s 0 ;:::;sn− 1 ) 2 Znq _, with noise which can be viewed as having
been sampled from_ σ_. Let_ S _be a GLWE secret key such that_ S= (S 0 ;:::;Sk− 1 ) 2 Rkq,N_. Let_

KSK=

```
n
CTi 2 GLevBS,ℓ(si)R
ℓ×(k+1)
q,N
```

```
o
0 ≤i≤n− 1
```

```
be a key switching key from s to S with noise sampled
```

_from_ σKSK_.
The variance of the_ j _-th coefficient of the noise in the output of Algorithm 10 is_

```
VarKS[j] =
```

##### 

```
^2 +n
```

##### 

```
q^2
12 B^2 ℓ
```

```
+cvar
```

##### 

```
Var(si) +n
```

##### 

```
q^2
12 B^2 ℓ
```

```
+cexp
```

##### 

```
E^2 (si)
```

##### 

```
j, 0
```

```
+n`^2 KSK
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### 

##### (10)

_where_ cvar _and_ cexp _are small constants that depend on exactly how the rounding operation is performed_

_and_ "(B;`) = 121 +4(B^1 +1)+^1 −(−B)

−ℓ
ℓ·(1+B−^1 )^2 2 O(1) _is a small correction coming from the distribution
of the digits output by Algorithm 9 , other decomposition algorithms would change this value slightly.
Here,_ j, 0 _is equal to 1 if_ j= 0 _, else it is zero.
The algorithmic complexity of Algorithm 10 is_

```
CKS=nC(ℓ)(BalDec) +`n(k+ 1)NC(mul)
+ ((`n1)(k+ 1)N)C(add):
```

##### (11)

_where_ C(mul) _denotes the complexity of performing a multiplication between two elements in_ Zq _, and_
C(BalDec) _denotes the complexity of the balanced decomposition outlined in Algorithm 9. The size (in
bits) of the key switching key_ Size(KSK) _is computed with the following formula:_

```
Size(KSK):=n`(k+ 1)Ndlog 2 (q)e
```

**Remark 19 (Stochastic Rouding)** _As an example of a particular rounding operation, if one rounds
to the nearest integer and if there is a tie (i.e., we are rounding a value that is exactly half way between
two integers), then we flip a random bit to determine which of the two closest integers is chosen, then
the constants are_ cvar=cexp=^16_. This rounding has the nice property that the expectation of the noise
coming from the rounding of a uniformly distributed integer in_ Zq _is zero. This can also be achieved
by deterministic methods of rounding which depend on_ q_._

**Remark 20 (Public functional Key Switch)** _It is possible to leverage the key switch algorithm to
also compute a public linear function_ f:Zαq!Rq,N _on several input LWE ciphertexts_ fctigαi=0−^1_. In
particular,_ f _must satisfy the following properties:_

2.5 Key Switching 2 THE TFHE SCHEME

- _for any_ (x;y) 2

#####

```
Zαq
```

#####  2

```
we have f(x+y) =f(x) +f(y) ;
```

- _for any_ t 2 Z _and any_ x 2 Zαq _we have_ f(tx) =tf(x)_._

_To perform a public functional key switch, one computes_

```
CT (0;; 0 ;f((b 0 ;;bα− 1 )))
```

```
nX− 1
```

```
i=0
```

##### D

```
CTi;BalDec(B,ℓ)(f((ai, 0 ;;ai,α− 1 )))
```

##### E

##### 

_If_ cti _encrypt the encoded message_ mfi _then_ CT _will encrypt_ f(mf 0 ;;^mα− 1 )_. It is given in
algorithmic form in Algorithm 11. This operation was first presented in [CGGI17, Algorithm 1] and
can be used to pack many LWE ciphertexts into a single GLWE ciphertext, see Section2.5.3for details
on this._

```
Algorithm 11: CT PublicKS(fctigαi=1;KSK;f)
```

```
Context:
```

```
(
s= (s 0 ;; sn− 1 ) 2 Znq: the input LWE secret key
S= (S 0 ; : : : ; Sk− 1 ) 2 Rkq,N: the output GLWE secret key
```

```
Input:
```

```
8
>><
```

```
>>:
```

```
fcti= (ai, 0 ;; ai,n− 1 ; bi) 2 LWEs(mfi)gαi=0−^1 withmfi 2 Zq
KSK=
```

```
n
CTi 2 GLevBS,ℓ(si)
```

```
o
0 ≤i≤n: public functional keyswitching key fromstoS
f:the public linear functionZαq!Rq,Nto be homomorphically evaluated
Output: CT 2 GLWES

f

mf 0 ;;m^α− 1

```

```
1 CT (0;; 0 ; f((b 0 ;; bα− 1 )))
Pn− 1
i=0
```

```
D
CTi;BalDec(B,ℓ)(f((ai, 0 ;; ai,α− 1 )))
```

```
E
```

```
2 return CT
```

We now specify in Algorithm 12 , a GLWE key switch that works the same way as the LWE
key switch of Algorithm 10. The dot product between the keyswitching key and the polynomial
decompositions can be performed using the FFT which significantly speeds up the execution time of
the GLWE key switch compared to the LWE key switch.

```
Algorithm 12: CTout GlweKeySwitch(CTin;KSK)
```

```
Context:
```

```
(
Sin=

Sin, 0 ;; Sin,kin− 1

2 Rkq,Nin :the input GLWE secret key
Sout 2 Rkq,Nout:the output GLWE secret key
```

```
Input:
```

```
8
<
:
```

```
CTin=

A 0 ;; Akin− 1 ; B

2 GLWESin
```

```

Mf
```

```

Rkq,Nin+1;withMf 2 Rq,N
KSK=
```

```
n
CTi 2 GLevBSout,ℓ(Sin,i)
```

```
o
0 ≤i≤kin− 1 :keyswitching key fromSintoSout
Output: CTout 2 GLWESout
```

```

Mf
```

```

```

```
1 CTout (0;; 0 ; B) 2 Rkq,Nout+1
2 for 0 ikin 1 do
3 CTout CTout
```

```
D
CTi;BalDec(B,ℓ)(Ai)
```

```
E
```

```
4 return CTout
```

**Theorem 7 (GLWE Key Switch)** _Let_ CT 2 GLWESin

##### 

```
Mf
```

##### 

```
Rkq,Nin+1 be a GLWE ciphertext en-
```

_crypting_ Mf 2 Rq,N _, under the GLWE secret key_ Sin= (Sin, 0 ;:::;Sin,kin− 1 ) 2 Rkq,Nin _, with noise coeffi-_

_cients which can be viewed as having been sampled from_ σ_. Let_ Sout 2 Rkq,Nout _be another GLWE secret_

2.5 Key Switching 2 THE TFHE SCHEME

_key and let_ KSK=

```
n
CTi 2 GLevBSout,ℓ(Sin,i)R
ℓ×(kout+1)
q,N
```

```
o
0 ≤i≤kin− 1
```

```
be a GLWE key switching key from
```

Sin _to_ Sout _with noise sampled from_ σKSK_.
The variance of the_ j _-th coefficient of the noise in the output of Algorithm 10 is_

```
VarKS[j] =^2 +kN
```

##### 

```
q^2
12 B^2 ℓ
+cvar
```

##### 

```
Var(Sin,i) +kN
```

##### 

```
q^2
12 B^2 ℓ
+cexp
```

##### 

```
E^2 (Sin,i)
```

```
+kN`^2 KSK
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### 

##### (12)

_where_ cvar _,_ cexp _and_ "(B;`) _are defined in Theorem 6 , and_ Var(Sin,i) _, respectively_ E^2 (Sin,i) _, is the
average of the variances, respectively mean squareds, of the coefficients of_ Sin,i_.
The algorithmic complexity of Algorithm 10 is_

```
CKS=kinNC(ℓ)(BalDec) +`kin(kout+ 1)Nlog(N)C(mul)
+ ((`kin1)(kout+ 1)N)C(add):
```

##### (13)

```
The size (in bits) of the key switching key Size(KSK) is computed with the following formula:
```

```
Size(KSK):=kin`(kout+ 1)Ndlog 2 (q)e
```

#### 2.5.3 Packing Key Switch

The idea of a packing keyswitch is to pack several LWE ciphertexts into a single GLWE ciphertext. It
takes as input a set of NLWE ciphertextsct 0 ;ct 1 ;;ctα− 1 as well as a set of indicesfijgαj=0−^1.
Given this set of indices and some public materialKSK, a packing keyswitch packs the messages

contained within the LWE ciphertexts into a single GLWE ciphertextGLWE

##### P

```
α− 1
j=0mfjX
```

```
ij
```

##### 

```
where
```

mfj 2 Zqis the encoded message encrypted inctj. We present the full algorithm in Algorithm 13.

```
Algorithm 13: CTout PackingKS(fctjgαj=0−^1 ;fijgαj=0−^1 ;KSK)
```

```
Context:
```

```
(
s= (s 0 ;; sn− 1 ) 2 Znq: the input LWE secret key
S= (S 0 ; : : : ; Sk− 1 ) 2 Rkq,N: the output GLWE secret key
```

```
Input:
```

```
8
<
:
```

```
fctj= (aj, 0 ;; aj,n− 1 ; bj) 2 LWEs(mfj)gαj=0−^1 withmfj 2 Zq
KSK=
```

```
n
CTi 2 GLevBS,ℓ(si)
```

```
o
0 ≤i≤n
: public functional keyswitching key fromstoS
Output: CT 2 GLWES
```

```
Pα− 1
j=0mfjX
ij
```

```
1 CT
```

```

0 ;; 0 ;
Pα− 1
j=0bjX
ij


Pn− 1
i=0
```

```
D
CTi;BalDec(B,ℓ)
```

```
P
α− 1
j=0aj,iX
ij
E
```

```
2 return CT
```

We have already seen one method that can be used to perform a packing keyswitch, namely the
public functional keyswitch from Remark 20. This packing key switch can be described easily with
the help of the LWE key switch (Algorithm 10 and Theorem 6 ), the multiplication by a power of
X(Theorem 2 ) and the addition (Theorem 1 ). In short, it takes as input several LWE ciphertexts,
keyswitches them using Algorithm 10 , multiplies each of them by a power ofX, and finally adds the
resulting ciphertexts together.
The noise in this case can be derived from Eq. ( 10 ). Definej,I, withI=fijgαj=0−^1 the set of indices
used, such thatj,Iis equal to 1 ifj2Iand zero otherwise. Then the noise of using Algorithm 10 to

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

perform a packing keyswitch has variance in thejth coefficient of

```
VarPKS[j] =
```

##### 

```
^2 +n
```

##### 

```
q^2
12 B^2 ℓ
```

```
+cvar
```

##### 

```
Var(si) +n
```

##### 

```
q^2
12 B^2 ℓ
```

```
+cexp
```

##### 

```
E^2 (si)
```

##### 

```
j,I
```

```
+ n`^2 KSK
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### 

The complexity is clearly times that of Eq. ( 11 ).

**Remark 21 (Alternative Packing Keyswitches)** _In [CDKS21], using automorphisms to compute
the trace function from algebraic number theory, the authors introduced a way to pack many LWE
ciphertexts into a single RLWE ciphertext (this can be naturally extended to a GLWE ciphertext).
While more efficient than the traditional packing key switch, there is less flexibility in the input and
output dimensions as the associated keys are related. Another approach that exploits GLWE ciphertexts
(called MLWE in the paper) is HERMES [BCK_ + _23 ]._

### 2.6 Building Blocks of the PBS

In this section, we define the building blocks of TFHE’s PBS as described in [CGGI16b,CGGI17,
CGGI20]. We have already seen one of these, namely the sample extract Algorithm 7. Further, we
need to introduce the modulus switch and the external product operations. Then, on top of the external
product, we build the _controlled mutex_ (CMux), and from theCMux, we explain how to perform a blind
rotate.

#### 2.6.1 Modulus Switch

The modulus switch takes as input an LWE ciphertext with some ciphertext modulusqand outputs an
LWE ciphertext with ciphertext modulusw. If the input ciphertext encrypts the encoded messagem ̃

then the output ciphertext will encrypt the encoded message

```
j
m ̃wq
```

```
m
```

. In the context of TFHE’sPBS,

w= 2NwithNthe polynomial size of the ringRq,N. There are two variants of the modulus switching
algorithm, indicated by the boolean valueDriftMitigation. If theDriftMitigationflag is set to true, then
the input ciphertext is modified in order to reduce the overall noise variance of the output ciphertext.
To do so, we need an additional public key calledDMK. The modifications to the input ciphertext
forDriftMitigation=Trueare described in Algorithm 15 and the modulus switching algorithm itself is
described in Algorithm 14.

**Drift Mitigation.** As mentioned in Section2.2.1, decryption produces a noisy version of the encoded
message contained in the ciphertext. The failure probability (pfail) represents the likelihood of obtaining
an incorrect message after decryption and decoding. As such, it largely depends on the magnitude of
the noise in the ciphertext.
The noise of the modulus switching is largely determined by the rounding errors. The vector of all
rounding errors that occur during modulus switching is called the drift vector. The additional noise
generated by the modulus switching operation arises from the dot product between the drift vector
and the secret key. As the size of the rounding errors depends on the mask of the input ciphertext,
an encryption of zero can be added to the input ciphertext to modify the mask without affecting
the underlying message. Hence by carefully selecting an appropriate encryption of zero, the rounding
errors can partially be mitigated, such that even though adding an encryption of zero increases the
noise of the input ciphertext of the modulus switching operation, the noise of the output ciphertext
will be reduced. More details on this technique can be found in [BJSW24].

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

```
Algorithm 14: ctout MS(ctin;w;DriftMitigation;DMK)
Context:
```

```
n
s= (s 0 ;; sn− 1 ) 2 Znq: the LWE secret key
```

```
Input:
```

```
8
>>>
<
>>>
:
```

```
ctin= (a 0 ;; an− 1 ; b) 2 LWEs(me)Znq+1
w: the output ciphertext modulus
DriftMitigation: Boolean value indicating if the drift mitigation technique should be applied
DMK: drift mitigation key, used whenDriftMitigationis True.
Output: ctout 2 LWEs
```

```
j
me·w
q
```

```
m
Znw+1
1 if DriftMitigation then
2 ct′=

a′ 0 ;; a′n− 1 ; b′

DriftMitigation(ctin;DMK)
3 else
4 ct′=

a′ 0 ;; a′n− 1 ; b′

ctin
5 for 0 i < n do
6 a′′i
```

```
hja′
i·w
q
```

```
mi
w
7 b′′
```

```
hjb′·w
q
```

```
mi
w
8 ctout
```

```

a′′ 0 ;; a′′n− 1 ; b′′
```

```

9 return ctout
```

To apply this technique, additional public key material is required, specifically a set of encryptions
of zero calledDMK=

##### 

ctz 2 LWEs(0)Znq+1
z∈[1,Z]. Additionally, an extra computation step is
needed: an encryption of zero must be added to the LWE ciphertext input before performing modulus
switching.
LetTdenote the bound on the maximum allowed modulus switch noise (in absolute value) andrthe
parameter such that the probability of the modulus switch noise lying in the interval[μ ̃r; ̃ ̃μ+r ̃]
[T;T], with ̃μand ̃computed as in Algorithm 15 , equals 1 erfc(r/

p
2). The total number of
encryptions of zero needed in the public key material is set such that the chance of not finding an
encryption of zero that makes the modulus switch noise satisfy the upper boundTis negligible.^4 The
usage of the drift mitigation is indicated with a boolean input valueDriftMitigationto the modulus
switching given in Algorithm 14 , which is by default set to true as the cost of adding an encryption of
zero is negligible compared to the benefits of the noise reduction it triggers.

**Theorem 8 (Modulus Switch)** _Let_ q _and_ w _be two ciphertext moduli. Let_ s _be an LWE secret key
such that_ s= (s 0 ;;sn− 1 )_. Let_ ctin _be an LWE ciphertext (Definition 3 ) such that_ ctin 2 LWEs(me)
Znq+1 _, so that it has ciphertext modulus_ q_.
The output of the modulus switch operation (Algorithm 14 ) is a ciphertext_ ctout _with ciphertext_

_modulus_ w _such that_ ctout 2 LWEs

```
j
m ̃wq
```

m
Znw+1_.
Let us assume that_ w _divides_ q _and that_ q/w _is even, as is the case for the modulus switch in the_
PBS _and denote the variance of the noise in_ ctin _by_ in^2_. If the boolean_ DriftMitigation _is false, then the
variance of the noise in_ ctout _is:_

```
Var(MS) =^2 in
```

```
w^2
q^2
```

##### +

##### 1

##### 12

```
+cvar
```

```
w^2
q^2
```

```
+n
```

##### 

##### 1

##### 12

```
+cvar
```

```
w^2
q^2
```

##### 

```
Var(si) +
```

##### 

##### 1

##### 12

```
+cexp
```

```
w^2
q^2
```

##### 

```
E^2 (si)
```

##### 

##### ; (14)

_otherwise, the boolean_ DriftMitigation _is true and, denoting the variance of the noise of the selected
fresh encryption of zero,_ ctindex _, by_ index^2 _, then the variance of the noise in_ ctout _is:_

(^4) If it happens that no encryption of zero results in a modulus switch noise that is lower than the upper bound, the
encryption of zero that is closest to the upper bound is taken. However, in practice, this should never happen as we can
choose the probability of not finding a candidate in the list to be negligible compared with the failure probabilitypfail.

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

```
Algorithm 15: ctout DriftMitigation(ctin;DMK)
```

```
Context:
```

```
(
s= (s 0 ;; sn− 1 ) 2 Zn: the input binary LWE secret key
T;r; Z:the drift parameters
```

```
Input:
```

```
(
ctin 2 LWEs(me)Znq+1;whereme= ∆m
DMKthe drift mitigation key
Output: ctout 2 LWEs(me)
1 index= 0
2 ctin= (ain, 0 ;; ain,n− 1 ; bin)
3 ̃μ
```

```
j
bin^2 Nq
```

```
mq
2 Nbin
```

```


Pn− 1
j=0
```

```
j
ain,j^2 qN
```

```
mq
2 Nain,j
```

```

^12
4 ̃^2
Pn− 1
j=0
```

```
j
ain,j^2 qN
```

```
m
q
2 Nain,j
```

```
 2
^14
5 Tbest=j ̃μj+r ̃
6 index= 0
7 while j ̃μj+r ̃Tandindex< Z do
8 index index+ 1
9 ctDriftMitigation= (a 0 ;; an− 1 ; b) Add(ctin;ctindex)
10 μ ̃
```

```
j
b^2 Nq
```

```
mq
2 Nb
```

```


Pn− 1
j=0
```

```
j
aj^2 qN
```

```
mq
2 Naj
```

```

^12
11  ̃^2
Pn− 1
j=0
```

```
j
aj^2 Nq
```

```
mq
2 Naj
```

```
 2
^14
12 Tindex=j ̃μj+r ̃
13 if Tindex<Tbest then
14 Tbest=Tindex
15 indexbest=index
16 if index=ZandTindex>Tandindexbest 6 = 0 then
17 ctDriftMitigation Add
```

```

ctin;ctindexbest
```

```

```

```
18 return ctout=ctDriftMitigation
```

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

Var(MS) =

#####

```
^2 in+^2 index
```

```
w^2
q^2
```

##### +

##### 1

##### 24

```
+cvar
```

```
w^2
2 q^2
```

##### +

```
n
2
```

##### 

##### 1

##### 12

```
+cvar
```

```
w^2
q^2
```

##### 

```
Var(si) +
```

##### 

##### 1

##### 12

```
+cexp
```

```
w^2
q^2
```

##### 

```
E^2 (si)
```

##### 

##### 

##### (15)

_Here again_ cvar _and_ cexp _are the same small constants that depend on the exact implementation of the
rounding operation_^5_.
The complexity of the modulus switch without drift mitigation is:_

CMS= (n+ 1)C(q,w)(Round): (16)
_Where_ C(q,w)(Round) _denotes the complexity of performing a modulus switch from_ q _to_ w _on a
single element of_ Zq_. If the average number of encryptions of zero we need to try before satisfying the
condition_ jμ ̃j+r ̃T _of the drift mitigation is given by_ avg^6 _, the complexity of the modulus switch
with drift mitigation is on average:_

```
CMS=avg(n+ 1)C(q,w)(Round) +avgC(Add): (17)
```

#### 2.6.2 External Product

In Algorithm 16 , we describe the external product. The external product takes as inputs a GGSW
ciphertext (Definition 8 ) encrypting a plaintextM 2 Rq,Nand a GLWE ciphertext (Definition 4 )

encrypting the encoded messageMfand returns a GLWE encryption ofMMf.

```
Algorithm 16: CTout ExternalProduct
```

##### 

```
CTin;CT
```

##### 

```
Context:
```

```
8
>>>
>>>
<
>>>
>>>
:
```

```
S= (S 0 ;; Sk− 1 ) 2 Rkq,N:the GLWE secret key
CTin= (A 0 ;; Ak− 1 ; B) 2 Rkq,N+1: the input GLWE ciphertext
CT=
```

```

CT 0 ;;CTk
```

```

2 R(q,Nk+1)×ℓ×(k+1):the input GGSW ciphertext
`:the decomposition level
B:the decomposition base
```

```
Input:
```

```
8
<
:
```

```
CTin 2 GLWES
```

```

Mf
```

```

;withMf 2 Rq,N
CT 2 GGSWBS,ℓ(M);withM 2 Rq,N
Output: CTout 2 GLWES
```

```

MMf
```

```

```

```
1 CTout
```

```
D
CTk;BalDec(B,ℓ)(B)
```

```
E
```

```
2 for 0 ik 1 do
3 CTout CTout+
```

```
D
CTi;BalDec(B,ℓ)(Ai)
```

```
E
```

```
4 return CTout
```

**Theorem 9 (External Product)** _Let_ CTin 2 GLWES

##### 

```
Mf
```

##### 

```
Rkq,N+1 be a GLWE ciphertext encrypt-
```

_ing the encoded message_ Mf 2 Rq,N _, under the GLWE secret key_ S= (S 0 ;:::;Sk− 1 ) 2 Rkq,N+1 _, with
noise coefficients which can be viewed as having been sampled from_ σ_._

```
Let M 2 Rq,N be a message and CT=
```

```
n
CTi 2 GLevBS,ℓ(MSi)R
ℓ×(k+1)
q,N
```

```
o
0 ≤i≤k− 1
```

```
be a GGSW
```

_ciphertext with noise sampled from_ σBSK _and where_ Sk= 1_._

(^5) An example of a rounding operation and its corresponding concrete values forcvarandcexpare given in Remark 19.
(^6) This value can be computed based on formula 6.2 of [BJSW24].

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

_In the special case where_ M _is a constant polynomial drawn uniformly from_ f 0 ; 1 g _and_ q/Bℓ _is
even, the variance of the noise of the coefficients after the external product is:_

```
Var(ExternalProduct) =
```

##### ^2

##### 2

```
+`(k+ 1)N
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### BSK^2 +

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

```
+kN
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

##### 

```
Var(si) +
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cexp
2
```

##### 

```
E^2 (si)
```

##### 

```
+FFTNoiseVar(B;`;k;N)
```

##### (18)

_where_ E(si) _and_ Var(si) _are the expectation and the variance of the secret key coefficients. The term_
FFTNoiseVar(B;`;k;N) _denotes the noise which is added due to the non-exact nature of the floating
point implementation of the FFT algorithm that is used to perform polynomial multiplication; cf.
Remark 22.
The complexity of the algorithm is:_

```
CExternalProduct= (k+ 1)NC(ℓ)(BalDec) +`(k+ 1)C(N)(FFT)
+ (k+ 1)`(k+ 1)NC(N)(multFFT)
+ (k+ 1)(`(k+ 1)1)NC(N)(addFFT)
+ (k+ 1)C(N)(iFFT)
```

##### (19)

_Where_ C(N)(FFT);C(N)(iFFT) _denote the cost of performing a size_ N _FFT and inverse FFT, respec-
tively, and_ C(N)(addFFT);C(N)(multFFT) _denotes the cost of performing addition and multiplication
of size_ N _in the Fourier domain._

**Remark 22 (FFT Noise)** _As mentioned in the above theorem, we assumed that the FFT algorithm
was used to perform polynomial multiplication, and as such the non-exact nature of the implementation
of this algorithm adds a noise term to the output ciphertext. In particular, this means the FFT noise
is implementation-dependent and can be the dominant noise term depending on the implementation
and the parameters used. Due to this fact, we will not state the formula for the FFT noise variance,_
FFTNoiseVar _, here explicitly. If an exact method for polynomial multiplication is used, then the variance
of the error after the external product would not contain the_ FFTNoiseVar(B;`;k;N) _term.
In the following Theorems, we will assume again that the FFT is used whenever we need to perform
polynomial multiplication. As such, the same comments also apply to these scenarios as well._

#### 2.6.3 CMUX

TheCMuxis a homomorphic selector. Given two GLWE ciphertexts encrypting two encoded poly-
nomial messagesMf 0 andMf 1 and a GGSW ciphertext encrypting a single bit (the selector bit),

theCMuxreturns a GLWE encryption ofgMβ. Intuitively, the algorithm homomorphically computes
gMβ= (Mf 1 Mf 0 ) +Mf 0.

This algorithm is built on top of the external product (Algorithm 16 and Theorem 9 ) and is the
cornerstone of TFHE’sPBS. The algorithm is detailed in Algorithm 17 and in the following theorem,
we give the noise formula for aCMux.

**Theorem 10 (CMUX)** _For_ i2 f 0 ; 1 g _, let_ CTi 2 GLWES

##### 

```
Mfi
```

##### 

```
Rkq,N+1 be two GLWE ciphertexts
```

_encrypting encoded messages_ Mfi 2 Rq,N _under the same GLWE secret key_ S= (S 0 ;:::;Sk− 1 ) 2 Rkq,N+1 _,
with noise whose coefficients can be seen as having been sampled from_ σ_. Let_ 2f 0 ; 1 g _be a bit. Let_

CT=

```
n
CTi 2 GLevBS,ℓ( Si)R
ℓ×(k+1)
q,N
```

```
o
0 ≤i≤k
be a GGSW with noise sampled from σBSK.
```

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

```
Algorithm 17: CTout CMux
```

##### 

##### CT 0 ;CT 1 ;CT

##### 

```
Context:
```

```
8
><
>:
```

```
S 2 Rkq,N:the GLWE secret key
`:the decomposition level
B:the decomposition base
```

```
Input:
```

```
8
>><
```

```
>>:
```

```
CT 02 GLWES
```

```

gM 0
```

```

;withgM 02 Rq,N
CT 12 GLWES
```

```

gM 1
```

```

;withgM 12 Rq,N
CT 2 GGSWBS,ℓ( );with 2f 0 ; 1 g
Output: CTout 2 GLWES
```

```

gMβ
```

```

```

```
1 CTout ExternalProduct
```

```

CT 1 CT 0 ;CT
```

```

+CT 0
2 return CTout
```

```
The variance of the noise coefficients of CTout in Algorithm 17 , assuming again that q/Bℓ is even:
```

```
Var(CMux) =^2 +`(k+ 1)N
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### ^2 BSK+

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

```
+kN
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

##### 

```
Var(si) +
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cexp
2
```

##### 

```
E^2 (si)
```

##### 

```
+FFTNoiseVar(B;`;k;N)
```

##### (20)

```
The complexity of Algorithm 17 is:
```

```
CCMux= (k+ 1)NC(ℓ)(BalDec) +`(k+ 1)C(N)(FFT)
+ (k+ 1)`(k+ 1)NC(N)(multFFT)
+ (k+ 1)(`(k+ 1)1)NC(N)(addFFT)
+ (k+ 1)C(N)(iFFT) + 2(k+ 1)NC(add):
```

##### (21)

#### 2.6.4 Blind Rotate

The blind rotate algorithm is one of the three main building blocks of thePBS, it is by far the most
complex and costly part. It consists in homomorphically rotating a lookup table. Given a GLWE
ciphertextCTencrypting an encoded messageMfand an LWE ciphertext modulo 2 Nencrypting an
encoded messageme with noisee, it returns a GLWE ciphertext encryptingMfX−me−e 2 Rq,N,
i.e., we rotate one ciphertext by the (negative) phase of another. We know that the constant term
ofMfX−me−eis the(me+emodN)-th coefficient ofMfup to sign, with the sign being positive if
0 me+e < Nand the sign being negative otherwise.
We need to introduce the concept of a redundant lookup table to deal with the noise terme. The
motivation behind these lookup tables will be explained in the next part when we will introduce the
PBSbut essentially they are used to perform the decoding operation.

**Definition 13 (Redundant Lookup Table)** _A_ redundant LUT _is a lookup table encoding a func-
tion_ f _, whose entries are redundantly represented inside the coefficients of a polynomial in_ Rq,N_. In
practice, the redundancy consists in an_ r _-times (with_ r _a system parameter) repetition of the entries_
f(i) _of the_ LUT _with a certain shift. We use the term_ mega-cell _for each block of successive redundant
values in an_ r _-redundant lookup table:_

2.6 Building Blocks of the PBS 2 THE TFHE SCHEME

```
Algorithm 18: ctout BlindRotate(ctin;BSK;CTf)
```

```
Context:
```

```
8
><
>:
```

```
s= (s 0 ;; sn− 1 ) 2 Zn: the LWE input secret key withsi U$ (f 0 ; 1 g)
S= (S 0 ; : : : ; Sk− 1 ) 2 Rkq,N: the output GLWE secret key
Mff 2 Rq,N:a redundantLUTfor a function:x7!f(x)
```

```
Input:
```

```
8
>><
```

```
>>:
```

```
ctin= (a 0 ;; an− 1 ; b) 2 LWEs(me)Zn 2 N+1;for someme 2 Z 2 N
BSK=
```

```
n
CTi 2 GGSWBS,ℓ(si)
```

```
on− 1
i=0
:a bootstrapping key fromstoS
CTf 2 GLWES
```

```

Mff
```

```

2 Rkq,N+1:an encrypted (possibly trivially) redundant LUT
Output: CTout 2 GLWES
```

```

MffX−me−e
```

```

, whereeis the noise inctin
1 CTout CTfX−b
2 for 0 in 1 do
3 CT 0 CTout
4 CT 1 CToutXai
5 CTout CMux
```

```

CT 0 ;CT 1 ;CTi
```

```

```

```
6 return CTout
```

```
Mff=X−r/2
```

```
NX/r− 1
```

```
i=0
```

```
Xi·r
```

##### 0

##### @

```
rX− 1
```

```
j=0
```

```
fg(i)Xj
```

##### 1

##### A (22)

_with_ fg(i) _an encoding of a message_ f(i)_. Sometimes we will write this lookup table using just the
mega-cell values as_ 

```
fg(0);;f^N
r^1
```

##### 

##### 

##### 

_The redundancy is used to perform the rounding operation needed to remove the noise during boot-
strapping._

The blind rotate operation requires some public key material, which we call the bootstrapping key
BSK. Informally, the bootstrapping key is a collection ofGGSWciphertexts, one for each element of
the input LWE secret key. Formally, we give the following definition:

**Definition 14** _Let_ B _and_ ` _be a pair of decomposition parameters, i.e., a decomposition base and
level. Given an input binary LWE secret key_ s= (s 0 ;;sn− 1 ) _and an output GLWE secret key_ S _,
the corresponding bootstrapping key is the set of_ GGSW _ciphertexts:_

##### BSK=

```
n
CT 2 GGSWBS,ℓ(si)
```

```
on− 1
i=0
```

##### 

```
We describe the blind rotate in Algorithm 18 and in Theorem 11.
```

**Theorem 11 (Blind Rotate)** _Let_ ctin 2 LWEs(me) _be an LWE ciphertext encrypting an encoded
message_ me _with ciphertext modulus_ 2 N _and LWE secret key_ s= (s 0 ;;sn− 1 ) 2 Zn _with all_ si

_binary. Let_ CTf 2 GLWES

##### 

```
Mff
```

##### 

```
Rkq,N+1 be a GLWE ciphertext encrypting a redundant LUT for a
```

_function_ f _with_ S 2 Rkq,N_. Let_ BSK _be a bootstrapping key from_ s _to_ S _with decomposition parameters_

B _and_ ` _, i.e.,_ BSK=

```
n
CTi 2 GGSWBS′,ℓ(si)
```

```
on− 1
i=0
```

_. Suppose further that the noises in all the GLWE_

_ciphertexts in_ BSK _have coefficients independently drawn from a distribution_ σBSK_. Also, suppose that_

2.7 PBS & Its Variants 2 THE TFHE SCHEME

_the noise in_ ctin _is_ e _and that the noise in_ CTf _can be seen as having been sampled from a distribution_
σf_._

```
Algorithm 18 outputs a ciphertext CTout 2 GLWES
```

##### 

```
MffX−me−e
```

##### 

```
and the variance of the coeffi-
```

_cients of the noise in_ CTout _is:_

```
Var(BR) =^2 f+n
```

##### 

```
`(k+ 1)N
```

##### 

##### B^2

##### 12

##### +"(B;`)

##### 

##### ^2 BSK+

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

```
+kN
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cvar
2
```

##### 

```
Var(Si,j) +
```

##### 

```
q^2
24 B^2 ℓ
```

##### +

```
cexp
2
```

##### 

```
E^2 (Si,j)
```

##### 

```
+nFFTNoiseVar(B;`;k;N)
```

##### (23)

_where_ Var(Si,j) _and_ E^2 (Si,j) _are the variance and expectation squared of the GLWE secret key coeffi-
cients. Note that the output noise does not depend on the noise in_ ctin_.
The complexity of the blind rotate is:_

```
CBR=nCCMux (24)
```

In aPBS,CTfis typically a trivial encryption ofMff(Definition 6 ) so that it can be computed
publically from knowledge off, in this caseσf is the constant zero distribution so^2 f= 0. If one
does not want the functionfto be public knowledge, thenCTfneeds to be given as part of the public
key material for allfthat are to be used.

**Remark 23 (Interest of the Modulus Switch in the** PBS **)** _To make the blind rotate work (Algo-
rithm 18 ), we need_ 2 N _to be equal to the ciphertext modulus of the input ciphertext, which is impractical
for_ q= 2^64_. As the complexity of the blind rotate depends on_ N _, we want_ N _as small as possible. To
this end, we perform a modulus switch before a blind rotate to reduce the ciphertext modulus. This
operation adds a lot of noise but allows to choose polynomial sizes much smaller than the size of_ q_._

### 2.7 PBS & Its Variants

The Programmable Bootstrapping (PBS) is a fundamental building block of TFHE, enabling both
noise reduction and homomorphic function evaluation on encrypted data. As previously discussed in
Remark 23 , one of the key challenges in implementing a PBS efficiently is handling the ciphertext mod-
ulus. The modulus switch step is crucial in reducing the ciphertext modulus before performing a blind
rotation, allowing smaller polynomial sizes while keeping the computational complexity manageable.
In this section, we present the classical PBS and explore its variants, which aim to optimize its
performance and extend its capabilities. We first formalize the PBS, detailing its major steps: the
modulus switch, the blind rotate, and the sample extract. We then introduce several variants, including
the multi-bit blind rotate, which enhances parallelizability, and the ManyLUT PBS, which enables
multiple function evaluations in a single PBS. Additionally, we discuss alternative approaches to blind
rotation and conclude with the concept of atomic patterns, which provide structured ways to compose
PBS-based operations efficiently.

#### 2.7.1 Programmable Bootstrap (PBS)

We have introduced in the previous parts all the building blocks needed to explain thePBS. The
Programmable Bootstrap operation [CGGI20,CJL+ 20 ,CJP21], orPBS, is an FHE algorithm that
achieves two goals. First, it resets the noise in a ciphertext to a fixed level (when certain conditions are
fulfilled) that is independent of the input noise level. Second, it allows to homomorphically evaluate,
at the same time, a lookup table on the encrypted message, i.e., apply an appropriate univariate

2.7 PBS & Its Variants 2 THE TFHE SCHEME

function. Such an operator takes as input an LWE ciphertext encrypting an encoding of the message
m, a bootstrapping keyBSK(Definition 14 ), an encryption (potentially trivial) of a redundant lookup

tableMff (Definition 13 ), and outputs an LWE ciphertext with a fixed level of noise encrypting an

encodingf](m)of the messagef(m)when the process is successful. When the redundancy matches
the encoding used, the algorithm outputs the correct output up to the failure probabilitypfailthat can
be estimated using the variance of the noise of the input LWE ciphertext and the parameters used
during blind rotate.
ThePBSis composed of three major steps: the modulus switch (Algorithm 14 and Theorem 8 ), the
blind rotate (Algorithm 18 and Theorem 11 ), and the sample extract (Algorithm 7 and Theorem 5 ).

```
Algorithm 19: ctout PBS
```

```

CTf;ctin;BSK;DriftMitigation;DMK
```

```

```

```
Context:
```

```
8
>>>
>>>
<
>>>
>>>
:
```

```
s= (s 0 ;; sn− 1 ) 2 Zn:the input binary LWE secret key
S′=
```

```

S′ 0 ; : : : ; S′k− 1
```

```

2 Rkq,N: a GLWE secret key whereS′i=
PN− 1
j=0s′i·N+jXjfor alli
s′= (s′ 0 ;; s′kN− 1 ) 2 ZkNq : the output LWE secret key
Mff 2 Rq,N:a redundantLUTfor a functionf
Band`are decomposition parameters
```

```
Input:
```

```
8
>>>
>>>
<
>>>
>>>
:
```

```
CTf 2 GLWES′
```

```

Mff
```

```

Rkq,N+1
ctin 2 LWEs(me)Znq+1;whereme= ∆m
BSK: a bootstrapping key fromstoS′
DriftMitigation: Boolean value indicating if the drift mitigation should be applied
DMK: drift mitigation key, only used whenDriftMitigationis True
Output: ctout 2 LWEs′
```

```

^f(m)
```

```

if we respect the requirements of Theorem 12
1 ctMS ModulusSwitch(ctin; 2 N;DriftMitigation;DMK)
2 CT BlindRotate
```

```

ctMS;BSK;CTf
```

```

3 ctout SE(CT;0)
4 return ctout
```

**Theorem 12 (Programmable Bootstrap (PBS))** _Let_ s= (s 0 ;;sn− 1 ) 2 Zn _be a binary LWE_

_secret key. Let_ S′=

#####

```
S 0 ′;:::;S′k− 1
```

##### 

```
2 Rkq,N be a GLWE secret key such that S′i=
```

##### PN− 1

```
j=0 s
′
i·N+jX
j , and
```

_denote by_ s′= (s′ 0 ;;s′kN− 1 ) _the corresponding flattened LWE secret key. Let_ BSK _be the associated
bootstrapping key (Definition 14 ) from_ s _to_ S′_. Assume further that_ q _is a power of two larger than_ N_.
Suppose we have an LWE ciphertext_ ctin 2 LWEs(me) _where_ me _is the encoding_ me= ∆m _of a
message_ m 2 Z 2 p _for some_ p _dividing_ N _and_ ∆ = 2 qp _, and suppose the noise in_ ctin _can be viewed as
having been sampled from the noise distribution_ σin_._

```
Let Mff be the Np -redundant lookup-table (Definition 13 ) for a function f:Zp!Zq and let CTf be
```

_a (potentially trivial) GLWE encryption of_ Mff_._

_Then Algorithm 19 outputs an LWE ciphertext_ ctout _under the secret key_ s′ _encrypting_ f](m) _, i.e.,
an encoding of_ f(m) _with a probability_ 1 pfail _if and only if both_

- _the input noise has variance satisfying_

```
in^2 <
```

##### ∆^2

```
4 z∗(pfail)^2
```

#####

##### 1

##### 1 +D

##### 

```
q^2
48 N^2
```

```
+cvar+n
```

##### 

```
q^2
96 N^2
```

##### +

```
cvar+cexp
4
```

##### 

```
D^2 index;
```

```
where D= 1 if DriftMitigation is true, else D= 0 , and where z∗(pfail) is the standard score
associated to the failure probability pfail given by
```

```
z∗(pfail) =
```

```
p
2 erf−^1 (1pfail)
```

2.7 PBS & Its Variants 2 THE TFHE SCHEME

```
and erf−^1 is the inverse of the error function erf(z) =√^2 π
```

```
Rz
0 e
```

```
−t^2 dt ;
```

- _and either_ 0 m < p _or both_ pm < 2 p _and_ f(m) =f(mp) _holds._

_The output noise after the_ PBS _is that of the blind rotate operation, that is we have_ Var(PBS) =
Var(BlindRotate) _from Eq._ ( 23 )_.
The complexity of Algorithm 19 is:_

```
CPBS=CMS+CBR+CSE: (25)
```

_The size (in bits) of the traditional bootstrapping key_ Size(BSK) _is computed with the following
formula:_
Size(BSK):=n`PBS(k+ 1)^2 Ndlog 2 (q)e:

**Remark 24 (Negacyclic Function)** _The second condition in Theorem 12 essentially states that we
can only use half the message space_ (Z 2 p) _when applying a_ PBS _with the exception that we can use the
full message space if_ f:Z 2 p!Zq _has the following property:_

```
f(x+p) =f(x) for all x 2 Z 2 p: (26)
```

_This is why, in general, we need to use at least one bit of padding in our encoding. If_ f _satisfies
Eq._ ( 26 ) _, then the function_ f _is said to be negacyclic and we do not require a padding bit for the PBS
to work. For odd message modulus (see Definition 9 ), it is also possible to use an arbitrary_ f _without
using a padding bit._

**Remark 25 (Independence of input and output encoding)** _We point out that the encoding used
in the input LWE of the PBS and the encoding used in the output LWE (determined via the encoding
in the redundant_ LUT _) are independent from one another. They can differ in their respective number
of padding bits_  _and in their respective message modulus_ p_. The output encoding can even use a
completely different style of encoding if so desired._

**Remark 26 (General ciphertext modulus)** _Theorem 12 assumes that the ciphertext modulus_ q _is
a power of two, this restriction can be lifted without any change to Algorithm 19 but the noise from
both the modulus switch and the blind rotate will be slightly different.
Furthermore, we do not actually require that the ciphertext modulus of the input LWE is_ q _as long
as we still correctly modulus switch this ciphertext to modulus_ 2 N _in the first step. For example, one
can use a PBS to change the ciphertext modulus from say_ 264 _to_ 2128 _using a bootstrapping key with
modulus_ 2128_._

**Remark 27 (Key Switch and PBS)** _More often than not, we apply a key switch before a_ PBS _with
the idea that we keyswitch to a smaller LWE dimension and thus have to perform fewer_ CMux _operations
during the blind rotate. For simplicity, we will sometimes refer to the sequential combination of a key
switch followed by a_ PBS _as a_ KS _-_ PBS_._

**Remark 28 (Using a carry space to enable multivariate function evaluation)** _Natively, the
PBS operation can only apply a univariate function, however by using an encoding with a carry space
it is possible to homomorphically compute a bivariate (or even multivariate) function such as multipli-
cation. This trick was proposed in [CZB_ + _22 ] and works for a suitably large carry space. Assuming the
carry spaces have been emptied, the idea is to concatenate two messages_ m 1 _and_ m 2 _(or more) respec-
tively encrypted in_ ct 1 _and_ ct 2 _by rescaling the first one with constant multiplication by_  2 +1 _(where_  2
_is the largest possible value that can be taken by_ m 2 _) and adding it to_ ct 2 _and finally computing a PBS
on the result. Once the two messages are concatenated into a single ciphertext, the bivariate function_

2.7 PBS & Its Variants 2 THE TFHE SCHEME

```
Inputs:
```

##### ?

##### 000

##### ?

##### 000

```
Shift: 0 00 000
```

```
Addition:^0
```

```
Result: 000
```

```
·β
```

##### +

```
PBS
```

```
Figure 6:Example of a bivariate function evaluation via a shift, an addition, and a PBS.
```

_can be simply evaluated as a univariate function on the concatenation of_ m 1 _and_ m 2_. A visual example
is given in Fig. 6. As mentioned, this gives one way to be able to perform a multiplication using a
single PBS operation. Regarding the functions computed in a bivariate bootstrapping, any bivariate
function_ g _can be written as a univariate one:_ g(a;b) =f(a+bm) _, with_ f _defined as in Remark 24 ,_
a 2 Nmsg _and_ b 2 Ncarry_. This gives the possibility of easily reusing the previous encoding for_ LUT_. The
signature of the algorithm is:_

```
ctout BivPBS(CTf;ctin 1 ;ctin 0 ;BSK;DriftMitigation;DMK)
```

#### 2.7.2 Multi-Bit Blind Rotate

We note that the blind rotation step of the PBS is not well suited to benefit from parallelism. In
this section, we give an alternative blind rotation algorithm that can make the PBS operation more
easily parallelizable. We refer to this variant as the _Multi-Bit Blind Rotate_ and it was introduced
in [ZYL+ 18 ]. It performs the blind rotation in chunks, whose length is referred to as the _grouping
factor_ and denotedgf. This variant is practically employed in the GPU implementation of the Blind
Rotate algorithm thanks to its better parallelizability, although its overall cost is higher than that
of the vanilla variant described in the previous section, which is used in the CPU implementation.
Furthermore, the size of the multi-bit bootstrapping key grows exponentially with the grouping factor.
Likewise to the vanilla variant, the multi-bit one aims to blindly multiply the input GLWE, which
holds theLUT, byX

```
Pa
isi. However, instead of applying one key bit per step viaCMux(cf. Line 5 of
```

Algorithm 18 ), here we apply a chunk ofgfbits via another update step (cf. Line 3 of Algorithm 20 ).
For instance, forgf= 2, the expanded multi-bit update step writes

```
CTout
```

##### 

```
X^0 CTi,∅+Xa^2 i+0CTi,{ 0 }+Xa^2 i+1CTi,{ 1 }+Xa^2 i+0+a^2 i+1CTi,{ 0 , 1 }
```

##### 

```
CTout (27)
```

whereCTi,∅ 2 GGSWBS,ℓ

#####

```
(1s 2 i+0)(1s 2 i+1)
```

##### 

```
,CTi,{ 0 } 2 GGSWBS,ℓ
```

#####

```
s 2 i+0(1s 2 i+1)
```

##### 

```
, etc.
In general, for a fixedi, note that exactly one of
```

```
n
CTi,J
```

```
o
J⊆{ 0 ,...,gf− 1 }
```

```
encrypts 1 : this is for
```

J ̄such thatQ
j∈J ̄si·gf+j

##### Q

```
j∈/J ̄(1si·gf+j) = 1, i.e.,
J ̄=fjjsi·gf+j= 1g; the others encrypt zero.
```

It follows that the newCToutencrypts the plaintext of the oldCToutmultiplied byX

```
P
j∈J ̄ai·gf+j=
```

X

```
Pgf− 1
j=0ai·gf+jsi·gf+j.
```

2.7 PBS & Its Variants 2 THE TFHE SCHEME

```
Algorithm 20: ctout MultiBitBlindRotate(ctin;MB-BSK;CTf)
```

```
Context:
```

```
8
>>>
><
>>>
>:
```

```
gf 2 N:grouping factor, typically a small integer2f 2 ; 3 ; 4 g;wheren=gfyfor somey 2 N
s= (s 0 ;; sn− 1 ) 2 Zn:the LWE input secret key withsi U$ (f 0 ; 1 g)
S= (S 0 ; : : : ; Sk− 1 ) 2 Rkq,N:the output GLWE secret key
Mff 2 Rq,N:a redundantLUTfor a functionx7!f(x)
```

```
Input:
```

```
8
>>>
>>>
<
>>>
>>>
:
```

```
ctin= (a 0 ;; an− 1 ; b) 2 LWEs(me)Zn 2 N+1;for someme 2 Z 2 N
MB-BSK=
```

```
n
CTi,J 2 GGSWBS,ℓ
```

```
Q
j∈Jsi·gf+j
```

```
Q
j∈/J(1si·gf+j)
```

```
o
J⊆{ 0 ,...,gf− 1 }
```

```
y− 1
```

```
i=0
```

```
:
a multi-bit bootstrapping key fromstoS
CTf 2 GLWES
```

```

Mff
```

```

Rkq,N+1:an encrypted (possibly trivially) redundantLUT
Output: CTout 2 GLWES
```

```

MffX−me−e
```

```

, whereeis the noise inctin
1 CTout CTfX−b
2 for 0 iy 1 do
3 CTout
```

```
P
J⊆{ 0 ,...,gf− 1 }X
```

```
P
j∈Jai·gf+jCTi,J
```

```

CTout
4 return CTout
```

**Remark 29 (Smaller Multi-Bit Bootstrapping Key)** _Since we know that for a fixed_ i _exactly_

_one of_ CTi,J _encrypts 1 and the rest encrypts zero, we know that their sum encrypts one. This can be_

_used to remove the need to provide_ CTi,J _for one chosen_ J _, say_ J=; _, and hence reduce the multi-bit
bootstrapping key size by_ ngf _GGSW ciphertexts in total. See [BMMP18] for more information. For the_

_case of_ gf= 2 _, Eq._ ( 27 ) _is replaced by_

CTout

##### 

```
(Xa^2 i1)CTi,{ 0 }+ (Xa^2 i+11)CTi,{ 1 }+
```

#####

```
Xa^2 i+a^2 i+1 1
```

##### 

```
CTi,{ 0 , 1 }
```

##### 

```
CTout+CTout:
```

#### 2.7.3 ManyLUT PBS

In Remark 24 we noted that for the PBS to be correct for a function that is not negacyclic we needed
at least one bit of padding in our encoding. That is 1 in the notation of Definition 9. When we
have more than one bit of padding we can compute a PBS with more than one function at once using
a single blind rotation. This idea was first proposed in [CLOT21] and we present a slightly modified
version of it here.
Suppose we have > 1 and functionsf 1 ;;f 2 π− 1 which we want to apply to the same encoded
messageme= ∆mwherem 2 Zp, then we can construct a redundant lookup table from thefiand
a suitably largeNfor which we can apply the blind rotation algorithm a single time and from the
resulting GLWE ciphertext we can sample extract 2 π−^1 LWE ciphertexts which with high probability

are encryptions of^fi(m).

The idea is to define a functionf:Z 2 π− (^1) p!Zqfromf 1 ;;f 2 π− 1 as follows:
f(x) =fy+1(z);wherex=py+zand 0 y < 2 π−^1 ; 0 z < p; (28)
and note for the lookup table associated tofthat everypmega-cells we go fromfi(z)tofi+1(z)for
some 0 z < p. During blind rotation we will rotate the lookup table bymmega-cells so that (for
small enough noise) sample extracting (with = 0) will give an encryption off 1 (m), sample extracting
position =N/2π−^1 will give an encryption off 2 (m), and so on for all positions that are multiples
ofN/2π−^1. Details are provided in Algorithm 21.

2.7 PBS & Its Variants 2 THE TFHE SCHEME

```
Algorithm 21:
```

```

ctout, 1 ;;ctout, 2 π− 1
```

```

PBSmanyLUT

ctin;BSK;CTf;DriftMitigation

```

```
Context:
```

```
8
>>>
>>>
>><
```

```
>>>
>>>
>>:
```

```
s= (s 0 ;; sn− 1 ) 2 Zn:the input binary LWE secret key
S′=
```

```

S′ 0 ; : : : ; S′k− 1
```

```

2 Rkq,N: a GLWE secret key whereS′i=
PN− 1
j=0s′i·N+jXjfor alli
s′= (s′ 0 ;; s′kN− 1 ) 2 ZkNq : the output LWE secret key
Mff 2 Rq,N:a redundantLUTfor a functionfderived fromf 1 ;; f 2 π− 1 using Eq. ( 28 )
Band`are decomposition parameters
DMKthe drift mitigation key
```

```
Input:
```

```
8
>>>
><
>>>
>:
```

```
ctin 2 LWEs(me)Znq+1;whereme= ∆mform 2 Zpand∆ =bq/(2πp)c
BSK: a bootstrapping key fromstoS′
CTf 2 GLWES′
```

```

Mff
```

```

Rkq,N+1
DriftMitigation: Boolean value indicating if the drift mitigation should be applied
Output: ctout,i 2 LWEs′
```

```

f^i(m)
```

```

for 1 i 2 π−^1 if we respect the requirements of Theorem 12
1 ctMS ModulusSwitch(ctin; 2 N;DriftMitigation;DMK)
2 CT BlindRotate
```

```

ctMS;BSK;CTf
```

```

3 for 0 i < 2 π do
4 ctout,i+1 SE
```

```

CT; i 2 πN− 1
```

```

```

```
5 return
```

```

ctout, 1 ;;ctout, 2 π− 1
```

```

```

We remark that this technique works equally well if instead of assuming we have more padding
bits we assume we have an empty carry space when using the message-and-carry type encoding. The
number of functions one can homomorphically compute simultaneously is the size of the carry space
(assuming we also have one bit of padding).

#### 2.7.4 Alternative Approaches to the Blind Rotate

The blind rotate (Section2.6.4) is the most computationally expensive part of PBS and has been the
focus of extensive research for optimization. One such improvement is the multi-bit version presented
in Section2.7.2. The approach in Algorithm 18 only works if the LWE secret key is binary but can be
extended to ternary of even larger keys as detailed in [JP22].
In all the approaches we have seen so far, the idea is to start from (a trivial encryption of)Xai
and use the bootstrapping key to compute an encryption ofXaisi and multiply the accumulator
by the result, this is referred to as the GINX approach [GINX16]. One can instead consider the
opposite, starting from an encryption ofXsi(which would be part of the bootstrapping key), compute
an encryption ofXaisiand multiply the accumulator by it, this approach is the AP style [AP14].
This style naturally benefits from the use of automorphisms and a number of works have explored
this [LMK+ 23 ,Lee24]. Note that here one must also modify the modulus switch to round to the
nearest odd integer as only odd powers can be computed using an automorphism; this idea is pushed
even further in [WWL+ 24 ].
Another idea to speed up the blind rotate operator is using a non-standard secret key distribu-
tion [LMSS23].

#### 2.7.5 Atomic Patterns

As introduced in [BBB+ 23 ], the notion of an _atomic pattern_ refers to a subgraph of FHE operations
that outputs one (or more) ciphertexts. These atomic patterns are typically constructed from basic
operations such as the _homomorphic dot product_ as defined in Theorem 4 , the _key switch_ as defined

2.7 PBS & Its Variants 2 THE TFHE SCHEME

in Theorem 6 , and the _programmable bootstrap_ as defined in Theorem 12. The atomic pattern typ-
ically used in TFHE-rs is from [CJP21], denoted byA(CJP), which is composed of a dot product of
several input ciphertexts, followed by a keyswitch, and completed with a PBS. An atomic pattern
has a matching input and output type, meaning that they can be chained together to build larger
computations.

##### 3 ARITHMETIC OVER LARGE HOM. INTEGERS

## 3 Arithmetic over Large Homomorphic Integers

To encode larger integers—i.e., those that do not fit into a singleLWEciphertext (see Section2.3)—
the radix decomposition is used. Programmatically, a radix ciphertext, denoted

##### 

ctand referred to
as a _homomorphic integer_ , is a list ofLWEciphertexts (aka. _blocks_ ), each encrypting a digit of the
decomposition of the original message. Essentially, we implement a homomorphic analogy to the
standard multiple precision arithmetic (like, e.g., the GMP Library^7 does in the clear), while instead
of cleartext machine words, which typically hold 32 or 64 bits of data, we deal withLWE-encrypted
chunks, which only hold small units of bits of data ( 2 bits in case of the default parameters of TFHE-rs).
In this section, we first describe the encoding in more detail, then we dive into algorithms for
individual arithmetic operations.

### 3.1 Radix Encoding of Large Homomorphic Integers

The radix encoding consists in decomposing a large integermmoduloΩ =

##### QB− 1

i=0^ iinto a list of digits
(mi)B−i=0^1 , such thatm=m 0 +m 1  0 +m 2  0  1 +:::+mB− 1 βB−Ω 1 and 0 mi< i. Each digit
miis individually encoded in the carry-message format introduced in Section2.3and encrypted into
anLWEciphertext. Together, these ciphertexts form the homomorphic integer

##### 

```
ct, which encryptsm
```

moduloΩ. The encoding ofmiis defined according to a pair( (^) i;pi) 2 N^2 of parameters, such that
2  (^) ipi, which respectively corresponds to the size of the message subspace and the carry-message
space (i.e., in the carry-message paradigm, we have (^) i=msgiandpi=msgicarryi); Fig. 7 gives a
visual representation of a toy example.
?
0
p 0
00
(^0)

##### 

```
e 0
```

```
ct 02 LWEs(me 0 )
```

##### ?

##### 0

```
p 1
```

```
00
```

(^1)

##### 

```
e 1
```

```
ct 12 LWEs(me 1 )
```

##### ?

##### 0

```
p 2
```

```
00
```

(^2)

##### 

```
e 2
```

```
ct 22 LWEs(me 2 )
```

Figure 7: Plaintext representation of a fresh radix-based large homomorphic integermof length

B= 3, where for all 0 i <Bwe have= 1, (^) i= 2^2 andpi= 22+2, hence working modulo
Ω = (2^2 )^3. For the encoding, it holds thatm=m 0 +m 1 4 +m 2  42. The symbol?represents the
padding bit needed for thePBS. For each block we havemfi=Encode

#####

```
mi; 2 1+2+2;q
```

##### 

##### 

**Definition 15 (Radix-Encoded Large Homomorphic Integer)** _Let_ Ω =

##### QB− 1

i=0^ i _, with_^ i^2
N> 1 _and_ B 2N _, be a message modulus,_ m 2 ZΩ _a message and let_ (mi)B−i=0^1 _be its respective radix
decomposition, i.e.,_

```
m=
```

##### B−X 1

```
i=0
```

```
mi
```

##### 0

##### @

```
iY− 1
```

```
j=0
```

(^) j

##### 1

##### A; (29)

_such that_ 0 mi< i_. Next, let_ (pi)B−i=0^1 _be the carry-message moduli as per Section2.3. We set_ mfi=
Encode(mi; 2 πpi;q) _with_  _the number of bits of padding and_ q _the_ LWE _ciphertext modulus. Finally,
we encrypt each_ mfi _, obtaining_ LWE _ciphertexts that form a vector_

##### 

ct _, referred to as a_ homomorphic
integer_.
To get back the original message_ m _, we simply recompose it using Eq._ ( 29 ) _where the_ mi _values are
obtained by decryption and decoding._

(^7) The GNU Multiple Precision Arithmetic Library,<https://gmplib.org>.

3.2 Notations and Assumptions 3 ARITHMETIC OVER LARGE HOM. INTEGERS

**Remark 30** _In practice, the restriction for_ Ω _is that it has to be a product of small bases. Indeed,
TFHE-like schemes do not scale well when one increases the precision, so the best practice is to keep_

pi 28_. Specifically, TFHE-rs employs by default the same parameters for all blocks:_ (^) i= 4 _and_
pi= 16 _, i.e.,_ msg=carry= 2^2 _, with_ = 1_._
To perform operations on radix-based integers, messages must be encoded and encrypted with the
same parameters to ensure compatibility. The majority of the arithmetic operations can be computed
by using a schoolbook approach (homomorphically mixing linear operations and PBSes) and by keeping
the degree of fullness (Definition 10 ) smaller thanpiin each block. When a carry subspace is full,
it needs to be propagated to the next block: this is done by extracting the carry and the message
(naïvely requiring twoPBSes) and adding the carry to the next block.
**Remark 31** _Signed integers are made using the two’s complement representation. Instead of con-
structing e.g. a 32-bit unsigned integer which is contained in the range_ [0; 232 1] _, a 32-bit signed
integer sits in the range_ [ 231 ; 231 1]_. An integer_ r _in this range is constructed as:_
r=r 31  231 +

##### X^30

```
i=0
```

```
ri 2 i:
```

**Remark 32** _The approach of splitting a message into multiple ciphertexts has already been proposed
for binary radix decomposition in FHEW [DM15] and TFHE [CGGI20], and for other representations
in [BST20], [GBA21], [KÖ22], [CZB_ + _22 ] and [LMP21]. However, none of them leverage carry buffers
to improve efficiency in computations on homomorphic integers—i.e., integers split across multiple
ciphertexts—by minimizing the number of bootstrappings.
In [GBA21], the authors propose two approaches to evaluate a generic_ LUT _over these multi-
ciphertexts inputs, namely the_ tree-based _and_ chaining-based _approaches (that we shorten as Tree-_
PBS _and Chained-_ PBS _), however, these methods become more and more inefficient as the number of
blocks goes increasing. The Chained-_ PBS _method is further generalized in [CZB_ + _22 ] to any function
in exchange for a larger plaintext space._

**Remark 33** _Alternatives to radix-based integer decomposition exist; in particular, decomposition based
on the_ Chinese Remainder Theorem _(CRT) is possible. However, for simplicity, we omit further details
in this paper._

### 3.2 Notations and Assumptions

Throughout Section 3 , we outline several algorithms on homomorphic integers and consider their
complexity. In particular, the complexity values given in this section denote a count of the number
of requiredPBSes to perform the algorithm. Moreover, we assume that the complexity of aPBSis
equivalent to that of a bivariatePBS, treating them as identical throughout using the notationCPBS.

#### 3.2.1 Example: Addition

Let us start with the algorithm computing the addition between two Radix-based ciphertexts (Algo-
rithm 22 ).

```
According to the algorithm signature
```

##### 

```
ct(out) Add
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

,Addtakes as input two ciphertext,
!
ct(1)and

##### 

```
ct(2), and outputs one ciphertext
```

##### 

ct(out). The algorithm is straightforward: for each of the
Bblocks of each homomorphic integer, an addition of LWE ciphertexts is computed, leveraging the
inherent linearity of the radix encoding used. The output ciphertext then consists of the resulting
LWE ciphertext blocks.

3.2 Notations and Assumptions 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 22: !ct(Add) Add
```

##### 

```
!ct(1);!ct(2)
```

##### 

```
Input: !ct(1);!ct(2)
Output: !ct(Add)
1 for i 2 [0;B1] do
2 ct(iAdd) ct(1)i +ct(2)i
3 return !ct(Add)
```

However, many details are hidden in this algorithm, including all cryptographic parameters such
as the ciphertext modulus, scaling factor∆, and others. To ensure algorithmic correctness, these
parameters must be the same for all the blocks. However, computational parameters, such as the
noise level and the degree of fullness, must also be constrained. For example, after performing an
addition, the noise level per block and the degree of fullness deviate from their nominal values, imposing
restrictions on subsequent operations. Consider the case of chaining multiple additions, where both the
noise level and the degree of fullness eventually reach their maximum values. To ensure computational
correctness, we must empty the carry buffers of the LWE ciphertexts and propagate them to the
next block—a process called carry propagation—before performing further computations. The cost
of this operation is significant and in the case of addition, it even dominates the overall computation
time. This leads to a second challenge: how can we benchmark operations when execution times
vary from microseconds to milliseconds, depending on whether carry propagation is applied? To
maintain consistency, we chose to enforce that the output encoding matches the input encoding in most
algorithms described in this section. Carry propagation can be seen as the integer equivalent of Boolean
gate bootstrapping in TFHE, resetting the encoding after each operation to preserve composability
and enable more realistic benchmarks.
In the following section, we outline the assumptions we make regarding the inputs and outputs of
each algorithm described below.

#### 3.2.2 Input/Output Assumptions

To simplify the notations of the large integer arithmetic algorithms in this section, we establish here
once and for all the assumptions we make about any large homomorphic integer. As detailed in the
previous section, a large homomorphic integer will be represented through a list ofLWEciphertexts,
where each one encrypts a digit of the radix decomposition of the large integer. EachLWEciphertext
of the list will be referred to as a block and the number of blocks will be indicated withB. The
blocks of the list are stored in little-endian order, that is, the least significant block (i.e., the block
that encrypts the least significant bits, calledLSBblock) is stored as the first element and the most
significant block is stored as the last element of the list (calledMSBblock).
As mentioned in Remark 30 , all blocks are assumed to use the same cryptographic parameters. An
input ciphertext

##### 

```
ct(1)is then denoted as:
```

!ct(1)=

##### 

```
ct(1) 0 ;:::;ct(1)B− 1
```

##### 

(^2) 
i∈[0,B−1]
LWEs(1)

##### 

```
Encode
```

##### 

```
m(1)i ; 2 π
```

```
(1)
msg(1)carry(1);q(1)
```

##### 

```
Z(n
```

```
(1)+1)×B(1)
q(1) ;
```

and likewise for

##### 

```
ct(2), etc.
Here is the list of assumptions:
```

3.2 Notations and Assumptions 3 ARITHMETIC OVER LARGE HOM. INTEGERS

- If an algorithm takes as input two (or more) ciphertexts

##### 

```
ct(1)and
```

##### 

```
ct(2), then:
```

```
msg=msg(1)=msg(2)
carry=carry(1)=carry(2)
q=q(1)=q(2)
=(1)=(2)
B=B(1)=B(2)
s=s(1)=s(2)
```

- The message and carry subspaces have the same power of two size:

```
msg=carry= 2κfor some 2 N
```

- The number of bits per block is a power of two:

```
log 2 (p) =log 2 (msgcarry) = 2κ
```

```
′
for some′ 2 N∗
```

- The carry subspace is assumed to be empty for any input. Using the degree notion (cf. Defini-
    tion 10 ):
       deg(ct) = max
          i∈[0,B−1]

```
fdeg(cti)g<msg
```

- At any point of an algorithm, the degree of a ciphertext never exceeds the maximal degree value:

```
deg(ct) = max
i∈[0,B−1]
```

```
fdeg(cti)g<msgcarry
```

- At any point in an algorithm, to preserve correctness, no ciphertext block is used as an operand
    of a dot product with a 2-norm larger than=

```
j
p− 1
msg− 1
```

```
k
(see Theorem 4 ).
```

The last point could be needed in the case where the ciphertext encrypts a Boolean value. In this, the
theoretical number of possible additions might be larger by only taking the degree into account. By
verifying all these constraints, we ensure the correctness of all our algorithms with a failure probability
smaller thanpfail(as defined in Theorem 12 ). If any exception is made on one of these assumptions, it
will be explicitly stated in the respective algorithm.

#### 3.2.3 Toolbox

We also use a variety of standard functions directly in our homomorphic integer algorithms, in partic-
ular:

**Generic Functions**

- X:append(y): the append method which operates on an existing listXand adds a new element
    yto the end ofX. Can also be used to append one list to another.
- len(X),X.len(): the length function which outputs the number of elements in a listX

3.2 Notations and Assumptions 3 ARITHMETIC OVER LARGE HOM. INTEGERS

- X.remove(i): removes theithelement from a listX, and shifts all elements with index> ito
    the left by one.
- X.pop(): removes the last element from a list and returns it.
- X//Y: quotient of integer division.
- Xy: shifts bits ofXto the left byyspaces.
- Xy: shifts bits ofXto the right byyspaces.
- isSigned(

##### 

```
ct): returns true if the homomorphic integer (
```

##### 

```
ct) represents a signed integer type or
false if it represents an unsigned integer.
```

```
Algorithm 23:
```

##### 

```
ct(out) BitExtraction
```

##### 

```
ct(in);offset;numBits
```

##### 

```
Input:
```

```
8
><
>:
```

```
!ct(in):a homomorphic integer
offset 2 [0;log 2 (msg)1]:bit index at which the extracted bit shall be placed in the output
numBits 2 [0;log 2 (Ω)1]:number of bits from!ct(in)to be extracted
Output: !ct(out)
1 for i 2 [0;numBits 1 ] do
2 blockIndex i//log 2 (msg)
3 bitIndex imod log 2 (msg)
4 lut GenLUT
```

```

(x)7!((xbitIndex)&1)offset
```

```

```

```
5 ct(iout) PBS
```

```

lut;ct(blockIndexin)
```

```

```

```
6 return !ct(out)
```

**Look-Up Table Generation Functions.** In some cases, the generation of theLUTis done be-
forehand. Taking as input any function fromf:Zp!Zp(or any subspaces inZp), the algorithm

GenLUToutputs a well-formedLUT 2 GLWES′

##### 

```
Mff
```

##### 

```
2 Rkq,N+1following Definition 13. We also abuse
```

the notation, by simply denoting this lookup table asLUTfwith

```
LUTf GenLUT
```

#####

```
f: (x)7!f(x)
```

##### 

##### 

Similarly in the case of bivariate functionsf(x;y), we apply Remark 28 , which leads to the following
signature:
LUTf GenLUT

#####

```
f: (x;y)7!f(x;y)
```

##### 

##### 

**PBS Functions.** Let us start by recalling the PBS signatures:

- From Algorithm 19 , the usual PBS is denoted byctout PBS(CTf;ctin;BSK;DriftMitigation);
- From Remark 28 ,ctout BivPBS

#####

```
CTf(x,y);ctin 1 ;ctin 0 ;BSK;DriftMitigation
```

##### 

```
is the signature of
the bivariate PBS.
```

In what follows, we simplify these: the PBS now takes as input either the function definition directly
or the associatedLUT, and one or two ciphertexts. The bootstrapping keyBSKand theDriftMitigation
are also abstracted away. In summary, these are the possibilities:

3.3 Bitwise Operations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

##### PBS

##### 

```
LUTf;
```

##### 

```
ct(1)
```

##### 

##### :=PBS

##### 

```
LUTf;
```

##### 

```
ct(1);BSK;DriftMitigation;DMK
```

##### 

##### 

##### PBS

##### 

```
f: (x)7!f(x);
```

##### 

```
ct(^1 )
```

##### 

```
:=LUTf GenLUT
```

#####

```
f: (x)7!f(x)
```

##### 

##### 

##### PBS

##### 

```
LUTf;
```

##### 

```
ct(1);BSK;DriftMitigation;DMK
```

##### 

##### 

```
BivPBS
```

##### 

```
LUTf;
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
:=BivPBS
```

##### 

```
LUTf;
```

##### 

```
ct(1);
```

##### 

```
ct(2);BSK;DriftMitigation;DMK
```

##### 

##### 

```
BivPBS
```

##### 

```
f: (x;y)7!f(x;y);
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
:=LUTf GenLUT
```

#####

```
f: (x;y)7!f(x;y)
```

##### 

##### 

```
BivPBS
```

##### 

```
LUTf;
```

##### 

```
ct(1);
```

##### 

```
ct(2);BSK;DriftMitigation;DMK
```

##### 

##### 

### 3.3 Bitwise Operations

In this subsection, we outline bitwise operations (such asAND;OR;XOR) for cases where both operands
are ciphertexts and where one operand is a ciphertext and the other a plaintext. We also consider
NOTin the encrypted case.

**Encodings.** Bitwise operations can be done in both _arithmetic_ and _logical_ mode, i.e., using the
original carry-message encoding (with empty carry bits, see Section2.3.1), or the Boolean encoding
(usually just a single block, see Section2.3.2). By default, we assume the logical mode (e.g., for
Boolean flags), however, we put the algorithms using the generic notation.

#### 3.3.1 Ciphertext-Ciphertext Addition

Bitwise operations are easy to implement as long asmsgis a power of 2 and the use of a bivariatePBS
is possible. Using the bivariatePBSallows for computing the bitwise operation in a block-by-block
manner since there are no dependencies across blocks like there would be with an arithmetic operation.
We present a generic bitwise algorithm forAND;OR;XORin Algorithm 24 where we use the symbol
⊛to denote any two-input Boolean operator. The bitwiseNOToperation can be done without using
thePBS, as presented in Algorithm 25.

```
Algorithm 24:
```

##### 

```
ct(out) Bitwise
```

##### 

##### ⊛(;)

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
Input:
```

```
(
⊛(;) :a binary bitwise operator
!ct(1);!ct(2):two homomorphic integers
Output: !ct(out): a homomorphic integer
1 for i 2 [0;B1] do
2 ct(iout) BivPBS
```

```

⊛(;);ct(1)i ;ct(2)i
```

```

```

```
3 return !ct(out)
```

**Theorem 13** _Let_

##### 

```
ct(1) and
```

##### 

```
ct(2) be two large homomorphic integers. Let ⊛ be a bitwise operator^8.
```

_Then,_ Decs

##### 

```
Bitwise
```

##### 

##### ⊛(;)

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=m(1)⊛m(2): The algorithmic complexity of Algorithm 24
```

_is:_
CBitwise=BCPBS

(^8) In TFHE-rs, we have implemented theOR;AND;XORoperators, and these are included in our benchmarks.

3.4 Equality 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 25: !ct(out) BitwiseNOT
```

##### 

```
!ct(in)
```

##### 

```
Input: !ct(in): a homomorphic integer
Output: !ct(out): a homomorphic integer
1 for i 2 [0;B1] do
2 ct(iout) ct(Triv)(msg1)ct(iin)
3 return !ct(out)
```

#### 3.3.2 Plaintext-Ciphertext Addition

When one of the inputs to a bitwise operation is a scalar (so unencrypted) we do not need to use the
bivariate PBS. Instead, we can use the usual PBS and construct theLUTs from the scalar.

```
Algorithm 26:
```

##### 

```
ct(out) Bitwise
```

##### 

##### ⊛(;)

##### 

```
ct(in);
```

##### 

```
Input:
```

```
8
><
>:
```

```
⊛(;) : a binary bitwise operator
!ct(in):a homomorphic integer
 2 ZΩ:a scalar
Output: !ct(out): a homomorphic integer
1 for i 2 [0;B1] do
2 ct(iout) PBS
```

```

⊛(; modmsg);ct(iin)
```

```

```

```
3  //msg
4 return !ct(out)
```

**Theorem 14** _Let_

##### 

```
ct(in) be a large homomorphic integer and  2 N be a scalar. Let ⊛ be a bitwise
```

_operator_^8_. Then,_ Decs

##### 

```
Bitwise
```

##### 

```
⊛(;);!ct(in);
```

##### 

```
=m(in)⊛. The algorithmic complexity of Algo-
```

_rithm 26 is:_
CBitwise=BCPBS

### 3.4 Equality

In this subsection, we introduce a method to check equality between two homomorphic integers and
between a homomorphic integer and a vector of plaintexts.

#### 3.4.1 Ciphertext-Ciphertext Equality

Equality works by using bivariatePBSon pairs of blocks from both inputs with theLUTdefined by
the function

```
eq(x;y)!
```

##### (

```
1 ifx=y;
0 otherwise:
```

This gives a list of blocks where each encrypted value in[0;1]:^9 The final part is to reduce this list
to one block that encrypts 1 if all blocks encrypt 1 , otherwise 0. Note that algorithms computing
the equalityEqand the differenceNeqare almost the same. The only changes lie in the functions

(^9) As each block only encrypts a Boolean value, theoretically, the number of values we can add together, denoted by
Chunksizein the algorithm could be larger. However, we do not leverage this, as performing additional additions could
exceed the noise threshold. For simplicity, we avoid this approach.

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

computed by eachPBS. We highlight them inside the algorithm by placing the not-equal code inside
a dashed box.

```
Algorithm 27: ct(Eq) Eq
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
ct(Neq) Neq
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
Input: !ct(1);!ct(2):two homomorphic integers
Output: ct(Eq): A one block ciphertext, encrypting a Boolean value
1 cmp fg
2 for i 2 [0;B1] do
3 cmp:append
```

```

BivPBS
```

```

eq(;);!ct(1)i ;!ct(2)i
```

```

cmp:append
```

```

BivPBS
```

```

!eq(;);!ct(1)i ;!ct(2)i
```

```

```

```
4 chunkSize
```

```
jp− 1
msg− 1
```

```
k
; //the maximum number of values we can add together
5 tmp fg
6 while len(cmp)> 1 do
7 (numFullChunks;rem) (len(cmp) //chunkSize;len(cmp)modchunkSize)
8 for i 2 [0;numFullChunks1] do
9 sum
PchunkSize− 1
j=0 cmpi·chunkSize+j
10 tmp:append(PBS(eq(;chunkSize);sum)) tmp:append(PBS(!eq(; 0 );sum))
11 sum
Plen(cmp)− 1
j=len(cmp)−remcmpj
12 tmp:append(PBS(eq(;rem);sum)) tmp:append(PBS(!eq(; 0 );sum))
13 cmp tmp
14 tmp fg
15 return cmp
```

**Theorem 15** _Let_

##### 

```
ct(1) and
```

##### 

```
ct(2) be two large homomorphic integers, and let l =
```

```
j
p− 1
msg− 1
```

```
k
be the
```

_maximal number of values we can add together. Then,_ Decs

##### 

```
Eq
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=eq(m(1);m(2)) 2 Z 2.
```

_The algorithmic complexity of Algorithm 27 is:_

```
CEq=
```

##### 0

##### @B+

```
⌈loglX(B)⌉− 1
```

```
i=0
```

##### 

##### B

```
li+1
```

##### 

##### 1

##### ACPBS

#### 3.4.2 Plaintext-Ciphertext Equality

The equality with a clear scalar uses the same logic as the equality with an encrypted value. One
improvement is to pack pairs of block into a single LWE ciphertext using the shift and add technique
(pack(a;b) = (amsg) +b), halving the number of blocks in the list that needs to be then reduced.
This is possible as we don’t need to use the bivariate PBS in this scenario. The complexity of the
algorithm becomes:

```
CEq=
```

##### 0

##### B

##### @

##### 

##### B

##### 2

##### 

##### +

```
dlogl(XdB 2 e)e−^1
```

```
i=0
```

##### &

```
B
2
```

##### 

```
li+1
```

##### '

##### 1

##### C

##### ACPBS

### 3.5 Carry Propagation

During arithmetic operations such as addition, subtraction, negation, or multiplication, the carry
buffers in each block gradually fill up. At some point, carry propagation becomes necessary, either

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

because the buffers are full and the next operation risks overflow, or because the next operation requires
an empty carry space.
Propagating carries is a very sequential operation as adding the carry stored in the blocknto the
blockn+ 1will modify the carry of blockn+ 1. The basic implementation of carry propagation using
a fully sequential algorithm is given in Algorithm 28. In it, starting from the block corresponding to
the least significant digit, we extract the value stored in the message space and the value stored in the
carry space using two separate PBSes. We then add the value that was in the carry space to the next
block before repeating this process. As such, the carry space in every block (except the first) needs to
be able to accommodate one further addition without the carry space overflowing; that is the block
can encrypt at most the valuecarry(msg1).

```
Algorithm 28: !ct(out) SequentialCarryPropagation
```

##### 

```
!ct(in)
```

##### 

```
Input: !ct(in): a homomorphic integer with partially filled carry buffers
Output: !ct(out): a homomorphic integer with empty carry buffers
1 ct(carry) ct(Triv)(0)
2 for i 2 [0;B1] do
3 ct(tmp) ct(iin)+ct(carry)
4 ct(iout) PBS
```

```

(x)7!xmodmsg;ct(tmp)
```

```

5 ct(carry) PBS

(x)7!x//msg;ct(tmp)

```

```
6 return !ct(out)
```

The sequential algorithm requires a total of 2 BPBSes (or 2 B 1 if the carry from the last block is
not needed). The latency corresponds to the time required to perform all thesePBSes sequentially. If
the twoPBSes in each iteration are executed in parallel, the latency isBtimes the latency of a single
PBS. In either case, the carry propagation algorithm’s latency scales linearly with both thePBStime
andB. Due to the high cost of thePBS, this operation remains slow even for relatively small values
ofB(e.g., 16).
When latency is a priority, a parallel algorithm can be used instead. In this section, we introduce
an algorithm that requires more computing power but is highly parallelizable, thereby reducing the
operation’s latency.

**Overview of specialised parallel implementation of carry propagation** For simplicity, we
assume that the carry space contains at most one filled bit and that, if the carry is one, the message
cannot reach its maximum value. This assumption does not reduce generality, as in the most common
use case, carry propagation is performed after summing two values, each at mostmsg 1. As a result,
the values encrypted in the resulting ciphertexts are smaller than 2 msg 2 , satisfying the previous
assumptions.
At a high level, the basic idea is to split the list of blocks into distinct groups of adjacent blocks,
compute some state information that depends only on blocks within a single group in parallel, compute
the carries between groups, and then propagate all this information to all blocks to give the result.
Suppose we want to determine whether blockbwithin groupgwill receive a carry during carry
propagation. Assume we have in encrypted form the carry (so either zero or one) of the previous group
g 1. We can then determine whether blockbof groupgwill receive a carry based on the so-called
group carry bit of groupg 1 and information about the values in the preceding blocks (i.e., blocks
0 ;:::;b 1 ) of groupg. Specifically, we need to know whether these preceding blocks generate a carry
that propagates to blockbor whether a carry from the previous group would propagate to this block.
What the algorithm will do is construct a ciphertext that encrypts a value inf 0 ; 1 ; 2 g, which we call the

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

propagation state, with the value 2 being if the previous blocks in the group generate and propagate
a carry to the blockb, 1 if the previous blocks would only propagate a carry from the previous group
and 0 otherwise, i.e., if any carry from the previous group or blocks in the current group would be
absorbed before reaching this block.
Once we have both the group carry of the previous group and the propagation state of the block,
we can compute via a PBS the value(xmodmsg) 1 , wherexmodmsgis the value encrypted in the
message part of blockbof groupg, and then add the group carry and the propagation state. Here, we
observe that if a carry is propagated to this block, the sum of the group carry and the propagation
state is either 2 or 3 ; otherwise, it is 0 or 1. Shifting the result of this sum to the right by one bit
produces a single bit that is 1 if a carry is propagated to this block (i.e., if the sum was either 2 or 3 ),
and 0 otherwise (i.e., if the sum was either 0 or 1 ). Note that this technique remains valid even if the
carry space consists of a single bit.

The algorithm consists of several steps, each of which is explained in further detail in the following
subsections. Below, we provide a simplified overview of each step:

```
1.First, blocks are grouped, and for each block (in parallel), a block state is computed along with
a cleaned version of the block (i.e., with the carry space emptied), shifted left by one bit.
```

```
2.Next, for each group (in parallel), an internal propagation of state information is performed to
compute the propagation states of each block, as well as unresolved information about the group
carries.
```

```
3.Then, the carry bit between groups is determined from the unresolved group carries. We pro-
vide two methods for this: one sequential and one using the Hillis-Steele scan, a parallel scan
algorithm.
```

```
4.Finally, in parallel, the group carries and propagation states are added to the cleaned and shifted
blocks, and the resulting blocks are shifted down by one bit to produce the final result.
```

#### 3.5.1 Step 1: Groups and Block States

Blocks are grouped in groups oflog 2 (p)elements (so with the standardp= 16, each group consists of
4 blocks). For each block, we will compute a so-called _block state_ which encodes information about the
value it encrypts. For each block, we will first associate it with an encoded value inf 0 ; 1 ; 2 gdepending
on whether the block

- generates a carry,
- may propagate the carry coming from the previous block,
- does neither, i.e., will absorb any input carry.

The encoding is 2 for _Generate_ , 1 for _Propagate_ and 0 for _Neither_. The way to compute this encoding is
by homomorphically comparing the encrypted valuexwithmsgusing aPBSby applying the function

```
unshifted_state(x) =
```

##### 8

##### ><

##### >

```
2 ; ifxmsg;
1 ; ifx=msg1;
0 ; otherwise:
```

This works because we have restricted the carry to be a single bit so it can only be propagated if it
is added to a value that is at leastmsg 1. Furthermore, a block cannot simultaneously generate a

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

carry and propagate a carry since then it would encrypt a value bigger than 2 msg 1 which would
break our assumption on the maximum value that a block can encrypt.
Instead of applyingunshifted_stateto each block, we will actually apply a shifted version where we
left-shift by the indexiused to index its position within its group, with the exception that for blocks
in the first group, the left-shift used isi 1. This shift is required so we can propagate these states
in the next step. Note that the first block of the first group is “left-shifted” by 1 bits by which we
mean right shift by 1 bit, i.e., dropping the least significant bit. This does not lose information since
the very first block does not receive any carry so it cannot propagate. Written another way, the very
first block of the homomorphic integer has a state that is given by the Boolean functionxmsg. In
summary, we use the following functions to determine the state of a given block, whereiis the index
of the block within its group:

```
State function for blocks in the first group: State function for all other blocks:
8
><
```

```
>:
```

```
2 (i1); ifxmsg;
1 (i1); ifx=msg 1 ;
0 ; otherwise.
```

##### (30)

##### 8

##### ><

##### >

```
2 i; ifxmsg;
1 i; ifx=msg 1 ;
0 ; otherwise.
```

##### (31)

We note that all these functions take the same form and depend only on the shift used. Further,
we point out that if we have a propagate state in the last block of a group then the state information
will occupy the padding bit. This is not a problem and will be dealt with later.
Finally, there is one further exception to how we compute the block states. The block state of
the very last block is only relevant if we want to check if an unsigned overflow happened as it would
only be used to compute the group carry of the last group. Since we do not need this group carry to
compute the final result, we avoid computing it as it can affect the latency of the next steps. This is
the reason for this exception. Instead, for the last block, we will shift the unshifted state by either 1
bit to the left if the last block is the only block in its group and otherwise by 2 bits to the left if there
are other blocks in the same group. More details will be given later in Section3.14.

Algorithm 30 describes how to compute all block states and how to clean and shift each block.
We leverage thePBSto compute all block states using the lookup tables defined in Algorithm 29 and
to generate the cleaned-shifted version of each block, this time using the functioncleanAndShift(x) =
(xmodmsg) 1.
For parameters withp 16 which have at least 4 bits in the combined message and carry space,
the manyLUT technique (see Algorithm 21 ) can be used, as only 3 out oflog 2 (p)bits are utilized.
Leveraging the manyLUT PBS reduces the number ofPBSoperations by half.

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 29: Block States Lookup Tables
1 Function CreateBlockStatesLuts(groupSize) :
Input: groupSize 2 N
Output: blockStatesLuts: list ofgroupSize+ 1lookup tables
2 blockStatesLuts
```

```
n
GenLUT
```

```

(x)7!xmsg
```

```
o
```

```
3 for i 2 [0;groupSize 1 ] do
4 Function LocalFn(x) :
5 if xmsg then
6 return 2 i
7 else if x=msg 1 then
8 return 1 i
9 else
10 return 0 i
11 blockStatesLuts:append

GenLUT

LocalFn()

```

```
12 return blockStatesLuts
```

```
Algorithm 30: Shifted block and block states computation
1 Function ComputeShiftedBlocksAndStates(!ct(in)) :
Input: !ct(in): a homomorphic integer with potentially non-empty carry spaces
Output:
```

```
(!
ct(shiftedBlocks)
!ct(blockStates) : two homomorphic integers (the integer encrypted has no real meaning)
2 groupSize blog 2 (p)c
3 blockStatesLuts CreateBlockStatesLuts(groupSize)
/*This can be done in parallel */
4 for i 2 [0;B1] do
5 groupIndex i//groupSize
6 indexInGroup imodgroupSize
7 if i=B 1 then
8 lut blockStatesLuts[3]
9 else if groupIndex= 0 then
10 lut blockStatesLuts[indexInGroup]
11 else
12 lut blockStatesLuts[indexInGroup+ 1]
13 ct(blockStates)[i] PBS
```

```

lut;!ct(iin)
```

```

```

```
14 ct(shiftedBlocks)[i] PBS
```

```

(x)7!(xmodmsg) 1 ;!ct(iin)
```

```

```

```
15 return !ct(shiftedBlocks);!ct(blockStates)
```

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

#### 3.5.2 Step 2: Propagation States and Group Carries CONTENTS CONTENTS

The next step is to update the block state information to form the _propagation states_ and compute
the so-called _group carries_ for each group, the idea of the group carries is the same as the block state
for a block but at the group level. Does the group generate a carry, if not, does it propagate a carry
from the previous group, or does it absorb any carry that might come from a previous group? Again,
the first group is special as it cannot propagate since it receives no carry.
Clearly, if the final block in a group generates a carry, so does the group itself and more generally, the
group will generate one if one of its blocks does and then all subsequent blocks in the group propagate.
Further, the group propagates if and only if all its blocks propagate. Any other combination of block
states is able to absorb a carry within the group since the carry is a single bit.
With this information, we can see that if we add all the (shifted) block states of the blocks in a
group (excluding the first group), then the group state will be ‘generate’ if the sum is at leastp(i.e.,
the padding bit is one), ‘propagate’ if the sum isp 1 , and otherwise will be ‘neither’ (any carry is
absorbed).
An example of all three cases when a group consists of two blocks is shown below. Recall that
block states are shifted based on their index within the group.

2 Propagate:

```
001
+ 010
Propagate 011
```

```
1 Generate, 1 Propagate:
```

```
010
+ 010
Generate 100
```

```
1 Generate, 1 Neither:
```

```
010
+ 000
Neither 010
```

For the first group, again there is no input carry so either the group generates a carry or it does
not. As we shifted by one less bit, when summing, we have that the group ‘generates’ if the sum is
at leastp 2. Thus for the first group, we have a resolved group carry and for the other groups, we have
only unresolved group carries.
Clearly, this technique is applicable to any of the blocks in a group, not just the last one, so in the
same manner, we will use the block states to compute the _propagation state_ of each block to determine
whether a carry (either from a previous block in the group or propagation of a carry from the previous
group) reaches that block. All we have to do is replacepin the above by 2 i+1whereiis the index of
the block in the group.
Thus, for each block that is not the first in a group, a new propagation state is computed using the
function given in Eq. ( 33 ) applied to the sum of the block states up to but not including that block in
a given group. For the first block in a group, only the group carry from the previous group matters,
in this case, we define the propagation state to be a trivial encryption of 1 , thus when we later add
the resolved group carry for the previous group to it, it will be 2 or 1 in line with what is required.
Again, for the blocks of the first group, the function is slightly different. As already pointed out,
the first block state is already resolved, thus the propagation states of the blocks in the first group will
be resolved at this step to either 2 or 0 , depending on if a carry needs to be added to this block or not.

```
Propagation state function for the first group
applied to the partial sum of the firstiblocks
(excludingi=log 2 (p)):
```

```
Propagation state function for all other groups
applied to the partial sum of the firstiblocks
(excludingi=log 2 (p)):
(
2 ; ifx 1 i;
0 ; otherwise.
```

##### (32)

##### 8

##### ><

##### >

```
2 ; ifx 2 i;
1 ; ifx= (2i) 1 ;
0 ; otherwise.
```

##### (33)

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

The group carry state of a group (excluding the first group) cannot be computed in this way due
to the fact that the padding bit may not be zero (since we are in the casei=log 2 (p)). Furthermore,
we will see in the next step that we will want to use a different function depending on which of the
two methods of resolving the group carry state is used.
The trick for dealing with a padding bit, which may be non-zero, is to note that if the padding bit
is one, then the resulting PBS will output an encryption of the negated value of the PBS where the
padding bit has zero instead. By being clever with the function we evaluate and adding a well-chosen
constant to the result, we will be able to differentiate the three cases. Something we now explain for
the two approaches.
In the case of the sequential algorithm, we want to use the usual encoding inf 0 ; 1 ; 2 gshifted by a
certain number of bits. To achieve this, we can first apply the function

```
sequential_state(x) =
```

##### (

```
0 ; ifx=p 1 ;
 1 (groupIndexmod(log 2 (p)1)); otherwise,
```

and then add the constant 1 (groupIndexmod(log 2 (p)1)). Here,groupIndexis the index of the
group in the list of all groups whose group carry state is unresolved (i.e., all groups except the first –
so the second group has index 0, the third group has index 1, etc.). An example forgroupIndex= 0
illustrating why this works is given below.

- Forx 2 [0;p2], the result of this operation will besequential_state(x) + 1 =1 + 1 = 0which
    means neither propagate nor generate.
- Forx=p 1 , the result will besequential_state(x) + 1 = 0 + 1which means propagate state.
- Forx 2 [p; 2 p2], the result will besequential_state(x) =sequential_state(xp) + 1 =
    (1) + 1 = 2which means generate, so all cases are correct.

We remark thatxcannot be 2 p 1 since this value cannot be reached by the sum of the block states
from step one.
In the case of the Hillis-Steele prefix scan method, we will change the encoding slightly and instead
apply the function

```
HS_state(x) =
```

##### (

```
2 ; ifx=p 1 ;
 1 ; otherwise,
```

and then add the clear constant 1 to the result. In this case, the resulting encoding is 2 for generate,
3 for propagate, and 0 for neither. Note that we do not associate 1 with any state due to a restriction
in thePBSapplyingHS_state. However, we will later associate 1 with a generate state as well.
Finally, the case of the group carry for the first group does not have the same issue with the padding
bit and we can directly apply a PBS with the Boolean functionxp 2 to give the resolved group carry
(we do not shift by 1 bit here as we want a value inf 0 ; 1 g).
The algorithm to construct all the necessary lookup tables in this step is given in Algorithm 31.
Computation of the propagation states and group carries is given in Algorithm 32. Each lookup table
is applied using a PBS and the output ciphertexts corresponding to propagation states will be added
to the appropriate block of

##### 

ct(shiftedBlocks)from Algorithm 30 , these are called _prepared blocks_. Note
that if the carry space consists of a single bit, then there is a chance that the padding bit of a prepared
block is flipped to one, this will not cause problems later however.

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 31: LUTs for computing group carries and propagation states
1 Function PrepareGroupLuts(groupSize;useSeq) :
Input:
```

```
(
groupSize 2 N
useSeq:a Boolean
```

```
Output:
```

```
8
>>>
<
>>>
:
```

```
firstGroupPropStateLuts
otherGroupPropStateLuts
firstGroupCarryLut
otherGroupCarryLuts
2 firstGroupPropStateLuts fg
3 for i 2 [0;groupSize2] do
4 firstGroupPropStateLuts:append
```

```

GenLUT
```

```

(x)7!((xi)&1) 2
```

```

```

```
5 otherGroupPropStateLuts fg
6 for i 2 [0;groupSize1] do
7 Function LocalFn(x) :
8 if x(2i) then
9 r 2
10 else if x= (2i) 1 then
11 r 1
12 else
13 r 0
14 return r
15 otherGroupPropStateLuts:append

GenLUT

LocalFn()

```

```
16 firstGroupCarryLut GenLUT
```

```

(x)7!(x(groupSize1))& 1
```

```

```

```
17 otherGroupCarryLuts fg
18 if useSeq then
19 for i 2 [0;groupSize2] do
20 Function LocalFn(x) :
21 if x=p 1 then
22 r 0
23 else
24 r ( 1 i)mod(2p)
25 return r
26 otherGroupCarryLuts:append
```

```

GenLUT
```

```

LocalFn()
```

```

```

```
27 else
28 Function LocalFn(x) :
29 if x=p 1 then
30 r 2
31 else
32 r  1 mod(2p)
33 return r
34 otherGroupCarryLuts:append
```

```

GenLUT
```

```

LocalFn()
```

```

```

```
35 return firstGroupPropStateLuts;otherGroupPropStateLuts;firstGroupCarryLut;otherGroupCarryLuts
```

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 32: Compute Propagation States And Group Carries
1 Function ComputePropagationStatesAndGroupCarries(groupSize;!ct(blockStates);useSeq) :
```

```
Input:
```

```
8
><
>:
```

```
groupSize 2 N
!ct(blockStates): a homomorphic integer (the integer encrypted has no real meaning)
useSeq;a Boolean
Output:
```

```
(!
ct(propStates): a homomorphic integer (the integer encrypted has no real meaning)
groupCarries:a list of single block (LWE) ciphetexts
2 (firstGroupPropStateLuts;otherGroupPropStateLuts;firstGroupCarryLut;otherGroupCarryLuts)
PrepareGroupLuts(groupSize;useSeq)
3 sums fg
4 numberofGroups
```

```
l B− 1
groupSize
```

```
m
```

```
/*This should be done in parallel */
5 for i 2 [0;numberofGroups1] do
6 sums:append
```

```
!
ct(i·blockStatesgroupSize)
```

```

```

```
/*This must be sequential */
7 for j 2 [1;groupSize1] do
8 if (igroupSize) +j <B 1 then
9 sums:append
```

```

sums:last() +!ct((blockStatesi·groupSize))+j
```

```

```

```
/*This should be done in parallel */
10 for i 2 [0;B2] do
11 groupIndex
```

```
j i
groupSize
```

```
k
```

```
12 indexInGroup imodgroupSize
13 if groupIndex= 0 then
14 if indexInGroup=groupSize 1 then
15 lut firstGroupCarryLut
16 else
17 lut firstGroupPropStateLuts[indexInGroup]
18 else if indexInGroup=groupSize 1 then
19 if useSeq then
20 lut otherGroupCarryLuts[(groupIndex1)mod(groupSize1)]
21 else
22 lut otherGroupCarryLuts 0
23 else
24 lut otherGroupPropStateLuts[indexInGroup]
25 sums[i] PBS(lut;sums[i])
26 if groupIndex 6 = 0andindexInGroup=groupSize 1 then
27 if useSeq then
28 sums[i] sums[i] +ct(Triv)(1(groupIndex1)mod(groupSize1))
29 else
30 sums[i] sums[i] +ct(Triv)(1)
```

```
31 !ct( 0 propStates) ct(Triv)(0)
32 groupCarries fg
/*Relabeling Step */
33 for i 2 [0;B2] do
34 if imodgroupSize=groupSize 1 then
35 groupCarries:append(sums[i])
36 !ct(ipropStates+1 ) ct(Triv)(1)
37 else
38 !ct(ipropStates+1 ) sums[i]
```

```
39 return !ct(propStates);groupCarries
```

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

#### 3.5.3 Step 3: Resolving the Group Carries

In this step, we want to resolve the group carries to take values inf 0 ; 1 g, depending on whether there
is a carry from that group or not. As previously mentioned, we propose two possible algorithms to
resolve the group carries:

- A sequential one that resolveslog 2 (p) 1 carries at each iteration, it uses a logic similar to the
    construction of the propagation states of the first group described earlier;
- A parallel algorithm based on Hillis-Steele prefix scan [HSJ86] which can be applied whenp 16.

The sequential algorithm is used for a smaller number of groups until Hillis-Steele becomes the faster
option, as determined later in Algorithm 36 using theuseSeqflag.
We again note that we are not computing or resolving a group carry for the last group as this may
slow down this step. Instead, if needed we can compute whether an overflow occurred later as will be
described in Section3.14.

**Sequential Group Carry Propagation.** This propagation works in essentially the same way the
propagation states were computed for the first group. We processlog 2 (p) 1 unresolved carries at a
time, compute their partial sums plus the resolved carry from the previous group, and check if this
result is larger than the appropriate power of two to resolve the group carry. Details are given in
Algorithm 33.

```
Algorithm 33: Sequential Group Carry Propagation
1 Function ResolveGroupCarriesSequentially(groupCarries;groupSize) :
Input:
```

```
(
groupCarries: a list of single block (LWE) ciphertexts
groupSize 2 N
Output: resolvedCarries:a list of single block (LWE) ciphertexts
2 luts fg
3 for i 2 [0;groupSize1] do
4 luts:append
```

```

GenLUT
```

```

(x)7!x(i+ 1)& 1
```

```

```

```
5 resolvedCarries fg
6 resolvedCarries:append(groupCarries[0])
7 j 1
8 lastIndex log 2 (p) 2
9 while j <groupCarries:len() do
10 array fg
11 array:append(resolvedCarries:last() +groupCarries[j])
12 j j+ 1
/*This needs to be done sequentially */
13 for i 2 [1;log 2 (p)2] do
14 if j <groupCarries:len() then
15 array:append(array:last() +groupCarries[j])
16 j j+ 1
17 else
18 lastIndex i 1
19 break
/*This can be done in parallel */
20 for i 2 [0;lastIndex] do
21 resolvedCarries:append(PBS(luts[i];array[i]))
22 return resolvedCarries
```

**Hillis-Steele Group Carry Propagation.** Here, we use the prefix scan parallel algorithm, however,
instead of doing an addition, we do a bivariatePBSwith the function described in Algorithm 34.

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 34: Lookup Table Function For Hillis-Steele Group Carry Propagation
1 Function HillisSteeleAdd(previous, current) :
Input:
```

```
(
previous2f 0 ; 1 ; 2 ; 3 g
current2f 0 ; 1 ; 2 ; 3 g
Output: result2f 0 ; 1 ; 2 ; 3 g
2 if current= 2 then
3 return 1
4 else if current= 3 then
5 if previous= 2 then
6 return 1
7 else
8 return previous
9 else
10 return current
```

Recall that for this method, we encoded our unresolved group carries as 3 for propagate, 1 or 2
for generate, and 0 for neither. Thus, we can write this function in the following manner to make it
clearer:

```
previous! N G G P
#current 0 1 2 3
N 0 0 0 0 0
G 1 1 1 1 1
G 2 1 1 1 1
P 3 0 1 1 3
```

Here, we see that we can only get as output a propagate state if the two inputs are also both propagate.
Otherwise, we are in a generate state unless the current state isNeitheror the current state isPropagate
and the previous state wasNeither.
By the end of the Hillis-Steele scan, allGeneratestates are going to be changed to 1 as it is the
value representing that a carry occurs. ThePropagatestates will also all be resolved to 0 or 1 since
the first group carry is already resolved, and this will get combined (via the bivariate PBS) with all
states eventually.

```
Algorithm 35: Hillis Steele Carry Propagation
1 Function ResolveGroupsCarriesUsingHillisSteele(groupCarries) :
Input: groupCarries: a list of single block (LWE) ciphertexts
Output: resolvedCarries:a list of single block (LWE) ciphertexts
2 lut GenLUT

HillisSteeleAdd(;)

/*This needs to be done sequentially */
3 for d 2 [0;dlog 2 (groupCarries:len())e1] do
/*This can be done in parallel */
4 for i 2
```

```

2 d;groupCarries:len() 1
```

```

do
5 groupCarries[i] BivPBS

lut;groupCarries[i 2 d];groupCarries[i]

```

```
6 resolvedCarries groupCarries
7 return resolvedCarries
```

#### 3.5.4 Step 4: Final Step

At this step, we have a list of _prepared blocks_ and the resolved group carries of each group. The last
step consists in adding the resolved group carry of groupito all the prepared blocks of the groupi+1,

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

and then using aPBSwith the function(x1)modmsgto finalize the block. Note that for the first
group, there is no group carry to add (so that the group carry to be added is a trivial encryption of
zero).
Further, in the case of a single bit for the carry space, we can have values up to 2 msg+1encrypted
after the sum which means the padding bit can be one. However, if the padding bit is one, then the
message and carry space contains the value 0 or 1, and the shift function results in the value zero
meaning that the negation applied due to the padding bit being one has no effect and there isn’t any
issue here.
This last step can be seen in Algorithm 36 which puts together Algorithm 30 , Algorithm 32 and
either Algorithm 33 or Algorithm 35 , to show to propagate the carries when they can occupy at most
one bit of the carry space.

```
Algorithm 36:
```

##### 

```
ct(out) PropagateSingleBitCarries
```

##### 

```
ct(in)
```

##### 

```
Input: !ct(in): a homomorphic integer with each block encrypting a value in[0; 2 msg2]
Output: !ct(out): a homomorphic integer with empty carry spaces
1 groupSize blog 2 (p)c
2 numCarries
```

```
l B
groupSize
```

```
m
 1
3 seqDepth
```

```
j
numCarries− 1
groupSize− 1
```

```
k
```

```
4 hillisSteeleDepth if numCarries= 0 then 0 else dlog 2 (numCarries)e
5 useSeq seqDepthhillisSteeleDepth or p< 16
6 !ct(shiftedBlocks);!ct(blockStates) ComputeShiftedBlocksAndStates
```

```
!
ct(in)
```

```

```

```
7 !ct(propStates);groupCarries
ComputePropagationStatesAndGroupCarries
```

```

groupSize;!ct(blockStates);useSeq
```

```

```

```
8 if numCarries= 0 then
9 resolvedCarries

ct(Triv)(0)
10 else if useSeq then
11 resolvedCarries ResolveGroupCarriesSequentially(groupCarries;groupSize)
12 else
13 resolvedCarries ResolveGroupsCarriesUsingHillisSteele(groupCarries)
/*This can be done in parallel */
14 for i 2 [0;B1] do
15 ct(iout) ct(ishiftedBlocks)+ct(ipropStates)+resolvedCarries
```

```
hj
i
groupSize
```

```
ki
```

```
16 ct(iout) PBS
```

```

(x)7!(x1)modmsg;ct(iout)
```

```

```

```
17 return !ct(out)
```

#### 3.5.5 Full-Fledged Carry Propagation

With the ability to propagate single-bit carries in a parallel manner, we can now give the final algorithm
which shows how to use this to propagate any size carries in parallel. This can be found in Algorithm 37.
The important observation is that when adding the carry space to the next message space, there is at
most one bit of further carry created, it is this carry that can then be propagated using the previous
algorithm. We can also avoid any unnecessary carry propagation by only starting at the first block
which has a non-empty carry space as determined by its degree of fullness.

#### 3.5.6 Complexity of Carry Propagation

We now give the complexities of the various carry propagation algorithms.

3.5 Carry Propagation 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 37: !ct(out) FullPropagate
```

##### 

```
!ct(in)
```

##### 

```
Input: !ct(in): a homomorphic integer with potentially non-empty carry spaces
Output: !ct(out): a homomorphic integer with empty carry spaces
/*Get the index of the first block with degree abovemsg 1 */
1 start FirstBlockWithDegreeAbove
```

```
!
ct(in);msg 1
```

```

```

```
2 for i 2 [0;start1] do
3 ct(iout) ct(iin)
4 ct(startout) PBS
```

```

(x)7!xmodmsg;ct(startin)
```

```

```

```
/*This can be done in parallel */
5 for i 2 [0;Bstart2] do
6 ct(imessages) PBS
```

```

(x)7!xmodmsg;ct(iin+)start+1
```

```

```

```
7 ct(icarries) PBS
```

```

(x)7!
```

```
j
x
msg
```

```
k
;ct(iin+)start
```

```

```

```
8 !ct(tmp) !ct(messages)+!ct(carries)
/*This will useBstart 1 in place ofB */
9 !ct(propagated) PropagateSingleBitCarries
```

```
!
ct(tmp)
```

```

```

```
/*This is just relabelling */
10 for i 2 [start+ 1;B1] do
11 ct(iout) ct(ipropagated−start− 1 )
12 return !ct(out)
```

**Theorem 16** _Let_ ct(in) _be a large homomorphic integer. Applying the sequential carry propagation of
Algorithm 28 to_

##### 

```
ct(in) has complexity CSequentialCarryPropagation=CPBS 2 B.
```

**Theorem 17** _Let_ ct(in) _be a large homomorphic integer where each block encrypts a value that is at
most_ 2 msg 2_. Let_ G=log 2 (p) _be the group size and_ L(B) =

##### B

```
G
```

##### 

_be the function returning the
number of groups given a number of blocks as input.
When using sequential inner propagation, the complexity of Algorithm 36 :_
CPropagateSingleBitCarries(B) =CBivPBS(B+B1 +max(0;L(B)2) +B)
_When using Hillis-Steele inner propagation, the complexity is:_
CPropagateSingleBitCarries(B) =CBivPBS(B+B1 +B) +CHillis−Steele(L(B)1)

_With_ CHillis−Steele(x) =CBivPBS

```
P⌈log 2 (x)⌉
d=0
```

#####

```
x 2 d
```

##### 

```
=CBivPBS
```

#####

```
(dlog 2 (x)e+ 1)x+ 1 2 1+⌈log^2 (x)⌉
```

##### 

##### 

**Theorem 18** _Let_ ct(in) _be a large homomorphic integer having_ B _blocks, assuming that we already
have used the carry space in the first block and we want to empty all the carries using Algorithm 37
then the complexity of this operation is:_

CFullPropagate(B) =CPBS(2B1) +CPropagateSingleBitCarries(B1):
Since Algorithm 37 benefits greatly from parallelism, it is also instructive to see how it compares
with Algorithm 28 in terms of the sequential depth of PBSes performed. In this regard, the sequential
algorithm has depthBPBSes while the parallel algorithm has depth either

```
4 +dmax(dB/log 2 (p)e 2 ;0)/(log 2 (p)1)e
```

if using the sequential algorithm to resolve group carries (Algorithm 33 ), or

```
4 +dlog 2 (max((dB/log 2 (p)e1);1))e
```

if using Hillis-Steele (Algorithm 35 ).

3.6 Addition 3 ARITHMETIC OVER LARGE HOM. INTEGERS

### 3.6 Addition

In TFHE, ciphertexts and plaintexts can be freely added as long as certain constraints are met –
namely, noise constraints and constraints on the degrees of fullness (see Section3.2.1). This addition
naturally extends to homomorphic integers. However, in the radix representation, addition becomes a
costly operation due to the need for carry propagation, as explained in Section3.5.

#### 3.6.1 Ciphertext-Ciphertext Addition

The addition of homomorphic integers is presented in Algorithm 38 , and Theorem 19 states its guaran-
tees and complexity. Note that the computations at each step of the loop can be performed in parallel
across all blocks.

```
Algorithm 38:
```

##### 

```
ct(Add) Add
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
Input: !ct(1);!ct(2):two homomorphic integers
Output: !ct(Add)
1 for i 2 [0;B1] do
2 ct(iAdd) ct(1)i +ct(2)i
3 return FullPropagate
```

```
!
ct(Add)
```

```

```

**Theorem 19** _Let_

##### 

```
ct(1) and
```

##### 

```
ct(2) be two large homomorphic integers. Then,
```

```
Decs
```

##### 

```
Add
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=m(1)+m(2) (modΩ)
```

_The algorithm’s complexity is:_ CAdd=CFullPropagate_._

#### 3.6.2 Plaintext-Ciphertext Addition

To add a plaintext to a homomorphic integer, we can directly use Algorithm 38 by interpreting the
plaintext as a trivial ciphertext (see Definition 6 ). In TFHE-rs, a more efficient version of this algorithm
is implemented when one operand is a trivial ciphertext and will be included in future versions of this
document.

### 3.7 Negation

Similarly to the case of addition, the negation operation on LWE ciphertexts extends naturally to
homomorphic integers. The negation itself is computationally inexpensive compared to the costly final
carry propagation required to reset the encoding. The full algorithm is presented in Algorithm 39 and
Theorem 20 states its guarantees and complexity.

3.8 Subtraction 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 39: !ct(Neg) Neg
```

##### 

```
!ct
```

##### 

```
Input: !ct:a large homomorphic integer
Output: !ct(Neg):a large homomorphic integer
1 zb 0
2 for i 2 [0;B1] do
3 block cti+zb
4 z max
```

```
nldeg(block)
msg
```

```
m
; 1
```

```
o
msg
5 ct(iNeg) (block) +z
6 zb z//msg
7 return FullPropagate
```

```
!
ct(Neg)
```

```

```

**Theorem 20** _Let_

##### 

```
ct be a large homomorphic integer encrypting m 2 ZΩ. Then,
```

```
Decs
```

##### 

```
Neg
```

##### 

```
ct
```

##### 

```
=m (modΩ):
```

_The algorithm’s complexity is_ CNeg=CFullPropagate_._

**Remark 34** _Similar to the addition algorithm, the negation algorithm can take as input a homomor-
phic integer with non-empty carry buffers. However, a certain amount of carry space must still be
available. The required carry space for negation depends on the degrees of fullness of the preceding
blocks (notably, some blocks may be completely full, i.e.,_ deg(cti) =carrymsg 1 _). As a result, it is
not possible to determine a tight bound on the degrees of fullness._

### 3.8 Subtraction

The subtraction presented in Algorithm 40 comes naturally from the homomorphic addition ofLWE
ciphertexts. Subtraction can be implemented in two ways:

- Adding the negation, i.e.,ab=a+ (b), only then the carries are propagated (once),
- Direct subtraction with a correcting term, then borrows are propagated.

In TFHE-rs, subtraction is performed by adding one operand to the negation of the second, except
for overflow detection in unsigned subtraction, where the second method is used instead (see Sec-
tion3.14.2).

```
Algorithm 40:
```

##### 

```
ct(Sub) Sub
```

##### 

```
ct(1);
```

##### 

```
ct(2)
```

##### 

```
Sub
```

##### 

```
ct(1);
```

##### 

```
Input:
```

```
n
!ct(1),!ct(2): two homomorphic integers !ct(1), v: a homomorphic integer and a scalar
Output: !ct(Sub):large homomorphic integer
1 !ct(Neg) Neg
```

```
!
ct(2)
```

```

; //without the finalFullPropagate()
```

```
2 !ct(Sub) Add
```

```
!
ct(1);!ct(Neg)
```

```
 !
ct(Sub) Add
```

```
!
ct(1);
```

```

```

```
3 return !ct(Sub)
```

```
Theorem 21 states the guarantees of Algorithm 40 and its complexity.
```

3.9 Block Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

**Theorem 21** _Let_

##### 

```
ct(1) and
```

##### 

ct(2) _be two large homomorphic integers that encrypt_ m(1) _and_ m(2) _,
respectively. Then,_

```
Decs
```

##### 

```
Sub
```

##### 

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=m(1)m(2) (modΩ)
```

_The algorithm’s complexity is_ CSub=CAdd_._

#### 3.8.1 Plaintext-Ciphertext Subtraction

To subtract a plaintext from a homomorphic integer, we can negate the scalar and use the plaintext-
ciphertext addition algorithm discussed in Section3.6.2.

### 3.9 Block Shifts and Rotations

Shifts and rotations can be done at two levels, _blocks_ or _bits_. Operating at the block level is often more
useful as a building block for other operations rather than for end users which will use the bit versions
as the blocks are abstracted. In this section, we describe plaintext-ciphertext and ciphertext-ciphertext
_block_ shifts while bit shifts are described in Section3.10. The operation descriptions in this section
begin with Plaintext-Ciphertext block shifts, as some of their subroutines are required to define the
Ciphertext-Ciphertext case.

#### 3.9.1 Plaintext-Ciphertext Block Shift & Rotation

Shifting and rotating blocks by a known amount are very cheap operations as everything happens in
the clear, and it involves no FHE operations. We present four algorithms for both left and right shift
and rotate in Algorithm 41.

3.9 Block Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 41: Block Shift / Rotate By A Clear Amount
```

```
Input:
```

```
(!
ct(in):a homomorphic integer
blockShift 2 [0;B1] :the number of blocks to shift
Output: !ct(out)
1 Function BlocksShiftLeft(!ct(in) , blockShift) :
2 for i 2 [0;BblockShift1] do
3 j i+blockShift
4 ct(iout) ct(jin)
5 for i 2 [BblockShift;B1] do
6 ct(iout) ct(Triv)(0)
7 return !ct(out)
8 Function BlocksShiftRight(!ct(in) , blockShift) :
9 for i 2 [0;blockShift1] do
10 ct(iout) ct(Triv)(0)
11 for i 2 [blockShift;B1] do
12 j iblockShift
13 ct(iout) ct(jin)
14 return !ct(out)
15 Function BlocksRotateLeft(!ct(in) , blockShift) :
16 for i 2 [0;B1] do
17 j (blockShift+i)modB
18 ct(iout) ct(jin)
19 return !ct(out)
20 Function BlocksRotateRight(!ct(in) , blockShift) :
21 for i 2 [0;B1] do
22 j (BblockShift+i)modB
23 ct(iout) ct(jin)
24 return !ct(out)
```

3.9 Block Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

#### 3.9.2 Ciphertext-Ciphertext Block Shift & Rotation

Shifting or rotating blocks by an encrypted amount introduces additional complexity, as it requires
FHE operations. The algorithm is based on the _Barrel shifter_ electronic circuit.
First, the bits of the integer encrypting the amount to shift or rotate by are extracted into individual
blocks using aPBS. Only the necessary bits are extracted, that is, only the bits in a position that is
in the range[0;dlog 2 (B)e], and these encrypted bits make up!ct(shiftBits).
Then, for each block in the ciphertext to be shifted/rotated, two bivariatePBSes are performed,
one to extract its newmessagevalue and another to extract the value that is meant to go to the next
block (calledcarryhere). Then, depending on the current iteration, thecarryis aligned with the
message, and the two are added together to form the new block value.
The algorithm for the set-up of the look-up table is outlined in Algorithm 42. The full algorithm
for encrypted block shift and rotation is outlined in Algorithm 43.

```
shiftMessage(x;shiftBit) =
```

##### (

```
x; ifshiftBit= 0
0 ; otherwise
```

```
shiftCarry(x;shiftBit) =
```

##### (

```
0 ; ifshiftBit= 0
x; otherwise
```

##### (34)

```
Algorithm 42: Auxiliary function for creation ofLUTfor the block shift
```

```
Input:
```

```
(
lastBlock 2 [0;msg1]
shiftBit2f 0 ; 1 g
Output: result2f 0 ; 1 g
1 Function PaddingBlockLut(lastBlock;shiftBit) :
2 if shiftBit= 1 then
3 signBit lastBlock// (msg// 2)
4 result= (msg1)signBit
5 else
6 result= 0
7 return result
```

3.9 Block Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 43: !ct(out) BlockBarrelShift
```

##### 

```
!ct(in);!ct(shiftBits);kind;range
```

##### 

```
Input:
```

```
8
>>>
<
>>>
:
```

```
!ct(in): a homomorphic integer
!ct(shiftBits):each block encrypts 1 bit, i.e.,!ct(shiftBits) ExtractBits()
kind2fLeftShift;LeftRotate;RightShift;RightRotateg: a type for the shift
range 2 [0;B1] :the size of the shift
Output: !ct(out)
1 j 0
2 for d 2 range do
3 if kind=RightShift AND isSigned(ct(in)) then
4 paddingBlock BivPBS
```

```

PaddingBlockLut(;);ct(B−in) 1 ;ct(jshiftBits)
```

```

```

```
5 else
6 paddingBlock ct(Triv)(0))
7 carries fg
8 for i 2 [0;B1] do
/*ct(shiftBits)is already shifted bymsg */
9 ct(iout) PBS
```

```

shiftMessage(;);ct(iin)+ct(jshiftBits)
```

```

//defined in Eq. ( 34 )
10 carries.append
```

```

PBS
```

```

shiftCarry(;);ct(iin)+ct(jshiftBits)
```

```

//defined in Eq. ( 34 )
11 j j+ 1;
12 switch kind do
13 case LeftShift do
14 carries BlocksRotateRight(carries;(1d)1)
15 for i 2 [0; 1 d] do
16 carries[i] paddingBlock
17 case RightShift do
18 carries BlocksRotateLeft(carries; 1 d)
19 for i 2 [B(1d);B1] do
20 carries[i] paddingBlock
21 case LeftRotate do
22 carries BlocksRotateRight(carries; 1 d)
23 case RightRotate do
24 carries BlocksRotateLeft(carries; 1 d)
25 for i 2 [0;B1] do
26 ct(iout) ct(iout)+carries[i]
```

```
/*Clean the noise */
27 for i 2 [0;B1] do
28 ct(iout) PBS
```

```

(x)7!xmodmsg;ct(iout)
```

```

```

```
29 return !ct(out)
The algorithms are defined as a specific call of theBlockBarrelShiftalgorithm, i.e.:
BlockLeftShift
```

```
!
ct(1);!ct(2)
```

```

=BlockBarrelShift(!ct(1);ExtractBits(!ct(2);log 2 (msg);dlog 2 (B)e);LeftShift;[ 0 ;dlog 2 (B)e])
```

```
BlockRightShift
```

```
!
ct(1);!ct(2)
```

```

=BlockBarrelShift(!ct(1);ExtractBits(!ct(2);log 2 (msg);dlog 2 (B)e);RightShift;[ 0 ;dlog 2 (B)e])
BlockLeftRotate
```

```
!
ct(1);!ct(2)
```

```

=BlockBarrelShift(!ct(1);ExtractBits(!ct(2);log 2 (msg);dlog 2 (B)e);LeftRotate;[ 0 ;dlog 2 (B)e])
```

BlockRightRotate

```
!
ct(1);!ct(2)
```

```

=BlockBarrelShift(!ct(1);ExtractBits(!ct(2);log 2 (msg);dlog 2 (B)e);RightRotate;[ 0 ;dlog 2 (B)e])
```

**Theorem 22** _Let_

##### 

```
ct(1) and
```

##### 

```
ct(2) be two large homomorphic integers.
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Then,
```

```
Decs
```

##### 

```
BlockLeftShift
```

##### 

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
= (m(1)msgm
```

```
(2)
) (modΩ);
```

```
Decs
```

##### 

```
BlockRightShift
```

##### 

```
!ct( 1 );!ct( 2 )
```

##### 

```
= (m(1)//msgm
```

```
(2)
) (modΩ);
```

```
Decs
```

##### 

```
BlockLeftRotate
```

##### 

```
!ct( 1 );!ct( 2 )
```

##### 

```
= (m(1)msgm
```

```
(2)
)j(m(1)//msgB−m
```

```
(2)
) (modΩ);
```

```
Decs
```

##### 

```
BlockRightRotate
```

##### 

```
!ct( 1 );!ct( 2 )
```

##### 

```
= (m(1)//msgm
```

```
(2)
)j(m(1)msgB−m
```

```
(2)
) (modΩ):
```

```
Let =dlog 2 (B)e be the number of bits that represent the number of block shifts.
```

##### C=CPBS

##### 

##### 

```
log 2 (msg)
```

##### 

##### + (2B) +B

##### 

_When using the ManyLUT_ PBS _(see Algorithm 21 ), which is possible only for_ msg> 2 _, the com-
plexity of unsigned left/right shifts and rotations, as well as signed left/right rotations and signed left
shifts, is given by:_

```
C=CPBS
```

##### 

##### 

```
log 2 (msg)
```

##### 

##### + (B) +B

##### 

##### 

_The complexity of the signed right shift is given by:_

##### C=CPBS

##### 

##### 

```
log 2 (msg)
```

##### 

##### + ((B+ 1)) +B

##### 

##### 

### 3.10 Bit Shifts and Rotations

In the previous section, we explored how to perform shifts and rotations on the blocks that compose a
homomorphic integer. In this section, we introduce a method for shifting and rotating directly on the
bits of the large integer, abstracting away the underlying block layout.

#### 3.10.1 Plaintext-Ciphertext Bit Shift & Rotation

Shifting or rotating bits by a known amount is a rather easy operation that only requires one layer of
bivariatePBSthat can be done in parallel.
The scalar can be decomposed with=log 2 (msg)k+rwherekgives by how many places the
blocks should be shifted/rotated andrgives by how many places the bits shall move between blocks.
By definition, we haver 2 [0;log 2 (msg)1], meaning that bits can only shift between adjacent blocks.
Thus, using a bivariatePBSbetween a block and its predecessor or successor (depending on the shift
direction) allows us to encode this behavior. As for shifting/rotating blocks, this operation is done in
the clear and does not requirePBS: it is cheap and fast.
The plaintext-ciphertext left shift is detailed in Algorithm 45 , and the plaintext-ciphertext left
rotation in Algorithm 46 , both of which rely on auxiliary functions defined in Algorithm 44. Similarly,
the plaintext-ciphertext right shift is detailed in Algorithm 48 , and the plaintext-ciphertext right
rotation in Algorithm 49 , both relying on auxiliary functions given in Algorithm 47.

**Remark 35 (Signed Integers)** _For signed integers, the right shift is slightly different: the_ arithmetic
_shift is performed. In the arithmetic shift, instead of filling the most significant bits with_ 0 _(as in the_
logical _shift), the most significant bits are filled with the value of the_ sign bit _, requiring to slightly adapt
the function used in the different_ PBS _es._

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

**Theorem 23** _Let_

##### 

```
ct(1) be a large homomorphic integer and let =log 2 (msg)k+r. Then,
```

```
Decs
```

##### 

```
LeftShift
```

##### 

```
ct(^1 );
```

##### 

```
=m(1)= (m(1) 2 υ) (modΩ);
```

```
Decs
```

##### 

```
RightShift
```

##### 

```
ct(^1 );
```

##### 

```
=m(1)= (m(1)// 2υ) (modΩ);
```

```
Decs
```

##### 

```
RightRotate
```

##### 

```
ct(^1 );
```

##### 

```
= (m(1))j(m(1)(log 2 (Ω)));
```

```
Decs
```

##### 

```
LeftRotate
```

##### 

```
ct(^1 );
```

##### 

```
= (m(1))j(m(1)(log 2 (Ω))):
```

_Setting_

```
(x) =
```

##### (

```
0 if x= 0;
1 otherwise ;
```

_the complexity of plaintext-ciphertext left or right shift is:_

```
CShift= (Bk)(r)CPBS;
```

_and the complexity of plaintext-ciphertext left or right is:_

```
CRot=B(r)CPBS
```

```
Algorithm 44: Auxiliary functions for plaintext-ciphertext left/right shift and rotate
```

```
Input:
```

```
(
previous;current;next 2 [0;msg1] :input domains to define the-functions
shiftBetweenBlocks 2 [0;log 2 (msg)1] : a known constant value
Output: result2f 0 ; 1 g
1 Function LeftShiftBetweenBlocks(previous, current, shiftBetweenBlocks) :
2 bitsFromPrevious previousshiftBetweenBlocks
3 bitsFromCurrent (currentshiftBetweenBlocks)modmsg
4 result=bitsFromCurrent OR bitsFromPrevious
5 return result
6 Function RightShiftBetweenBlocks(current, next, shiftBetweenBlocks) :
7 bitsFromCurrent currentshiftBetweenBlocks
8 bitsFromNext (nextshiftBetweenBlocks)modmsg
9 result=bitsFromCurrent OR bitsFromPrevious
10 return result
```

#### 3.10.2 Ciphertext-Ciphertext Bit Shift & Rotation

For the shifts and rotations by an encrypted amount, the computations are more complex, and there
are a few different possibilities depending onmsg:

- Ifmsg= 2, then each block encrypts a single bit, thus bit shift/rotation is the same as block
    shift/rotation.
- Iflog 2 (msg)is a power of 2, an algorithm based on the decomposition described in Section3.10.1
    can be used.
- Otherwise, a barrel shifter can be applied.

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 45: !ct(out) LeftShift
```

##### 

```
!ct(in);
```

##### 

```
Input:
```

```
(!
ct(in):homomorphic integer
 2 [0;Blog 2 (msg)1]
Output: !ct(out): homomorphic integer
1 blockShift //log 2 (msg)
2 shiftBetweenBlocks mod log 2 (msg)
3 if shiftBetweenBlocks= 0 then
4 !ct(out) BlocksShiftRight(!ct(in);blockShift)
5 else
6 !ct(out)
```

```

ct(Triv)(0);;ct(Triv)(0)
```

(^) blockShift
**7** !ct(out):append

PBS

(x)7!(xshiftBetweenBlocks)modmsg;!ct( 0 in)

**8 for** i 2 [1;BblockShift1] **do
9** lut GenLUT

(prev;curr)7!LeftShiftBetweenBlocks(prev;curr;shiftBetweenBlocks)

**10** !ct(out):append

BivPBS

lut;ct(iin−) 1 ;ct(iin)

**11 return** !ct(out)
Recall that we have assumed thatlog 2 (msg)is a power-of-2 so the third case is not applicable here.
**Remark 36 (Signed Integers.)** _As with the plaintext-ciphertext shift (Section3.10.1), the right shift
will perform an arithmetic shift, thus the functions used in that case need to be modified slightly._
Given an encryption

##### 

ct(amount)ofm(amount), the amount we want to shift/rotate by, we can homo-
morphically decompose it into encryptions of

```
numBitShift=m(amount)mod log 2 (msg), and of
numBlockShift=m(amount)//log 2 (msg):
```

As seen above, these computations involve homomorphic division and modulo operations, both of
which have not yet been described. By analogy with hardware and big-integer libraries, they can be
assumed to be among the most time-consuming operations. However, since we know thatlog 2 (msg)
is a power of two, the integer division//is equivalent to a bit shift, and the modulo operationmod
corresponds to a _bitwise AND_ (&).
Thus, to perform ciphertext-ciphertext shifting or rotation, we first computenumBitShiftand
numBlockShiftas explained above. Then, we shift the bits bynumBitShift, followed by shifting the
blocks bynumBlockShift. SincenumBitShift 2 [0;log 2 (msg)1]and each block encrypts a value in
[0;msg1], the bivariatePBScan be used to shift bits between blocks.
Unlike the plaintext-ciphertext version of this algorithm, performing the shift requires twoPBS
operations per block, along with an addition.

```
LetamountBlockbe the first block of
```

##### 

ct(amount). Informally, to perform a ciphertext-ciphertext left
shift, we first compute, for each block in

##### 

ct(in), the new messages after applyingnumBitShiftusing the
function:
(x;amountBlock)7!(x(amountBlockmod log 2 (msg)))modmsg:
Next, we compute the carries using the function:

```
(x;amountBlock)7!x(log 2 (msg)(amountBlockmod log 2 (msg))):
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 46:
```

##### 

```
ct(out) LeftRotate
```

##### 

```
ct(in);
```

##### 

```
Input:
```

```
(!
ct(in):homomorphic integer
 2 [0;Blog 2 (msg)1]
Output: !ct(out): homomorphic integer
1 blockShift //log 2 (msg)
2 shiftBetweenBlocks mod log 2 (msg)
3 !ct(out) BlocksRotateRight(!ct(out))
4 blockShift //msg
5 shiftBetweenBlocks modmsg
6 !ct(out) BlocksRotateRight(!ct(out))
7 if shiftBetweenBlocks 6 = 0 then
8 lut GenLUT
```

```

(prev;curr)7!LeftShiftBetweenBlocks(prev;curr;shiftBetweenBlocks)
```

```

```

```
9 for i 2 [0;B1] do
10 prev if i= 0 then B 1 else i 1
11 ct(iout) BivPBS
```

```

lut;ct(prevout);ct(iout)
```

```

```

```
12 return !ct(out)
```

```
Algorithm 47: Auxiliary functions plaintext-ciphertext right rotate and shift
Input: mostSignificantBlock 2 [0;msg1] :input domain to define the-functions
Output: result 2 [0;msg1]
1 Function RightShiftSignedBlock(mostSignificantBlock) :
2 signBit mostSignificantBlock// (msg// 2)
3 padding (msg1)signBit
4 fullBlock (paddingmsg)jmostSignificantBlock
5 result= (fullBlockshiftBetweenBlocks)modmsg
6 return result
Input: mostSignificantBlock 2 [0;msg1] :input domain to define the-functions
Output: padding 2 [0;msg1]
7 Function RightShiftSignedPaddingBlocks(mostSignificantBlock) :
8 signBit mostSignificantBlock// (msg// 2)
9 padding (msg1)signBit
10 return padding
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 48: !ct(out) RightShift
```

##### 

```
!ct(in);
```

##### 

```
Input:
```

```
(!
ct(in):homomorphic integer
 2 [0;Blog 2 (msg)1]
Output: !ct(out):homomorphic integer
1 blockShift //log 2 (msg)
2 shiftBetweenBlocks mod log 2 (msg)
3 if shiftBetweenBlocks= 0 then
4 !ct(out) BlocksShiftLeft(!ct(in))
5 else
6 !ct(out) fg
7 lut GenLUT
```

```

(current;next)7!RightShiftBetweenBlocks(current;next;shiftBetweenBlocks)
```

```

```

```
8 for i 2 [blockShift;B 2 ] do
9 !ct(out):append
```

```

BivPBS
```

```

lut;!ct(iin);!ct(iin+1)
```

```

```

```
10 if isSigned(!ct(in)) then
11 lut GenLUT

RightShiftSignedBlock()

```

```
12 !ct(out):append
```

```

PBS
```

```

lut;!ct(B−in) 1
```

```

```

```
13 lut GenLUT

RightShiftSignedPaddingBlocks()

```

```
14 ct(PaddingBlock) PBS
```

```

lut;!ct(B−in) 1
```

```

```

```
15 else
16 lut GenLUT
```

```

(x)7!(xshiftBetweenBlocks)modmsg
```

```

```

```
17 !ct(out):append
```

```

PBS
```

```

lut;!ct(B−in) 1
```

```

```

```
18 ct(PaddingBlock) ct(Triv)(0)
/*Complete with padding blocks */
19 for i 2 [0;blockShift 1 ] do
20 !ct(out):append

ct(PaddingBlock)

```

```
21 return !ct(out)
```

Finally, these carries are correctly aligned with their corresponding messages and added together.
Note that while it is possible to merge the bit shift operation with the first iteration of the block
shift—at the cost of adding a few morePBSoperations—this may not always improve performance.
If the hardware cannot handle the increased concurrency, somePBScomputations will be delayed,
resulting in a latency similar to that of the non-merged approach.

The auxiliary functions defining the look-up tables used for merged shift bits and first block shift
are given in Algorithms 50 and 51. The bit shift and bit rotation are given in Algorithm 54 , with the
details for the first round of the Barrel shift with encrypted power of 2 given in Algorithm 52 and the
details of the subsequent rounds in Algorithm 53. The bit shift and bit rotation functions also use the
block Barrel shift for the power of 2 , as defined in Algorithm 43.

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 49: !ct(out) RightRotate
```

##### 

```
!ct(in);
```

##### 

```
Input:
```

```
(!
ct(in):homomorphic integer
 2 [0;Blog 2 (msg)1]
Output: !ct(out): homomorphic integer
1 blockShift //log 2 (msg)
2 shiftBetweenBlocks mod log 2 (msg)
3 !ct(out) BlocksRotateLeft(!ct(in))
4 if shiftBetweenBlocks 6 = 0 then
5 !ct(out) fg
6 lut GenLUT
```

```

(current;next)7!RightShiftBetweenBlocks(current;next;shiftBetweenBlocks)
```

```

```

```
7 for i 2 [0;B1] do
8 next if i=B 1 then 0 else i+ 1
9 !ct(out):append
```

```

BivPBS
```

```

lut;!ct(iout);!ct(nextout)
```

```

```

```
10 return !ct(out)
```

```
Algorithm 50: Auxiliary functions forLUTfor power of 2 shifts (part 1)
Input: block;firstAmountBlock;previous 2 [0;msg1] : input domains to define the-functions
Output: result 2 [0;msg1]
1 Function MsgForBlock(block , firstAmountBlock , kind) :
2 shiftWithinBlock firstAmountBlockmod log 2 (msg)
3 shiftToNextBlock (firstAmountBlock//log 2 (msg))mod 2
4 switch kind do
5 case LeftShift OR LeftRotate do
6 result (blockshiftWithinBlock)modmsg
7 case RightShift OR RightRotate do
8 result (blockshiftWithinBlock)modmsg
9 if shiftToNextBlock= 1 then
10 result= 0
11 return result
12 Function MsgForNextBlock(previous, firstAmountBlock, kind) :
13 shiftWithinBlock firstAmountBlockmod log 2 (msg)
14 shiftToNextBlock (firstAmountBlock//log 2 (msg))mod 2
15 if shiftToNextBlock= 1 then
/*We get the message part of the previous block */
16 switch kind do
17 case LeftShift OR LeftRotate do
18 result= (previousshiftWithinBlock)modmsg
19 case RightShift OR RightRotate do
20 result= (previousshiftWithinBlock)modmsg
21 else
/*We get the carry part of the previous block */
22 switch kind do
23 case LeftShift OR LeftRotate do
24 result=previous(log 2 (msg)shiftWithinBlock)
25 case RightShift OR RightRotate do
26 result=previous(log 2 (msg)shiftWithinBlock)modmsg
```

```
27 return result
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 51: Auxiliary functions forLUTfor power of 2 shifts (part 2)
Input: block;firstAmountBlock;previous 2 [0;msg1] : input domains to define the-functions
Output: result 2 [0;msg1]
1 Function MsgForNextNextBlock(previousPrevious;firstAmountBlock;kind) :
2 shiftWithinBlock firstAmountBlockmod log 2 (msg)
3 shiftToNextBlock (firstAmountBlock//log 2 (msg))mod 2
4 if shiftToNextBlock= 1 then
/*We get the carry part of the previous block */
5 switch kind do
6 case LeftShift OR LeftRotate do
7 result= (previousPrevious(log 2 (msg)shiftWithinBlock))
8 case RightShift OR RightRotate do
9 result= (previousPrevious(log 2 (msg)shiftWithinBlock))modmsg
10 else
/*Nothing reaches that block */
11 result= 0
12 return result
13 Function MsgForBlockSignedRight(block;firstAmountBlock;kind) :
14 shiftWithinBlock firstAmountBlockmod log 2 (msg)
15 shiftToNextBlock (firstAmountBlock//log 2 (msg))mod 2
16 signBitPos log 2 (msg) 1
17 signBit (previoussignBitPos)& 1
18 paddingBlock (msg1)signBit
19 if shiftToNextBlock= 1 then
20 result=paddingBlock;
21 else
22 block (paddingBlocklog 2 (msg))jblock;
23 result= (blockshiftWithinBlock)modmsg
24 return result
25 Function MsgForNextBlockSignedRight(previous;firstAmountBlock;kind) :
26 shiftWithinBlock firstAmountBlockmod log 2 (msg)
27 shiftToNextBlock (firstAmountBlock//log 2 (msg))mod 2
28 signBitPos log 2 (msg) 1
29 signBit (previoussignBitPos)& 1
30 paddingBlock (msg1)signBit
31 if shiftToNextBlock= 1 then
32 previous (paddingBlocklog 2 (msg))jprevious;
33 result= (previousshiftWithinBlock)modmsg
34 else
35 result= (previous(log 2 (msg)shiftWithinBlock))modmsg
36 return result
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 52: Auxiliary function for first round of Barrel Shift with encrypted power of 2
```

```
Input:
```

```
8
>>>
<
>>>
:
```

```
!ct(value);!ct(amount):homomorphic integers
kind2fLeftShift;LeftRotate;RightShift;RightRotateg
maxNumBitsThatTellShift: generally equals tolog 2 (log 2 (Ω))
i.e., the number of bits to represent the maximum possible shift
Output: messagesForBlock;messagesForNextBlock;messagesForNextNextBlock=
fctgB:lists ofBblocks of encrypted values
1 Function DoFirstRound(!ct(value);!ct(amount);kind;maxNumBitsThatTellShift) :
2 offset log 2 (msg)
3 shiftBits ExtractBits(!ct(amount);offset;maxNumBitsThatTellShift)
4 messageBitsPerBlock log 2 (msg)
5 messages fg
6 for i 2 [0;B1] do
7 if isSigned
```

```
!
ct(value)
```

```

AND kind=RightShift ANDi=B 1 then
8 lut GenLUT
```

```

(block;firstAmountBlock)7!
MsgForBlockSignedRight(block;firstAmountBlock;kind)
```

```

```

```
9 else
10 lut GenLUT
```

```

(block;firstAmountBlock)7!MsgForBlock(block;firstAmountBlock;kind)
```

```

```

```
11 messages:append
```

```

BivPBS
```

```

lut;!ct(value);!ct( 0 amount)
```

```

```

```
12 messagesForNextBlock fg
13 switch kind do
14 case RightShift do
15 messagesForNextBlock:append

ct(Triv)(0)

16 range [1;B1]
17 case LeftShift do range [0;B2];
18 case LeftRotateORRightRotate do range [0;B1];
19 for range do
20 if isSigned
```

```
!
ct(value)
```

```

andkind=RightShift ANDi=B 1 then
21 LUT GenLUT
```

```

(block;firstAmountBlock)7!
MsgForBlockSignedRight(block;firstAmountBlock;kind)
```

```

```

```
22 else
23 lut GenLUT
```

```

(block;firstAmountBlock)7!MsgForBlock(block;firstAmountBlock;kind)
```

```

```

```
24 messages:append
```

```

BivPBS
```

```

lut;!ct(value);!ct( 0 amount)
```

```

```

```
25 if kind=LeftShift then
26 messagesForNextBlock:append

ct(Triv)(0)

```

```
27 messagesForNextNextBlock fg
28 switch kind do
29 case RightShift do
30 messagesForNextNextBlock:append
```

```

ct(Triv)(0)
```

```

31 messagesForNextNextBlock:append
```

```

ct(Triv)(0)
```

```

32 range [2;B1]
33 case LeftShift do range [0;B3];
34 case LeftRotate OR RightRotate do range [0;B1];
35 for i 2 range do
36 lut GenLUT
```

```

(block;firstAmountBlock)7!MsgForBlock(block;firstAmountBlock)
```

```

```

```
37 messages:append
```

```

BivPBS
```

```

lut;!ct(ivalue);!ct( 0 amount)
```

```

```

```
38 if kind=LeftShift then
39 messagesForNextNextBlock:append

ct(Triv)(0)

40 messagesForNextNextBlock:append

ct(Triv)(0)

```

```
41 return messagesForBlock;messagesForNextBlock;messagesForNextNextBlock
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 53:
```

##### 

```
ct(out) BitBarrelShiftPowerOfTwo
```

##### 

```
ct(value);
```

##### 

```
ct(amount);kind;maxBits
```

##### 

```
Input:
```

```
8
><
>:
```

```
!ct(value);!ct(amount):homomorphic integers
kind2fLeftShift;LeftRotate;RightShift;RightRotateg
maxBits 2 N
Output: !ct(out):homomorphic integer
1 Function BitBarrelShiftPowerOfTwo(!ct(value);!ct(amount);kind;maxBits) :
2 messagesForBlock;messagesForNextBlock;messagesForNextNextBlock
DoFirstRound
```

```
!
ct(value);!ct(amount);kind;maxBits
```

```

```

```
3 for i 2 [0;B1] do
4 messagesi messagesForBlocki+messagesForNextBlocki+messagesForNextNextBlocki
5 messageBitsPerBlock log 2 (msg)
6 shiftBits ExtractBits(amount;offset=messageBitsPerBlock;maxBits)
7 numBitThatTellsShiftWithinBlocks log 2 (messageBitsPerBlock)
8 numBitsAlreadyDone numBitThatTellsShiftWithinBlocks+ 1
9 shiftBits

shiftBitsnumBitsAlreadyDone;;shiftBitslen(shiftBits)
10 !ct(out)=BlockBarrelShift(messages;shiftBits;[1;maxBitsnumBitsAlreadyDone])
11 return !ct(out)
```

```
Algorithm 54:
```

##### 

```
ct(out) BitRotateShift
```

##### 

```
ct(value);
```

##### 

```
ct(amount);kind
```

##### 

```
Input:
```

```
(!
ct(value);!ct(amount):homomorphic integers
kind2fLeftShift;LeftRotate;RightShift;RightRotateg
Output: !ct(out):homomorphic integer
1 if msg= 2 then
2 shiftBits ExtractBits(amount)
3 !ct(out) BlockBarrelShift(!ct(value);shiftBits;kind;dlog 2 (B)e)
4 else
5 n dlog 2 dlog 2 (Ω)ee
6 !ct(out) BitBarrelShiftPowerOfTwo(!ct(value);!ct(amount);kind; n)
7 return !ct(out)
```

3.10 Bit Shifts and Rotations 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Theorem 24 states the guarantees of Algorithm 54 and its complexity.
```

**Theorem 24** _Let_

##### 

```
ct(1) and
```

##### 

ct(2) _be two large homomorphic integers. Let_ =dlog 2 (Blog 2 (msg))e=
dlog 2 (dlog 2 (Ω)e)e _be the number of bits required to represent the number of bits encrypted by_

##### 

ct(1)_.
For example, with_ msg= 4 _,_ B= 4 _, the number of bits encrypted by_

##### 

ct(1) _is_ Blog 2 (msg) = 8 _so we
allow shifts in the range_ [0;7] _meaning that any shift can be represented using_ dlog 2 (Blog 2 (msg))e= 3
_bits. Thus,_  _allows us to extract the appropriate amount of bits from_

##### 

ct(2) _to perform the shift in the
allowed range.
Then,_

```
Decs
```

##### 

```
LeftShift
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=m(1)(m(2)mod) = (m(1) 2 m
```

```
(2)
) (modΩ);
```

```
Decs
```

##### 

```
RightShift
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
=m(1)(m(2)mod) = (m(1)// 2m
```

```
(2)
) (modΩ);
```

```
Decs
```

##### 

```
RightRotate
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
= (m(1)(m(2)mod))j(m(1)(log 2 (Ω)m(2)));
```

```
Decs
```

##### 

```
LeftRotate
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 )
```

##### 

```
= (m(1)(m(2)mod))j(m(1)(log 2 (Ω)m(2))):
```

```
If msg> 2 , the algorithmic complexity is:
```

```
CShift=CPBS(+ ( 2 B) +B)
```

```
When using the ManyLut PBS is possible (only for msg> 4 ) we have:
```

- _For left/right bit shift of unsigned integers as well as left shift of signed integers:_

```
CShift=CPBS
```

##### 

```
 1 log 2 (msg)
log 2 (msg)
```

##### 

```
+B+ (B1) + (B2) + (( 1 log 2 (msg)B) +B
```

##### 

- _For right shift of signed integers:_

```
CShift=CPBS
```

##### 

```
 1 log 2 (msg)
log 2 (msg)
```

##### 

```
+B+ (B1) + (B2) + (( 1 log 2 (msg)(B+ 1)) +B
```

##### 

- _For left and right rotations, either signed or unsigned:_

```
CShift=CPBS
```

##### 

```
 1 log 2 (msg)
log 2 (msg)
```

##### 

```
+ (3B) + ( 1 log 2 (msg)))B+B
```

##### 

##### 

3.11 Absolute Value 3 ARITHMETIC OVER LARGE HOM. INTEGERS

### 3.11 Absolute Value

The absolute value can only be applied to _signed_ homomorphic integers, producing an _unsigned_ ho-
momorphic integer. In fact, this operation is redundant for unsigned integers, as they are inherently
non-negative. The algorithm for computing the absolute value of a signedlog 2 (Ω)-bit integermis
given in Algorithm 55 , and Theorem 25 states its guarantees and complexity. Informally, it consists of
the following steps:

```
1.Perform an arithmetic right shift (cf. Remark 35 ) onmbylog 2 (Ω) 1 bits, producing the output
mask.
```

```
2.Computem+mask.
```

```
3.Compute an XOR between(m+mask)andmask.
```

As an example, consider a signed 8-bit representation of 6 , which is written 111110102. Performing
the arithmetic right shift by 7 bits (step 1 ) and recalling that in the signed case, it fills the empty bits
with a copy of the sign bit (here it is 1 ), we obtainmask= 11111111 2. The next step is to perform
addition between these two values, i.e.,

```
111110102 + 11111111 2 = 11111001 2 ;
```

which yields 7. Finally, we retrieve the absolute value by performing a bitwiseXORbetween 7 and
mask, which yields:
111110012  111111112 = 00000110 2 ;

which is 6 in the unsigned representation, as expected.

```
Algorithm 55:
```

##### 

```
ct(Abs) Abs
```

##### 

```
ct(1)
```

##### 

```
Input: !ct(1):a signed large homomorphic integer
Output: !ct(Abs): an unsigned large homomorphic integer
1 numBits Blog 2 (msg)
2 !ct(mask)=RightShift(!ct(1);numBits 1 )
3 !ct(sum)=Add
```

```
!
ct(1);!ct(mask)
```

```

```

```
4 !ct(Abs)=Bitwise
```

```

XOR;!ct(mask);!ct(sum)
```

```

```

```
5 return !ct(Abs)
```

**Theorem 25** _Let_ !ct(1) _be a signed large homomorphic integer that encrypts_ m(1) 2

##### 

##### Ω 2 ;Ω 2  1

##### 

_Then,_

```
Decs
```

##### 

```
Abs
```

##### 

```
ct(^1 )
```

##### 

##### =

```
m(1)
2 [0;Ω1]:
```

_The algorithmic complexity is:_

```
CAbs=CShift+CAdd+CBitwise
```

### 3.12 Sum

While summation is a useful operation on its own, it also serves as a crucial building block for other
operations, such as multiplication. A naïve implementation would simply accumulate the result into a

3.12 Sum 3 ARITHMETIC OVER LARGE HOM. INTEGERS

variable and perform carry propagation after each addition. However, this approach would be highly
costly and fail to fully utilize the available space in the carry buffers.
Given a list of homomorphic integers where each block encrypts a value less thanmsg, the expression
p− 1
msg− 1 represents the maximum number of ciphertexts that can be summed before completely filling
their carry buffers. Once the carry buffers are full, the next step is to extract the message part
(xmodmsg) and the carry part (x//msg) of each block usingPBS. These are then used to create
two new integer ciphertexts, (

##### 

```
ct(m)and
```

##### 

ct(c)), which can be added back to the list of integers to be
summed. This process can be repeated until fewer thanmsgp−−^11 ciphertexts remain in the list. At this

point,

##### 

```
ct(m)and
```

##### 

ct(c)are added together and a carry propagation happens.
Blocks withdeg= 0have as only value possible 0, which does not affect the final sum value, thus
these blocks can be skipped. This small optimization is beneficial mainly in the multiplication to avoid
redundant work as the terms have some trivial ciphertexts. To make writing the algorithm slightly
simpler, the data is processed as columns of blocks as opposed to processing lines of radix ciphertexts.

**Theorem 26** _Let_

```
n
!ct(0);!ct(n−1)
o
be a list of n large homomorphic integers. Then,
```

```
Decs
```

##### 

```
Sum
```

```
n!
ct(^0 );
```

##### 

```
ct(n−^1 )
```

```
o
=
```

```
nX− 1
```

```
i=0
```

```
m(i)modΩ:
```

```
Let S=msgp−−^11 be the maximum number of ciphertexts that can be summed before completely filling
```

_their carry buffers. Let_ ci _be the number of blocks in the column at index_ i_. Let_ C(x) =

```
l
max{x−S, 0 }
S− 1
```

```
m
```

_be a function that computes how many steps happen given a number of blocks.
The complexity of the partial sum is:_

```
CPartialSum
```

```
n!
ct(0);
```

##### 

```
ct(n−1)
```

```
o
=CPBS
2
c 0 +
```

##### B−X 2

```
i=1
```

```
C(ci+C(ci− 1 ))
```

##### 

```
+C(cB− 1 +C(cB− 2 ))
```

##### 

##### 

```
The complexity of the sum is:
```

```
CSum
```

```
n!
ct(0);
```

##### 

```
ct(n−1)
```

```
o
=CPartialSum
```

```
n!
ct(0);
```

##### 

```
ct(n−1)
```

```
o
+CPBS(2B) +CPropagation:
```

_Most of the time, the input of the sum is a list of_ n _homomorphic integers with each of them having
the same number of blocks_ B_._

We present theSumalgorithm in Algorithm 57 , along with required subroutines in Algorithm 56.
We also require usage of theFullPropagatealgorithm, which is outlined in Algorithm 37.

3.13 Multiplication 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 56: Auxiliary functions used inSum
1 Function PartialSum(terms) :
Input: terms=
```

```
n!
ct(0); :::;!ct(n−1)
```

```
o
: list of homomorphic integers
Output: !ct(partialSum): homomorphic integer
2 numTerms len(terms)
/*Re-organize as columns */
3 columns
```

```
n
fg 0 ;;fgB− 1
```

```
o
```

```
4 for i 2 [0;B1] do
5 for j 2 [0;numTerms 1 ] do
6 if deg(terms[j][i]) 6 = 0 then
7 columns[i]:append(terms[j][i])
```

```
/*Process columns */
8 chunkSize
```

```
j
p− 1
msg− 1
```

```
k
```

```
9 while any(len(column)>chunkSize for column in columns) do
10 for i 2 [0;B1] do
11 if len(column[i])chunkSize then continue
12 numChunks len(column[i]) //chunkSize
13 for k 2 [0;numChunks 1 ] do
14 sum
PchunkSize− 1
j=0 columns[i]:pop()
15 m; c (PBS((x)7!xmodmsg;sum);PBS((x)7!x//msg;sum))
16 columns[i]:append(m)
17 if i <B 1 then columns[i+ 1]:append(c)
```

```
/*Go back to radix form */
18 !ct(partialSum) fg
19 for i 2 [0;B1] do
20 !ct(partialSum):append
```

```
Plen(columns[i])− 1
j=0 columns[i][j]
```

```

```

```
21 return !ct(partialSum)
```

```
Algorithm 57:
```

##### 

```
ct Sum
```

```
n!
ct(0);;
```

##### 

```
ct(n−1)
```

```
o
```

```
Input:
```

```
n!
ct(0);;!ct(n−1)
```

```
o
: list of homomorphic integers
Output: !ct(Sum): homomorphic integer
1 return FullPropagate
```

```

PartialSum
```

```
n!
ct(0);;!ct(n−1)
```

```
o
```

### 3.13 Multiplication

Multiplication is a core operation on homomorphic integers. Below, we present two types of multipli-
cation: multiplication between two homomorphic integers and multiplication between a homomorphic
integer and a plaintext. Both implementations of the multiplication only compute the _half multiplica-
tion_ , that is, for aB-by-Bmultiplication, the output hasBblocks.

#### 3.13.1 Ciphertext-Ciphertext Multiplication

The ciphertext-ciphertext multiplication (Algorithm 58 ) uses a standard schoolbook algorithm. This
algorithm can be understood as having two phases: (1) the creation of so-called _terms_ and (2) the
summation of these terms. The _term_ creation phase involves numerousPBSes to compute block-by-
block (digit-by-digit) multiplications. Since the product of two blocks exceedsmsg, it would occupy

3.13 Multiplication 3 ARITHMETIC OVER LARGE HOM. INTEGERS

part of the carry space in a way that prevents summation of the blocks (assumingmsg=carry). Thus,
the complete result of the multiplication of two blocks is done using two bivariatePBSes to get the
result directly split in to two blocks with the functions:

```
messageOfBlockMul(x;y)7!(xy)modmsg;
carryOfBlockMul(x;y)7!(xy) //msg:
```

For aB 1 -by-B 2 multiplication, we obtainB 1  B 2 ciphertexts from bivariatePBSes. The **Sum**
algorithm (Algorithm 57 ) can later take advantage of the empty carry buffers in these ciphertexts.

```
Algorithm 58:
```

##### 

```
ct(Mul) Mul
```

##### 

```
ct(lhs);
```

##### 

```
ct(rhs)
```

##### 

```
Input: !ct(lhs);!ct(rhs): two homomorphic integers
Output: !ct(out): a homomorphic integer
1 terms fg
2 for i 2 [0;B1] do
3 if deg
```

```

ct(irhs)
```

```

= 0 then continue
4 term
```

```

ct(Triv)(0)
```

(^) i
**5 for** j 2 [0;Bi1] **do
6** term:append

BivPBS

messageOfBlockMul(;);ct(irhs);ct(jlhs)

**7** terms:append(term)
**8 if** msg> 2 **then
9 for** i 2 [0;B2] **do
10 if** deg

ct(irhs)

= 0 **then** continue
**11** term

ct(Triv)(0)
(^) i+1
**12 for** j 2 [0;Bi2] **do
13** term:append

BivPBS

carryOfBlockMul(;);ct(irhs);ct(jlhs)

**14** terms:append(term)
**15 return** Sum(terms)
Theorem 27 states the guarantees and complexity of Algorithm 58.
**Theorem 27** _Let_

##### 

```
ct(1) and
```

##### 

ct(2) _be two large homomorphic integers. Then the output of Algorithm 58
satisfies:_

```
Decs
```

##### 

```
Mul
```

##### 

```
!ct( 1 );!ct( 2 )
```

##### 

```
=Decs
```

##### 

```
!ct( 1 )
```

##### 

```
Decs
```

##### 

```
!ct( 2 )
```

##### 

```
modΩ:
```

_The algorithmic complexity is:_

```
CMultiplication=CPBSB 1 B 2 +CPartialSum
```

##### 

##### 1 ; 2 ;;(1 + 2B)

#####  

```
+CPBS(2B3) +CPropagation(B2)
```

#### 3.13.2 Plaintext-Ciphertext Multiplication

The plaintext-ciphertext multiplication uses the same principle as the standard binary multiplication
recalled in Algorithm 59 and is presented in Algorithm 60.

3.14 Detecting Overflows 3 ARITHMETIC OVER LARGE HOM. INTEGERS

**Algorithm 59:** Binary Multiplication
**Input:** x; y 2 Z
**Output:** xy 2 Z
**1** result 0
**2 for** i 2 [0;numBitsOf(y)1] **do
3 if** y&1 = 1 **then
4** result result+x
**5** y y 1
**6** x x 1
**7 return** result
The algorithm leverages the fact that the scalar is a known value; it means that we can decompose
it into bits and use them as decision bits in an if statement. Another trick is that the standard binary
multiplication ofxyessentially creates a list of all possible shifted values ofx:allShifts = [x
 i for i in range(nbBits(x))]and then uses the bits ofyto select whether a particular
shift value has to be added to the partial sum. In the case of our radix representation,allShifts
can be computed more efficiently than calling ourLeftShiftmultiple times for all needed values.
In fact, we have for a shiftsthatshiftAmount= (smod log 2 (msg))+klog 2 (msg)for somek. Thus,
it suffices to compute all the shifts fors 2 [0;log 2 (msg)1], then only aBlocksShiftLeftneeds
to be applied to get any desired shift.

```
Algorithm 60:
```

##### 

```
ct(ScalarMul) ScalarMul
```

##### 

```
ct(lhs);
```

##### 

```
Input:
```

```
(!
ct(lhs): a homomorphic integer
 2 ZΩ
Output: !ct(ScalarMul): a homomorpic integer
1 preComputedShifts fg
2 for bitShift 2 [ 0 ;log 2 (msg)] 1 do
3 preComputedShifts:append
```

```

LeftShift
```

```
!
ct(lhs);bitShift
```

```

```

```
4 terms fg
5 bitLen numBitsOf()
6 for i 2 [0;bitLen 1 ] do
7 if (& 1 ) = 1 then
8 bitShift;blockShift imod log 2 (B); i//log 2 (B)
9 terms:append(BlocksShiftRight(preComputedShifts[bitShift];blockShift))
10   1 ;
11 return Sum(terms)
```

### 3.14 Detecting Overflows

When a user calls an operation with overflow detection, an additional (encrypted) boolean is returned
alongside the result computation. This boolean represents whether an overflow is present or not, and
therefore allows a user to detect overflows in their computation flow. In this section, we consider three
cases: addition in Section3.14.1, subtraction in Section3.14.2, and multiplication in Section3.14.3.

#### 3.14.1 Addition

The method to detect overflows is different for signed and unsigned integers. The computations to
determine if an overflow occurs can be performed during the carry propagation (Section3.5) as the carry
is needed to know if an overflow occurred. For both signed and unsigned integers, the overflow check

3.14 Detecting Overflows 3 ARITHMETIC OVER LARGE HOM. INTEGERS

computation adds 2PBSes and these can be scheduled to be done in parallel to other computations of
the carry propagation, meaning that the latency is not degraded assuming the hardware can handle
this extra added concurrency.

**Unsigned.** For unsigned the idea is simple, it suffices to compute the carry from the most significant
block of the homomorphic integer. The way it is done is that after the first step (Section3.5.1)

the block that encodes the state of the most significant block

##### 

ct(B−blockStates 1 )is kept instead of being
discarded, this block is called the overflow block and currently encrypts a value inf 0 ; 2 ; 4 gdepending
on if the last block is _neither_ , _propagate_ or _generate_ , respectively. After the propagation states have
been computed in the second step (Section3.5.2), the last propagation state

##### 

ct(B−propStates 1 )is added
(using the regular LWE addition) to the overflow block. Once the group carries have been resolved in
step three (Section3.5.3), the last group carry, in other words, the carry that the last group takes as
input, is also added to the overflow block (this will be taken as a trivial encryption of zero if there are
no group carries). Lastly, while the final step (Section3.5.4) is being performed, aPBSis done on the
overflow block with the following function:

```
(x)7!(x2)& 1 :
```

The result of thisPBSis the overflow flag, a ciphertext that encrypts 0 if there was no overflow or 1
if there was an overflow, using the Boolean encoding (Section2.3.2).

**Signed.** For signed integers, as they use the two’s complement, the _overflow flag_ is computed. This
flag is obtained by doing anXORbetween the input carry to the last bit and the output carry of the
last bit, which can be achieved by aPBS. In case the blocks encrypt more than one bit (msg> 2 ),
some adaptations need to be made.
The computation of the overflow flag needs to be done in two steps, both of which can be executed
in parallel to other computations.
The first overflow flag step consists in computing a ciphertext, called the overflow block, that
will hold a value that, when later combined with the input carry to the last block, will allow us to
conclude on the overflow. It can be done in parallel with the step where the block states are computed
(Section3.5.1).
To compute the overflow block, a bivariatePBSis done between the most significant blocks of both
inputs, the function used to generate the lookup table is given in Algorithm 61. ThisPBSwill result in
a ciphertext that stores the 2 possible outcomes for the overflow depending on the input carry which
is still unknown at this time.

3.14 Detecting Overflows 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 61: Signed overflow preparation
1 Function OverflowGivenCarry(lhs;rhs; c) :
Input:
```

```
(
lhs;rhs 2 [0;msg1]^2 : The possible values of most significant blocks of signed integers
c2f 0 ; 1 g:The carry received
Output: 1 if there is an overflow, 0 otherwise
2 mask (msg// 2) 1
3 lhslow lhs&mask
4 rhslow rhs&mask
5 outputCarry (lhs+rhs+c) //msg
6 inputCarryToLastbit (lhslow+rhslow+c) // (msg// 2)
7 return outputCarry 6 =inputCarryToLastbit
8 Function OverflowFlagPreparation(lhs;rhs) :
Input: lhs;rhs 2 [0;msg1]^2 : The possible values of most significant blocks of signed integers
Output: 1 if there is an overflow, 0 otherwise
9 b 0 OverflowGivenCarry(lhs;rhs;1)
10 b 1 OverflowGivenCarry(lhs;rhs;0)
11 return (b 1 3)j(b 0 2)
```

After the propagation states have been computed in the second step (Section3.5.2), the last
propagation state is added (using the regular LWE addition) to the overflow block.
Once the group carries have been resolved in the third step (Section3.5.3), the last carry, in other
words the carry that the last group takes as input is added to the overflow block.
Lastly, while final step (Section3.5.4) is being done, aPBSis done on the overflow flag with the
following function used to generate the lookup table:

```
Algorithm 62: Signed overflow finalization
1 Function OverflowFlagFinalization(x) :
Input: x 2 [0;msg1]:The possible value of most significant blocks of signed integer
Output: 1 if there is an overflow, 0 otherwise
2 inputCarry (x1)& 1
3 doesOverflowIfCarryIsOne (x3)& 1
4 doesOverflowIfCarryIsZero (x2)& 1
5 if inputCarry= 1 then
6 return doesOverflowIfCarryIsOne
7 else
8 return doesOverflowIfCarryIsZero
```

#### 3.14.2 Subtraction

**Signed.** For _signed integers_ , subtraction is done by adding the negated term, then the carry propa-
gation follows. The overflow detection is the same as for the addition of signed integers.

**Unsigned.** To get the overflow alongside the subtraction of _unsigned integers_ , we instead use sub-
traction with borrow for which the borrow of the last block indicates the overflow.
To achieve subtraction with borrow, normally each pair of blocks is subtracted, however, these
subtractions might themselves underflow, setting the padding bit value to 1 which needs to be avoided.
Thus, a _correcting term_ of the valuemsgis added.
Given two blocksaandbthat encrypt a value in[0;msg1], the result of subtraction is in
[(msg1);msg1], therefore, by addingmsg, we find the final result in[1; 2 msg1]. This allows

3.15 Conditionals 3 ARITHMETIC OVER LARGE HOM. INTEGERS

us to apply aPBSon each block to get the _borrow state_ :

```
unshifted_state(x) =
```

##### 8

##### ><

##### >

```
Borrows ifx <msg;
Propagates ifx=msg;
Neither otherwise;
```

where the values forBorrows,Propagates,Neithervalues are described in Section3.5(Generatebecomes
Borrows), as the borrow propagation follows a similar pattern.
As with the carry propagation (Eq. ( 30 )), the borrows computed need to be shifted to the correct
position depending on the block’s group. The cleanAndShift function changes slightly to only be a
shift function: Shift(x) =x 1. That is because the extra bit abovemsgwill serve as an overflow
protection when the compute borrow may later be subtracted from the block.
Now that we have the _block states_ we can use the same algorithm described in the second phase of
the carry propagation (Section3.5.2) with however one difference for how _prepared blocks_ are created.
In the carry propagation, we took the _shifted blocks_ and added the _propagation states_ , i.e.prepared_blocki=
shifted_blocksi+propagation_statesi. This works as the _propagation states_ are seen as carries whereas
in the subtraction with borrow we see them as borrows, thus in the subtraction with borrow, the
formula changes to:

```
prepared_blocki=shifted_blocksipropagation_statesi+ 1
The way it works is that the equation above is equivalent to
```

prepared_blocki=shifted_blocksi+ (propagation_statesi+ 1):
Aspropagation_statesi2f 0 ; 1 ; 2 gwe have thatpropagation_statesi+ 12f 1 ; 0 ; 1 gmore specifi-
cally:

- 2 : Generates becomes 1 , meaning we will properly subtract a borrow from the next block
- 1 : Propagates becomes 0 , meaning if a borrow ( 1 ) comes from a previous block, it will get
    propagated as 0 1 = 1
- 0 : Neither (Absorb) becomes 1 , meaning if a borrow ( 1 ) it will be absorbed as 1 1 = 0

Once that is done the borrow between groups (which is still encoded in thef 0 ; 1 ; 2 gset) can be
propagated using the same methods described for the carry propagation (Section3.5.3)
Finally the last step (Section3.5.4) is also similar, the change is that the borrow is subtracted (as
opposed to the carry which is being added). To get the overflow indicator, the borrow out of the last
block must be computed.

#### 3.14.3 Multiplication

TFHE-rs supports an overflowing variant of multiplication, where we apply similar techniques as
discussed in Section3.14.2. Full details will be given in a later version of the document.

### 3.15 Conditionals

One of the main differences between FHE computation and classical computation is that conditional
statements cannot be used to dynamically trim computations based on encrypted values. Instead, every
possible branch must be executed before selecting the correct one. While FHE allows for conditional
execution, all branches must be computed in parallel, after which the appropriate result is selected,

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

allowing computation to proceed. ThePBSprovides an efficient mechanism to select the correct
branch based on an encrypted condition, enabling support for conditional statements such asifand
if-else.
Algorithm 63 details how to perform homomorphicif-elsestatements, while Theorem 28 states
its guarantees and complexity. Note that Algorithm 63 also supportsifstatements, provided they
are rewritten asif-elsestatements before calling the algorithm. In Algorithm 63 , the lastPBSwith
theidentityfunction is there to decrease the noise of the output of the algorithm.

```
Algorithm 63: !ct(out) Select
```

##### 

```
ct(condition);!ct(true);!ct(false)
```

##### 

```
Input:
```

```
8
><
>:
```

```
ct(condition):encrypted boolean
!ct(true):homomorphic integer
!ct(false):homomorphic integer
```

```
Output:
```

```
(!
ct(out):homomorphic integer that encrypts the same value as!ct(true)ifm(cond)=true;
or!ct(false)ifm(cond)=false
1 ct(invertedCondition) BitwiseNOT
```

```

ct(condition)
```

```

```

```
2 lut GenLUT
```

```

(cond;value)7!condvalue
```

```

```

```
3 for i 2 [0;B1] do
4 ct(t) BivPBS
```

```

lut;ct(condition);ct(itrue)
```

```

```

```
5 ct(f) BivPBS
```

```

lut;ct(invertedCondition);ct(ifalse)
```

```

```

```
6 ct(iresult) PBS
```

```

identity;ct(t)+ct(f)
```

```

```

```
7 return !ct(result)
```

**Theorem 28** _Let_

##### 

```
ct(true) and
```

##### 

```
ct(false) be two large homomorphic integers, and ct(condition) an LWE
```

_ciphertext with_ Decs

##### 

```
!ct(condition)
```

##### 

```
2f 0 ; 1 g
```

```
Decs
```

##### 

```
Select
```

##### 

```
ct(condition);
```

##### 

```
ct(true);
```

##### 

```
ct(false)
```

##### 

##### =

##### 8

##### <

##### 

```
Decs
```

##### 

```
ct(true)
```

##### 

```
; if Decs
```

##### 

```
ct(condition)
```

##### 

##### = 1

```
Decs
```

##### 

```
ct(false)
```

##### 

```
; if Decs
```

##### 

```
ct(condition)
```

##### 

##### = 0

```
The algorithmic complexity is: CSelect= 3BCPBS
```

### 3.16 Comparisons

Comparison algorithms (<;;>;) in TFHE-rs are based on the subtraction algorithm. In particular,
to perform a comparison we essentially perform a subtraction with borrow, and propagate the borrow
to compute the result.
For unsigned integers, computing the overflowing subtraction (Algorithms 40 and 61 ) is sufficient,
as the overflow flag directly corresponds to the result of the comparison. Specifically,aboverflows
if and only ifa < b. For signed integers, it is slightly less direct, but the algorithm still relies on
subtraction. However, for both variants, comparison uses fewerPBSes than the full borrow/carries
propagation algorithm, as only the final borrow is needed (unlike the actual result of subtraction).
There are four supported comparison operations, each of which can be evaluated using overflowing
subtraction:

- a < b() overflows(ab),
- ab () :(b < a) () :(overflows(ba)),
- a > b() b < a() overflows(ba),

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

- ab () ba() :(overflows(ab)).

As described above,(a < b)is determined by computing the overflow of(ab). Similarly,(ab)
can be expressed as:(b < a), meaning that we can use the same approach with an additionalNOT
operation. The same principle applies to(a > b)and(ab)by swapping the order of the operands.
This ensures that all comparisons follow the same algorithm, differing only in whether the inputs are
swapped and/or the boolean result is negated. The complete comparison algorithm for both signed
and unsigned integers is presented in Algorithm 69. We now briefly outline the differences between
the unsigned and signed cases.

**Unsigned.** As outlined, in the unsigned case, computing the subtraction overflow flag is enough.
Moreover, computing the overflow flag for unsigned integer subtraction is as simple as computing the
output borrow of the last block. The borrow state is computed using the functionComputeGroup-
BorrowStateoutlined in Algorithm 65.

**Signed.** For signed integers, the algorithm is more complex: both the overflow flag and the result-
ing sign bit have to be computed andXORed together, which requires additionalPBSes compared
to the unsigned case. This additional step in Algorithm 69 can be seen in thePrepareSigned-
Checkfunction in Algorithm 67. However, when blocks encrypt 4 bits or more (i.e.,p 16 ), both
these computations can be merged into one in a way similar to the overflow detection in case of the
addition of signed numbers. This idea is shown in theSignedCmpResultPreparationfunction
in Algorithm 64.

```
Algorithm 64: Comparison: auxiliary functions to generateLUTs specific to signed integers
1 Function IsLessThanGivenCarry(lhs;rhs; c) :
```

```
Input:
```

```
8
><
>:
```

```
lhs 2 [0;msg1]
rhs 2 [0;msg1]
c2f 0 ; 1 g
Output: f 0 ; 1 g
2 mask (msg// 2) 1
3 lhslow lhs&mask
4 rhslow rhs&mask
5 outputBorrow lhs<(rhs+c)
6 inputBorrowToLastBit lhslow<(rhslow+c)
7 r lhsrhs+msg
8 signBit r// (msg// 2)
9 overflowFlag inputBorrowToLastBitoutputBorrow
10 return overflowFlagsignBit
11 Function SignedCmpResultPreparation(lhs;rhs) :
12 b 1 IsLessThanGivenCarry(lhs;rhs;1)
13 b 0 IsLessThanGivenCarry(lhs;rhs;0)
14 return (b 1 3)j(b 0 2)
```

**Theorem 29** _Let_

##### 

```
ct(1) and
```

##### 

ct(2) _be two large homomorphic integers, and let_  _denote an arbitrary
condition in_ f<;;>;g_. Then the output of Algorithm 69 satisfies:_

```
Decs
```

##### 

```
Compare
```

##### 

```
ct(^1 );
```

##### 

```
ct(^2 );
```

##### 

##### =

##### (

```
1 if Decs
```

##### 

```
ct(^1 )
```

##### 

```
Decs
```

##### 

```
ct(^2 )
```

##### 

##### 

```
0 otherwise.
```

```
Let groupSize=log 2 (p) and numGroup=
```

```
l
B
groupSize
```

```
m
```

_. For unsigned comparisons, the complexity is:_

```
CUnsignedCmp= (B+numGroup+numGroup1)CPBS;
```

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

_for signed comparisons, the complexity is:_

```
CSignedCmp= ((B+ 1) + (numGroup+ 1) +)CPBS;
```

_where_ =

##### (

```
1 if numGroup= 1;
numGroup 1 otherwise.
```

**Remark 37 (Plaintext-Ciphertext Comparison)** _To perform a plaintext-ciphertext comparison,
we can use the same algorithm by treating the plaintext as a trivial ciphertext (see Definition 6 ). A
more efficient version is available in TFHE-rs and will be included in future versions of this document._

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 65: Comparison: auxiliary function to compute the group borrow state
1 Function ComputeGroupBorrowState(invertResult;groupSize;blockStates) :
```

```
Input:
```

```
8
>><
```

```
>>:
```

```
invertResult2ftrue;falseg
groupSize 2 N
blockStates=
```

```

ct(jblockState)
```

```
B− 1
j=0:aB-tuple of block states
Output: groupPropagationStates=
```

```

ct(igroupState)
```

```
ng− 1
i=0
:ang=
```

```
l
B
groupSize
```

```
m
-tuple of group states
2 numGroups
```

```
l
B
groupSize
```

```
m
```

```
3 numBlocksInLastGroup BmodgroupSize
4 numCarries numGroups 1
5 seqDepth (numCarries1) // (groupSize1)
6 hillisSteeleDepth if numCarries= 0 then 0 else dlog 2 (numCarries)e
7 useSeq seqDepthhillisSteeleDepth
/*The last group may not be full */
8 if numGroups= 1 then
9 lastGroupLut GenLUT
```

```

(x)7!((x(numBlocksInLastGroup1))&1)invertResult
```

```

```

```
10 lastGroupLutCorrector 0
11 else
/*There are at least 2 groups, the first and the last having specificLUTs */
12 firstGroupLut GenLUT
```

```

(x)7!(x(log 2 (p)1))& 1
```

```

```

```
13 otherGroupLuts fg
14 if useSeq then
15 for i 2 [0;groupSize2] do
16 otherGroupLuts:append
```

```

GenLUT
```

```

(x)7!((x 6 =p1)i)mod 2 p
```

```

```

```
17 else
18 Function LocalFn(x) :
19 if x=p 1 then return 2 else return  1
20 otherGroupLuts:append
```

```

GenLUT
```

```

LocalFn()
```

```

```

```
21 if numBlocksInLastGroup = groupSize then
22 lastGroupLut otherGroupLuts[0]
23 lastGroupLutCorrector 1
24 else
25 Function LocalFn(x) :
26 if x(1numBlocksInLastGroup) then
27 return 2
28 else if x= (1numBlocksInLastGroup) 1 then
29 return 1
30 else
31 return 0
32 lastGroupLut GenLUT
```

```

LocalFn()
```

```

33 lastGroupLutCorrector 0
34 groupPropagationStates fg
35 for i 2 [0;numGroups1] do
36 sum
Pmin(B− 1 ,(i+1)·groupSize)
j=i·groupSize ct
```

```
(blockState)
j
37 if i=numGroups 1 then
38 result PBS(lastGroupLut;sum)
39 result result+lastGroupLutCorrector
40 else if i= 0 then
41 result PBS(firstGroupLut;sum)
42 else
43 index if useSeq then (i1)mod(groupSize1) else 0
44 result PBS(otherGroupLuts[index];sum)
45 corrector if useSeq then 1 ((i1)mod(groupSize1)) else 1
46 result result+corrector
47 groupPropagationStates:append(result)
48 return groupPropagationStates 89
```

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 66: Comparison: auxiliary functions to compare two values given a borrow
1 Function IsXLessThanYGivenInputBorrow(lastXBlock;lastYBlock;borrow) :
```

```
Input:
```

```
8
><
>:
```

```
lastXBlock 2 [0;msg1]
lastYBlock 2 [0;msg1]
borrow2f 0 ; 1 g
Output: result2ftrue;falseg
2 lastBitPos log 2 (msg) 1
3 mask (1lastBitPos) 1
4 xWithoutLastBit lastXBlock&mask
5 yWithoutLastBit lastYBlock&mask
6 inputBorrowToLastBit xWithoutLastBit<(yWithoutLastBit+borrow)
7 result lastXBlock+lastYBlock+borrow
8 outputSignBit (resultlastBitPos)& 1
9 outputBorrow lastXBlock<(lastYBlock+borrow)
10 overflowFlag inputBorrowToLastBitoutputBorrow
11 return outputSignBitoverflowFlag
```

```
Algorithm 67: Comparison: auxiliary functions to prepare the signed check
1 Function PrepareSignedCheck(ct(B−lhs) 1 ;ct(B−rhs) 1 ) :
Input:
```

```
(
ct(B−lhs) 1 :the most significant block of a signed homomorphic integer
ct(B−rhs) 1 :the most significant block of a signed homomorphic integer
```

```
Output:
```

```
(
ct(out):a block that encryptsm2f 0 ; 4 ; 8 ; 12 gifmsg 4
ct(out,1);ct(out,2)

:a pair of blocks that encryptm(i)2f 0 ; 1 gotherwise
2 if msg 4 then
3 Function LocalFn(x; y) :
/*Calls Algorithm 66 */
4 b 0 IsXLessThanYGivenInputBorrow(x; y;0)
5 b 1 IsXLessThanYGivenInputBorrow(x; y;1)
6 return ((b 1 1)jb 0 ) 2
7 return BivPBS
```

```

LocalFn;ct(B−lhs) 1 ;ct(B−rhs) 1
```

```

```

```
8 else
9 b 0 BivPBS
```

```

(x; y)7!IsXLessThanYGivenInputBorrow(x; y;0);ct(B−lhs) 1 ;ct(B−rhs) 1
```

```

```

```
10 b 1 BivPBS
```

```

(x; y)7!IsXLessThanYGivenInputBorrow(x; y;1);ct(B−lhs) 1 ;ct(B−rhs) 1
```

```

```

```
11 return (b 0 ; b 1 )
```

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 68: Comparison: auxiliary function to finalize comparison
1 Function FinishCompare(groupBorrows;preparedSignedCheck;useSequentialAlgorithm;invertResult) :
```

```
Input:
```

```
8
>>>
><
>>>
>:
```

```
groupBorrows=
```

```

ct(igroupBorrow)
```

```

:a tuple of group borrows
preparedSignedCheck:result ofPrepareSignedCheckfor signed numbers, otherwiseNone
useSequentialAlgorithm2ftrue;falseg:flag if the sequential algorithm is to be used
invertResult2ftrue;falseg:flag if the result is to be inverted
Output: ct(result):encrypted boolean
2 lastGroupBorrowState groupBorrows:pop()
3 if groupBorrows:isEmpty() then
4 return lastGroupBorrowState
5 else if useSequentialAlgorithm then
/*Algorithm 33 */
6 resolvedBorrows ResolveGroupCarriesSequentially(groupBorrows)
7 else
/*Algorithm 35 */
8 resolvedBorrows ResolveGroupsCarriesUsingHillisSteele(groupBorrows)
9 if preparedSignedCheck=None then
10 result lastGroupBorrowState+resolvedBorrows:last()
11 result PBS((x)7!((x1)&1)invertResult;result)
12 return result
13 else
14 if len(preparedSignedCheck) = 1 then
15 if len(resolvedBorrows)> 0 then
16 lastGroupBorrowState lastGroupBorrowState+resolvedBorrows:last()
17 result lastGroupBorrowState+preparedSignedCheck
18 Function LocalFn(block) :
19 index if len(resolvedBorrows)> 0 then 0 else 1
20 inputBorrw (blockindex)& 1
21 if inputBorrow= 1 then
22 return (block3)& 1
23 else
24 return (block2)& 1
25 result PBS(LocalFn;result)
26 return result
27 else
28 if len(resolvedBorrows)> 0 then
29 lastGroupBorrowState lastGroupBorrowState+resolvedBorrows:last()
30 lastGroupBorrowState PBS((x)7!x 1 & 1 ;lastGroupBorrowState)
31 result if lastGroupBorrowState then resolvedBorrows[1] else resolvedBorrows[0]
32 if invertResult then
33 return BitwiseNOT(result)
34 else
35 return result
```

3.16 Comparisons 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 69: ct(Compare) Compare
```

##### 

```
ct(lhs);
```

##### 

```
ct(rhs);kind
```

##### 

```
Input:
```

```
(!
ct(lhs);!ct(rhs):homomorphic integers
kind2fLess;LessOrEqual;Greater;GreaterOrEqualg
Output: ct(Compare):encrypted boolean
1 switch kind do
2 case Less do
3 lhs;rhs;invertResult
```

```
!
ct(lhs);!ct(rhs);false
```

```

```

```
4 case LessOrEqual do
5 lhs;rhs;invertResult
```

```
!
ct(rhs);!ct(lhs);true
```

```

```

```
6 case Greater do
7 lhs;rhs;invertResult
```

```
!
ct(rhs);!ct(lhs);false
```

```

```

```
8 case GreaterOrEqual do
9 lhs;rhs;invertResult
```

```
!
ct(lhs);!ct(rhs);true
```

```

```

```
10 subBlocks fg
11 for i 2 [0;B1] do
12 subBlocks:append(lhsirhsi)
13 groupSize blog 2 (p)c
14 firstGroupLuts;otherGroupLuts CreateBlockStatesLuts(groupSize); //Algorithm 29
15 blocksStates fg
16 for i 2 [0;B 1 isSigned(lhs)] do
17 groupIndex;indexInGroup (i//groupSize; imodgroupSize)
18 if groupIndex= 0 then
19 blocksStates:append(PBS(firstGroupLuts[indexInGroup];subBlocksi))
20 else
21 blocksStates:append(PBS(otherGroupLuts[indexInGroup];subBlocksi))
```

```
22 groupBorrows ComputeGroupBorrowState(invertResult;groupSize;blocksStates); //Algorithm 65
23 maybePreparedSignedCheck None
24 if isSigned(lhs) then
25 maybePreparedSignedCheck PrepareSignedCheck(lhsB− 1 ;rhsB− 1 ); //Algorithm 67
/*Final call to Algorithm 68 */
26 return FinishCompare(groupBorrows;maybePreparedSignedCheck;useSequentialAlgorithm;invertResult)
```

3.17 Division and Modulo 3 ARITHMETIC OVER LARGE HOM. INTEGERS

### 3.17 Division and Modulo

The division and modulo operations are the most challenging of the basic arithmetic operations.

#### 3.17.1 Ciphertext-Ciphertext Division & Modulo

**Unsigned.** Division by an unsigned encrypted value is based on the binary long division algorithm,
adapted to the homomorphic setting. Recall that for binary digits, when computing the digits of the
quotient in this method, the next digit can be computed as the Booleanrdwheredis the divisor
andris the value of the current remainder; then we branch on the value of this digit.
This branching typically requires the evaluation of anifstatement, and, although conditional
execution is not possible, theifstatement can be replaced with aselect(Section3.15). However,
this means the computations inside theifhave to be done anyway. Thus, to make the algorithm FHE
friendly, the comparison can be removed by computing the overflow of the subtractionrdsincerd
overflows if and only ifd > rwhich is if and only if:(rd).
Another improvement that can be made is splitting the remainderrand divisordin two. As we
know, the remainder starts with all bits set to 0 and at each iteration, one more bit is set, we know
which part ofrconsists of 0 bits, allowing us to take the same part ofdcompare it with 0 and do
a properoverflowing_subtractionfor the other part and later combine the two together. This
is something that is worth doing as it reduces the depth of the carry propagation needed during each
iteration. This brings us to the division algorithm implemented inside TFHE-rs, which is presented in
Algorithm 71 and uses Algorithm 70 as a subroutine.

```
Algorithm 70: Auxiliary function for the division
1 Function SplitAt(!ct(in);bitIndex) :
```

```
Input:
```

```
8
><
>:
```

```
!ct(in):a homomorphic integer
bitIndex 2 [0;log 2 (Ω)1]:bit index at which!ct(in)shall be split.
The bit at indexbitIndexis contained in the second part
Output: !ct(low);!ct(high):
two homomorphic integers encrypting the lower and higher order bits of the input
2 blockIndex bitIndex//log 2 (msg)
3 if (bitIndexmod log 2 (msg)) = 0 then
4 return
```

```
n!
ct( 0 in);!ct(blockIndexin) − 1
```

```
o
;
```

```
n!
ct(blockIndexin) ;;!ct(B−in) 1
```

```
o
```

```
5 else
6 !ct(low)
```

```
n!
ct( 0 in);!ct(blockIndexin) − 1
```

```
o
```

```
7 !ct(blockIndexlow) PBS
```

```

(x)7!xmod(1(bitIndexmod log 2 (msg)));!ct(blockIndexin)
```

```

```

```
8 !ct(high) RightShift
```

```
n!
ct(blockIndexin) ;;!ct(B−in) 1
```

```
o
;bitIndexmod log 2 (msg)
```

```

```

```
9 return !ct(low);!ct(high)
```

3.17 Division and Modulo 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 71:
```

##### 

```
!ct(q);!ct(r)
```

##### 

```
UnsignedDivRem
```

##### 

```
!ct(n);!ct(d)
```

##### 

```
Input:
```

```
(!
ct(n):a homomorphic integer encrypting the numerator of the division
!ct(d):a homomorphic integer encrypting the denominator of the division
```

```
Output:
```

```
!
ct(q);!ct(r)
```

```

:two homomorphic integers encryptingm(q); m(r)where
m(n)=m(q)m(d)+m(r)with 0 m(r)< m(d)
1 !ct(r)
```

```

ct(Triv)(0)
i∈[0,B−1]
2 dlen numBitsOf
```

```
!
ct(d)
```

```

```

```
3 for i 2 [0; dlen] do
4 !ct(r) LeftShift
```

```
!
ct(r); 1
```

```

```

```
/*Set the LSB ofrequal to thei-th bit ofn */
5 ct( 0 r) ct( 0 r)+PBS
```

```

i-th bit ofx;!ct(n)
```

```

```

```
6 !ct(highr);!ct(lowr) SplitAt(!ct(r); i)
7 !ct(highd);!ct(lowd) SplitAt(!ct(d); i)
8 !ct(r.new);ct(overflow) overflowingSub(!ct(lowr);!ct(lowd))
9 ct(overflow) Bitwise
```

```

OR;ct(overflow);Neq
```

```
!
ct(highd); 0
```

```

```

```
10 !ct(r) Select
```

```

ct(overflow);!ct(r);!ct(r.new)
```

```

```

```
/*Set thei-th bit ofq; simplified */
11 ct(iq) Neg

ct(overflow)

```

```
12 return
```

```
!
ct(q);!ct(r)
```

```

```

**Theorem 30** _Let_

##### 

```
ct(n);
```

##### 

ct(d) _be two large homomorphic integers encrypting unsigned values_ m(n) _and_
m(d) _, respectively. Then,_

```
UnsignedDivRem
```

##### 

```
ct(n);
```

##### 

```
ct(d)
```

##### 

##### =

##### 

```
ct(q);
```

##### 

```
ct(r)
```

##### 

```
with
```

```
Decs
```

##### 

```
ct(q)
```

##### 

```
=m(n)//m(d) and Decs
```

##### 

```
ct(r)
```

##### 

```
=m(n)modm(d):
```

**Signed.** The signed integer division relies on the unsigned division, with a few pre-processing and
post-processing steps. The algorithm used for signed integers gives a division that rounds towards 0.

3.17 Division and Modulo 3 ARITHMETIC OVER LARGE HOM. INTEGERS

```
Algorithm 72:
```

##### 

```
!ct(q);!ct(r)
```

##### 

```
SignedDivRem
```

##### 

```
!ct(n);!ct(d)
```

##### 

```
Input:
```

```
(!
ct(n):a homomorphic integer encrypting the numerator of the division
!ct(d):a homomorphic integer encrypting the denominator of the division
```

```
Output:
```

```
!
ct(q);!ct(r)
```

```

:two homomorphic integers encryptingm(q); m(r)where
m(n)=m(q)m(d)+m(r)with m(d)< m(r)< m(d)
1 !ct(d+) Abs
```

```
!
ct(d)
```

```

```

```
2 !ct(n+) Abs
```

```
!
ct(n)
```

```

```

```
/*position within the most significant block */
3 signBitPos log 2 (msg) 1
4
```

```
!
ct(q);!ct(r)
```

```

UnsignedDivRem
```

```
!
ct(n+);!ct(d+)
```

```

```

```
5 lut GenLUT
```

```

(x; y)7!(xsignBitPos) 6 = (ysignBitPos)
```

```

```

```
6 signBitsAreDifferent BivPBS
```

```

lut;!ct(B−n) 1 ;!ct(B−d) 1
```

```

```

```
7 !ct(q) select
```

```

signBitsAreDifferent;!ct(q);!ct(q)
```

```

```

```
8 numeratorIsNegative PBS
```

```

(x)7!xsignBitPos;!ct(B−n) 1
```

```

```

```
9 r select
```

```

numeratorIsNegative;!ct(r);!ct(r)
```

```

```

```
10 return !ct(q);!ct(r)
```

#### 3.17.2 Plaintext-Ciphertext Division & Modulo

Dividing by a clear value is significantly more efficient due to the large performance gap between clear
operations and FHE operations. A common optimization used by compilers and big-integer libraries
is to replace division with multiplication by the precomputed inverse.
Our division and modulo by known value are using the algorithm described in _Division by Invariant
Integers using Multiplication_ [GM94]. The computation of the inverse happens in the clear, and then
the actual division/modulo uses operations already defined in this document.

## 4 LWE Ciphertext Compression

## 4 LWE Ciphertext Compression

The _expansion factor_ of an LWE ciphertextct 2 LWEs(me)Znq+1is given by the ratio between
the ciphertext size and the plaintext size. As an example, the default parameters in TFHE-rs are
V1_0_PARAM_MESSAGE_2_CARRY_2_KS_PBS_TUNIFORM_2M128and usen= 879andq = 2^64
meaning that the ciphertext size is 880 64 = 56; 320 bits. In this parameter set, each ciphertext
encrypts just two bits, meaning that the expansion factor is 56 ;320/2 = 28; 160. As we can see, LWE
ciphertexts have a large expansion factor and it is therefore desirable to compress ciphertexts at various
stages of FHE computation. There are three types of compression which are particularly useful:

```
1.Input ciphertext compression: this comprises of compressing LWE ciphertexts, typically gener-
ated by a client, which are to be sent to a server for homomorphic computation.
```

```
2.Intermediate ciphertext compression: this comprises of compressing LWE ciphertexts during
computation (i.e., these ciphertexts are both outputs of previous computation and are required
to be inputs to future computation). This compression occurs entirely on the server.
```

```
3.Output ciphertext compression: this comprises of compressing LWE ciphertexts which contain
the result of a homomorphic computation, and are to be sent back to the client for decryption.
```

Note that input and output compression are typically aimed at saving _bandwidth_ , whilst interme-
diate compression is aimed at saving _storage_. The input ciphertext compression technique in TFHE-rs
uses a CSPRNG to compress the random mask terms to a seed, and is explained in Remark 5.

### 4.1 Input Ciphertext Compression

In Table 1 , we present all relevant object sizes for the TFHE-rs scheme. In particular, we outline
(theoretical) sizes of all secret keys, ciphertexts, and evaluation keys used as part of the TFHE scheme,
including those for fresh ciphertexts compressed using a CSPRNG (see Remark 5 ).

```
Object
```

```
Size (bits)
Uncompressed CSPRNG Compressed
s n —
S kN —
LWEciphertext (n+ 1)log 2 q +log 2 q
GLWEciphertext (k+ 1)Nlog 2 q +Nlog 2 q
GLevciphertext `(k+ 1)Nlog 2 q +`Nlog 2 q
GGSWciphertext `(k+ 1)^2 Nlog 2 q +`(k+ 1)Nlog 2 q
BSK n`BSK(k+ 1)^2 Nlog 2 q +n`BSK(k+ 1)Nlog 2 q
MB-BSK 2 gfngf`BSK(k+ 1)^2 Nlog 2 q + 2gfngf`BSK(k+ 1)Nlog 2 q
LWEKSK `KSK(n+ 1)kNlog 2 q +kN`KSKlog 2 q
PKSK n`PKS( ̄k+ 1)N ̄log 2 q +n`PKSN ̄log 2 (q)
DMK Z(n+ 1)log 2 q +Zlog 2 q
```

Table 1:Sizes of ciphertexts and evaluation key material in TFHE,denotes the bit size of the seed
used in the CSPRNG.

4.2 Intermediate Ciphertext Compression 4 LWE CIPHERTEXT COMPRESSION

### 4.2 Intermediate Ciphertext Compression

The most complex setting for compression is the intermediate case. Here, the ciphertexts to be
compressed are assumed to be the output of a previous bootstrap (and therefore have a higher noise
level than fresh LWE ciphertexts). These ciphertexts must be stored for future computation, under the
constraint that we want the storage requirement to be as small as possible (i.e, the expansion factor
should be minimized). The technique used within the TFHE-rs library for this is a combination of a
packing-keyswitch and a modulus switch. In particular, given a homomorphic integer!ctwhich is made
up of individual LWE ciphertexts encrypting the messagesm 0 ;m 1 ;mB− 1 , the packing keyswitch
packs these messages into a GLWE ciphertext encrypting the message

##### PB− 1

```
j=0mjX
ijfor a given set
```

of indices 0 i 0 < i 1 << iB− 1 N 1.

```
CTpack PackingKS(fctjgB−j=0^1 ;fijgB−j=0^1 ;KSK) 2 GLWE
```

##### 0

##### @

##### B−X 1

```
j=0
```

```
mjXij
```

##### 1

##### A

```
which generates a GLWE encryption of
```

##### PB− 1

```
j=0mjX
```

```
ij. This is followed by a modulus switch to
```

the storage modulusq ̄(note that TFHE-rs typically choosesq ̄= 2N):

CTCompress MS(ctin;q ̄)
to produce the output, compressed, GLWE ciphertextCTCompress. Decompression is computed via
a combination of sample extractions and blind rotations. This technique allows us to packB N
LWE ciphertexts (the default parameters in the TFHE-rs library support compression of up to 256
individual LWE ciphertexts into a single GLWE ciphertext) of original sizeB(n+ 1)log 2 (q)into
a single GLWE ciphertext of size( ̄k+ 1)N ̄log 2 ( ̄q)where( ̄k;N; ̄q ̄)are compression-specific GLWE
parameters. When using the right parameters, the expansion factor can be reduced from 28 ; 160 to
30 for 256 LWEs packed into a single GLWE ciphertext. We note that lower expansion factors can
be achieved (either by packing a greater number of LWE ciphertexts, or via a different parameter
optimization) however the associated cost can increase. The core flow of our compression algorithm
can be seen in Fig. 8.

```
LWEkN
```

```
· · ·
```

```
GLWE( ̄k, ̄q)N ̄ LWE( ̄k ̄qN) ̄ LWE(2k ̄N ̄N) GLWEk,N LWEkN
```

```
SEi MS BR(D) SE
```

##### PKS

##### &MS

##### IN: OUT

```
usual FHE computations
```

Figure 8: Computation flow for the intermediate compression: a list of LWE’s at the input, the
compressed ciphertext given in a bold box, thei-th decompressed sample at the output. PKS+MS
denotes the application of a packing keyswitch and modulus switch operation,SEdenotes a sample
extraction,MSdenotes a modulus switch,BR(D)denotes blind rotation with the usage of a decompres-
sion key. Ciphertexts’ dimensions (and polynomial degrees) are given as a subscript, and the modulus
is the default one (i.e.,q) unless stated explicitly in the superscript. The dimensions (and polynomial
degrees) are:kNfor the input LWE’s,(k; ̄N ̄)forPKS. The modulus is denoted using the superscript,
and the storage modulusq ̄is typically set to be 2 Nin TFHE-rs.

We now define the compression and decompression algorithms. Here, we consider the packing and
unpacking of a _single_ homomorphic integer of generic size. We note that several homomorphic integers
can be packed together, simply by viewing them as a single large integer. For a generic description of

4.2 Intermediate Ciphertext Compression 4 LWE CIPHERTEXT COMPRESSION

the packing keyswitch algorithm, which is a major subroutine used in the packing, we refer the reader
to Algorithm 13.

```
Algorithm 73: CTCompress Compress(
```

##### 

```
ct(in);KSK;I;qstg)
```

```
Input:
```

```
8
>>>
<
>>>
:
```

```
!ct(in):a homomorphic integer
KSK:a packing keyswitching key
I=fijgB−j=0^1 :the indices used to pack the messages, with 0 i 0 < i 1 << iB− 1 < N
qstg:the storage modulus
Output: CT(out) 2 GLWES
```

```
P
B− 1
j=0mfjX
ij

R(qkstg+1)
1 CT(pack) PackingKS(!ct(in);I;KSK)
2 CT(Compress) MS
```

```

CT(pack); qstg
```

```

```

```
3 return CT(Compress)
```

```
Algorithm 74: !ct(Decompress) Decompress(CT;BSK;I)
```

```
Input:
```

```
8
><
>:
```

```
CT(in):a compressed GLWE ciphertext
BSK:a decompression (bootstrapping) key
I=fijgB−j=0^1 :the indices of the packed messages, with 0 i 0 < i 1 << iB− 1 < N
Output: !ct(out): a homomorphic integer
1 for 0 jB 1 do
2 ct(ext) SE
```

```

CT(in); ij
```

```

```

```
3 ct(ms) MS
```

```

ct(ext); 2 N
```

```

4 CT(BRot) BlindRotate

ct(ms);BSK

```

```
5 ct(jout) SE
```

```

CT(BRot); 0
```

```

```

```
6 return !ct(out)
```

**Theorem 31 (LWE Ciphertext Compression)** _Let_

##### 

ct _be a homomorphic integer containing_ B
_blocks, with_

##### 

cti=LWEs(mi) _,_ 0 i B  1 _, let_ I=fijgB−j=0^1 _be the set of indices for the packed
messages, with_ 0 i 0 < i 1 << iB− 1 N 1_. Then:_

```
Compress(
```

##### 

```
ct) =GLWES
```

##### 0

##### @

##### B−X 1

```
j=0
```

```
mjXij
```

##### 1

##### A

```
The algorithmic complexity is: CCompress=CPKS+CMS
```

**Theorem 32 (LWE Ciphertext Decompression)** _Let_ CT=GLWES

##### P

```
B− 1
j=0mjX
ij
```

##### 

```
be a com-
```

_pressed ciphertext containing_ B _LWE ciphertexts,_ I =fijgB−j=0^1 _be the set of indices for the packed
messages, with_ 0 i 0 < i 1 << iB− 1 N 1_. Then:_

```
Decompress(CT) =fLWEs(mfj)gB−j=0^1
```

```
The algorithmic complexity is: CDecompress=CSE+CMS+CBR+CSE
```

**Remark 38 (Output Ciphertext Compression)** _TFHE-rs uses the same technique as the inter-
mediate ciphertext compression method for output ciphertext compression._

## 5 Benchmarks and Parameters

## 5 Benchmarks and Parameters

In this section, we will outline the different parameters used inside TFHE-rs and their associated
benchmarks with the various available operations. We are only giving benchmarks assuming the usual
Gaussian distribution to simplify comparison with other work in the literature. Note that in TFHE-rs,
the default distribution is TUniform to better fit some use-cases. TUniform benchmarks can be found
online and we refer tohttps://docs.zama.ai/tfhe-rs/get-started/benchmarksfor these benchmarks. As
a general rule of thumb, switching to a TUniform distribution for the error leads to timings which are
roughly between 5 to 10 percent slower. All benchmarks presented in this section are CPU-based, and
were performed on AWS using ahpc7a.96xlarge^10 machine.

### 5.1 Bootstrapping Related Benchmarks

We begin by looking at benchmarks for thePBSoperation, which is the atomic unit of circuit cost
when evaluating algorithms using the TFHE scheme. In particular, we consider benchmarks of the
_classical bootstrapping_ in Section5.1.1(cf. Algorithm 19 ), as well as the _multi-bit bootstrapping_ in
Section5.1.2(cf. Algorithm 20 ).

#### 5.1.1 Classical Bootstrapping Performance

The first benchmark of interest, presented in Table 2 , is to consider the cost of the programmable
bootstrapping operation for a variety of precisionslog 2 (p)(we consider 2, 4, 6, and 8 in this section).
In particular we measure the cost of the _keyswitch_ (KS), the _programmable bootstrap_ (PBS)as well as
combination of the two(KS-PBS). For theKS-PBSoperation, we consider three separate timings: the
latency of theKS-PBSoperation, an amortized “per-bit” cost of theKS-PBSoperation, and the latency
of theKS-PBSoperation when combined with the drift mitigation techniques outlined in Algorithm 15.
A key observation in Table 2 is that, for most failure probabilities, the amortized cost of theKS-PBS
operation—defined as theKS-PBSlatency divided by the number of bits bootstrapped—is optimal
when choosinglog 2 (p) = 4. This makesp= 2^4 a strong candidate for constructing homomorphic
integers, whose benchmarks are discussed in Section5.2.

(^10) <https://aws.amazon.com/ec2/instance-types/hpc7a/>

5.1 Bootstrapping Related Benchmarks 5 BENCHMARKS AND PARAMETERS

```
pfail Operation
```

```
log 2 (p)
2 4 6 8
```

```
2 −^40 KS 677 s 2.29 ms 20.9 ms 53.3 ms
2 −^40 PBS 5.77 ms 10.7 ms 94.2 ms 471 ms
2 −^40 KS-PBS 7.01 ms 13.6 ms 109 ms 513 ms
2 −^40 KS-PBS(amortized) 7.01 ms 6.80 ms 36.33 ms 128.25 ms
2 −^64 KS 1.28 ms 2.39 ms 15.4 ms 130 ms
2 −^64 PBS 8.33 ms 11.2 ms 99.2 ms 641 ms
2 −^64 KS-PBS 9.74 ms 14.2 ms 116 ms 791 ms
2 −^64 KS-PBS(amortized) 10.2 ms 7.14 ms 38.67 ms 158.5 ms
2 −^128 KS 1.16 ms 5.28 ms 30.3 ms 154 ms
2 −^128 PBS 8.67 ms 25.2 ms 212 ms 1.26 s
2 −^128 KS-PBS 10.4 ms 31.1 ms 238 ms 1.58 s
2 −^128 KS-PBS(amortized) 10.4 ms 12.4 ms 47.33 ms 316.0 ms
2 −^128 KS-PBS(drift) 10.4 ms 14.6 ms 122 ms 1.41s
2 −^128 KS-PBS(drift, amortized) 10.4 ms 7.3 ms 20.3 ms 176.3 ms
```

Table 2:Benchmarks for a variety of atomic patterns for different failure probabilities with parameters
using a Gaussian distribution. The parameter set in each case corresponds toX_X_Yin Table 13 , where
X=log 2 (p)/2is the precision used andY=log 2 (pfail). The timings labelled “KS-PBS(amortized)”
denote the timing of theKS-PBSdivided by the number of bits in the corresponding operation,
i.e., it represents log^1
2 (p)
KS-PBS. “KS-PBS(drift)” denotes the latency of theKS-PBSoperation

when evaluating theKS-PBSoperation in combination with the drift mitigation technique outlined in
Algorithm 15.

#### 5.1.2 MultiBit Bootstrapping Performance

In this section, we focus on the timings of parallelized variant of bootstrapping called the Multi-Bit
PBS, described in Algorithm 20. Unlike in the usual bootstrapping, another parameter called the
_grouping factor_ (gf)has to be taken into account in these benchmarks. In Table 3 , we show the
timings depending on the input plaintext precision given as a bit size(log 2 (p))and the grouping factor
(gf). We note that in Table 3 , forpfail< 2 −^128 , the drift mitigation technique is not used in any of the
benchmarks.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

```
pfail Operation gf= 2 log^2 gf(p=) = 2 3 gf= 4 gf= 2 loggf^2 (p=) = 4 3 gf= 4 gf= 2 loggf^2 (p=) = 6 3 gf= 4 gf= 2 loggf^2 (p=) = 8 3 gf= 4
2 −^40 KS 669 us 1.16 ms 1.2 ms 1.23 ms 1.31 ms 1.34 ms 15.5 ms 14.1 ms 15.1 ms 52.9 ms 51.1 ms 76.6 ms
2 −^40 MB-PBS 5.91 ms 3.27 ms 3.19 ms 8.16 ms 5.05 ms 4.06 ms 51.3 ms 35.2 ms 25.5 ms 363 ms 205 ms 156 ms
2 −^40 KS-MB-PBS 5.74 ms 4.54 ms 5.01 ms 8.55 ms 6.46 ms 5.83 ms 63.7 ms 47.1 ms 54.1 ms 442 ms 270 ms 258 ms
2 −^64 KS 1.07 ms 1.22 ms 1.52 ms 1.8 ms 1.39 ms 1.93 ms 15.7 ms 15.4 ms 15.5 ms 130 ms 125 ms 130 ms
2 −^64 MB-PBS 7.98 ms 4.37 ms 3.23 ms 8.83 ms 6.46 ms 4.1 ms 49.7 ms 41.4 ms 26.8 ms 395 ms 289 ms 216 ms
2 −^64 KS-MB-PBS 8.19 ms 5.17 ms 5.51 ms 10.7 ms 7.37 ms 6.51 ms 66.8 ms 53.8 ms 44.9 ms 533 ms 396 ms 389 ms
2 −^80 KS 1.25 ms 1.14 ms 1.52 ms 4.28 ms 4.24 ms 4.33 ms 29.9 ms 31.0 ms 30.8 ms 191 ms 156 ms 286 ms
2 −^80 MB-PBS 5.96 ms 4.89 ms 3.27 ms 11.7 ms 7.68 ms 5.82 ms 101 ms 68.6 ms 56.0 ms 785 ms 515 ms 412 ms
2 −^80 KS-MB-PBS 7.24 ms 5.36 ms 7.06 ms 16.2 ms 15.0 ms 12.0 ms 125 ms 97.5 ms 87.6 ms 834 ms 616 ms 637 ms
2 −^128 KS 1.21 ms 1.2 ms 1.78 ms 4.06 ms 3.94 ms 4.24 ms 30.6 ms 30.5 ms 31.2 ms 167 ms 148 ms 159 ms
2 −^128 MB-PBS 8.39 ms 5.17 ms 4.53 ms 14.7 ms 11.9 ms 9.26 ms 103 ms 81.2 ms 62.6 ms 845 ms 635 ms 525 ms
2 −^128 KS-MB-PBS 7.26 ms 6.09 ms 6.39 ms 18.9 ms 16.0 ms 13.5 ms 131 ms 104 ms 94.3 ms 1.07 s 859 ms 606 ms
```

Table 3:Performance comparison of different operations grouped by precision across different grouping
factorsgf=2,gf=3, andgf=4 considered in theMB-PBS. Bold values indicate the smallest timing
for each operation perlog 2 (p)column. All parameter sets used are outlined in Table 14

Recall that in the case ofgf> 1 , we see an increase in the bootstrapping key size, as can be seen
in Table 1. Table 3 shows that, as the precision increases, theKS-MB-PBSoperation is consistently
fastest in the case ofgf= 4. However, in this case, the bootstrapping keys are significantly larger (by

a factor of^2

gf
gf) which can be cumbersome to deal with when deploying advanced FHE protocols, par-
ticularly including those utilising threshold decryption, where keysize should be minimized to benefit
the associated MPC protocols.

### 5.2 Integer Benchmarks

In this section, we consider the benchmark of homomorphic integer operations. In particular, the
algorithms presented in Section 3. In Section5.2.1, we discuss how to choose the best carry-message
modulus to build large integers. Next, in Section5.2.2, we consider the benchmark of plaintext-
ciphertext operations (where algorithms have one encrypted input and one plaintext input) as well
as ciphertext-ciphertext operations (where algorithms have only encrypted inputs) in Section5.2.3.
We also look at benchmark timings for the compression and decompression algorithms (Algorithms 73
and 74 respectively). Finally, we present all parameter sets used in Section5.4.

#### 5.2.1 Choosing the Best Precision and Bootstrapping Algorithm

The first step in constructing efficient homomorphic integers (and circuits for evaluating them) is
selecting an appropriate carry-message moduluspfor their representation. Suppose we want to evaluate
operations on encryptedu64values. In that case, we must determine the most efficient way to split
this integer into smaller blocks for evaluation. As an example, we could split the 64-bit values into 32
blocks of 2-bits, or 16 blocks of 4-bits, etc. The message modulusmsgdirectly impacts the required
PBS precision, and therefore the latency of the individual bootstraps. Whilst choosing a lower precision
bootstrap can lead to significantly faster PBS evaluation, it typically also leads to a circuit containing
substantially more PBS operations, and can therefore lead to a negative trade-off.
To illustrate this with an example, Table 4 shows that the number of PBS operations required for
a 64-bit multiplication circuit can range from as low as 341 (when using a message modulusmsg= 2^4
and a carry moduluscarry= 2^4 , i.e., a carry-message modulusp= 2^8 ) to as high as 6068 (when
using a message modulusmsg= 2^1 ). However, the optimal choice is a message modulusmsg= 2^2 , as
indicated by the latency results for multiplication in Table 5.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

```
Parameters
```

```
log 2 (Ω)
8 16 32 64
```

```
1 _ 1 _ 64 92 372 1504 6068
2 _ 2 _ 64 25 116 455 1772
3 _ 3 _ 64 13 59 184 687
4 _ 4 _ 64 6 22 94 341
```

Table 4:The number of programmable bootstrapping operations in alog 2 (Ω)-bit multiplication circuit,
for a variety of PBS precisions. In a parameter set _X_Y_Z_ , _X_ refers to thelog 2 of the carry modulus,
_Y_ refers to thelog 2 of the message modulus and _Z_ refers to thelog 2 of the failure probability.

```
Parameters
```

```
log 2 (Ω)
```

```
4 8 16 32 64 128 256
```

```
Bitwise AND
```

```
1 _ 1 _ 64 14.9 ms 15.3 ms 16.1 ms 17.3 ms 18.8 ms 19.7 ms 34.2 ms
2 _ 2 _ 64 18.9 ms 18.6 ms 19.0 ms 19.4 ms 20.3 ms 22.0 ms 24.6 ms
4 _ 4 _ 64 762 ms 844 ms 920 ms 1.03 s 1.09 s 1.12 s 1.15 s
```

```
Addition
```

```
1 _ 1 _ 64 46.6 ms 124 ms 178 ms 370 ms 898 ms 1.43 s 2.85 s
2 _ 2 _ 64 32.1 ms 52.5 ms 58.1 ms 79.9 ms 101 ms 161 ms 173 ms
4 _ 4 _ 64 798 ms 1.64 s 2.78 s 3.09 s 3.23 s 4.17 s 4.8 s
```

```
Multiplication
```

```
1 _ 1 _ 64 67.2 ms 164 ms 331 ms 653 ms 1.46 s 3.98 s 12.3 s
2 _ 2 _ 64 38.5 ms 94.4 ms 136 ms 210 ms 381 ms 1.1 s 3.65 s
4 _ 4 _ 64 745 ms 1.65 s 3.68 s 5.01 s 7.49 s 17.1 s 40.9 s
```

Table 5:Comparison of Bitwise AND, Addition, and Multiplication timings depending on the input size
chunks and number of chunks. All parameter sets used have a failure probability satisfyingpfail 2 −^64.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

```
Parameters
```

```
log 2 (Ω)
```

```
4 8 16 32 64 128 256
```

```
Bitwise AND
```

```
1 _ 1 _ 128 15.0 ms 15.7 ms 16.6 ms 18.0 ms 19.1 ms 20.9 ms 37.3 ms
2 _ 2 _ 128 22.8 ms 19.7 ms 20.2 ms 20.9 ms 22.3 ms 24.7 ms 27.3 ms
4 _ 4 _ 128 1.47 s 1.63 s 1.73 s 1.78 s 1.89 s 2.04 s 2.08 s
```

```
Addition
```

```
1 _ 1 _ 128 61.6 ms 123 ms 249 ms 517 ms 1.02 s 2.03 s 3.96 s
2 _ 2 _ 128 42.6 ms 62.8 ms 59.2 ms 85.5 ms 105 ms 174 ms 190 ms
4 _ 4 _ 128 1.51 s 3.22 s 5.1 s 5.13 s 5.76 s 8.12 s 8.34 s
```

```
Multiplication
```

```
1 _ 1 _ 128 76.2 ms 176 ms 347 ms 723 ms 1.67 s 4.54 s 13.7 s
2 _ 2 _ 128 41.5 ms 101 ms 146 ms 219 ms 400 ms 1.13 s 3.77 s
4 _ 4 _ 128 1.39 s 3.25 s 6.82 s 9.14 s 19.0 s 52.4 s 184 s
```

Table 6:Comparison of Bitwise AND, Addition, and Multiplication timings depending on the input
size chunks and number of chunks. All parameter sets used have a failure probability satisfying
pfail 2 −^128.

We also observe that addition is most efficient when using a message modulus ofmsg= 2^2 and
a carry modulus ofcarry= 2^2. This is also the best representation for the bitwise AND operator
when handling large precisions (e.g., more than 256 in this case). These findings suggest that a 4-bit
carry-message modulus is a good starting point.
In Table 7 , we compare classical bootstrapping with multi-bit bootstrapping using a grouping factor
of 3, both forp= 2^4. While multi-bitPBSis faster for small precisions, it quickly becomes slower
than the classical approach for larger precisions (i.e., whenlog 2 (Ω) 64 ).
For this reason, the default representation of homomorphic 64-bit integers in TFHE-rs uses a carry
modulus ofcarry= 2^2 and a message modulus ofmsg= 2^2 , relying on classicalPBSrather than later
variants like multi-bitPBS.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

```
Parameters
```

```
log 2 (Ω)
```

```
4 8 16 32 64 128 256
```

```
Bitwise AND
```

```
2 _ 2 _ 64 18.9 ms 18.6 ms 19.0 ms 19.4 ms 20.3 ms 22.0 ms 24.6 ms
2 _ 2 _ 64 _MultBit3 13.8 ms 12.6 ms 13.6 ms 17.3 ms 26.8 ms 45.1 ms 93.6 ms
```

```
Addition
```

```
2 _ 2 _ 64 32.1 ms 52.5 ms 58.1 ms 79.9 ms 101 ms 161 ms 173 ms
2 _ 2 _ 64 _MultBit3 24.1 ms 37.1 ms 40.6 ms 64.8 ms 106 ms 201 ms 395 ms
```

```
Multiplication
```

```
2 _ 2 _ 64 38.5 ms 94.4 ms 136 ms 210 ms 381 ms 1.1 s 3.65 s
2 _ 2 _ 64 _MultBit3 25.3 ms 69.9 ms 133 ms 359 ms 1.15 s 4.14 s 15.8 s
```

Table 7:Comparison of Bitwise AND, Addition, and Multiplication timings depending on the boot-
strapping method and number of chunks. Both parameter sets are chosen so thatpfail 2 −^64.

#### 5.2.2 Plaintext-Ciphertext Operation Benchmarks

Next, we consider benchmarks for the plaintext-ciphertext integer operations. In particular, we
present the latencies for a variety of algorithms presented in Section 3 for precisions log 2 (Ω) 2
f 4 ; 8 ; 16 ; 32 ; 64 ; 128 ; 256 g. Each benchmark is linked to an individual algorithm (or section) in the
text, so that the reader knows which benchmarks correspond to which algorithms. Moreover, the
names of the benchmarks match the name used in the TFHE-rs library directly, so that a user can
re-run each benchmark on their own machine (or different hardware, for comparison) if desired.

```
log 2 (Ω)
Operation Reference 4 8 16 32 64 128 256
unsigned_overflowing_scalar_add_parallelized Section3.14 38.2 ms 53.2 ms 56.3 ms 57.0 ms 78.6 ms 105 ms 177 ms
unsigned_overflowing_scalar_sub_parallelized Section3.14 37.9 ms 53.0 ms 56.9 ms 57.9 ms 79.9 ms 106 ms 178 ms
unsigned_scalar_add_parallelized Section3.6.2 35.9 ms 51.7 ms 56.2 ms 57.0 ms 78.6 ms 104 ms 172 ms
unsigned_scalar_bit{and,or,xor}_parallelized Algorithm 26 18.1 ms 19.0 ms 19.3 ms 19.7 ms 21.3 ms 23.1 ms 25.0 ms
unsigned_scalar_div_parallelized Section3.17.2 78.8 ms 137 ms 189 ms 269 ms 451 ms 833 ms 2.15 s
unsigned_scalar_{eq,ne}_parallelized Section3.4.2 17.0 ms 34.6 ms 35.4 ms 56.0 ms 54.4 ms 75.8 ms 78.1 ms
unsigned_scalar_{gt,ge,lt,le}_parallelized Algorithm 69 14.3 ms 36.7 ms 35.5 ms 54.7 ms 74.0 ms 94.5 ms 138 ms
unsigned_scalar_mul_parallelized Algorithm 60 37.9 ms 75.6 ms 118 ms 161 ms 222 ms 419 ms 1.04 s
unsigned_scalar_rem_parallelized Section3.17.2 146 ms 268 ms 344 ms 492 ms 719 ms 1.27 s 2.86 s
unsigned_scalar_{left,right}_shift_parallelized Algorithms 45 and 48 19.8 ms 18.9 ms 19.3 ms 19.9 ms 20.6 ms 22.1 ms 24.8 ms
unsigned_scalar_rotate_{left,right}_parallelized Algorithms 46 and 49 20.3 ms 19.3 ms 19.4 ms 19.9 ms 21.4 ms 22.7 ms 24.9 ms
unsigned_scalar_sub_parallelized Section3.8.1 35.2 ms 54.3 ms 58.0 ms 58.5 ms 80.2 ms 106 ms 175 ms
```

Table 8: Benchmarks for integer plaintext-ciphertext operations. The parameter set in each case
corresponds to 2 _2_ 64 in Table 13. In particular, this means that the failure probability is 2 −^64.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

```
log 2 (Ω)
Operation Reference 4 8 16 32 64 128 256
unsigned_overflowing_scalar_add_parallelized Section3.14 38.8 ms 55.7 ms 62.9 ms 64.4 ms 89.2 ms 115 ms 189 ms
unsigned_overflowing_scalar_sub_parallelized Section3.14 39.9 ms 56.1 ms 62.1 ms 64.8 ms 89.4 ms 114 ms 187 ms
unsigned_scalar_add_parallelized Section3.6.2 36.5 ms 55.5 ms 60.4 ms 62.7 ms 87.5 ms 112 ms 180 ms
unsigned_scalar_bit{and, or, xor}_parallelized Algorithm 26 16.1 ms 18.3 ms 20.5 ms 21.4 ms 22.9 ms 24.6 ms 27.1 ms
unsigned_scalar_div_parallelized Section3.17.2 76.9 ms 145 ms 197 ms 274 ms 459 ms 830 ms 2.36 s
unsigned_scalar_{eq,ne}_parallelized Section3.4.2 14.9 ms 34.6 ms 34.7 ms 54.9 ms 57.0 ms 80.1 ms 82.3 ms
unsigned_scalar_{ge,gt,le,lt}_parallelized Algorithm 69 14.8 ms 39.6 ms 37.3 ms 55.0 ms 78.0 ms 99.4 ms 145 ms
unsigned_scalar_mul_parallelized Algorithm 60 42.0 ms 77.0 ms 127 ms 175 ms 240 ms 457 ms 1.09 s
unsigned_scalar_rem_parallelized Section3.17.2 151 ms 277 ms 370 ms 527 ms 770 ms 1.44 s 3.17 s
unsigned_scalar_{left,right}_shift_parallelized Algorithms 45 and 48 21.1 ms 20.4 ms 20.4 ms 21.3 ms 23.1 ms 24.8 ms 26.8 ms
unsigned_scalar_rotate_{left,right}_parallelized Algorithms 46 and 49 21.2 ms 19.2 ms 20.6 ms 21.3 ms 22.8 ms 24.6 ms 26.4 ms
unsigned_scalar_sub_parallelized Section3.8.1 35.0 ms 56.1 ms 62.2 ms 63.9 ms 88.4 ms 114 ms 185 ms
```

Table 9: Benchmarks for integer plaintext-ciphertext operations. The parameter set in each case
corresponds to 2 _2_ 128 in Table 13. In particular, this means that the failure probability is 2 −^128.

5.2 Integer Benchmarks 5 BENCHMARKS AND PARAMETERS

#### 5.2.3 Ciphertext-Ciphertext Operation Benchmark

Next, we consider benchmarks for the ciphertext-ciphertext integer operations. In particular, we
present the latencies for a variety of algorithms presented in Section 3 for precisions log 2 (Ω) 2
f 4 ; 8 ; 16 ; 32 ; 64 ; 128 ; 256 g. Each benchmark is linked to an individual algorithm (or section) in the
text, so that the reader knows which benchmarks correspond to which algorithms. Moreover, the
names of the benchmarks match the name used in the TFHE-rs library directly, so that a user can
re-run each benchmark on their own machine (or different hardware, for comparison) if desired.

```
log 2 (Ω)
Operation Reference 4 8 16 32 64 128 256
unsigned_add_parallelized Algorithm 38 32.1 ms 52.5 ms 58.1 ms 79.9 ms 101 ms 161 ms 173 ms
unsigned_bitnot Algorithm 25 3.19μs 7.26μs 14.2μs 26.7μs 52.3μs 100 μs 204 μs
unsigned_bit{and,or,xor} Algorithm 24 18.9 ms 18.9 ms 19.7 ms 20.3 ms 21.8 ms 22.9 ms 25.0 ms
unsigned_div_rem_parallelized Algorithm 71 292 ms 667 ms 1.49 s 3.39 s 7.87 s 19.6 s 52.1 s
unsigned_if_then_else_parallelized Algorithm 63 28.5 ms 28.9 ms 29.4 ms 31.0 ms 31.9 ms 34.3 ms 49.2 ms
unsigned_{left, right}_shift_parallelized Algorithm 30 19.7 ms 59.9 ms 79.2 ms 100 ms 128 ms 169 ms 235 ms
unsigned_{lt, le, gt, ge}_parallelized Algorithm 69 36.6 ms 36.1 ms 55.6 ms 74.5 ms 94.8 ms 136 ms 166 ms
unsigned_mul_parallelized Algorithm 58 38.5 ms 94.4 ms 136 ms 210 ms 381 ms 1.1 s 3.65 s
unsigned_{eq,ne}_parallelized Algorithm 27 36.9 ms 36.3 ms 55.6 ms 55.0 ms 76.3 ms 77.6 ms 99.9 ms
unsigned_neg_parallelized Algorithm 39 32.1 ms 48.7 ms 57.0 ms 78.0 ms 103 ms 162 ms 183 ms
unsigned_overflowing_add_parallelized Section3.14 34.8 ms 56.5 ms 58.7 ms 79.2 ms 101 ms 162 ms 173 ms
unsigned_overflowing_mul_parallelized Section3.14 108 ms 151 ms 181 ms 268 ms 509 ms 1.45 s 4.94 s
unsigned_overflowing_sub_parallelized Section3.14 38.9 ms 56.6 ms 55.3 ms 79.2 ms 100 ms 166 ms 178 ms
unsigned_rotate_{left, right}_parallelized Section3.9 19.2 ms 57.5 ms 77.1 ms 98.9 ms 132 ms 179 ms 245 ms
unsigned_sub_parallelized Algorithm 40 36.0 ms 56.4 ms 58.8 ms 80.3 ms 102 ms 162 ms 174 ms
unsigned_sum_ciphertexts_parallelized_5_ctxts Algorithm 57 37.7 ms 77.2 ms 76.9 ms 97.8 ms 121 ms 188 ms 223 ms
unsigned_sum_ciphertexts_parallelized_10_ctxts Algorithm 57 58.4 ms 95.5 ms 99.8 ms 123 ms 149 ms 237 ms 297 ms
unsigned_sum_ciphertexts_parallelized_20_ctxts Algorithm 57 76.4 ms 117 ms 119 ms 146 ms 193 ms 288 ms 419 ms
```

Table 10: Benchmarks for integer ciphertext-ciphertext operations. The parameter set in each case
corresponds to 2 _2_ 64 in Table 13. In particular, this means that the failure probability is 2 −^64.

```
log 2 (Ω)
Operation Algorithm / Section 4 8 16 32 64 128 256
unsigned_add_parallelized Algorithm 38 42.6 ms 62.8 ms 59.2 ms 85.5 ms 105 ms 174 ms 190 ms
unsigned_bitnot Algorithm 25 3.37μs 5.35μs 10.3μs 19.8μs 39.1μs 83.9μs 184 μs
unsigned_bit{and,or,xor}_parallelized Algorithm 24 22.8 ms 20.9 ms 20.2 ms 21.3 ms 22.7 ms 24.8 ms 27.3 ms
unsigned_div_rem_parallelized Algorithm 71 293 ms 684 ms 1.52 s 3.51 s 8.42 s 21.0 s 54.6 s
unsigned_if_then_else_parallelized Algorithm 63 27.9 ms 30.6 ms 31.4 ms 33.7 ms 34.5 ms 38.0 ms 49.6 ms
unsigned_{left, right}_shift_parallelized Algorithm 30 20.4 ms 59.5 ms 83 ms 109 ms 141 ms 186 ms 255 ms
unsigned_{lt, le, gt, ge}_parallelized Algorithm 69 39.1 ms 36.9 ms 56.3 ms 77.8 ms 101 ms 147 ms 177 ms
unsigned_mul_parallelized Algorithm 58 41.5 ms 101 ms 146 ms 219 ms 400 ms 1.13 s 3.77 s
unsigned_{eq, ne}_parallelized Algorithm 27 35.7 ms 35.9 ms 54.9 ms 57.4 ms 79.1 ms 81.5 ms 105 ms
unsigned_neg_parallelized Algorithm 39 33.4 ms 48.8 ms 62.6 ms 84.6 ms 106 ms 171 ms 189 ms
unsigned_overflowing_add_parallelized Section3.14 40.5 ms 59.2 ms 63.6 ms 86.1 ms 107 ms 175 ms 188 ms
unsigned_overflowing_mul_parallelized Section3.14 111 ms 153 ms 181 ms 274 ms 526 ms 1.51 s 5.13 s
unsigned_overflowing_sub_parallelized Section3.14 39.1 ms 58.3 ms 61.7 ms 84.7 ms 109 ms 172 ms 192 ms
unsigned_rotate_{left, right}_parallelized Section3.9 20.4 ms 61.1 ms 83.0 ms 110 ms 142 ms 191 ms 258 ms
unsigned_sub_parallelized Algorithm 40 39.9 ms 59.5 ms 60.1 ms 83.3 ms 110 ms 176 ms 191 ms
unsigned_sum_ciphertexts_parallelized_5_ctxts Algorithm 57 39.0 ms 73.6 ms 82.2 ms 106 ms 134 ms 199 ms 237 ms
unsigned_sum_ciphertexts_parallelized_10_ctxts Algorithm 57 57.8 ms 98.4 ms 108 ms 130 ms 161 ms 250 ms 319 ms
unsigned_sum_ciphertexts_parallelized_20_ctxts Algorithm 57 83.5 ms 124 ms 130 ms 156 ms 206 ms 314 ms 432 ms
```

Table 11: Benchmarks for integer ciphertext-ciphertext operations. The parameter set in each case
corresponds to 2 _2_ 128 in Table 13. In particular, this means that the failure probability is 2 −^128.

5.3 Compression and Decompression Benchmarks 5 BENCHMARKS AND PARAMETERS

### 5.3 Compression and Decompression Benchmarks

In this section, we outline the benchmarks for the compression (Algorithm 73 ) and decompression
(Algorithm 74 ) algorithms.

```
log 2 (Ω)
Operation Algorithm pfail 8 16 32 64 128 256 512
Compress 73 2 −^64 1.74 ms 1.99 ms 2.42 ms 3.33 ms 4.78 ms 5.89 ms 9.57 ms
Decompress 74 2 −^64 16.9 ms 17.1 ms 17.5 ms 17.9 ms 18.6 ms 22.0 ms 39.6 ms
Compress 73 2 −^128 2.72 ms 2.15 ms 2.59 ms 3.46 ms 4.88 ms 5.89 ms 9.65 ms
Decompress 74 2 −^128 16.7 ms 17.8 ms 18.0 ms 18.4 ms 18.9 ms 22.3 ms 41.9 ms
```

Table 12:Benchmarks of compression and decompression operations applied to homomorphic integers
of various sizes, evaluated for two different failure probabilities.

5.4 Parameters 5 BENCHMARKS AND PARAMETERS

### 5.4 Parameters

In Tables 13 and 14 , we present all parameter sets considered as part of the benchmarks, listed alongside
their respective failure probabilities.
As before, the naming convention of the parameter sets informs the reader about the carry modulus,
message modulus and the failure probability. In particular, a classical parameter setX_X_Yrepresents
homomorphic integers with a carry modulus ofcarry= 2Xand a message modulus ofmsg= 2X, with a
failure probability of 2 −Y. For the Multibit parameter setsX_X_Y_MultBitW,Wcorresponds to the
grouping factor.

```
Parameter Set
```

```
LWE Dimension
```

```
(n
)
```

```
LWE Noise
```

```
(
n)
```

```
GLWE Dimension
```

```
(k
)
```

```
Polynomial Size
```

```
(N
)
```

```
GLWE Noise
```

```
(
N
)
```

```
PBS Base Log
```

```
(
PBS
)
```

```
PBS Level
```

```
(`
PBS
)
```

```
KS Base Log
```

```
(B
KS
)
```

```
KS Level
```

```
(
KS
)
```

```
Total Encryptions of Zero
```

```
(Z
)
```

```
Avg Number of Zeros
```

```
(avg
)
```

```
Probability of failure
```

```
(p
fail
)
```

```
1 _ 1 _ 40 750 1 : 514  10 −^5 3 512 1 : 952  10 −^11 17 1 4 3 - - 2 −^40.^004
2 _ 2 _ 40 796 6 : 846  10 −^6 1 2048 2 : 845  10 −^15 23 1 3 5 - - 2 −^40.^489
3 _ 3 _ 40 925 7 : 393  10 −^7 1 8192 2 : 168  10 −^19 15 2 3 6 - - 2 −^40.^298
4 _ 4 _ 40 1096 3 : 868  10 −^8 1 32768 2 : 168  10 −^19 15 2 4 5 - - 2 −^40.^107
1 _ 1 _ 64 781 8 : 868  10 −^6 4 512 2 : 845  10 −^15 23 1 4 3 - - 2 −^64.^01
2 _ 2 _ 64 833 3 : 616  10 −^6 1 2048 2 : 845  10 −^15 23 1 3 5 - - 2 −^64.^014
3 _ 3 _ 64 977 3 : 014  10 −^7 1 8192 2 : 168  10 −^19 15 2 3 6 - - 2 −^64.^177
4 _ 4 _ 64 1110 3 : 038  10 −^8 1 32768 2 : 168  10 −^19 11 3 2 11 - - 2 −^64.^014
1 _ 1 _ 128 838 3 : 317  10 −^6 4 512 2 : 845  10 −^15 23 1 5 3 1444 17 2 −^128.^979
2 _ 2 _ 128 866 2 : 046  10 −^6 1 2048 2 : 845  10 −^15 23 1 3 5 1446 17 2 −^128.^839
3 _ 3 _ 128 1007 1 : 796  10 −^7 1 8192 2 : 168  10 −^19 15 2 3 7 1455 17 2 −^128.^857
4 _ 4 _ 128 1098 3 : 737  10 −^8 1 65536 2 : 168  10 −^19 11 3 3 7 2961 34 2 −^128.^676
```

```
Table 13:Typical (Gaussian) computation parameter sets used inside TFHE-rs.
```

5.4 Parameters 5 BENCHMARKS AND PARAMETERS

```
Parameter Set
```

```
Grouping factor (gf)LWE Dimension
```

```
(n
)
```

```
LWE Noise
```

```
(
n)
```

```
GLWE Dimension
```

```
(k
)
```

```
Polynomial Size
```

```
(N
)
```

```
GLWE Noise
```

```
(
N
)
```

```
PBS Base Log
```

```
(
PBS
)
```

```
PBS Level
```

```
(`
PBS
)
```

```
KS Base Log
```

```
(B
KS
)
```

```
KS Level
```

```
(
KS
)
```

```
Probability of failure
```

```
(p
fail
)
1 _ 1 _ 64 _MultBit2 2 748 1 : 567  10 −^5 2 1024 2 : 845  10 −^15 22 1 4 3 2 −^66.^162
1 _ 1 _ 64 _MultBit3 3 747 1 : 594  10 −^5 2 1024 2 : 845  10 −^15 22 1 4 3 2 −^64.^618
1 _ 1 _ 64 _MultBit4 4 712 2 : 916  10 −^5 1 2048 2 : 845  10 −^15 22 1 3 4 2 −^65.^871
2 _ 2 _ 64 _MultBit2 2 872 1 : 845  10 −^6 1 2048 2 : 845  10 −^15 22 1 4 4 2 −^64.^148
2 _ 2 _ 64 _MultBit3 3 912 9 : 252  10 −^7 1 2048 2 : 845  10 −^15 22 1 5 3 2 −^64.^095
2 _ 2 _ 64 _MultBit4 4 872 1 : 845  10 −^6 1 2048 2 : 845  10 −^15 22 1 4 4 2 −^64.^122
3 _ 3 _ 64 _MultBit2 2 978 2 : 963  10 −^7 1 8192 2 : 168  10 −^19 14 2 3 6 2 −^64.^212
3 _ 3 _ 64 _MultBit3 3 978 2 : 963  10 −^7 1 8192 2 : 168  10 −^19 14 2 3 6 2 −^64.^232
3 _ 3 _ 64 _MultBit4 4 980 2 : 862  10 −^7 1 8192 2 : 168  10 −^19 14 2 3 6 2 −^64.^633
4 _ 4 _ 64 _MultBit2 2 1122 2 : 470  10 −^8 1 32768 2 : 168  10 −^19 8 4 2 11 2 −^64.^021
4 _ 4 _ 64 _MultBit3 3 1119 2 : 601  10 −^8 1 32768 2 : 168  10 −^19 8 4 2 11 2 −^64.^022
4 _ 4 _ 64 _MultBit4 4 1124 2 : 386  10 −^8 1 32768 2 : 168  10 −^19 8 4 2 11 2 −^64.^007
```

```
Table 14:Typical (Gaussian, Multi-bit) computation parameter sets used inside TFHE-rs.
```

5.4 Parameters 5 BENCHMARKS AND PARAMETERS

In Table 15 we present the parameters used for the compression and decompression algorithm
benchmarks outlined in Table 12 .COMPRESS_Xcorresponds to the parameters used for compression
with a failure probability of 2 −X

```
Parameter Set
```

```
BR Base Log
```

```
(B
br
)
```

```
BR Level
```

```
(`
br
)
```

```
PKS Base Log
```

```
(B
pks
)
```

```
PKS level
```

```
(`
pks
)
```

```
Polynomial Size
```

```
(N
)
```

```
GLWE Noise
```

```
(
N
)
(B
)
```

```
GLWE Dimension
```

```
(k
)
```

```
Number of LWEs
```

```
(
)
```

```
Storage Modulus
```

```
(q
stg
)
```

```
Probability of failure
```

```
(p
fail
)
COMPRESS_ 64 1 23 2 6 256 1 : 340  10 −^15 4 256 12  2 −^64
COMPRESS_ 128 1 23 2 6 256 1 : 340  10 −^15 4 256 12  2 −^128
```

```
Table 15:Typical compression parameters used inside TFHE-rs.
```

For completeness, in Table 16 , we present typical parameters used in TFHE-rs when using a
TUniform noise distribution.

5.4 Parameters 5 BENCHMARKS AND PARAMETERS

```
Parameter Set
```

```
LWE Dimension
```

```
(n
)
```

```
LWE Noise
```

##### (B

```
n)
```

```
GLWE Dimension
```

```
(k
)
```

```
Polynomial Size
```

##### (N

##### )

```
GLWE Noise
```

##### (B

```
N
)
```

```
PBS Base Log
```

##### (

```
PBS
)
```

```
PBS Level
```

##### (`

```
PBS
)
```

```
KS Base Log
```

##### (B

```
KS
)
```

```
KS Level
```

##### (

```
KS
)
```

```
Total Encryptions of Zero
```

##### (Z

##### )

```
Avg Number of Zeros
```

```
(avg
```

```
)
```

```
Probability of failure
```

```
(p
fail
)
```

1 _1_ (^40) T 799 48 3 512 30 17 1 3 4 - - 2 −^40.^525
2 _2_ (^40) T 839 47 1 2048 17 23 1 3 5 - - 2 −^57.^015
3 _3_ (^40) T 958 44 1 8192 3 15 2 3 6 - - 2 −^50.^002
4 _4_ (^40) T 1077 41 1 32768 3 15 2 3 7 - - 2 −^41.^009
1 _1_ (^64) T 839 47 4 512 17 23 1 4 3 - - 2 −^72.^226
2 _2_ (^64) T 879 46 1 2048 17 23 1 3 5 - - 2 −^72.^178
3 _3_ (^64) T 998 43 1 8192 3 15 2 3 6 - - 2 −^64.^454
4 _4_ (^64) T 1117 40 1 32768 3 11 3 1 22 - - 2 −^64.^037
1 _1_ (^128) T 879 46 4 512 17 23 1 5 3 1437 15 2 −^144.^044
2 _2_ (^128) T 918 45 1 2048 17 23 1 4 4 1449 17 2 −^129.^153
3 _3_ (^128) T 1077 41 1 8192 3 15 2 4 5 1459 17 2 −^128.^771
4 _4_ (^128) T 1117 40 1 65536 3 11 3 3 7 2948 31 2 −^141.^493
Table 16:Typical (TUniform) computation parameter sets used inside TFHE-rs.

## 6 Find Out More

## 6 Find Out More

By explaining the algorithms and integer arithmetic of TFHE-rs, this document aims to support
developers building backends for homomorphic evaluations based on the TFHE scheme in general, and
more specifically, the variant implemented in TFHE-rs. However, as advancements in the FHE space
might raise additional questions, more supporting resources can be found through the following links:

- additional information on the TFHE-rs library:<https://docs.zama.ai/tfhe-rs>
- codebase of the TFHE-rs library:<https://github.com/zama-ai/tfhe-rs>
- general information on the TFHE scheme:<https://www.tfhe.com/about>
- support channels for all technical questions: <https://community.zama.ai/andhttps://discord>.
    com/invite/zama
- contact point for collaborations or any other inquiries:hello@zama.ai

##### REFERENCES REFERENCES

## References

[ACPS09] Benny Applebaum, David Cash, Chris Peikert, and Amit Sahai. Fast cryptographic prim-
itives and circular-secure encryption based on hard learning problems. In Shai Halevi,
editor, _Advances in Cryptology – CRYPTO 2009_ , volume 5677 of _Lecture Notes in Com-
puter Science_ , pages 595–618, Santa Barbara, CA, USA, August 16–20, 2009. Springer
Berlin Heidelberg, Germany.

[AP14] Jacob Alperin-Sheriff and Chris Peikert. Faster bootstrapping with polynomial error. In
Juan A. Garay and Rosario Gennaro, editors, _Advances in Cryptology – CRYPTO 2014,
Part I_ , volume 8616 of _Lecture Notes in Computer Science_ , pages 297–314, Santa Barbara,
CA, USA, August 17–21, 2014. Springer Berlin Heidelberg, Germany.

[BBB+23] Loris Bergerat, Anas Boudi, Quentin Bourgerie, Ilaria Chillotti, Damien Ligier, Jean-
Baptiste Orfila, and Samuel Tap. Parameter optimization and larger precision for
(T)FHE. _Journal of Cryptology_ , 36(3):28, July 2023.

[BCK+23] Youngjin Bae, Jung Hee Cheon, Jaehyung Kim, Jai Hyun Park, and Damien Stehlé.
HERMES: Efficient ring packing using MLWE ciphertexts and application to transci-
phering. In Helena Handschuh and Anna Lysyanskaya, editors, _Advances in Cryptology_

_- CRYPTO 2023, Part IV_ , volume 14084 of _Lecture Notes in Computer Science_ , pages
37–69, Santa Barbara, CA, USA, August 20–24, 2023. Springer, Cham, Switzerland.

[BGGJ20] Christina Boura, Nicolas Gama, Mariya Georgieva, and Dimitar Jetchev. CHIMERA:
combining ring-lwe-based fully homomorphic encryption schemes. _J. Math. Cryptol._ ,
14(1), 2020.

[BGV12] Zvika Brakerski, Craig Gentry, and Vinod Vaikuntanathan. (leveled) fully homomorphic
encryption without bootstrapping. In _Innovations in Theoretical Computer Science 2012,
Cambridge, MA, USA, January 8-10, 2012_ , 2012.

[BIP+22a] Charlotte Bonte, Ilia Iliashenko, Jeongeun Park, Hilder V. L. Pereira, and Nigel P. Smart.
Final: Faster fhe instantiated with ntru and lwe. In Shweta Agrawal and Dongdai Lin, ed-
itors, _Advances in Cryptology – ASIACRYPT 2022_ , pages 188–215, Cham, 2022. Springer
Nature Switzerland.

[BIP+22b] Charlotte Bonte, Ilia Iliashenko, Jeongeun Park, Hilder V. L. Pereira, and Nigel P. Smart.
FINAL: Faster FHE instantiated with NTRU and LWE. In Shweta Agrawal and Dongdai
Lin, editors, _Advances in Cryptology – ASIACRYPT 2022, Part II_ , volume 13792 of
_Lecture Notes in Computer Science_ , pages 188–215, Taipei, Taiwan, December 5–9, 2022.
Springer, Cham, Switzerland.

[BJRW23] Katharina Boudgoust, Corentin Jeudy, Adeline Roux-Langlois, and Weiqiang Wen. On
the hardness of module learning with errors with short distributions. _Journal of Cryptol-
ogy_ , 36(1):1, January 2023.

[BJSW24] Olivier Bernard, Marc Joye, Nigel P. Smart, and Michael Walter. Drifting towards better
error probabilities in fully homomorphic encryption schemes. Cryptology ePrint Archive,
Paper 2024/1718, 2024.

[BMMP18] Florian Bourse, Michele Minelli, Matthias Minihold, and Pascal Paillier. Fast homomor-
phic evaluation of deep discretized neural networks. In Hovav Shacham and Alexandra
Boldyreva, editors, _Advances in Cryptology – CRYPTO 2018, Part III_ , volume 10993 of

##### REFERENCES REFERENCES

```
Lecture Notes in Computer Science , pages 483–512, Santa Barbara, CA, USA, August 19–
23, 2018. Springer, Cham, Switzerland.
```

[BST20] Florian Bourse, Olivier Sanders, and Jacques Traoré. Improved secure integer comparison
via homomorphic encryption. In _CT-RSA_. Springer, 2020.

[CDKS21] Hao Chen, Wei Dai, Miran Kim, and Yongsoo Song. Efficient homomorphic conversion
between (ring) LWE ciphertexts. In Kazue Sako and Nils Ole Tippenhauer, editors, _ACNS
21: 19th International Conference on Applied Cryptography and Network Security, Part I_ ,
volume 12726 of _Lecture Notes in Computer Science_ , pages 460–479, Kamakura, Japan,
June 21–24, 2021. Springer, Cham, Switzerland.

[CGGI16a] Ilaria Chillotti, Nicolas Gama, Mariya Georgieva, and Malika Izabachène. Faster fully
homomorphic encryption: Bootstrapping in less than 0.1 seconds. In Jung Hee Cheon and
Tsuyoshi Takagi, editors, _Advances in Cryptology – ASIACRYPT 2016, Part I_ , volume
10031 of _Lecture Notes in Computer Science_ , pages 3–33, Hanoi, Vietnam, December 4–8,

2016. Springer Berlin Heidelberg, Germany.

[CGGI16b] Ilaria Chillotti, Nicolas Gama, Mariya Georgieva, and Malika Izabachène. Faster fully ho-
momorphic encryption: Bootstrapping in less than 0.1 seconds. In _Advances in Cryptology_

_- ASIACRYPT 2016_ , 2016.

[CGGI16c] Ilaria Chillotti, Nicolas Gama, Mariya Georgieva, and Malika Izabachène. TFHE: Fast
fully homomorphic encryption library.<https://tfhe.github.io/tfhe/>, August 2016.

[CGGI17] Ilaria Chillotti, Nicolas Gama, Mariya Georgieva, and Malika Izabachène. Faster packed
homomorphic operations and efficient circuit bootstrapping for TFHE. In _Advances in
Cryptology - ASIACRYPT 2017_ , 2017.

[CGGI20] Ilaria Chillotti, Nicolas Gama, Mariya Georgieva, and Malika Izabachène. TFHE: Fast
fully homomorphic encryption over the torus. _Journal of Cryptology_ , 33(1):34–91, 2020.
Earlier versions in ASIACRYPT 2016 and 2017.

[CJL+20] Ilaria Chillotti, Marc Joye, Damien Ligier, Jean-Baptiste Orfila, and Samuel Tap. Con-
crete: Concrete operates on ciphertexts rapidly by extending TfhE. In _WAHC 2020_ ,
2020.

[CJP21] Ilaria Chillotti, Marc Joye, and Pascal Paillier. Programmable bootstrapping enables
efficient homomorphic inference of deep neural networks. In _Cyber Security Cryptography
and Machine Learning - 5th International Symposium, CSCML 2021_ , volume 12716 of
_Lecture Notes in Computer Science_. Springer, 2021.

[CLOT21] Ilaria Chillotti, Damien Ligier, Jean-Baptiste Orfila, and Samuel Tap. Improved pro-
grammable bootstrapping with larger precision and efficient arithmetic circuits for tfhe.
In _ASIACRYPT 2021_. Springer, 2021.

[CZB+22] Pierre-Emmanuel Clet, Martin Zuber, Aymen Boudguiga, Renaud Sirdey, and Cédric
Gouy-Pailler. Putting up the swiss army knife of homomorphic calculations by means
of tfhe functional bootstrapping. Cryptology ePrint Archive, Report 2022/149, 2022.
<https://ia.cr/2022/149>.

[DM15] Léo Ducas and Daniele Micciancio. FHEW: Bootstrapping homomorphic encryption in
less than a second. In _Advances in Cryptology – EUROCRYPT 2015, Part I_ , volume 9056
of _Lecture Notes in Computer Science_ , pages 617–640. Springer, 2015.

##### REFERENCES REFERENCES

[GBA21] Antonio Guimarães, Edson Borin, and Diego F. Aranha. Revisiting the functional boot-
strap in TFHE. _IACR Trans. Cryptogr. Hardw. Embed. Syst._ , 2021(2), 2021.

[Gen09] Craig Gentry. Fully homomorphic encryption using ideal lattices. In Michael Mitzen-
macher, editor, _41st Annual ACM Symposium on Theory of Computing_ , pages 169–178,
Bethesda, MD, USA, May 31 – June 2, 2009. ACM Press.

[GINX16] Nicolas Gama, Malika Izabachène, Phong Q. Nguyen, and Xiang Xie. Structural lattice
reduction: Generalized worst-case to average-case reductions and homomorphic cryp-
tosystems. In Marc Fischlin and Jean-Sébastien Coron, editors, _Advances in Cryptology –
EUROCRYPT 2016, Part II_ , volume 9666 of _Lecture Notes in Computer Science_ , pages
528–558, Vienna, Austria, May 8–12, 2016. Springer Berlin Heidelberg, Germany.

[GM94] Torbjörn Granlund and Peter L. Montgomery. Division by invariant integers using mul-
tiplication. In _Proceedings of the ACM SIGPLAN 1994 Conference on Programming
Language Design and Implementation_ , PLDI ’94, page 6172, New York, NY, USA, 1994.
Association for Computing Machinery.

[GSW13] Craig Gentry, Amit Sahai, and Brent Waters. Homomorphic encryption from learning
with errors: Conceptually-simpler, asymptotically-faster, attribute-based. In _Advances in
Cryptology – CRYPTO 2013, Part I_ , volume 8042 of _Lecture Notes in Computer Science_ ,
pages 75–92. Springer, 2013.

[HSJ86] W Daniel Hillis and Guy L Steele Jr. Data parallel algorithms. _Communications of the
ACM_ , 29(12):1170–1183, 1986.

[Joy21] Marc Joye. Balanced non-adjacent forms. In Mehdi Tibouchi and Huaxiong Wang,
editors, _Advances in Cryptology - ASIACRYPT 2021 - 27th International Conference on
the Theory and Application of Cryptology and Information Security, Singapore, December
6-10, 2021, Proceedings, Part III_ , volume 13092 of _Lecture Notes in Computer Science_ ,
pages 553–576. Springer, 2021.

[Joy24] Marc Joye. TFHE public-key encryption revisited. In Elisabeth Oswald, editor, _Topics in
Cryptology – CT-RSA 2024_ , volume 14643 of _Lecture Notes in Computer Science_ , pages
277–291, San Francisco, CA, USA, May 6–9, 2024. Springer, Cham, Switzerland.

[JP22] Marc Joye and Pascal Paillier. Blind rotation inăfully homomorphic encryption
withăextended keys. In Shlomi Dolev, Jonathan Katz, and Amnon Meisels, editors,
_Cyber Security, Cryptology, and Machine Learning_ , pages 1–18, Cham, 2022. Springer
International Publishing.

[KÖ22] Jakub Klemsa and Melek Önen. Parallel operations over TFHE-encrypted multi-digit
integers. In _Proceedings of the Twelfth ACM Conference on Data and Application Security
and Privacy_ , pages 288–299, 2022.

[LATV12] Adriana López-Alt, Eran Tromer, and Vinod Vaikuntanathan. On-the-fly multiparty
computation on the cloud via multikey fully homomorphic encryption. _Proceedings of the
Annual ACM Symposium on Theory of Computing_ , 05 2012.

[Lee24] Yongwoo Lee. Lmkcdey revisited: Speeding up blind rotation with signed evaluation keys.
_Mathematics_ , 12(18), 2024.

[Lib24] Benoît Libert. Vector commitments withăproofs ofăsmallness: Short range proofs
andămore. In Qiang Tang and Vanessa Teague, editors, _Public-Key Cryptography – PKC
2024_ , pages 36–67, Cham, 2024. Springer Nature Switzerland.

##### REFERENCES REFERENCES

[LMK+23] Yongwoo Lee, Daniele Micciancio, Andrey Kim, Rakyong Choi, Maxim Deryabin, Jieun
Eom, and Donghoon Yoo. Efficient FHEW bootstrapping with small evaluation keys, and
applications to threshold homomorphic encryption. In Carmit Hazay and Martijn Stam,
editors, _Advances in Cryptology – EUROCRYPT 2023, Part III_ , volume 14006 of _Lecture
Notes in Computer Science_ , pages 227–256, Lyon, France, April 23–27, 2023. Springer,
Cham, Switzerland.

[LMP21] Zeyu Liu, Daniele Micciancio, and Yuriy Polyakov. Large-precision homomorphic sign
evaluation using fhew/tfhe bootstrapping. Cryptology ePrint Archive, Report 2021/1337,
2021.<https://ia.cr/2021/1337>.

[LMSS23] Changmin Lee, Seonhong Min, Jinyeong Seo, and Yongsoo Song. Faster TFHE boot-
strapping with block binary keys. In Joseph K. Liu, Yang Xiang, Surya Nepal, and
Gene Tsudik, editors, _ASIACCS 23: 18th ACM Symposium on Information, Computer
and Communications Security_ , pages 2–13, Melbourne, VIC, Australia, July 10–14, 2023.
ACM Press.

[LPR10] Vadim Lyubashevsky, Chris Peikert, and Oded Regev. On ideal lattices and learning with
errors over rings. In Henri Gilbert, editor, _Advances in Cryptology - EUROCRYPT 2010_ ,
volume 6110 of _Lecture Notes in Computer Science_. Springer, 2010.

[LS15] Adeline Langlois and Damien Stehlé. Worst-case to average-case reductions for module
lattices. _Designs, Codes and Cryptography_ , 75(3):565–599, 2015.

[MM11] Daniele Micciancio and Petros Mol. Pseudorandom knapsacks and the sample complexity
of LWE search-to-decision reductions. In Phillip Rogaway, editor, _Advances in Cryptology_

_- CRYPTO 2011_ , volume 6841 of _Lecture Notes in Computer Science_ , pages 465–484,
Santa Barbara, CA, USA, August 14–18, 2011. Springer Berlin Heidelberg, Germany.

[MP12] Daniele Micciancio and Chris Peikert. Trapdoors for lattices: Simpler, tighter, faster,
smaller. In David Pointcheval and Thomas Johansson, editors, _Advances in Cryptology –
EUROCRYPT 2012_ , volume 7237 of _Lecture Notes in Computer Science_ , pages 700–718,
Cambridge, UK, April 15–19, 2012. Springer Berlin Heidelberg, Germany.

[Reg05] Oded Regev. On lattices, learning with errors, random linear codes, and cryptography.
In Harold N. Gabow and Ronald Fagin, editors, _Proceedings of the 37th Annual ACM
Symposium on Theory of Computing, 2005_. ACM, 2005.

[Reg09] Oded Regev. On lattices, learning with errors, random linear codes, and cryptography.
_Journal of the ACM_ , 56(6):34:1–34:40, 2009. Earlier version in STOC 2005.

[SSTX09] Damien Stehlé, Ron Steinfeld, Keisuke Tanaka, and Keita Xagawa. Efficient public key
encryption based on ideal lattices. In _Advances in Cryptology – ASIACRYPT 2009_ ,
volume 5912 of _Lecture Notes in Computer Science_ , pages 617–635. Springer, 2009.

[vDGHV10]Marten van Dijk, Craig Gentry, Shai Halevi, and Vinod Vaikuntanathan. Fully homo-
morphic encryption over the integers. In Henri Gilbert, editor, _Advances in Cryptology –
EUROCRYPT 2010_ , pages 24–43, Berlin, Heidelberg, 2010. Springer Berlin Heidelberg.

[WWL+24] Ruida Wang, Yundi Wen, Zhihao Li, Xianhui Lu, Benqiang Wei, Kun Liu, and Kunpeng
Wang. Circuit bootstrapping: Faster and smaller. In Marc Joye and Gregor Leander,
editors, _Advances in Cryptology – EUROCRYPT 2024, Part II_ , volume 14652 of _Lec-
ture Notes in Computer Science_ , pages 342–372, Zurich, Switzerland, May 26–30, 2024.
Springer, Cham, Switzerland.

##### REFERENCES REFERENCES

[XZD+23] Binwu Xiang, Jiang Zhang, Yi Deng, Yiran Dai, and Dengguo Feng. Fast blind rotation
for bootstrapping FHEs. In Helena Handschuh and Anna Lysyanskaya, editors, _Advances
in Cryptology – CRYPTO 2023, Part IV_ , volume 14084 of _Lecture Notes in Computer
Science_ , pages 3–36, Santa Barbara, CA, USA, August 20–24, 2023. Springer, Cham,
Switzerland.

[ZYL+18] Tanping Zhou, Xiaoyuan Yang, Longfei Liu, Wei Zhang, and Ningbo Li. Faster boot-
strapping with multiple addends. _IEEE Access_ , 6:49868–49876, 2018.
