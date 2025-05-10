# OSPU 开发路线图与状态追踪

## 阶段一: PoC 核心功能 (MVP)

- [-] **任务 1**: 在 `src/registers.rs` 中定义所有加密寄存器结构。
  - [x] **子任务 1.1**: 为所有寄存器统一定义类型别名 `type OSPUReg = tfhe::integer::RadixCiphertext;`。
  - [x] **子任务 1.2**: 明确所有 `RadixCiphertext` 实例将使用 `PARAM_MESSAGE_2_CARRY_2_KS_PBS` 参数集。**理由**: 根据 `tfhe-rs-handbook` 的基准测试，此参数集在整数运算的性能和功能（如并行进位传播）之间提供了最佳平衡。
  - [-] **子任务 1.3**: 定义一个结构体 `OSPURegisters` 来统一管理所有的寄存器实例。

- [ ] **任务 2**: 在 `src/isa/` 目录中，为每条基础指令（Base ISA）创建一个独立的模块文件。
  - [ ] **子任务 2.1 (ADD/SUB)**: 实现 `add.rs` 和 `sub.rs`。操作直接调用 `server_key.add()` 和 `server_key.sub()`。这些函数内部利用了高效的并行进位传播算法。
  - [ ] **子任务 2.2 (MUL)**: 实现 `mul.rs`。采用 Schoolbook 乘法算法。这将涉及一个嵌套循环，对操作数的每个块（digit）执行 `B*B` 次 `bivariate_pbs` 调用，然后高效地对部分积求和。
  - [ ] **子任务 2.3 (CMP/MUX)**: 实现 `cmp.rs` 和 `mux.rs`。操作直接调用 `server_key.eq()` 和 `server_key.if_then_else()`，它们是基于 PBS 的原子操作。
  - [ ] **子任务 2.4 (LOAD/OUT/INC_CTR)**: 实现 `load.rs`, `out.rs`, `inc_ctr.rs`。这些是线性操作，不涉及 PBS，噪声增长可控。

- [ ] **任务 3**: 实现特殊指令 `AnchorMatch` 和 `KeySwitch`。
  - [ ] **子任务 3.1 (AnchorMatch)**: 在 `src/isa/anchor_match.rs` 中实现。
    - [ ] **3.1.1**: 将 `AnchorMatch` 分解为两个并行的同态比较：`CMP(EXT_MR_reg, exp_mr)` 和 `CMP(INT_CTR_reg, exp_ctr)`。
    - [ ] **3.1.2**: 对两个比较操作返回的加密布尔结果，执行一次同态 `AND`（通过一次二元 PBS 实现）。
    - [ ] **3.1.3**: **[性能关键]** 两个并行的 `CMP` 操作必须使用 `rayon::join` 来执行，以最大化利用 CPU 核心，显著降低延迟。
  - [ ] **子任务 3.2 (KeySwitch)**: 在 `src/isa/key_switch.rs` 中实现。
    - [ ] **3.2.1**: PoC 阶段的 `KeySwitch` 实现为一个简单的 `LOAD` 操作，将新的 ClientKey 明文加载到 `NCK_reg`。实际的密钥切换逻辑由 Host 在 OSPU 外部通过状态重加密完成。

- [ ] **任务 4**: 编写完整的单元测试 (`tests/isa_tests.rs`) 和集成测试 (`tests/e2e_flow.rs`)。

- [ ] **任务 5**: 在 `src/main.rs` 中构建一个模拟 Host 与 OSPU 交互的命令行程序。

- [ ] **任务 6**: 最终分析与项目总结报告。

## 阶段二: 完整系统演进 (Future Work)

- [ ] **MPC 集成**: 研究并集成一个 Rust MPC 框架（如 `mpc-net`），以替代 PoC 中的明文密钥注入，实现分布式的 `KeySwitch`。
- [ ] **ZKP 状态验证**: 探索使用 Rust ZKP 库（如 `arkworks`）为 `AnchorMatch` 提供零知识证明，允许 Host 在不访问任何秘密的情况下验证状态转换的正确性。
- [ ] **硬件加速**: 评估并集成 `tfhe-rs-cuda` 后端，以利用 GPU 加速 FHE 计算。
- [ ] **物理安全集成**: 实现 DBRW 算法，将 OSPU 的加密状态与硬件指纹绑定。

## 关键发现与决策记录

- **决策 1 (核心数据结构)**: OSPU 的所有加密寄存器将基于 `tfhe::integer::RadixCiphertext` 实现，并统一采用 `2-bit message + 2-bit carry` 的块编码方案。此决策基于 `tfhe-rs-handbook` 的详尽性能分析，旨在最大化算术运算的效率。
- **决策 2 (性能优化策略)**: 所有数据并行的 FHE 操作（尤其是在 `AnchorMatch` 和 `MUL` 中）必须使用 `rayon` 库进行并行化，这是确保 OSPU 性能达到可接受水平的关键。
- **发现 1 (物理安全路径)**: `dbrw-pdf.md` 中描述的 DBRW (Dual-Binding Random Walk) 算法为 OSPU 实现物理抗克隆提供了具体的技术路线。它通过绑定硬件熵和环境指纹来防止状态机被复制。
- **发现 2 (架构理论基础)**: `dsm.md` 中描述的去中心化状态机 (DSM) 框架是 OSPU 设计哲学的理论基石。其"双边状态隔离"和"自验证状态演进"的核心思想，使 OSPU 能够摆脱对全局共识的依赖。
- **发现 3 (高级功能启发)**: DSM 框架中的"确定性 Limbo 保险库 (DLV)"概念——即延迟密钥派生——为未来设计完全在 FHE 加密域内执行的、有条件的、安全 `KeySwitch` 操作提供了重要启发。可以设想一个 FHE 电路，只有在 `AnchorMatch` 验证成功后，新密钥的解密密钥才能在 FHE 电路内部被计算出来。
