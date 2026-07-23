---
name: ikevss-zh-friendly
description: "你装了一堆 AI 技能，但名字全是英文，不知道哪个是干嘛的？这个技能帮你一键翻译成中文，一眼看懂每个技能是干嘛的。扫描环境后生成能力清单，让 AI 知道你有哪些技能和工具，多智能体协作时每个 agent 都能看清全局。当用户没设置 agent 或 agent 太少时，根据已有的技能和工具推荐合适的人设。支持 Claude Code、TRAE Work、Codex、Cursor 等 13+ AI 工具。默认只预览不改文件，确认了才动手。"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# ikevss-zh-friendly

你装了一堆 AI 技能，但名字全是英文，不知道哪个是干嘛的？
这个技能帮你一键翻译成中文，一眼看懂。

---

## 触发条件（出现任一即激活）

用户说"这些英文技能看不懂" / "帮我翻译一下技能" / "汉化"
用户说"我装了哪些技能？" / "AI 环境里有什么工具？"
用户说"agent 太少了，推荐几个" / "有没有适合我的智能体"
用户说"有些技能连描述都没有" / "帮我补上 description"
用户说"更新一下能力清单" / "扫描一下环境"
用户想查看某个工具的汉化效果（加 `--dry-run`）

---

## 第一阶段：扫描环境（自动执行）

用户触发后，自动扫描本机已安装的 AI 工具。

### 扫描路径

| 工具 | 路径 | 格式 |
|------|------|------|
| Claude Code | `~/.claude/skills/*/SKILL.md` + `commands/` + `agents/` | Markdown |
| TRAE Work | `~/.trae/skills/*/SKILL.md` + `user_rules/` | Markdown |
| Codex | `~/.codex/skills/*/SKILL.md` + `agents/*.toml` | Markdown + TOML |
| OpenCode | `~/.config/opencode/skills/*/SKILL.md` + `command/` | Markdown |
| Cursor | `~/.cursor/skills/*/SKILL.md` | Markdown |
| Windsurf | `~/.codeium/windsurf/skills/*/SKILL.md` | Markdown |
| WorkBuddy | `~/.workbuddy/skills/*` | Markdown |
| OpenClaw | `~/.openclaw/skills/*/SKILL.md` | Markdown |
| Antigravity | `~/.gemini/antigravity/skills/` + `extensions.json` | 只读扫描 |

### 分类每个文件

扫描完成后，将每个文件分类为：

| 情况 | 处理 |
|------|------|
| A：已有英文 description | 标记"待汉化" |
| B：无 description，正文有内容 | 标记"待生成" |
| C：无 description，正文无内容 | 标记"跳过" |
| D：description 中文字符 > 50% | 标记"已是中文，跳过" |
| E：文件是符号链接 | 标记"需确认" |
| F：存在 `_meta.json` | 标记"marketplace 来源，有更新风险" |

### 输出扫描摘要

```
扫描完成：
- 待汉化：N 个
- 待生成 description：N 个
- 已是中文（跳过）：N 个
- 符号链接（需确认）：N 个
- marketplace 来源（有风险）：N 个
- 跳过（无内容）：N 个
```

---

## 第二阶段：汉化 / 生成（用户确认后执行）

### 情况 A：汉化已有 description

不是机翻。每个描述里加上 3-5 个你可能会说的中文口语表达。

**示例：**

| 英文原版 | 翻译后 |
|---------|--------|
| Generate mermaid diagrams for documentation | 生成 Mermaid 图表 \| 画流程图、时序图、架构图、mermaid、diagram |
| Deploy the application to production | 部署应用 \| 帮我发布、上线、deploy、推代码 |
| Code review skill for pull requests | 代码审查 \| review、检查代码、帮我看看代码、code review |

**翻译规则：**
- 主描述：中文自然语言
- 保留英文触发关键词（用 `|` 分隔）
- 加上口语化表达（"帮我..."、"画个..."、"做个..."）
- 句式统一：动词 + 对象 + 补充说明
- 标点统一：中文标点

### 情况 B：生成缺失 description

从正文提取：标题 + 第一段 + 关键章节名，生成中文 description + 英文触发词。

**示例：**

```markdown
原文正文：
# Executive Summary Generator
Generates C-suite ready executive summaries from lengthy reports...

生成 description：
"高管摘要生成 | 给老板汇报、executive summary、C-suite、总结报告"
```

### 情况 E：符号链接处理

检测到符号链接时，暂停并给用户三个选项：

| 选项 | 说明 |
|------|------|
| A | 修改源文件（一次汉化，所有工具受益，但会破坏锁文件哈希） |
| B | 跳过（保持原样） |
| C | 断开链接，创建独立副本后汉化（安全但需要额外磁盘空间） |

### 情况 F：marketplace 来源

存在 `_meta.json` 时提示用户：
> "此 skill 来自 marketplace，汉化内容可能在更新时被英文原版覆盖。建议用 `--overlay` 模式。"

---

## 第三阶段：校验（每个文件修改后立即执行）

每个文件修改后立即校验：

| 检查项 | 通过条件 |
|--------|---------|
| frontmatter 合法性 | `---` 开闭配对完整 |
| `name` 字段未变 | 与备份一致 |
| 工具名/权限/参数未变 | 与备份一致 |
| description 长度 | ≤ 1024 字符（Codex 限制） |
| Agent 正文 | 逐字符比对，完全一致 |

校验失败时：
- 立即从备份恢复该文件
- 标记为 `✗ [FAILED]`
- 继续处理下一个文件

---

## 第四阶段：输出报告

```
处理完成。

[汇总]
- 汉化成功：N 个
- 生成 description 成功：N 个
- 跳过（已是中文）：N 个
- 跳过（无内容）：N 个
- 失败：N 个
- 备份目录：~/.claude/backup-i18n-YYYYMMDD-HHmmss/

[逐文件日志]
✓ [SKILL]   ~/.claude/skills/example/SKILL.md
✓ [COMMAND] ~/.claude/commands/deploy.md
✓ [AGENT]   ~/.claude/agents/Explore.md  # 仅 frontmatter，System Prompt 未改变
⚠ [SKIP]    ~/.claude/skills/devils-advocate/SKILL.md：已是中文
✗ [FAILED]  ~/.claude/skills/broken/SKILL.md：frontmatter 解析失败

[marketplace 风险提示]
- 以下 skill 来自 marketplace，汉化内容可能在更新时被覆盖：
  - codex/skills/example（有 _meta.json）

[建议验证]
1. 输入 / 查看 slash command 菜单是否正常
2. 用一个中文自然语言任务触发已汉化的 skill，确认行为无变化
3. 检查 Agent 文件，确认正文 System Prompt 完全未变
4. 如有异常，从备份目录恢复对应文件
```

---

## 第五阶段：能力扫描（scan 子命令）

当用户传了 `--suggest-agents` 或 agents 数量 < 5 时触发。

### 推断用户领域

读取以下信息推断用户工作领域：
- `~/.claude/CLAUDE.md` 中的工作关键词
- 已装 skills 的 description 词频
- `~/.claude/.remember/` 下的记忆文件（如存在）
- `~/.claude/settings.json` 中的 MCP/plugin 配置

### 推荐 agent 人设

从 [agency-agents-zh](https://github.com/ikevss/agency-agents-zh) 仓库按领域推荐：

| 用户领域 | 推荐部门 |
|---------|---------|
| 金融投资 | `finance/`、`specialized/` |
| 内容创作与营销 | `marketing/`、`paid-media/` |
| 软件工程 | `engineering/`、`testing/`、`security/` |
| 产品设计 | `design/`、`product/` |
| 项目管理 | `project-management/`、`strategy/` |

每个部门推荐 3-5 个，总数上限 15 个。

### 安装

用户确认后：
1. 从 GitHub Raw 下载对应 .md 文件
2. 检查 `~/.claude/agents/{filename}.md` 是否已存在（存在则跳过）
3. 校验 frontmatter 至少含 `name` 和 `description`
4. 写入 `~/.claude/agents/{filename}.md`
5. 提示："✅ 已安装 N 个 agent。无需重启，立即可用。"

---

## 边缘场景处理

| 场景 | 策略 | 提示语 |
|------|------|--------|
| 目录不存在 | 跳过，不报错 | — |
| 目录存在但为空 | 跳过，不报错 | — |
| frontmatter 无法解析 | 跳过，标记失败 | "frontmatter 解析失败，已跳过" |
| 用户中断（Ctrl+C） | 保存已处理进度 | "已处理 N 个，下次继续" |
| 备份目录已存在 | 使用时间戳区分 | — |
| 备份失败 | 立即停止 | "备份失败，任务已停止" |
| marketplace 更新覆盖 | 提示用户 | "更新后请重新运行 zh" |
| 符号链接目标不存在 | 跳过，标记失败 | "符号链接目标不存在" |

---

## 质量检查清单（自检 5 条）

输出前必须检查：

- [ ] 每个汉化 description 包含中文描述 + 英文触发词
- [ ] 没有改原文件的 `name` 字段
- [ ] 没有改 Agent 文件的 System Prompt 正文
- [ ] marketplace 来源已提示更新风险
- [ ] 备份目录创建成功

---

## 完整示例

**用户输入：**
```
帮我把这些英文技能翻译成中文
```

**执行过程：**

1. 扫描 `~/.claude/skills/` 发现 15 个 skill
2. 分类：12 个待汉化，1 个已是中文，2 个无内容跳过
3. 备份到 `~/.claude/backup-i18n-20260723-140522/`
4. 逐文件汉化 + 校验
5. 输出报告

**输出：**
```
扫描完成：
- 待汉化：12 个
- 已是中文（跳过）：1 个
- 跳过（无内容）：2 个

处理完成。

[汇总]
- 汉化成功：12 个
- 失败：0 个
- 备份目录：~/.claude/backup-i18n-20260723-140522/

[逐文件日志]
✓ [SKILL]   ~/.claude/skills/brainstorming/SKILL.md
✓ [SKILL]   ~/.claude/skills/skill-creator/SKILL.md
✓ [SKILL]   ~/.claude/skills/find-skills/SKILL.md
...（共 12 条）
⚠ [SKIP]    ~/.claude/skills/devils-advocate/SKILL.md：已是中文

[建议验证]
1. 输入 / 查看 slash command 菜单是否正常
2. 用中文触发一个已汉化的 skill，确认行为无变化
```
