# 跨工具汉化规则

> zh 子命令对各工具执行写入时的具体规则。
> SKILL.md 的"第三步"按 scope 加载本文件获取工具特定逻辑。

## 通用规则（所有工具适用）

以下规则对所有工具生效，工具特定规则在各工具章节中补充：

- 扩写式翻译：中文描述 + ` | ` + 英文触发关键词
- 绝不翻译：name、目录名、工具名、权限规则、参数占位符、路径、URL、代码块
- 补充缺失 description：从正文提取标题+第一段+关键章节名，生成双语 description
- 修改前备份，修改后校验 frontmatter 合法性

---

## Claude Code（主运行环境，读写）

- 扫描路径：`~/.claude/skills/`、`commands/`、`agents/`
- 写入：直接用 Edit 工具修改 description 字段
- Agents：只改 frontmatter，正文 System Prompt 逐字符校验
- 跳过：plugins 目录、非 Markdown 文件、frontmatter 无法解析的文件

---

## Codex

- 扫描路径：`~/.codex/skills/*/SKILL.md`
- **排除**：`.system/`（官方内置，更新会覆盖）、`vendor_imports/`（deprecated）、`plugins/cache/`（运行时缓存）
- 额外可汉化字段：`metadata.short-description`（UI 简短描述）
- description 长度限制：≤ 1024 字符（`quick_validate.py` 硬限制）
- 已是中文检测：description 中文字符占比 > 50% 则跳过
- Agent 文件（`~/.codex/agents/*.toml`）：TOML 格式，V2 不支持写入，只读扫描

---

## OpenCode

- 扫描路径：`~/.config/opencode/skills/*/SKILL.md`、`~/.config/opencode/command/*.md`
- **绝不修改** `opencode.json`（启动时严格校验 JSON schema，任何改动都会导致启动失败）
- `model`、`category` 字段不翻译（是机器标识符）
- `triggers` 字段的值不翻译（触发词数组）
- marketplace 检测：存在 `_meta.json` → 提示更新可能覆盖
- Agent 目录：`~/.config/opencode/agents/`（复数）可写，`~/.config/opencode/agent/`（单数）**绝不碰**

---

## TRAE Work

- 扫描路径：`~/.trae/skills/*/SKILL.md`（99 个，格式与 Claude Code 同构）
- SKILL.md frontmatter 字段与 Claude Code 一致，汉化逻辑完全复用
- `~/.trae/user_rules/*.md`：用户自定义规则，自然语言部分可翻译
- **不修改**：`builtin_skills/`（内置）、`mcps/*.json`（MCP 配置）、`extensions/extensions.json`
- `~/.trae-cn/` 变体：如果存在，用相同规则处理 `skills/` 和 `user_rules/`

---

## OpenClaw

- 扫描路径：`~/.openclaw/skills/`、`~/.openclaw/workspace/skills/`
- **符号链接高风险**：`~/.openclaw/skills/` 下约 80 个符号链接指向 `~/.agents/skills/`，修改会影响所有工具
- 非标准 frontmatter 兼容：先试 `---` 分隔，失败则 fallback 到裸 `key: value` 行
- `model`、`category`、`triggers` 字段不翻译
- 锁文件 `.skills_store_lock.json` 不修改（哈希变化会破坏 marketplace 更新）

---

## 纯 Skill 平台（Cursor / Windsurf / WorkBuddy / 变体）

这些平台只有 `skills/` 目录，汉化逻辑与 Claude Code 一致：

| 平台 | 路径 |
|------|------|
| Cursor | `~/.cursor/skills/` |
| Windsurf | `~/.codeium/windsurf/skills/` |
| WorkBuddy | `~/.workbuddy/skills/` |
| OpenClaw 变体 | `~/.nanobot/skills/`、`~/.picoclaw/skills/` 等（按需检测） |

扫描时先检测目录是否存在，不存在则跳过。符号链接检测同 OpenClaw。
