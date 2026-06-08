获取仓库基本元信息（许可证、语言分布、创建时间、默认分支、归档状态、stars、forks、open/closed issue 数量）。

→ `overview_result.json`：`{ license, languages, created_at, default_branch, archived, stars, forks, open_issues, closed_issues, open_prs }`

输入：`owner/repo`。

## 1. 许可证

首选：`gh api repos/<owner>/<repo>` → `license.spdx_id`。

回退：`gh api repos/<owner>/<repo>/license` → `license.spdx_id`；再失败检查 `package.json`（`license` 字段）、`Cargo.toml`。全部失败设 `null`。

## 2. 语言分布

`gh api repos/<owner>/<repo>/languages` → `{ 语言: 字节数 }`，计算各语言字节占比（一位小数）。占比 < 0.1% 省略。失败设 `null`。

## 3. 仓库元信息

`gh api repos/<owner>/<repo>`，提取：
- `created_at`、`default_branch`、`archived`
- `stargazers_count`、`forks_count`
- `open_issues_count` — **含 open PR，非纯 issue 数**

任一字段缺失设 `null`。

## 4. Issue 与 PR 数量

```bash
# open_issues_count 含 issues + PRs
gh api repos/<owner>/<repo> --jq ".open_issues_count"

# 纯 open PR 数
gh api "search/issues?q=repo:<owner>/<repo>+is:pr+is:open&per_page=1" --jq ".total_count"

# open_issues = open_issues_count - open_prs

# closed issues 数
gh api "search/issues?q=repo:<owner>/<repo>+is:issue+is:closed&per_page=1" --jq ".total_count"
```

用减法而非 `search/issues is:open` — REST `open_issues_count` 确定，search `total_count` 不可靠。

报告格式：`94 open / 971 closed · 99 open PRs`

## 5. 错误处理

- 仓库不存在（404）→ 终止。
- 字段获取失败 → `null`，记录原因。
