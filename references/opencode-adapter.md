# OpenCode 适配器

> 本文件定义 OpenCode 平台的扫描路径、汉化规则和排除规则。
> 与 Claude Code 的差异用 ⚠️ 标注。

## 扫描路径

### Skills
- 全局：`~/.config/opencode/skills/*/SKILL.md`
- 部分 skill 有 `_meta.json`（marketplace 元数据，含 version、slug、ownerId）

### Commands
- 全局：`~/.config/opencode/command/*.md`（⚠️ **注意目录名是 `command` 不是 `commands`**）

### Agents
- 全局：`~/.config/opencode/agents/*.md`（⚠️ **复数形式 `agents/`，是用户可写的人设文件**）
- ⚠️ **绝不能碰** `~/.config/opencode/agent/`（单数形式，是 OpenCode 内部路由系统）

### MCP 服务器
- 配置文件：`~/.config/opencode/opencode.json`（`mcp` 键）和 `~/.opencode/opencode.json`
- ⚠️ opencode.json 启动时严格校验 JSON schema（`$schema: "https://opencode.ai/config.json"`），**只读不写**

### Providers
- 配置文件：`~/.config/opencode/opencode.json`（`provider` 键）
- 包含 provider 名称和模型列表，**只读不写**

## 排除规则

1. `~/.config/opencode/agent/`（单数，内部路由系统）
2. `~/.config/opencode/context/`（上下文注入系统，非技能配置）
3. `_meta.json` 文件（marketplace 元数据，不修改）
4. `opencode.json` 配置文件本身（**绝不修改**，会破坏启动校验）

## 汉化规则

### 可翻译字段（frontmatter）
- `description` — 主要触发字段，扩写式翻译
- `argument-hint` — 如存在，翻译

### 绝不翻译的内容（除通用列表外，额外增加）
- `model` 字段值（如 `"reasoning"`，是模型标识）
- `category` 字段值（如 `"testing"`，是分类标识）
- `triggers` 字段的值（触发词数组）
- `source`、`risk`、`date_added`、`version`、`homepage`、`slug` 字段值
- `disable-model-invocation` 字段值

### marketplace 来源检测
- 存在 `_meta.json` → 此 skill 来自 marketplace
- 汉化后提示："marketplace 更新可能覆盖汉化内容"
- `_meta.json` 中的 `version` 信息在 scan 时可读取用于能力清单版本标注

### Agents 处理
- `~/.config/opencode/agents/` 格式与 Claude Code 的 `.claude/agents/` 基本同构
- frontmatter 含 `mode: subagent`、`color` 等 OpenCode 特有字段
- V1 仅 scan 只读扫描，不写入 agent 人设
