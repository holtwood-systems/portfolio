# 2026-11-06 tiny-llm 竞态修复与 sanitizer 证据

- **tiny-llm**：首份 compute-sanitizer 证据（`64254fb`）：memcheck
  42/42 干净；racecheck 报 3 hazards（online-softmax 归约路径）且
  SplitKv oracle 插桩下数值失败（0.083 vs 容差 0.002）。
- **根因与修复**（`9afe0c0`）：SASS 定位为真实竞态——
  `block_reduce_max_dyn` 末尾读 `red[0]`（`LDS [0x200]`）与下一次
  `block_reduce_sum_dyn` 开头写 `red[wid]`（`STS [R14+0x200]`）之间
  缺 BAR.SYNC，快 warp 可在慢 warp 读取前覆写。修复为两个归约函数
  各加收尾 `__syncthreads()`。修复后 racecheck **0 hazards**、
  memcheck 0 errors、gpu-gate **252/252**、TPOT 2.2694ms（+0.1%，
  噪声内）。
- **附带发现**：旧 build 缓存里 `CMAKE_CUDA_ARCHITECTURES=52` 会在
  增量 reconfigure 时被当作"用户指定"而静默退回 sm_52（实测 44 项
  门禁失败、SplitKv max|diff|=1.25）。CMakeLists 加固为按值识别
  污染（52 = nvcc 遗留默认）+ `CACHE FORCE` 持久化 native。
- **meta 同步**：证据索引与 README 的 tiny-llm 段落补 sanitizer 与
  竞态修复事实；"sanitizer 全绿"从此可作为有证据的说法。

## 笔记判定

本批 meta 改动为证据导航同步，无新的工作区治理决策。技术决策在
tiny-llm 的 `.agents/notes/implemented/bug-fix/`（竞态根因与修复、
arch 缓存二次陷阱）。
