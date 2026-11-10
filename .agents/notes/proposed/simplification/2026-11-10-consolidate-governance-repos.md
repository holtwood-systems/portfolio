# Agent Note: 合并 portfolio 与 open-infra-ai 双治理仓

Status: proposed

## Problem

`portfolio` 与 `open-infra-ai` 目录树几乎全同（`diff -r` 排除 .git 后
仅 8 处差异），都自称组织 landing 页与「唯一权威注册表」。漂移已实际
发生且包含实质内容分叉：

- `README.md`：portfolio 已有技术叙事、项目卡片与私有实验边界段，
  open-infra-ai 仍是旧版且测试口径停留在已废弃的「615 项」。
- `docs/cross-repo-contracts.md`：portfolio 为 ABI v3，
  open-infra-ai 仍为 ABI v2。
- `docs/evidence-index.md`：portfolio 有「已失效 / 不可引用证据」段，
  open-infra-ai 没有，7B canary 仍可能被当有效证据引用。
- `docs/portfolio-graduation-and-interview-proof.md`：portfolio 记录
  面试材料已于 2026-11-06 迁回私有仓；open-infra-ai 仍写「interview/
  是历史归档」，且整套面试排练材料仍在公开仓托管——迁移只执行了一半。
- `LEARNING_PATH.md`：kvtier 结果包状态不一致。

近 30 个提交里 portfolio 26 个、open-infra-ai 22 个是 docs 同步提交，
大部分为同一批变更在两个仓的重复搬运。「三处同步」的声明已被自身
违反，双仓结构是当前治理成本的最大来源。

## Proposal

保留 `portfolio` 为唯一治理仓（命名与职责最贴合）。分两步收敛：

1. 本轮：open-infra-ai 全量同步为 portfolio 当前状态，并按
   2026-11-06 已落地决策移除其 `interview/`（私有存档已核验存在于
   career-prep `archive/legacy-2026-11-09/`）；先消除漂移与半迁移。
2. 下轮：open-infra-ai 内容收敛为一页重定向 README 指向 portfolio，
   GitHub 上 archive 该仓；组织 profile 与各仓 README 的组织入口
   链接同步更新。状态注册表、证据索引、跨仓契约此后只维护一份。

## Alternatives considered

- **保持双仓、加同步脚本**——能降低漏同步概率，但「两个权威注册表」
  的语义冲突仍在，且同步脚本本身成为新的维护面，否决。
- **保留 open-infra-ai、退役 portfolio**——open-infra-ai 命名偏生态
  门户，不如 portfolio 贴合「作品集治理」职责；且 portfolio 内容更新、
  `interview/` 已迁出，保留它的改动面最小，否决。
- **把所有技术仓并入单一 monorepo**——一次消灭全部跨仓同步，但破坏
  各仓独立的 CI、发布节奏与 git 历史边界，远超本问题所需，否决。

## Acceptance criteria

- `diff -r portfolio open-infra-ai`（排除 .git/仓名自述）仅剩有意差异。
- `interview/` 不出现在任何公开仓；私有存档完整可溯。
- 公开仓中不再存在「两个权威注册表」；docs 同步类提交归零。

## Risks

- open-infra-ai 可能被外部链接（简历、issue、历史文档）引用：重定向
  期保留一页 landing README 写明去向，不立即删仓。
- archive 后 GitHub topics 与组织 README 可能残留旧链接：归档前统一
  改向 portfolio，避免死链。
- 失去「第二份拷贝」的表面冗余：实际冗余由 git 历史与本地克隆提供，
  不构成真实风险。
