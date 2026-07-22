# 能力清单章节模板

> 本模板定义写入 CLAUDE.md 的能力清单章节格式。
> scan 子命令生成能力清单时，按此模板填充内容。

## 格式规范

- 用 HTML 注释标记作为边界，便于后续精准替换
- 总计不超过 **50 行**（含标题、分隔符、注释标记）
- 超过 50 行时：每个工具只列前 N 个 name，标注"共 M 个，仅列出前 N 个"
- 只列 name，不加 description（description 在 skill/agent 定义文件中，模型会自动读取）
- 按工具分组，每组用 `###` 标题

## 模板

```
<!-- CAPABILITY_LIST_START -->
## 当前 AI 环境能力清单
> 自动生成于 {DATE}，运行 /ikevss-zh-friendly scan 刷新

### Claude Code
- Skills ({N}): {name1}, {name2}, ...
- Agents ({N}): {name1}, {name2}, ...
- Commands ({N}): {name1}, {name2}, ...
- MCP ({N}): {name1}, {name2}, ...

### Codex
- Skills ({N}): {name1}, {name2}, ...
- Agents ({N}): {name1}, {name2}, ...

### OpenCode
- Skills ({N}): {name1}, {name2}, ...
- Agents ({N}): {name1}, {name2}, ...
- MCP ({N}): {name1}, {name2}, ...
- Providers ({N}): {name1}, {name2}, ...

### Antigravity
- Extensions ({N}): {name1}, {name2}, ...
- MCP ({N}): {name1}, {name2}, ...

### TRAE Work
- Skills ({N}): {name1}, {name2}, ...
- Agents(user_rules) ({N}): {name1}, {name2}, ...
- MCP ({N}): {name1}, {name2}, ...
- Extensions ({N}): {name1}, {name2}, ...

### OpenClaw / 变体
- Skills ({N}): {name1}, {name2}, ...
- Agents(skill内嵌) ({N}): {name1}, {name2}, ...

### 其他平台
- Cursor ({N}): {name1}, {name2}, ...
- Windsurf ({N}): {name1}, {name2}, ...
- WorkBuddy(Agents:SOUL.md) ({N}): {name1}, {name2}, ...

### CLI 工具
- 已检测 ({N}): {name1}, {name2}, ...
<!-- CAPABILITY_LIST_END -->
```

## 行数控制

如果某个工具的 skill 列表过长，使用以下截断格式：
```
- Skills (90+): meeting-prep-brief, china-stock-analysis, ...（仅列出前 20 个）
```

如果总计超过 50 行，按以下优先级保留：
1. Claude Code（主运行环境，全部保留）
2. 其他工具：每个最多保留 5 个 name

## 使用说明

scan 子命令按以下步骤使用本模板：
1. 按模板格式生成内容
2. 用 `{DATE}` 替换为当前日期
3. 用实际的 name 列表替换占位符
4. 检查总行数是否 ≤ 50，超过则截断
5. 输出预览给用户确认
6. 用户确认后写入 CLAUDE.md
