分析 Issue 处理质量。先 Open 后 Closed。

→ `issues_result.json`：`{ open_critical: [{ number, severity, summary }], closed_stats: { total, resolved, unresolved }, response_quality }`

## Phase 1：Open Issues

```bash
gh api "search/issues?q=repo:<owner>/<repo>+is:issue+is:open+created:>=<1mo前>&per_page=15&sort=created&order=desc" --jq ".items[] | {number, title, state, created_at, updated_at, user: .user.login}"
```

Issue 总数 >100 时缩窄时间窗：先 `1w前`，不够补 `2w前`。>500 时取 10 条。

**严重性判断：**
- 🔴 critical：数据丢失、安全漏洞、崩溃、竞态、回归、内存泄漏、核心功能不可用
- 🟡 high：功能严重受限、API 行为错误、性能严重下降
- ⚪ medium 及以下：跳过

**high/critical 深挖：** `gh issue view <N> --repo <owner>/<repo> --json title,body,state,comments` 读完整帖，评估：维护者参与度（bot 快速回复不算）、可重现性、修复进展（PR/label/milestone）。

## Phase 2：Closed Issues

```bash
gh api "search/issues?q=repo:<owner>/<repo>+is:issue+is:closed+created:>=<1mo前>&per_page=15&sort=created&order=desc" --jq ".items[] | {number, title, state, created_at, updated_at, user: .user.login}"
```

`gh issue view <N> --repo <owner>/<repo> --json comments` 读最后 3-5 条评论。

**关闭分类与统计：**
- resolved：有 PR/commit 引用、维护者确认、问题已解答
- unresolved：stale bot 关闭、wontfix 无理由、重复指向未解决 issue

**响应质量：** 自然语言总结回复速度、讨论深度、拒绝理由、bot vs 人类。
