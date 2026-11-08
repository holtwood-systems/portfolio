# 2026-11-08 证据链接与项目卡片数字同步

- **kvtier 链接**：`README.md` 与 `docs/evidence-index.md` 中共 5 条
  `blob/master` 链接改为 `blob/main`。kvtier 仓库只有 `main` 分支
  （本地 `branch -a` 与 `git ls-remote --heads` 均确认），原链接在 GitHub
  上 404，导致作品集证据链的核心入口不可达。
- **LEARNING_PATH**：删去“尚无完成审计的 GPU 结果包”的过期描述，改为
  已存在 2026-10-26 WSL2/3060 与 2026-11-05 原生 Linux/4090 两个审计结果包，
  与 kvtier README、状态注册表和证据索引一致。
- **项目卡片**：trifuse 的 `GPU 123/123` 更新为最新 2026-11-06 RTX 4090
  快照的 `pytest 130/131（bf16 SGEMM 容差失败留档）`；cuflash 的
  `81/81 测试` 更新为同快照的 `串行 ctest 71/71（1 skip）`。卡片上方的
  “数字来自各仓最近一次归档快照”声明现在与所列数字一致。

## 验证

- 5 条 kvtier 目标路径在本地 `kvtier` 工作树中逐一存在。
- 更新后的测试数字逐项对照
  `trifuse/docs/performance/results/2026-11-06-rtx4090/report.md` 与
  `cuflash/docs/performance/results/2026-11-06-rtx4090/report.md`。
- 未运行 GitHub 上的实际 HTTP 访问检查；分支事实来自本地与远端引用列表。

## 笔记判定

机械性链接与数字同步，不改变工作区治理或架构决策，按仓库约定不新增
Agent Note。

## 2026-11-08 追加：测试计数口径纠正

- **paged-serving 615 → 301 passed / 1 ignored**：`615` 来自 2026-10-04 的
  CI 汇总（`msrv` 与 `check` 两个 job 运行同一批用例后再加 doc tests），
  不是单次命令的唯一用例数，无法从当前提交复现。现改为本地单次
  `cargo test --locked` 口径（commit `b6af619`；doc 17、lib 177）。
  `README.md` 项目卡片与证据摘要同步；旧 changelog 保留 615 作为历史记录，
  不回写。
- **今日本地提交**（未推送，portfolio 数字暂以本地 commit 为准）：
  `paged-serving@b6af619`、`trifuse@10cd88c`、`cuda-foundations@d8dc35a`、
  `cuflash@fcbcff2`、`tiny-llm@12a03f8`。
- **验证边界**：paged-serving 与 trifuse 已跑通本地 CPU 测试；cuda-foundations
  的 MemoryPool 用真实头文件 host harness 验证；cuflash 与 tiny-llm 已在
  CUDA 12.2.2 devel 容器中完整编译（tiny-llm 261 项用例，GGUF/量化 18 项
  host 测试通过）。真实 GPU 全量门禁、cuflash 并发数值对拍、tiny-llm 真实
  Q4_0 模型 e2e 仍未运行，相关声明保持 E2/E3 边界。
