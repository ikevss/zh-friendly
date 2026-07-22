# Codex 适配器

> 本文件定义 OpenAI Codex 平台的扫描路径、汉化规则和排除规则。
> 与 Claude Code 的差异用 ⚠️ 标注。

## 扫描路径

### Skills
- 用户级：`~/.codex/skills/*/SKILL.md`

### Agents
- 用户级：`~/.codex/agents/*.toml`（⚠️ **TOML 格式，不是 Markdown**）

### Plugins
- 缓存：`~/.codex/plugins/cache/`（仅扫描，不修改）

## 排除规则

以下路径不扫描、不汉化：

1. `~/.codex/skills/.system/`（官方内置，含 imagegen、openai-docs、plugin-creator、review-agent、skill-creator、skill-installer，更新会覆盖）
2. `~/.codex/vendor_imports/`（deprecated，更新会覆盖）
3. `~/.codex/plugins/cache/`（运行时缓存，修改无意义）
4. `~/.codex/.codex-global-state.json`（Electron 状态文件，无描述文本）
5. `~/.codex/sessions/`（会话数据）
6. `~/.codex/.sandbox/`（沙箱日志）

## 汉化规则

### 可翻译字段（frontmatter）
- `description` — 主要触发字段，必须扩写式翻译
- `metadata.short-description` — ⚠️ **Codex 特有**，UI 芯片上显示的简短描述

### 绝不翻译的内容（除 Claude Code 通用列表外，额外增加）
- `license` 字段值
- `metadata` 中的 `version`、`author`、`risk_level`、`requires_login`、`default_install`、`requires_mcp`、`tier` 等技术标识符
- `quick_validate.py` 限制 `description` 最大 **1024 字符**（含空格），汉化后需检查长度

### Agents 特殊处理
- ⚠️ Codex agent 文件是 **TOML 格式**，结构为：
  ```toml
  name = "AgentName"
  description = "..."
  developer_instructions = "\n# Agent Personality\n..."
  ```
- **V1 不支持 Codex agent 写入**（TOML 格式解析/序列化风险高）
- scan 子命令可以只读扫描并提取 `name` + `description` 纳入能力清单

### 已有中文 skill 的处理
- 部分 skill 的 description 已经是中文（如 `devils-advocate`、`longbridge-*` 系列）
- 扫描时检测：如果 description 中文字符占比 > 50%，标记为"已是中文"并跳过
- 但 `longbridge-*` 等中英混合的 skill 仍需检查是否需要补充英文触发词

### marketplace 更新风险
- Codex 正在从 skills 迁移到 plugins 体系
- 用户级 skills（`~/.codex/skills/`）汉化后，后续 Codex 更新不会覆盖
- 但如果通过 `skill-installer` 重新安装，会覆盖汉化内容
