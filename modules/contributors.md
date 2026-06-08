分析贡献者构成，区分人类、Bot、AI Agent。

→ `contributor_map.json`：`{ "<username>": { "type": "human|bot", "evidence": "..." } }`，供后续模块过滤 bot。

## 1. 建立贡献者列表

`gh api "repos/<owner>/<repo>/commits?since=<90天前>&per_page=100"` → 按 `author.login` 聚合去重。commits 较多时加 `&page=2`。

补全：`gh api "search/commits?q=repo:<owner>/<repo>+committer-date:>=<90天前>&per_page=30&sort=committer-date&order=desc" --jq ".items[].author.login"`。

**NEVER** 用 `repos/<owner>/<repo>/contributors`（org 仓库常返回 404）。

合并去重，每人记录：username（小写）、commit_count、first/last seen、shas（最多 5 个样本）。

## 2. 分类

用户名含 `[bot]`、`dependabot`、`renovate`、`-bot`、`swe-agent`、`copilot`、`agent` 等 → `bot`；其余 → `human`。

`gh api` 返回 author 可能自带 `[bot]` 后缀。项目自家的 AI agent 也标 `bot`。

## 3. 产出

每人输出 `type` + `evidence`。写入 `contributor_map`。输出人类/Bot 数量统计 + 一句话结论。

## 4. 边缘情况

- 多 email 同人：当 `author.login` 为 null（commits 未关联 GitHub 账户）时，按 `commit.author.email` 精确匹配去重——同 email 视为同一人。不同 email 即使 name 相同也分开记录（无法可靠判断同一人）。
- 截断（>100 commits/90d）：用 search/commits 补全