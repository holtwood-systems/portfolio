# 跨仓张量与运行时契约（live 版）

> 本文是组织级语义契约的 **live 版本**，派生自只读审计快照
> [`docs/organization-audit/2026-08-13/03-cross-repo-contracts.md`](organization-audit/2026-08-13/03-cross-repo-contracts.md)。
> 快照记录当时事实，不改写；契约演进在本文进行。
> 与审计快照的差异：§10 Backend 边界已按实际决策落地为 tiny-llm ⇄ paged-serving
> 的 C ABI 双源契约（见 §10.1）。

## 1. 目的

本文定义五个仓库讨论同一推理概念时必须共享的语义。它不是要求所有实现共享一种物理布局，而是要求每个边界都明确声明布局，必要的转换可定位、可测试。

## 2. 维度命名

统一使用：

| 符号 | 含义 |
|---|---|
| `B` | batch size |
| `S` | 当前输入 token 数或 query length |
| `T` | cache 中可见的总 key/value length |
| `Hq` | query heads |
| `Hkv` | key/value heads |
| `D` | head dimension |
| `C` | hidden dimension，通常 `Hq * D` |
| `L` | Transformer layer 数 |
| `V` | vocabulary size |

要求 `Hq % Hkv == 0`。GQA group size 为 `G = Hq / Hkv`。

## 3. 逻辑布局

### 3.1 模型投影边界

Transformer GEMM 的推荐逻辑输出：

- Q：`[B, S, Hq, D]`
- K：`[B, S, Hkv, D]`
- V：`[B, S, Hkv, D]`

允许物理存储为扁平 `[B*S, H*D]`，但进入 attention 前必须显式传递 strides，或转换成 attention kernel 声明的布局。禁止仅靠注释假定另一种 reshape。

### 3.2 Prefill attention 边界

跨仓文档和 fixture 的规范布局选为 head-major：

- Q：`[B, Hq, S, D]`
- K/V：`[B, Hkv, T, D]`
- O：`[B, Hq, S, D]`

在无历史 prefix 的普通 prefill 中 `T == S`。若实现内部使用 token-major，应提供显式 adapter 并单测 adapter。

### 3.3 Decode attention 边界

- Q：`[B, Hq, 1, D]`
- K/V cache：`[B, L, Hkv, Tcapacity, D]` 的逻辑视图
- O：`[B, Hq, 1, D]`

具体 cache 可以 paged/block 化，但逻辑索引必须完整包含：

```text
(sequence, layer, kv_head, position, dim)
```

缺少任一维都会导致请求、层、头或位置之间相互覆盖。

## 4. GQA 契约

连续分组时，query head 到 KV head 的映射为：

```text
group_size = Hq / Hkv
kv_head = query_head / group_size
```

等价公式可以不同，但 fixture 必须覆盖非 MHA 情况，例如 `Hq=14, Hkv=2`。只测试 `Hq == Hkv` 不能声称支持 GQA。

## 5. RoPE 契约

组织内选用与 Llama/Qwen 常见 `rotate_half` 一致的 half-split 语义：

```text
x = [x_first, x_second]
rotate_half(x) = [-x_second, x_first]
y = x * cos + rotate_half(x) * sin
```

### 5.1 Cache 形状

为减少歧义，规范 fixture 的 `cos/sin` 使用完整 `D`：

```text
freqs = outer(position, inv_freq)        # [S, D/2]
emb   = concat(freqs, freqs, dim=-1)     # [S, D]
cos   = cos(emb)
sin   = sin(emb)
```

禁止在 half-split kernel 中使用 `repeat_interleave(freqs, 2)`；后者对应 interleaved pair 排列。

实现可以只存 `[S,D/2]`，但 API 名称或类型必须明确为 half cache，不能与完整 `[S,D]` 契约混用。

### 5.2 应用位置

- RMSNorm 后、attention score 前。
- 对 Q 的 `Hq` 个 head 应用。
- 对 K 的 `Hkv` 个 head 应用。
- V 不应用 RoPE。
- prefill 的 position 为序列绝对位置；decode 使用 cache 尾部的绝对 position。

## 6. KV Cache 可见性

每个 decode step 的推荐事务语义：

1. 对所有 layer 计算当前 token 的 K/V。
2. 每层 K/V 写入该层 cache 的 `position=current_length`。
3. 当前层 attention 可读取 `0..=current_length`。
4. 所有 layer 成功后，sequence length 只增加一次。
5. 任一层失败时，不得把部分提交的长度暴露给下一次调度。

如果实现选择先提交再计算，也必须通过事务/回滚或状态标记保证失败一致性。

Paged cache 的 block table 只负责 `logical_position -> physical_block+offset`，layer 不应被折叠进同一个无 layer key 的内容表。

## 7. 数值契约

### 7.1 累加精度

- FP16/BF16 attention score、softmax statistics、RMS variance 默认 FP32 累加。
- 输出可转换回输入 dtype。
- tolerance 必须按 dtype、sequence length 和算子定义，不使用全组织单一阈值。

### 7.2 Softmax

必须使用稳定形式：减去 row max 后指数化。因果 mask 的不可见位置逻辑上为 `-inf`，全 mask row 的行为必须有定义。

### 7.3 量化

每个量化 tensor 必须附带：

- 原始逻辑 shape。
- 磁盘 block format。
- 反量化公式和 block size。
- runtime 重量化 group size。
- 是否发生转置，以及转置前后 shape。

## 8. Sampling 契约

采样处理顺序必须固定并测试：

1. repetition penalty（如果支持）。
2. temperature；`temperature=0` 明确定义为 greedy。
3. top-k。
4. top-p。
5. RNG sample。

如果某个参数没有实现，公共 API 必须拒绝非默认值或明确忽略策略；不能验证参数后静默执行 greedy。

## 9. 流式文本契约

tokenizer decode 不是一般意义上的逐 token 纯函数。SSE/streaming 必须维护每个请求的 decoder 状态：

- 处理 byte fallback 和不完整 UTF-8。
- 处理跨 token merge/normalization。
- 最终拼接的所有 chunk 必须与一次性 decode 完全相同。

验收性质：

```text
concat(stream_decode(tokens)) == decode(tokens)
```

至少覆盖 ASCII、中文、emoji、byte fallback 和特殊 token。

## 10. Backend 边界

Serving 与 runtime 的窄接口应围绕批次和状态，而非名为 GPU 的具体实现：

```text
create_model(model_config, weights)
create_sequence(request_id, capacity)
prefill(batch_descriptors) -> logits/status
decode(batch_descriptors) -> logits/status
cancel(sequence)
metrics()
```

### 10.1 已落地决策：C ABI 双源契约（ABI v3）

tiny-llm（数据面）与 paged-serving（控制面）采用**同进程 C ABI 静态链接**：

- 契约双源：[`tiny-llm/include/tiny_llm/ffi.h`](https://github.com/holtwood-systems/tiny-llm/blob/master/include/tiny_llm/ffi.h)
  ⇄ [`paged-serving/src/tiny_llm_ffi.rs`](https://github.com/holtwood-systems/paged-serving/blob/master/src/tiny_llm_ffi.rs)。
- `TinyLlmConfig` 为 9 个 int 的 repr(C) 布局；paged-serving 侧有布局守卫测试
  （`size_of == 9*4`）锁定一致性。
- 分页 KV 策略 1（block_tables + scatter/gather）为默认路径；
  `max_num_blocks == 0` 表示策略 2。
- KV 生命周期语义：后端管理序列分配/释放（`tinyllm_allocate_sequence` /
  `tinyllm_free_sequence`），调度侧驱动。**策略 1 下 `num_tokens` 仅登记序列、
  不参与容量预留**——分页容量权威是共享 block pool + `max_seq_len`；
  策略 2（连续 KV）下 `num_tokens` 才是真实的每序列预留容量。
- `tinyllm_step` 的 `next_tokens` 至少容纳 `num_sequences` 个 `int`。当
  `logprobs_k == 0` 时 `logprobs` 可为空；当 `logprobs_k > 0` 时调用方必须提供
  至少 `num_sequences * logprobs_k * 2` 个 `float`。第 `s` 个序列、第 `j` 个候选
  从 `((s * logprobs_k + j) * 2)` 开始，依次存储以 `float` 表示的 `token_id` 与
  `logprob`。`logprobs_k < 0`、超过词表大小或请求输出但缓冲区为空均为参数错误。
- **逐序列状态（ABI v3）**：`tinyllm_step` 新增必填出参
  `seq_status[num_sequences]`。返回值 `TLLM_OK` 仅表示调用级执行完成；
  `seq_status[i] == TLLM_SEQ_OK` 时 `next_tokens[i]` 有效，`TLLM_SEQ_ERR`
  表示该序列失败且**不株连** batch 内其他序列。`TLLM_ERR` 保留给调用级
  违约（空指针、logprobs_k 越界、内部缓冲分配失败等）。序列级失败后该
  序列的 KV/position 不推进（序列级原子性），可安全重试或释放。
- **已声明但未生效的参数**：`positions` 在两种策略下均被后端忽略——
  序列位置由后端 `SeqState` 内部跟踪。chunked prefill / prefix caching
  等需要调用方控制位置的功能必须走下一次 ABI breaking change。
- 策略 1 的 decode 路径默认经 **CUDA Graph** 重放（kernel 地址固化、
  token/位置/可见长度经固定 device buffer 每步 H2D 更新；
  `TLLM_FFI_CUDA_GRAPHS=0` 显式关闭）。这消除的是 launch 开销，
  **不等于 Transformer 层的计算批量化**——各序列仍逐序列执行 layer
  forward，batch 只作用于末端 norm + LM head + argmax。
- 差分验证：`tiny-llm/tests/test_ffi.cpp`（策略 1 vs 策略 2 逐 token 差分）、
  `tiny-llm/tests/test_ffi_paged_dispatch.cpp`（含逐序列失败隔离用例）、
  `paged-serving/tests/tiny_llm_backend.rs`、`paged-serving/tests/tiny_llm_text_e2e.rs`。

#### 控制面 ↔ 数据面对账不变量

- 调度器 `max_batch_size` 不得超出后端 `ExecutorCapabilities::max_batch_seqs`；
  引擎在构造时调用 `Scheduler::reconcile_backend_batch_cap` 收敛。
- `ExecutionOutput` 允许为调度集合的真子集：缺失的 `seq_id` 由引擎判定为
  逐序列失败（`fail_sequences` 只作用于失败子集），成功序列照常推进。
- 后端能力声明（`ExecutorCapabilities`）：`sampling`、`logprobs`、
  `retry_safe`、`max_batch_seqs`。请求携带后端不支持的能力时（如
  `params.logprobs.is_some()` 而后端 `logprobs == false`），引擎在 submit
  阶段显式拒绝，不允许静默降级。

## 11. 契约变更规则

- 更改布局或 RoPE convention 属于 breaking change。
- 必须同时更新 fixture schema、reference generator 和所有消费者。
- PR 描述必须给出变更前后 shape/stride 示例。
- 不能只更新注释而保留不同的索引实现。
- `ffi.h` / `tiny_llm_ffi.rs` 的 ABI 或结构体布局变化同为 breaking change：
  先改本文档与双源代码，同批提交，两仓 CHANGELOG 各记一条。
