# GitHub Research

一个 Oh My Pi / Claude Code skill，输入 `owner/repo`，自动调研 GitHub 项目并输出结构化评估报告。

## 安装

下载 `github-research.skill`，解压到 skills 目录：

**用户级**（所有项目可用）：
```
~/.omp/agent/skills/github-research/
```

**项目级**（仅当前项目）：
```
<项目>/.omp/skills/github-research/
```

重启终端即生效。

## 前置条件

- [GitHub CLI](https://cli.github.com/) 已安装并认证

## 使用

对话中直接说「调研 `owner/repo`」即可，例如：

> 帮我调研一下 sharkdp/bat 项目

AI 会自动执行 6 个分析模块，最终产出报告。

## 报告内容

| 章节 | 内容 |
|------|------|
| 项目概览 | license、语言、stars、forks、issue/PR 数量 |
| README 质量 | 完整性、安装说明、示例、API 文档（🟢🟡🔴）|
| 技术架构 | 技术栈、包管理、目录结构、架构模式 |
| Issue 分析 | open 严重问题 + closed 处理质量 |
| Commit 分析 | 活跃度、message 规范、diff 合理性 |
| 贡献者分析 | 人类 vs Bot 分类 |
| 综合评估 | 总体评分、Vibe Coding 程度、推荐度、关键风险 |

## Vibe Coding 检测

项目是否疑似 AI 批量生成代码？综合以下信号：

- 大量相同 commit message
- diff 与 message 不匹配且无解释
- 同一作者 1 小时内 10+ 条密集提交
- 功能变更缺少测试覆盖
