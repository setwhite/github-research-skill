分析 commit 历史质量。先扫 message + 活跃度，再对红旗钻取 diff。

→ `commits_result.json`：`{ activity, message_quality, diff_reasonableness, red_flags }`

## Phase 1：扫描 commit

两窗 `gh api search/commits`：

```bash
# 7 天活跃度
gh api "search/commits?q=repo:<owner>/<repo>+committer-date:>=<7天前>&per_page=15&sort=committer-date&order=desc" --jq ".items[] | {sha: .sha[0:7], message: .commit.message, author: .author.login, date: .commit.committer.date}"

# 30 天趋势
gh api "search/commits?q=repo:<owner>/<repo>+committer-date:>=<30天前>&per_page=15&sort=committer-date&order=desc" --jq ".items[] | {sha: .sha[0:7], message: .commit.message, author: .author.login, date: .commit.committer.date}"
```

**作者过滤：** bot（contributor_map 标记）和 merge commit（`Merge` 开头）跳过 message 评估，但计入活跃度和红旗。

### Message 质量（仅 human）

- Conventional Commits 遵循度
- Body 是否解释 WHY（不限于 WHAT）

**红旗：** `fix`/`update`/`changes`/`wip` 单独成句；`fix bug`/`fix issue` 僵尸 message；批量相同 message；空 message。
**绿旗：** 标题清晰、Body 解释动机、关联 issue/PR 引用。

### 活跃度

从采样窗口内的 commit 时间戳计算：**日均 = 采样条数 ÷ 窗口天数**，标注窗口范围（如「7 天窗口日均 2.1 条」）。**NEVER** 估算仓库总 commit 数——search API 的 `total_count` 不可靠，REST API 不提供该字段。禁止通过翻页计数或 `git` 命令推断总量。

其他指标：突发（同一作者 1h 内 10+ 条 → AI 批量）、断档（>6 月 → 间歇维护；>1 年 + 开放 issue → abandonware）。

## Phase 2：钻取 diff

选最可疑 3-5 个红旗 commit。

```bash
# 文件级统计（首选）
gh api repos/<owner>/<repo>/commits/<sha> --jq "{message: .commit.message, stats: .stats, files: [.files[]? | {filename, changes, additions, deletions}]}"

# 完整 diff（按需）
gh api repos/<owner>/<repo>/commits/<sha> --jq ".files[].patch"
```

**评估：**
- 规模：20+ files 或 500+ 行且 message 无法解释 → 🔴。≤5 files、≤100 行 → 🟢。lockfile 不计。
- 测试：功能 commit 含测试 → 🟢；缺失 → 🔴。
- 焦点：同时改逻辑+格式化+重命名+依赖 → 粒度过粗。diff 与 message 不符。
- "震楼器"：大段结构重复代码，仅变量名替换。

## 注意事项

- 无红旗时 `red_flag_commits` 空数组
- 无 contributor_map 时默认 human
- `items[].author` 可能 null（无 GitHub 关联）→ 用 `items[].commit.author.name`
