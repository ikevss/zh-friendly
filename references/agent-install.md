# Agent 人设安装规则

> scan 子命令的"第四步：人设推荐"中，从 agency-agents-zh 仓库安装 agent 到各工具的规则。

## 仓库信息

- 仓库：`https://github.com/ikevss/agency-agents-zh`
- 内容：268 个中文 agent 人设，按 19 个部门分类
- 格式：Markdown + YAML frontmatter（name、description、emoji、color）
- 下载 URL：`https://raw.githubusercontent.com/ikevss/agency-agents-zh/main/{path}`

## 部门 → 目录映射

| 用户选择 | 仓库目录 |
|---------|---------|
| 金融投资 | `finance/`、`specialized/` |
| 内容创作与营销 | `marketing/`、`paid-media/` |
| 软件工程 | `engineering/`、`testing/`、`security/` |
| 产品设计 | `design/`、`product/` |
| 数据分析 | `specialized/`、`gis/` |
| 项目管理 | `project-management/`、`strategy/` |
| 游戏开发 | `game-development/` |
| 法务合规 | `legal/`、`hr/` |

## Claude Code 安装（主目标）

- 目标路径：`~/.claude/agents/{name}.md`
- 格式：Markdown + YAML frontmatter（与仓库一致，无需转换）
- 检查：同名文件已存在则跳过并提示
- 校验：frontmatter 至少含 `name` 和 `description`
- 安装后立即可用（agent 懒加载，无需重启）

## OpenCode 安装（V2）

- 目标路径：`~/.config/opencode/agents/{name}.md`（**复数 agents/，不是单数 agent/**）
- 格式：Markdown + YAML frontmatter，需额外加 `mode: subagent` 字段
- 示例：
  ```yaml
  ---
  name: 金融数据分析师
  description: 专业财务建模与数据分析...
  mode: subagent
  color: cyan
  ---
  ```
- **绝不碰** `~/.config/opencode/agent/`（单数，内部路由系统）

## Codex 安装（V3，暂不实现）

- 目标路径：`~/.codex/agents/{name}.toml`
- 格式：TOML（不是 Markdown）
- 转换规则：
  ```toml
  name = "AgentName"
  description = "..."
  developer_instructions = "\n# Agent Personality\n...(正文内容，换行转 \\n，引号转 \\")"
  ```
- 关键转义：正文中的 `\n` → `\\n`，`"` → `\"`，`\` → `\\`
- 风险高，需要严格测试后才启用

## WorkBuddy 安装

- WorkBuddy 的 agent 人设机制不同于其他工具：它没有 `agents/` 目录
- 人设定义在工作区顶层文件中：
  - `SOUL.md` — 核心人格（价值观、行为准则、沟通风格）
  - `IDENTITY.md` — 身份信息（名字、角色、特质）
  - `USER.md` — 用户画像（随交互积累）
- 安装方式：将 agency-agents-zh 的 agent 人设内容写入 `SOUL.md` 和 `IDENTITY.md`
- **注意**：WorkBuddy 的 SOUL.md 是全局人格，不是独立 agent。写入前备份原文件
- 检测方式：`test -f ~/.workbuddy/SOUL.md`

## OpenClaw 安装

- OpenClaw 没有独立的 agents 目录，agent 功能通过 skill 内嵌实现
- agent 人设定义在 skill 的 SKILL.md 正文中，通过 `sessions_spawn()` 机制调用
- 安装方式：将 agency-agents-zh 的 agent 内容作为新 skill 写入 `~/.openclaw/skills/{name}/SKILL.md`
- SKILL.md 正文需包含完整的 agent 人设定义（身份、使命、规则、工作流）
- 检测方式：`test -d ~/.openclaw/skills/`
- **注意**：OpenClaw 的"agent"本质是 skill，安装后会出现在 skills 列表而非独立的 agent 列表

## 安装数量限制

- 每次推荐最多 **15 个** agent
- 按部门分组展示，每个部门 3-5 个
- 用户可选择"全部安装"、"逐个选择"或"取消"
- 安装后提示用户可运行 `zh` 汉化新 agent 的 description

## 安装后更新能力清单

安装完成后自动重跑 scan 的第二步和第三步，将新 agent 纳入能力清单。
