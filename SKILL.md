---
name: github-research
description: 调研 GitHub 项目——评估代码质量、issue 处理、commit 规范、贡献者水平、vibe coding 程度，输出结构化报告
---

# GitHub Research

多维度调研 GitHub 项目，评估维护水平与 vibe coding 程度，输出结构化报告。

## 硬规则

1. **数据获取只用 `gh` CLI** — `gh api`/`gh issue view`/`gh pr view`。**禁止用 `read` 读 `github.com`/`raw.githubusercontent.com` URL**（reader-mode 不稳定）。聚合阶段读本地 `*_result.json` 除外。
2. **`search/commits` MUST 带 `q`** — 裸调用静默返回空。格式：`q=repo:<owner>/<repo>+committer-date:>=<date>`。
3. **截断用时间窗替代大 limit** — limit 30 实出 15-20。open/closed 分开查各 `limit: 15`；精细时缩窄 `since`。

## 执行流程

| 顺序 | 模块 | 产出 |
|------|------|------|
| 串行 1 | Overview → `modules/overview.md` | `overview_result.json` |
| 串行 2 | Contributors → `modules/contributors.md` | `contributor_map.json` |
| 并行 | README → `modules/readme.md` | `readme_result.json` |
| 并行 | Architecture → `modules/architecture.md` | `architecture_result.json` |
| 并行 | Issues → `modules/issues.md` | `issues_result.json` |
| 并行 | Commits → `modules/commits.md` | `commits_result.json` |
| 聚合 | 读所有 JSON，撰写报告 | `github-research-report.md` |

产出到 `<owner>-<repo>/` 目录，全小写、kebab-case（如 `vercel/ai` → `vercel-ai/`，`esengine/DeepSeek-Reasonix` → `esengine-deepseek-reasonix/`）。

## 报告结构（7 章，每节末附一句总结）

1. 项目概览 — license、语言、创建时间、归档状态、stars、forks、issue 数（`open/closed`）、open PR
2. README 质量 — 完整性、示例、安装、API 文档
3. 技术架构 — 技术栈、运行时、包管理、目录结构、架构模式
4. Issue 分析 — open 严重问题 + closed 处理质量
5. Commit 分析 — 活跃度、message 格式、diff 合理性
6. 贡献者分析 — 人类/Bot 分类
7. 综合评估 — 🟢🟡🔴 评分 + vibe coding 程度 + 推荐度 + 关键风险

**Vibe coding 信号**：大量相同 commit message、diff 与 message 不匹配且无解释、同一作者 1h 内 10+ 条密集提交、功能变更缺测试覆盖。综合判断，非单一规则。

## 聚合

读所有 `*_result.json`，按报告结构撰写。每个结论附可验证引用（issue `#N`、commit SHA 前 7 位、文件路径）。
