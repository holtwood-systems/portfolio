# 作品集证据索引

本页只负责跨仓导航。原始数据、复现命令和限制由产生证据的技术仓维护，
不在 meta 仓复制第二份结果。

新正式结果包的最小 provenance、artifact hash、状态晋级和失效规则见
[`evidence-artifact-lifecycle.md`](evidence-artifact-lifecycle.md)；
机器可读约束见 [`evidence-manifest.schema.json`](evidence-manifest.schema.json)。

作品集的跨仓审计、项目分层与后续 P0/P1/P2 开发验收见
[`portfolio-audit-and-development-roadmap.md`](portfolio-audit-and-development-roadmap.md)。
将证据安全地转化为简历声明、现场演示和逐仓面试答辩的门槛见
[`portfolio-graduation-and-interview-proof.md`](portfolio-graduation-and-interview-proof.md)。

## 证据层级

1. **真实实验结果**：绑定 commit、模型/输入、硬件与软件环境，保留原始输出和汇总。
2. **真实硬件正确性**：在记录的 GPU 环境上通过差分、属性或端到端测试，
   但不自动构成性能结论。
3. **CPU/CI 回归**：证明接口、调度、参考模型或文档未回退，不替代 CUDA 运行时验证。
4. **脚手架或报告模板**：只表示实验已可准备；在产生绑定环境的结果前，不得称为已完成实验。

## 当前可导航证据

| 仓库 | 可审计入口 | 能证明什么 | 不能证明什么 |
|------|--------------|--------------|----------------|
| [tiny-llm](https://github.com/holtwood-systems/tiny-llm) | [CUDA Graphs A/B 报告](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-08-23-cuda-graphs-ab.md) · [summary JSON](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/data/2026-08-23-cuda-graphs-ab.summary.json) · [raw JSONL](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/data/2026-08-23-cuda-graphs-ab.raw.jsonl) | 报告所述硬件、模型、shape 和 decode 设置下的配对 A/B | 其他 GPU、模型、长上下文或生产 Serving 性能 |
| tiny-llm paged attention | [9/14 三路 DPA](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-09-14-rtx5070ti-dpa.md) · [9/15 split-KV](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-09-15-rtx5070ti-splitkv.md) · [raw 重算工具](https://github.com/holtwood-systems/tiny-llm/blob/b9cbdf922dcfaf0dfc8c704bd285b4b04e757f51/scripts/summarize_dpa.py) | RTX 5070 Ti、固定 GQA 与 shape 的 kernel 取址/分片比较；含正确性、原始计时、回退和未收敛记录 | TPOT/TTFT/Serving 收益、通用最优 split |
| tiny-llm GPU correctness | [252/252 非 skip 门禁报告](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-10-26-gpu-gate.md) · [脱敏原始日志](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/data/2026-10-26-gpu-gate.log) | RTX 3060 Laptop 上两种真实 GGUF 模型与不同 GQA 几何的本地正确性回归；commit、模型 hash 和零 skip 可审计 | 远端持续 GPU CI、Sanitizer、跨架构兼容性或性能结论 |
| tiny-llm decode profiler | [ncu 归因报告](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-10-26-ncu-decode-profile.md) · [汇总 CSV](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/data/2026-10-26-ncu-decode-kernels.csv) | 固定模型无 Graph decode 窗口的逐 kernel 耗时与 SpeedOfLight 归因；区分量化 matmul、带宽受限 LM head 和其他 kernel | 未采样的完整 timeline、launch gap、精确 TPOT 分解或其他模型/GPU；CSV 汇总不是可打开的 `.ncu-rep` |
| [paged-serving](https://github.com/holtwood-systems/paged-serving) | [P1 历史基线](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-09-04-RTX3060Laptop-paged-serving) · [P2 流式矩阵](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-09-04-RTX3060Laptop-paged-serving-p2-streaming) · [批量末端后处理 canary](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-09-05-RTX3060Laptop-paged-serving-p2-batch-postprocess-canary) · [当前正式矩阵](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-09-07-RTX3060Laptop-paged-serving-p2-batch-postprocess-streaming) · [评测入口](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving) | 绑定 RTX 3060 Laptop、模型 SHA-256、双仓 commit、原始请求与报告的真实 CUDA closed-loop/Poisson 观察；当前 21-run 矩阵验证当前提交上的首文本 TTFT、Poisson seed、成功率与 429 边界 | 跨 GPU/模型/引擎的通用容量或生产 SLO；当前 c2/c8 与三档 Poisson 仍有未收敛 TTFT p95，且没有配对基线，不能形成批量末端后处理的速度比值；canary 为 `n=1`、无预热，不能作性能结论 |
| paged-serving real backend | [2026-10-26 真实 tiny-llm 后端日志](https://github.com/holtwood-systems/paged-serving/blob/master/tests/evidence/2026-10-26-real-backend.log) | 当前双仓提交与固定模型下，strategy 1 KV 的真实后端测试 8/8 通过、0 ignored | 持续 GPU CI、真实网络故障注入、稳定 SLO 或性能收益 |
| paged-serving 故障注入 | [2026-11-06 证据包](https://github.com/holtwood-systems/paged-serving/blob/master/tests/evidence/2026-11-06-fault-injection/README.md) | 真实后端客户端侧故障 4/4：mid-SSE RST、慢消费者溢出（RSS 有界）、64 并发洪峰 200×8+429×56、两时点断连；首跑抓出准入契约缺陷并修复（batch 超限 500 → 准入 429） | 真实网络抖动/丢包；慢消费者背压梯度；Retry-After 语义 |
| [cuflash](https://github.com/holtwood-systems/cuflash) | [RTX 4090 快照（含 GBench JSON）](https://github.com/holtwood-systems/cuflash/blob/master/docs/performance/results/2026-11-06-rtx4090/report.md) · [benchmark 口径](https://github.com/holtwood-systems/cuflash/blob/master/docs/performance/benchmarks.md) | sm_89-only cubin、52 项 GBench 原始数据、ctest 串行 71/71；并行泄漏假阳性留档 | 通用加速比或生产库等价性；无 profiler 产物 |
| [trifuse](https://github.com/holtwood-systems/trifuse) | [RTX 4090 快照（含原始 JSON）](https://github.com/holtwood-systems/trifuse/blob/master/docs/performance/results/2026-11-06-rtx4090/report.md) | rmsnorm_rope 18/18、gated_mlp 8/8 正确与逐 shape 原始报告；pytest 130/131（bf16 SGEMM 容差失败如实保留） | 跨 GPU 性能优势；autotuner 未启用 |
| [cuda-foundations](https://github.com/holtwood-systems/cuda-foundations) | [RTX 3060 实测页](https://github.com/holtwood-systems/cuda-foundations/blob/master/docs/en/benchmarks/rtx3060-laptop-2026-08-17.md) · [RTX 4090 实测页（含原始 JSON）](https://github.com/holtwood-systems/cuda-foundations/blob/master/docs/en/benchmarks/rtx4090-2026-11-06.md) | 教学 kernel 的本机优化阶梯（GEMM 5371→6983 GFLOPS）与 cuBLAS 差距；Softmax/RoPE/Conv 原始数据 | 生产算子库或推理系统性能；Python bindings 默认构建在本机不可用 |
| [kvtier](https://github.com/holtwood-systems/kvtier)（active） | [单卡 L2 回载基线](https://github.com/holtwood-systems/kvtier/blob/master/bench/repro-31600/evidence/2026-10-26-wsl2-l2-baseline/README.md) · [结果 JSON](https://github.com/holtwood-systems/kvtier/blob/master/bench/repro-31600/evidence/2026-10-26-wsl2-l2-baseline/results.json) · [本地补丁](https://github.com/holtwood-systems/kvtier/blob/master/bench/repro-31600/evidence/2026-10-26-wsl2-l2-baseline/sglang-43375-local.patch) · [指针诊断](https://github.com/holtwood-systems/kvtier/blob/master/bench/repro-31600/evidence/2026-10-26-wsl2-l2-baseline/wsl2-host-pointer-diagnostic.out) | WSL2/RTX 3060 Laptop、Qwen2.5-0.5B 上 6 轮 host 命中与回载的逐请求证据及配对 TTFT；保留失败包、固定上游 commit 和补丁 hash | #31600 完整复现、自研 HiCache engine、跨模型/GPU 性能、多卡/L3/EAGLE、decode 吞吐或默认 CUDA Graph 配置 |
| kvtier 原生 Linux | [L2 回载基线 README](https://github.com/holtwood-systems/kvtier/blob/main/bench/repro-31600/evidence/2026-11-05-linux-rtx4090-l2-baseline/README.md) · [结果 JSON](https://github.com/holtwood-systems/kvtier/blob/main/bench/repro-31600/evidence/2026-11-05-linux-rtx4090-l2-baseline/results-20261105T144330Z.json) · [server 日志](https://github.com/holtwood-systems/kvtier/blob/main/bench/repro-31600/evidence/2026-11-05-linux-rtx4090-l2-baseline/server-startup.log) | 原生 Linux/RTX 4090 上 8 轮 host 命中 4223/4224、配对 R/P 中位 0.375；**kernel IO 后端无需 PR #43375 补丁**，证实 WSL2 指针问题为平台特有；toolchain 偏离（torch 2.11+cu128、源码 sgl-kernel sm_89 裁剪）逐条记录 | 同上，另加：不代表官方 wheel/pinned torch 下可复现，不代表其他 IO 后端或多卡 |
| tiny-llm RTX 4090 | [sm_89 结果报告](https://github.com/holtwood-systems/tiny-llm/blob/master/docs/performance/results/2026-11-05-rtx4090.md) · [arch 修复笔记](https://github.com/holtwood-systems/tiny-llm/blob/master/.agents/notes/implemented/bug-fix/2026-11-05-cuda-arch-default-never-applies.md) | sm_89 门禁 252/252、decode graphs TPOT 2.267ms/441tok/s、LM head 282µs、DPA 30 shape bitwise；含 sm52 失败批、ncu ERR_NVGPUCTRPERM 与 cudaEvent 替代归因 | 与 3060/5070 Ti 数字不构成优化前后对比；ncu 完整 profile 缺位；非持续 GPU CI |
| paged-serving RTX 4090 | [干扰包](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-11-05-NVIDIA-GeForce-RTX-4090) · [独占复测包](https://github.com/holtwood-systems/paged-serving/tree/master/benchmarks/serving/results/2026-11-05-RTX4090-exclusive-gpu) | 真实 tiny-llm 后端 closed-loop：首包记录外部干扰下 c1/c4/c8 未收敛；独占包四档全收敛、~300 tok/s 饱和 | 跨 GPU/引擎容量结论；干扰包与独占包不可合并为单一数值 |

### 整改分支的控制面回归

[第二批 `paged-serving@59d90c8`](https://github.com/holtwood-systems/paged-serving/commit/59d90c84aa0ea849322c18d3c741f8f9eef34dc9)
复用 PR #23 的取消实现，加入有界文本 mailbox、独立终态和无转发任务的多候选合并。
Rust 1.88 本地通过 255 个默认测试与 17 个 doc tests；24 个服务内联测试和 45 个
HTTP/SSE 测试重复 10 轮通过。[实现与测试边界](https://github.com/holtwood-systems/paged-serving/blob/59d90c84aa0ea849322c18d3c741f8f9eef34dc9/.agents/notes/implemented/feature/2026-10-04-bounded-events-and-cancellation.md)
覆盖无文本取消、慢消费者、成功排空、末步投递失败和资源回基线。这是 CPU 控制面证据，
不是新的 CUDA/Serving 性能包；该批已随 PR #24 合入默认分支，真实网络压力仍待补。

[第三批 `paged-serving@3039093`](https://github.com/holtwood-systems/paged-serving/commit/3039093ddc2fb6ddcab9508e4024bc84a61816c7)
实现类型化取消和按候选的独立计数，JSON/准入/后端/SSE 错误按 HTTP 请求去重，
包括未读取 body 的后端失败；慢消费者溢出属于失败，末步溢出不改引擎已成功的事实。
Rust 1.88 本地通过 260 个默认测试与 17 个 doc tests，25 个服务内联与 48 个
HTTP/SSE 测试重复 10 轮通过。[指标单位与兼容性](https://github.com/holtwood-systems/paged-serving/blob/3039093ddc2fb6ddcab9508e4024bc84a61816c7/.agents/notes/implemented/bug-fix/2026-10-04-typed-cancellation-and-metrics.md)
明确 Rust source-breaking change，以及取消保持 500 信封但不计 HTTP errors。
该提交的 [CI](https://github.com/holtwood-systems/paged-serving/actions/runs/37170403585)
在 2026-10-04 完成，Rust 1.88 MSRV 与 stable 检查均通过。
新增用例验证 CPU 控制面，不新增 TCP 故障注入、CUDA 回收或性能数据；完整测试中的
既有 loadgen TCP/SSE 回归仍通过，不能据此称新取消/背压实现已完成真实网络压力验收。

[第四批 `paged-serving@69dafbe`](https://github.com/holtwood-systems/paged-serving/commit/69dafbe9e4f726fa8f5b472666e4679945b516b3)
补齐真实 loadgen 二进制到 JSONL/summary 落盘的 CLI 回归，修复预热推进测量 RNG 和
测量输入顺序的问题。Poisson 使用绝对 deadline；原始请求分别记录计划时间与客户端
dispatch，目标到达率不等于服务端实际到达率。
[执行口径与验证](https://github.com/holtwood-systems/paged-serving/blob/69dafbe9e4f726fa8f5b472666e4679945b516b3/.agents/notes/implemented/testing/2026-10-04-loadgen-cli-reproducibility.md)
包含 264 个默认测试与 17 个 doc tests 本地通过，4 个 CLI 用例重复 10 轮，60 个子进程
通过。50% token coverage 的 tok/s 保持 null；100% coverage 按 measurement wall
计算。历史结果没有补写新字段，21-run 正式包结构校验仍通过；不作为新的 GPU 性能结论。
该提交的 [CI](https://github.com/holtwood-systems/paged-serving/actions/runs/37172586197)
在 2026-10-04 完成，Rust 1.88 MSRV 与 stable 检查均通过，head 对应 `69dafbe`。

[第五批 `paged-serving@a7fef1e`](https://github.com/holtwood-systems/paged-serving/commit/a7fef1e523132fe5e52bf22141afdc97db53b682)
联合重算原始请求、summary 与 metadata；绘图拒绝不兼容系列，也不只平均有可信
token 吞吐的部分重复。36 个标准库测试通过，5 个存量包的 66 个 run 只读重验通过；
三个正式包满足正式门槛，两个 canary 保持基础校验。
[校验规则与限制](https://github.com/holtwood-systems/paged-serving/blob/a7fef1e523132fe5e52bf22141afdc97db53b682/.agents/notes/implemented/testing/2026-10-04-serving-result-semantics.md)
区分数据矛盾和 `non_converged`；9/7 的 c2/c8、三档 Poisson 未收敛与 83 个 429
完整保留，核心 CSV 均值与历史记录一致。新增图表只写临时目录，历史 raw/报告/图未改。
这是数据内部一致性与引用门禁，不是新的性能实验，不证明真实 CUDA 取消回收或生产 SLO。
该提交的 [CI](https://github.com/holtwood-systems/paged-serving/actions/runs/37187094976)
在 2026-10-04 完成，`serving-evidence`、Rust 1.88 MSRV 和 stable 检查均通过。

## 旗舰系统当前缺口

`tiny-llm + paged-serving` 已有跨语言正确性链路及多份真实 CUDA serving 结果包；
但下列缺口关闭前，仍不宣称通用容量、跨引擎排名或生产成熟度：

- 当前 2026-09-07 矩阵中，closed c1/c4 通过 TTFT p95 与吞吐的 10% 重复波动检查，
  c2/c8 与三档 Poisson 的 TTFT p95 仍未通过；Poisson 0.64 / 1.28 req/s 还分别出现
  9 / 74 个 HTTP 429；
- 发压机与服务同机，未采集 GPU 时钟/功耗，且没有独立网络发压机；
- 尚无 llama-server/vLLM 的同模型、同上下文、同工作负载对照；
- direct/split-KV 的已有结果是 kernel 级；默认 legacy、split 关闭，不能据此宣称
  Serving 或整个模型加速；
- 取消/有界队列/指标与结果门禁通过独立代理审阅后，已由
  [PR #24](https://github.com/holtwood-systems/paged-serving/pull/24) 合入默认分支
  （2026-10-05 北京时间，merge `e60a315`）。真实 CPU TCP 回归与文本 EOF/终态竞态
  修复已有证据；原生 GPU 登记/显存回收和网络负载调优仍需验证。较早的
  [PR #23](https://github.com/holtwood-systems/paged-serving/pull/23) 已于 2026-10-25
  以 superseded 关闭，master 实现为准；
- 当前 tiny-llm FFI 的 Transformer layer forward 仍逐序列执行；正常 greedy 的末端已改为
  GPU batch final RMSNorm / LM head / argmax 与一次 batch token 回传，但 logprobs 仍走主机
  logits 路径，不能把调度 batch 当作 fused compute batch；当前矩阵没有配对基线，不能将
  任何差异归因于这项末端后处理；
- KV 利用率采样、prefix cache、抢占、chunked prefill 与 token 级 ITL 仍未实现。

## 引用规则

- 履历和面试只引用技术仓中已归档的结果，并保留硬件和 workload 限定。
- 正确性测试数不替代性能证据；单 kernel benchmark 不替代端到端 Serving 结果。
- 无法回溯到原始结果的历史数字不进入履历、首页或新的性能声明。
