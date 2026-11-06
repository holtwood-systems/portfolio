# 2026-11-06 serving 故障注入 + 三仓 sm_89 基线

- **paged-serving**：新增 `tests/fault_injection/inject.py`（客户端侧
  网络故障：mid-SSE RST、慢消费者溢出、64 并发洪峰、两时点断连），
  4/4 实验断言通过：取消计数+1、资源回零、停顿期并发探针正常、
  RSS +26.8 MiB 有界、洪峰 200×8+429×56。**首跑即抓出准入契约缺陷**：
  `PAGED_SERVING_TINY_LLM_MAX_SEQS` 只约束 executor，调度器
  `max_num_seqs/max_batch_size` 无联动，过载在 batch 组层变 500；
  修复为 `create_engine` 把两上限夹到后端容量（单测 2 例 + 真实
  后端 4/4 回归）。证据 `tests/evidence/2026-11-06-fault-injection/`。
- **trifuse**：首份带原始报告的硬件快照（`docs/performance/results/
  2026-11-06-rtx4090/`）：rmsnorm_rope 18/18、gated_mlp 8/8 正确
  （最高 31.8 TFLOPS），pytest 130/131——`test_sgemm_bf16` 容差失败
  如实保留（0.3% 相对差，同机 1024³ 复现误差为 0，shape/seed 相关）。
- **cuflash**：首份符合复验协议的包（`docs/performance/results/
  2026-11-06-rtx4090/`）：sm_89-only cubin、GBench JSON 52 项
  （FP16/BF16 forward 比 Naive FP32 快 10–20×，decode kv65536 0.246ms）、
  ctest 串行 71/71；并行 -j8 的泄漏测试假阳性（cudaMemGetInfo 并发
  污染）与隔离复跑通过均留档。
- **cuda-foundations**：第二页实测（`docs/en/benchmarks/
  rtx4090-2026-11-06.md`）：GEMM 阶梯 2048³ Naive 5371→DoubleBuffer
  6983 GFLOPS、Softmax/RoPE/Conv 原始 JSON 归档；ctest 36/36；如实
  记录 `TC_BUILD_PYTHON=ON` 的既有编译缺陷与 OFF 绕过。
- **meta 同步**：证据索引更新 trifuse/cuflash/cuda-foundations 三行并
  新增 paged-serving 故障注入行；README 与毕业门槛的逐仓段落随后续
  批次同步（本轮以索引为准）。

## 笔记判定

本批 meta 改动为证据导航同步，无新的工作区治理决策。技术决策分别
保存在 paged-serving（准入契约修复、故障注入测试策略）、trifuse /
cuflash / cuda-foundations（证据归档惯例、构建绕过）的
`.agents/notes/`。
