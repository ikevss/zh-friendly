# 纯 Skill 平台通用适配器

> 本文件覆盖所有只有 `skills/` 目录、没有 agents/commands/MCP/plugins 的平台。
> 这些平台共用同一套扫描和汉化规则，仅路径不同。

## 覆盖平台与路径

| 平台 | Skills 路径 | 本机状态 |
|------|-----------|---------|
| OpenClaw 变体 - NanoBot | `~/.nanobot/skills/` | 按需检测 |
| OpenClaw 变体 - PicoClaw | `~/.picoclaw/skills/` | 按需检测 |
| OpenClaw 变体 - memUBot | `~/.memubot/skills/` | 按需检测 |
| OpenClaw 变体 - MaxClaw | `~/.maxclaw/skills/` | 按需检测 |
| OpenClaw 变体 - CoPaw | `~/.copaw/skills/` | 按需检测 |
| OpenClaw 变体 - AutoClaw | `~/.autoclaw/skills/` | 按需检测 |
| OpenClaw 变体 - KimiClaw | `~/.kamiclaw/skills/` | 按需检测 |
| OpenClaw 变体 - QClaw | `~/.qclaw/skills/` | 按需检测 |
| OpenClaw 变体 - EasyClaw | `~/.easyclaw/skills/` | 按需检测 |
| Cursor | `~/.cursor/skills/` | ✅ 存在 |
| Windsurf | `~/.codeium/windsurf/skills/` | ✅ 存在（70+ 个） |
| Gemini CLI | `~/.gemini/skills/` | ✅ 存在 |
| WorkBuddy | `~/.workbuddy/skills/` | ✅ 存在（14 个） |

## 扫描策略

扫描时按顺序检测每个路径：
1. 检查目录是否存在（`test -d`）
2. 不存在 → 跳过，不报错
3. 存在 → 扫描 `*/SKILL.md` 和 `*.md`（扁平文件）
4. 后续处理与 Claude Code adapter 一致

## 排除规则

- 非 Markdown 文件
- `_meta.json`、`config.json`、`README.md` 等辅助文件
- 锁文件（`*lock*`、`*lock*.json`）

## 汉化规则

与 Claude Code adapter 完全一致：
- 可翻译：`description`、`when_to_use`、`argument-hint`、Markdown 正文自然语言
- 绝不翻译：`name`、工具名、权限、参数、路径、URL、代码块
- 扩写式翻译：中文描述 + ` | ` + 英文触发词

### 符号链接检测
- 与 OpenClaw adapter 一致：检测到符号链接时提示用户选择

### 非标准 frontmatter 兼容
- 部分 OpenClaw 变体的 skill 可能使用非标准 frontmatter
- 解析策略同 OpenClaw adapter：先试标准 `---`，失败则 fallback 到裸 `key: value`

## scan 子命令说明

这些平台在能力清单中统一归入"其他平台"分组：
```
### 其他平台
- Cursor (1): officecli
- Windsurf (70+): lark-*, longbridge-*, ...
- WorkBuddy (14): a-stock-data, canvas-design, ...
```

OpenClaw 变体如果存在，归入"OpenClaw / 变体"分组。
