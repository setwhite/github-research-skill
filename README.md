# GitHub Research

AI Skill — 多维度调研 GitHub 项目，评估代码质量、Issue 处理、Commit 规范、贡献者水平、Vibe Coding 程度，输出结构化 Markdown 报告。

## 项目结构

```
github-research/
├── SKILL.md                 # Skill 定义 + 执行流程 + 硬规则
└── modules/
    ├── overview.md          # 仓库元信息（license、语言、stars、issue 数）
    ├── contributors.md      # 贡献者人/Bot 分类
    ├── readme.md            # README 文档质量四维评分
    ├── architecture.md      # 技术栈、包管理、架构模式
    ├── issues.md            # Issue 处理质量（open 严重 + closed 分类）
    └── commits.md           # Commit 历史质量（message、diff、红旗）
```

## 前置条件

- [GitHub CLI](https://cli.github.com/) (`gh`) 已安装并认证
- 对目标仓库有读取权限

## 用法

输入 `owner/repo`，AI agent 按 6 模块管线自动执行，产出到 `<owner>-<repo>/` 目录（全小写、kebab-case）：

```
vercel-ai/
├── overview_result.json
├── contributor_map.json
├── readme_result.json
├── architecture_result.json
├── issues_result.json
├── commits_result.json
└── github-research-report.md
```

## 执行管线

| 阶段 | 模块 | 产出 |
|------|------|------|
| 串行 | Overview | 仓库元信息 |
| 串行 | Contributors | 贡献者人/Bot 分类映射 |
| 并行 | README | 文档质量四维评分 |
| 并行 | Architecture | 技术栈、包管理、架构模式 |
| 并行 | Issues | open 严重问题 + closed 处理质量 |
| 并行 | Commits | message 质量、diff 合理性、红旗标记 |
| 聚合 | — | 7 章结构化报告 |

Overview → Contributors 串行（后者依赖 contributor_map 供后续模块过滤 bot），其余 4 模块并行执行，最后聚合所有 JSON 撰写报告。

## 报告结构

1. **项目概览** — license、语言、创建时间、归档状态、stars、forks、issue 数（`open/closed`）、open PR
2. **README 质量** — 完整性、安装说明、示例、API/配置文档
3. **技术架构** — 技术栈、运行时、包管理、目录结构、架构模式
4. **Issue 分析** — open 严重问题 + closed 处理质量
5. **Commit 分析** — 活跃度、message 格式、diff 合理性
6. **贡献者分析** — 人类/Bot 分类统计
7. **综合评估** — 🟢🟡🔴 评分 + Vibe Coding 程度 + 推荐度 + 关键风险

每节末尾附一句话总结，每个结论附可验证引用（Issue `#N`、Commit SHA、文件路径）。

## Vibe Coding 检测

综合以下信号判断，非单一规则：

- 大量相同 commit message
- diff 与 message 不匹配且无解释
- 同一作者 1h 内 10+ 条密集提交
- 功能变更缺测试覆盖

## 数据来源

所有数据通过 `gh api` / `gh issue view` / `gh pr view` 获取，不依赖 GitHub Web 抓取。search API 的 `total_count` 不可靠，用 REST 端点或时间窗替代大 limit。
