# Claude Code 适配器

> 本文件定义 Claude Code 平台的扫描路径、汉化规则和排除规则。
> SKILL.md 的 zh/scan 流程通过 Read 工具加载本文件获取具体规则。

## 扫描路径

### Skills
- 全局：`~/.claude/skills/*/SKILL.md`
- 项目级：`.claude/skills/*/SKILL.md`（仅扫描当前项目）

### Commands
- 全局：`~/.claude/commands/*.md`
- 项目级：`.claude/commands/*.md`

### Agents
- 全局：`~/.claude/agents/*.md`
- 项目级：`.claude/agents/*.md`

### MCP 服务器
- 配置文件：`~/.claude/settings.json`（`mcpServers` 键）

## 排除规则

以下路径不扫描、不汉化：

1. 任何包含 `plugins`、`.plugins` 的路径
2. Claude Code 官方内置 bundled skills
3. 非 Markdown 文件
4. frontmatter 无法解析的文件
5. 明显不是 Claude Code 配置的 Markdown 文件

## 汉化规则

### 可翻译字段（frontmatter）
- `description` — 主要触发字段，必须扩写式翻译
- `when_to_use` — 如存在，翻译
- `argument-hint` — 如存在，翻译

### 可翻译内容（Markdown 正文）
- 自然语言说明、步骤、解释、注意事项
- 列表项中的描述文字
- 表格中的说明列

### 绝不翻译的内容
- 文件名、目录名
- `name` 字段值
- `allowed-tools`、`tools` 字段值
- `user-invocable`、`disable-model-invocation`、`model`、`effort`、`context`、`agent`、`paths`、`shell` 字段值
- `hooks` 字段的结构和值
- 所有工具名称（Bash、Read、Write、Edit、MultiEdit、Grep、Glob、Task、Skill、WebFetch、WebSearch）
- 所有权限规则（如 `Bash(git add )`、`Bash(gh )`、`Skill(commit)`）
- 所有参数占位符（`$ARGUMENTS`、`$0`、`$1`、`$2`、`$ARGUMENTS[0]`）
- 所有文件路径
- 所有 URL
- 所有动态 Shell 注入命令（`!` + 反引号形式）
- 所有 fenced code block 中的代码内容
- YAML 结构、缩进、列表格式、引号风格

### Agents 特殊处理
- 仅翻译 frontmatter 中的 `description` 和 `when_to_use`
- 第二个 `---` 之后的全部正文（System Prompt）**一个字符都不改**
- 修改后必须逐字符比对正文，有变化立即从备份恢复

### 翻译风格
- 使用简体中文，技术文档风格
- 必须做扩写式翻译：中文描述 + ` | ` + 英文触发关键词
- 保留必要的英文技术术语（参见 `references/glossary.md`）
- 句式统一：动词 + 对象 + 补充说明
- 标点统一：中文标点
- 不扩写原文没有的能力，不改变触发含义

### 缺失 description 补充规则
- 从正文提取：标题 + 第一段 + 关键章节名
- 生成格式：中文描述 + ` | ` + 英文触发关键词
- 验证 frontmatter 完整性（`---` 开闭配对）
- 写入前 dry-run 预览
