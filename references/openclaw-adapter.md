# OpenClaw 适配器

> 本文件定义 OpenClaw 平台的扫描路径、汉化规则和排除规则。
> ⚠️ OpenClaw 是**纯 skill 架构**，没有独立的 agents/commands/MCP/plugins 目录。

## 扫描路径

### Skills（两种来源）
1. 用户级：`~/.openclaw/skills/*/SKILL.md` 或 `~/.openclaw/skills/*.md`（扁平文件）
2. 市场级：`~/.openclaw/workspace/skills/*/SKILL.md`

### 共享池（符号链接来源）
- `~/.openclaw/skills/` 下约 80 个符号链接指向 `~/.agents/skills/`
- 这些是跨工具共享的 skill，修改会影响所有引用它的工具

## 排除规则

1. `~/.openclaw/workspace/skills/.skills_store_lock.json`（市场安装锁文件，**绝不修改**）
2. `~/.agents/.skill-lock.json`（跨工具锁文件，**绝不修改**）
3. 非 SKILL.md 的辅助文件（`_meta.json`、`config.json`、`README.md` 等）

## 符号链接检测与处理

**这是 OpenClaw 适配器最关键的安全机制。**

检测方法：
```
使用 Bash 的 test -L 判断文件是否为符号链接
```

处理策略：
- 扫描到符号链接时，在输出中显示：`⚠️ [符号链接] path -> target`
- 提示用户："修改源文件会影响所有引用它的工具（Claude Code、OpenClaw 等）"
- 提供两个选项：
  - A）修改源文件（一次汉化，所有工具受益，但会破坏 `~/.agents/.skill-lock.json` 中的哈希校验）
  - B）跳过（保持原样）
- 默认行为：跳过符号链接，除非用户明确选择 A

## 汉化规则

### 可翻译字段（frontmatter）
- `description` — 主要触发字段，扩写式翻译

### 绝不翻译的内容（除通用列表外，额外增加）
- `model` 字段值（如 `"reasoning"`，是模型标识）
- `category` 字段值（如 `"testing"`，是分类标识）
- `triggers` 字段的值
- `metadata` 中的技术标识符

### 非标准 frontmatter 兼容
- `plugin-architecture/SKILL.md` 没有使用标准 `---` 分隔符
- 解析策略：
  1. 先尝试标准 YAML frontmatter（`---` 开闭）
  2. 如果失败，尝试在文件头部查找裸 `key: value` 行（连续的非空行，直到遇到空行或 Markdown 标题）
- 这种 fallback 解析只用于读取，生成的 description 仍写入标准 `---` frontmatter 格式

### 锁文件兼容
- 汉化不修改锁文件中的哈希值
- 这意味着下次 `skillhub update` 时可能触发哈希不一致警告
- 在输出报告中提示用户："汉化后如遇 marketplace 更新冲突，请选择保留本地版本"

## scan 子命令说明

scan 时 OpenClaw 侧只提取 skills 信息（name + description），因为：
- 无独立 agents 目录
- 无独立 commands 目录
- 无 MCP 配置
- 无 plugins 系统

能力清单中 OpenClaw 部分会比其他工具简洁。
