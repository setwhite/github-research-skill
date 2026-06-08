分析 README 质量。

→ `readme_result.json`：`{ completeness, installation, examples, api_docs }` — 每项含 `rating` + `evidence`

## 1. 获取 README

首选：`gh api repos/<owner>/<repo>/readme --jq ".content" | base64 -d`。

>500 行截断时先 `--jq ".size"` 确认大小再完整解码。

回退：依次试 `?ref=main`、`?ref=master`；或 `gh api repos/<owner>/<repo>/contents/README.md?ref=main --jq ".content" | base64 -d`，再试 `README`、`Readme.md`、`readme.md`、`docs/README.md`。

不存在则全部 `🔴 缺失`，结束。

## 2. 评估维度

统一 `🟢`/`🟡`/`🔴`，每项附 `evidence`。

**完整性** — 「是什么」「为什么」「谁用」：
- `🟢` 三项清晰 / `🟡` 缺 1-2 项 / `🔴` 无实质描述

**安装说明** — 前置依赖、平台差异、可复现命令。Docker 项目看 compose/run：
- `🟢` 完整 / `🟡` 有步骤但缺条件 / `🔴` 缺失或错误

**示例** — 可复制的代码片段、在线 demo：
- `🟢` 完整 + demo / `🟡` 不全 / `🔴` 无示例

**API/配置文档** — 内联 API、外部链接、类型定义、配置项：
- `🟢` 完善 / `🟡` 有但粗略 / `🔴` 无文档

## 3. 输出

写入 `readme_result` + 一句话总结（聚焦文档对用户的实际帮助程度）。

## 4. 按项目类型调整

CLI 看重 `--help` 和参数说明，库看重 API 文档，UI 看重 demo 链接，底层系统看重架构图。在 evidence 中说明判断依据。
