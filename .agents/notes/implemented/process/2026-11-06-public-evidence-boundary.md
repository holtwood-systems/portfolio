# Agent Note: 公开作品集不再引用无公开 artifact 的私有实验数字

Status: implemented

## Problem

`portfolio/README.md` 的“2026-11 实验线（2×RTX 4090 48GB）”列出了手写
FlashAttention、softmax、DDP/NCCL 等六组数字，但标注“结果包整理中”，
没有公开结果包、原始数据或 commit 链接；其中 DDP 数字还会被读成
“分布式经验”。这与本仓“每个数字绑定 artifact”的规则冲突，也与
`portfolio-graduation-and-interview-proof.md` 中“分布式仅理论、未实测”
的边界冲突。

同一天，paged-serving 的 7B serving canary 因 provenance 缺失与基线矛盾
被标记 invalid；虽然它不在 portfolio README 的项目卡片里，但
`docs/evidence-index.md` 需要明确它不可引用，避免后续从技术仓链接时
误入公开索引。

## Decision

1. 从公开 README 移除 2×4090 实验数字表，只保留一段“私有 interview lab，
   未形成公开可复现 artifact，不作为公开声明”的说明。
2. 在 `docs/evidence-index.md` 新增“已失效 / 不可引用证据”：
   - paged-serving `2026-11-06-7b-serving`：invalid/unverified；
   - 2026-11 私有 2×4090 实验线：未公开，不作为分布式经验。
3. 在项目卡片下注明：测试数/性能数字来自各仓最近一次归档快照，硬件、
   套件、日期不同，以链接报告为准，不横向排名。
4. `interview/NUMBERS_CARD.md` 加 Historical snapshot 横幅，避免 2026-08
   的 3060 数字被当作 2026-11 当前状态。
5. 不在本轮公开私有实验数字，也不新建 `experiments` 仓；等稳定硬件、
   公开结果包、限制说明齐全后再入册。

## Alternatives considered

- **保留数字但加“私有/未验证”标签**：读者仍会记住 24.7x、1.37x 等数字，
  而无法点进证据；这比不写更容易产生误读，否决。
- **为 2×4090 新建公开 experiments 仓**：能承载结果，但需要新仓治理、
  artifact 清理和长期维护；当前不应为一次私有练习新增第八个技术仓，
  否决。
- **把私有仓链接放到 README**：私有仓对面试官不可见，链接等于死链；
  且会让公开页面依赖不可访问资源，否决。
- **只删 README 表格、不改 evidence-index**：技术仓链接仍可能把 invalid
  7B 包带进公开索引；需要同时维护“不可引用”清单，否决。

## Consequences

- 公开首页少了一组“看起来很强”的分布式数字；换来的是每个留下的数字
  都能点进 artifact，面试追问时不会出现“结果包还没整理”的断点。
- 私有实验仍保留在个人执行仓，未来可在稳定 2×4090 上补齐公开结果包；
  本决策不删除任何原始记录。
- 7B canary 的负结果继续保留在 paged-serving，便于后续重跑对照；但不会
  出现在 portfolio 的“可导航证据”中。
- 面试材料需要以技术仓当前结果包为准；`interview/` 只作历史演练材料。

## Verification

- `README.md` 不再包含 2×4090 的六组数字表；只保留私有实验边界说明。
- `docs/evidence-index.md` 的“已失效 / 不可引用证据”包含 7B 包和私有
  2×4090 实验线。
- `interview/NUMBERS_CARD.md` 顶部有 Historical snapshot 横幅。
- `npm run verify-notes` 通过。
