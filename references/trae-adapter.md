# TRAE Work 适配器

> 本文件定义字节跳动 TRAE Work 平台的扫描路径、汉化规则和排除规则。
> TRAE 是国内用户常用的 AI 编程工具，skill 体系与 Claude Code 基本同构。

## 扫描路径

### Skills
- 用户级：`~/.trae/skills/*/SKILL.md`（本机 99 个，SKILL.md + YAML frontmatter 格式）
- 内置：`~/.trae/builtin_skills/`（含 `_shared` 子目录）

### User Rules（类似 Commands）
- 用户规则：`~/.trae/user_rules/*.md`
- 格式：纯 Markdown，内容是用户自定义的指令/约束（如"始终用中文回复"）

### MCP 服务器
- 配置文件：`~/.trae/mcps/*.json`（每个 MCP 服务器一个 JSON 文件）
- 格式：`{ "command": "...", "args": [...], "env": {...} }`

### Extensions（VS Code 兼容扩展）
- 列表文件：`~/.trae/extensions/extensions.json`
- 提取字段：`identifier.id`、`version`

### TRAE-cn 变体
- 中国版配置：`~/.trae-cn/`（含独立的 skills、mcps、plugins、memory 等）
- 如果存在，一并扫描 `~/.trae-cn/skills/` 和 `~/.trae-cn/mcps/`

## 排除规则

1. `~/.trae/builtin_skills/_shared/`（内置共享模块，更新会覆盖）
2. `~/.trae/extensions/` 目录下的子文件夹（运行时缓存）
3. `~/.trae/binaries/`（二进制文件）
4. `~/.trae/argv.json`（启动参数）

## 汉化规则

### 可翻译字段（frontmatter）
- `description` — 主要触发字段，扩写式翻译
- `argument-hint` — 如存在，翻译

### 可翻译内容（User Rules）
- `~/.trae/user_rules/*.md` 中的自然语言说明
- 这些是用户自己写的规则，汉化有助于理解

### 绝不翻译的内容（除通用列表外，额外增加）
- `allowed-tools` 中的 Bash 权限规则（如 `Bash(npx agent-browser:*)`）
- MCP 配置文件中的 `command`、`args`、`env` 值
- Extensions 的 `identifier`、`version`、`metadata` 字段

### 特殊注意
- TRAE 的 SKILL.md 格式与 Claude Code 几乎一致（name/description/allowed-tools），汉化逻辑可复用
- TRAE-cn 可能有已中文化的 skill，扫描时检测中文字符占比 > 50% 则跳过
- user_rules 文件通常是用户个性化设置，汉化时需保守——只翻译明显是英文的规则

## scan 子命令说明

scan 时 TRAE Work 提取以下能力信息：
1. **skills/** → name + description（99 个）
2. **user_rules/** → 文件名 + 内容摘要
3. **mcps/** → MCP 服务器名（从 JSON 文件名提取）
4. **extensions/** → 扩展名 + 版本

能力清单中 TRAE Work 归入"其他平台"分组，但由于 skill 数量多（99 个），会按优先级截断显示。
