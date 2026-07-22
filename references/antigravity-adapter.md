# Antigravity (Gemini) 适配器

> 本文件定义 Google Antigravity 平台的扫描规则。
> ⚠️ **Antigravity 仅支持只读扫描，不修改任何文件。**

## 扫描路径

### Extensions（VS Code 兼容扩展）
- 列表文件：`~/.antigravity/extensions/extensions.json`
- 提取字段：`identifier.id`（扩展 ID）、`version`
- ⚠️ **绝不修改此文件**，会破坏扩展加载

### MCP 服务器
- 配置文件：`~/.gemini/antigravity/mcp_config.json`（`mcpServers` 键）
- 格式：`{ "mcpServers": { "name": { "command": "...", "args": [...] } } }`
- 可能存在符号链接：`~/.gemini/antigravity/mcp_config.json` → `~/.gemini/config/mcp_config.json`

### Skills（角色文件）
- 目录：`~/.gemini/antigravity/skills/`
- 格式：SKILL.md + YAML frontmatter（与 Claude Code 基本同构）
- 额外字段：`risk`、`source`、`date_added`
- 已有约 200+ 个角色文件，**大部分已有中文 description**

### 不纳入扫描的路径
- `~/.gemini/antigravity/brain/`（会话级工作产物，非能力描述）
- `~/.gemini/antigravity/knowledge/`（当前为空）
- `~/.gemini/antigravity/antigravity_state.pbtxt`（protobuf 状态文件，无用户可读文本）
- `~/.gemini/antigravity/context_state/`（上下文状态）
- `~/.gemini/antigravity/conversations/`（对话历史）
- `~/.gemini/antigravity/browser_recordings/`（浏览器录制）

## scan 子命令的数据提取

scan 时从以下来源提取能力信息：

1. **extensions.json** → `identifier.id` + `version`，作为"已安装扩展"列表
2. **mcp_config.json** → MCP 服务器名，作为"MCP 服务"列表
3. **skills/** → `name` + `description`（只读提取，不修改）

## 安全约束

| 操作 | 是否允许 |
|------|---------|
| 读取 extensions.json | ✅ |
| 读取 mcp_config.json | ✅ |
| 读取 skills/ | ✅ |
| 修改 extensions.json | ❌ **绝不**（会破坏扩展加载） |
| 修改 mcp_config.json | ❌ V1 不修改 |
| 修改 skills/ | ❌ V1 不修改 |
| 修改 pbtxt 状态文件 | ❌ **绝不** |
