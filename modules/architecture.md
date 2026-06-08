分析技术架构、技术栈、架构模式。

→ `architecture_result.json`：`{ tech_stack, runtime, package_manager, dir_structure, architecture_pattern }`

## 1. 数据来源

按优先级：README → 目录结构（根目录及 `src/`、`packages/`、`apps/`、`cmd/`、`lib/`，最多 3 层）→ 清单文件。

清单文件统一用：

```bash
gh api repos/<owner>/<repo>/contents/<path>?ref=main --jq ".content" | base64 -d
```

先 `?ref=main`，再 `?ref=master`；都失败标注 `[UNREADABLE]`。并行读取，不存在的跳过。

## 2. 分析

**包管理器** — lockfile 映射：`bun.lock`→bun、`package-lock.json`→npm、`yarn.lock`→yarn、`pnpm-lock.yaml`→pnpm、`Cargo.lock`→cargo、`go.sum`→go modules。用 `gh api repos/<owner>/<repo>/contents/<lockfile>?ref=main` 探测存在性。

**技术栈** — 从依赖文件提取包名，按框架/数据库/消息/基础设施/工具链分类。仅列实际依赖。无清单文件时用后缀推断，标注 `[INFERENCE]`。

**架构模式** — README + 目录结构 + 配置文件判断：
- Monorepo：`workspaces`、`lerna.json`/`nx.json`/`turbo.json`、`Cargo.toml [workspace]`、`go.work`
- 分层：`controllers`/`services`/`models` 或 `features/`/`modules/`
- 前后端分离：`frontend/` + `backend/` 或 `client/` + `server/`
- 微服务：多个 `services/` 各有独立配置
- 插件架构：`plugins/`/`extensions/` 目录

README 优先，目录验证。不匹配时自行描述，标注 `[INFERENCE]`。输出如 `"monorepo (turborepo) + layered"`。

**目录结构摘要** — 50-150 字：顶层分类、子目录用途、包/workspace 数量。

## 3. 边缘情况

- 无清单文件：后缀推断，`package_manager` 设 `none`，标 `[INFERENCE]`
- 超大仓库（500+ 项）：只读前 2 层
- 多语言：识别主运行时，次要标 `[tooling]`
