---
name: ikevss-zh-friendly
description: "批量汉化 AI 工具的 skills/agents/commands/MCP 的描述文本（不是翻译代码注释或项目文档），扫描环境中已安装的技能、agent 和 MCP（不是配置或调试它们），并更新 CLAUDE.md 中的能力清单（不是修复 CLAUDE.md 格式）。当用户说看不懂英文技能描述、想翻译技能成中文、想知道环境里有哪些工具、想更新能力清单、想推荐安装 agent、想补充缺失的 description 时触发。即使用户没有明确说'汉化'，只要他们在讨论技能/工具/agent/MCP 的描述文本翻译或环境能力盘点，就应该触发。注意：不适用于翻译代码注释、编写翻译 API、调试 agent system prompt、配置 MCP 服务器、创建新技能。"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# ikevss-zh-friendly

让 AI 工具说流利中文，让中文用户用得更顺手。

## 资源路径

本技能的参考文件位于 `~/.claude/skills/ikevss-zh-friendly/` 下，按需用 Read 工具加载：

- **适配器**（各工具的扫描/汉化规则）：`references/{tool}-adapter.md`
  - 可用：`claude-code`、`codex`、`opencode`、`antigravity`、`trae`、`openclaw`、`skill-only`
- **跨工具汉化规则**（各工具写入时的特定规则）：`references/zh-rules.md`
- **Agent 安装规则**（从仓库安装人设到各工具）：`references/agent-install.md`
- **术语表**（翻译风格统一）：`references/glossary.md`
- **模板**（能力清单格式）：`assets/capability-section.md`

## 子命令路由

根据用户传入的 args 参数执行不同流程：

- **args 包含 "zh"** → 执行汉化流程（见下方"§ 汉化流程"）
- **args 包含 "scan"** → 执行能力扫描流程（见下方"§ 扫描流程"）
- **无 args 或 args 包含 "help"** → 显示以下帮助信息，然后停止：

```
ikevss-zh-friendly — 让 AI 工具说流利中文

用法：
  /ikevss-zh-friendly zh    批量汉化 skills/commands/agents 的 description
  /ikevss-zh-friendly scan  扫描环境能力，更新 CLAUDE.md 能力清单

zh 选项：
  --scope <工具>     claude-code | codex | opencode | trae | openclaw | cursor | windsurf | all（默认 all）
  --dry-run          只预览不执行（默认行为）
  --confirm          确认执行
  --overlay          非破坏性汉化：新增 description_zh 字段，保留英文原版 description
  --include-shared   包含共享池 ~/.agents/skills/
  --no-fill-missing  跳过补充缺失 description

scan 选项：
  --scope <工具>        claude-code | codex | opencode | antigravity | trae | openclaw | all（默认 all）
  --suggest-agents      启用人设推荐
  --no-suggest          禁用人设推荐
  --include-shared      包含共享池能力
```

---

## § 汉化流程（zh）

当 args 包含 "zh" 时，按以下流程执行。

### 前置检查

1. 解析 args 中的选项：`--scope`、`--dry-run`、`--confirm`、`--overlay`、`--include-shared`、`--no-fill-missing`
2. 如果 `--dry-run` 和 `--confirm` 都未指定，默认 `--dry-run`
3. 确定扫描范围：
   - `--scope all`（默认）：Claude Code + Codex + OpenCode + TRAE Work + OpenClaw + Cursor + Windsurf + WorkBuddy + 共享池（如有 --include-shared）
   - `--scope claude-code`：仅 Claude Code
   - 其他：按指定工具

### 第一步：加载适配器并扫描

按 scope 逐工具执行：

1. 用 Read 工具读取对应适配器文件（路径见上方"资源路径"）：
   - Claude Code：`references/claude-code-adapter.md`
   - Codex：`references/codex-adapter.md`
   - OpenCode：`references/opencode-adapter.md`
   - TRAE Work：`references/trae-adapter.md`
   - Antigravity：`references/antigravity-adapter.md`（只读扫描）
   - OpenClaw：`references/openclaw-adapter.md`
   - 纯 Skill 平台（Cursor/Windsurf/WorkBuddy/变体）：`references/skill-only-adapter.md`

2. 按适配器中定义的"扫描路径"，用 Glob 找到所有目标文件

3. 对每个文件用 Read 读取 frontmatter，分类为：
   - **情况 A**：已有英文 description → 标记"待汉化"
   - **情况 B**：无 description，正文有内容 → 标记"待生成"
   - **情况 C**：无 description，正文无内容 → 标记"跳过"
   - **情况 D**：description 中文字符占比 > 50% → 标记"已是中文，跳过"
   - **情况 E**：文件是符号链接（`test -L`）→ 标记"需确认"
   - **情况 F**：存在 `_meta.json` → 标记"marketplace 来源，有更新风险"

4. 输出扫描摘要：
   ```
   扫描完成：
   - 待汉化：N 个
   - 待生成 description：N 个
   - 已是中文（跳过）：N 个
   - 符号链接（需确认）：N 个
   - marketplace 来源（有风险）：N 个
   - 跳过（无内容）：N 个
   ```

### 第二步：备份

1. 生成备份目录名：`~/.claude/backup-i18n-{YYYYMMDD-HHmmss}/`（用 Bash `date` 命令获取时间，格式中不用冒号）
2. 检查 `~/.claude/backup-i18n-*` 目录数量，如果 ≥ 3，删除最旧的
3. 将所有"待汉化"和"待生成"的文件复制到备份目录，保持原始目录结构
4. 备份失败则立即停止，输出错误信息

### 第三步：汉化 / 生成

如果 `--dry-run` 模式，只输出预览不执行修改。预览格式：
```
[预览] 将修改以下文件：
  ✓ [SKILL]   ~/.claude/skills/example/SKILL.md
    description: "Create mermaid diagrams" → "生成 Mermaid 图表 | 画流程图、时序图、mermaid、diagram"
  ✓ [COMMAND] ~/.claude/commands/deploy.md
    description: "Deploy the application" → "部署应用 | 帮我发布、上线、deploy、推代码"
  ✓ [AGENT]   ~/.claude/agents/Explore.md
    description: "General-purpose agent..." → "通用探索智能体 | 浏览代码、搜索文件、explore、find"
```

如果 `--confirm` 模式，逐文件执行：

**Agents 特殊处理：**
Agent 文件的正文是精心调校的 System Prompt，改动任何字符都可能改变 agent 的行为模式。
所以只翻译 frontmatter 中面向人类的 description 和 when_to_use，正文保持原样——
这部分是给模型看的指令，不是给人看的说明。
修改后用逐字符比对确认正文完全未变，有变化立即从备份恢复。
1. 读取术语表 `references/glossary.md` 作为术语参考
2. 翻译 description：扩写式中文 + ` | ` + 英文触发关键词
3. 如存在 `when_to_use`、`argument-hint`，一并翻译
4. 用 Edit 工具最小化 diff 修改，只改 description 值，不动其他字段
5. 如有 Markdown 正文需要翻译，逐段翻译自然语言部分，保留代码块、路径、URL 不动

**翻译风格要求：**
- 纯翻译 ❌："生成 Mermaid 图表"
- 扩写式 ✅："生成 Mermaid 图表 | 画流程图、时序图、架构图、mermaid、diagram、chart"
- 句式：动词 + 对象 + 补充说明
- 口语触发词覆盖用户最可能说的 3-5 种说法
- 标点统一：中文标点

**生成 display_name（中文显示名）：**
对于没有 `display_name` 字段的 skill，根据 description 生成一个简短的中文显示名（3-10 个字）：
- 格式：`display_name: "中文显示名"`，写入 frontmatter
- 不影响 `name` 字段（保持英文 kebab-case）
- 中文显示名用于 / 斜杠菜单中的人类可读展示

**--overlay 模式（非破坏性汉化）：**
当用户传了 `--overlay` 参数时，不覆盖原 `description`，而是在 frontmatter 中新增 `description_zh` 字段：
```yaml
---
name: example-skill
description: "Generate mermaid diagrams"  # 英文原版保留
description_zh: "生成 Mermaid 图表 | 画流程图、时序图、mermaid、diagram"  # 中文新增
---
```
这样 marketplace 更新不会覆盖中文，工具切换时可回退到英文。但 Claude Code 不会自动识别 `description_zh`，实际触发仍依赖原 `description`。

**跨工具汉化规则：**
各工具的特定写入规则（排除路径、额外字段、特殊限制）见 `references/zh-rules.md`。
按 scope 加载对应工具的规则章节。

**情况 B（补充缺失 description）：**
1. 读取文件正文，提取：标题 + 第一段 + 关键章节名
2. 生成中文 description + ` | ` + 英文触发关键词
3. 用 Edit 工具在 frontmatter 中插入 `description:` 字段
4. 验证 frontmatter 的 `---` 开闭配对完整
5. 如果生成的 description 包含英文引号或冒号，用双引号包裹整个值以避免 YAML 解析错误

**情况 E（符号链接）：**
1. 输出符号链接信息：`⚠️ [符号链接] path -> target`
2. 提示："修改源文件会影响所有引用它的工具（Claude Code、OpenClaw 等）"
3. 提供三个选项：
   - A）修改源文件（一次汉化，所有工具受益，但会破坏锁文件中的哈希校验）
   - B）跳过（保持原样，默认）
   - C）断开链接，创建独立副本后汉化（安全但需要额外磁盘空间）
4. 选项 C 的实现：`cp` 符号链接目标文件到链接位置（覆盖链接），汉化副本，提示"已断开并创建独立副本"

**情况 F（marketplace 来源）：**
1. 提示："此 skill 来自 marketplace，汉化内容可能在更新时被英文原版覆盖"
2. 继续汉化（让用户知情后决定）

### 第四步：校验

每个文件修改后立即校验，因为一个被破坏的 frontmatter 会导致整个技能无法加载：
1. frontmatter 仍是合法 YAML（检查 `---` 开闭配对）
2. `name` 字段值未变（name 是技能加载器的索引 key，变了等于删除技能）
3. 工具名、权限规则、参数占位符未变（这些是机器解析的指令，翻译会破坏执行）
4. description 长度 ≤ 1024 字符（Codex 的 hard limit，超长会被截断）
5. Agents 文件：提取正文与备份逐字符比对
   - 完全一致 → 通过
   - 有变化 → 立即从备份恢复，标记为失败

校验失败时：
- 从备份恢复该文件
- 标记为 `✗ [FAILED]`，记录原因
- 继续处理下一个文件

### 第五步：输出报告

```
处理完成。

[汇总]
- 汉化成功：N 个
- 生成 display_name：N 个
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

[失败文件]
- path：原因

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

## § 扫描流程（scan）

当 args 包含 "scan" 时，按以下流程执行。

### 前置检查

1. 解析 args 中的选项：`--scope`、`--suggest-agents`、`--no-suggest`、`--target`、`--include-shared`
2. `--target` 默认为 `~/.claude/CLAUDE.md`
3. 确定扫描范围（默认 all）

### 第一步：扫描全部工具能力

按 scope 逐工具执行只读扫描。加载对应适配器文件获取扫描路径。

**Claude Code 扫描项：**
1. 用 Glob `~/.claude/skills/*/SKILL.md` 找到所有 skill，用 Read 提取 name + description
2. 用 Glob `~/.claude/commands/*.md` 找到所有 command，提取 name + description
3. 用 Glob `~/.claude/agents/*.md` 找到所有 agent，提取 name + description
4. 用 Read 读取 `~/.claude/settings.json`，提取 `mcpServers` 的 key 列表

**Codex 扫描项：**
1. 用 Glob `~/.codex/skills/*/SKILL.md`，排除 `.system/`，提取 name + description
2. 用 Glob `~/.codex/agents/*.toml`，提取 name + description（TOML 格式）
3. agent 总数计入能力清单

**OpenCode 扫描项：**
1. 用 Glob `~/.config/opencode/skills/*/SKILL.md`，提取 name + description
2. 用 Glob `~/.config/opencode/command/*.md`，提取 name + description
3. 用 Glob `~/.config/opencode/agents/*.md`，提取 name + description
4. 用 Read 读取 `~/.config/opencode/opencode.json`，提取 `mcp` 服务器名和 `provider` 名
5. 读取 `~/.opencode/opencode.json`（如存在），提取项目级 MCP 和 agent 配置

**Antigravity 扫描项：**
1. 用 Read 读取 `~/.antigravity/extensions/extensions.json`，提取扩展名列表
2. 用 Read 读取 `~/.gemini/antigravity/mcp_config.json`，提取 MCP 服务器名
3. 用 Glob `~/.gemini/antigravity/skills/*/SKILL.md`（如存在），提取 name + description

**TRAE Work 扫描项：**
1. 用 Glob `~/.trae/skills/*/SKILL.md`，提取 name + description（99 个，格式与 Claude Code 一致）
2. 用 Glob `~/.trae/user_rules/*.md`，提取文件名（其中 agent-*.md 计入 agent 统计）
3. 用 Glob `~/.trae/mcps/*.json`，提取 MCP 服务器名
4. 用 Read 读取 `~/.trae/extensions/extensions.json`，提取扩展名列表
5. 如果 `~/.trae-cn/skills/` 存在，一并扫描

**CLI 工具扫描项：**
CLI 工具是 AI 环境中重要的可执行能力。用 Bash `which` 或 `command -v` 检测以下常用 CLI 是否存在：
  - 版本管理：`git`、`gh`（GitHub CLI）
  - 包管理：`npx`、`npm`、`pnpm`、`yarn`、`pip`、`cargo`
  - 运行时：`node`、`python`、`python3`、`ruby`
  - 容器/云：`docker`、`kubectl`、`aws`、`gcloud`
  - 构建/测试：`make`、`cmake`、`go`、`rustc`
  - 文档/其他：`pandoc`、`ffmpeg`、`curl`、`jq`
检测方式：用 Bash 执行 `command -v {tool} && echo "✅ {tool}" || echo ""` 批量检测
每个找到的 CLI 工具记录 name（如 `git`、`node`、`docker`）

**OpenClaw 扫描项：**
1. 用 Glob `~/.openclaw/skills/*/SKILL.md` 和 `~/.openclaw/skills/*.md`，提取 name + description
2. 用 Glob `~/.openclaw/workspace/skills/*/SKILL.md`，提取 name + description
3. 统计符号链接数量
4. 识别含 agent 角色的 skill（sessions_spawn 机制），计入 agent 数量

**其他纯 Skill 平台扫描项（使用 skill-only 适配器）：**
1. 用 Read 读取 `references/skill-only-adapter.md` 获取路径列表
2. 对每个路径检测目录是否存在（`test -d`）
3. 存在的目录扫描 `*/SKILL.md` 和 `*.md`，提取 name + description
4. 路径列表：`~/.cursor/skills/`、`~/.codeium/windsurf/skills/`、`~/.gemini/skills/`、`~/.workbuddy/skills/`
5. OpenClaw 变体（按需检测）：`~/.nanobot/skills/`、`~/.picoclaw/skills/`、`~/.memubot/skills/`、`~/.maxclaw/skills/`、`~/.copaw/skills/`、`~/.autoclaw/skills/`、`~/.kamiclaw/skills/`、`~/.qclaw/skills/`、`~/.easyclaw/skills/`

### 第二步：生成能力清单

1. 用 Read 读取模板：`assets/capability-section.md`
2. 按模板格式，用第一步收集的数据填充各工具段落
3. 日期替换为当前日期（用 Bash `date +%Y-%m-%d`）
4. 每个工具只列 name 列表（不加 description，避免过长）
5. 能力清单分组：
   - Claude Code：Skills / Agents / Commands / MCP
   - Codex：Skills / Agents
   - OpenCode：Skills / Agents / MCP / Providers
   - Antigravity：Extensions / MCP
   - TRAE Work：Skills / Agents(user_rules) / MCP / Extensions
   - OpenClaw / 变体：Skills（含检测到的变体）
   - 其他平台：Cursor / Windsurf / WorkBuddy(Agents:SOUL.md)（各自独立一行）
   - CLI 工具：git, node, python, docker, ...（检测到的列表）
6. 检查总行数：
   - ≤ 50 行 → 完整保留
   - > 50 行 → 每个工具最多保留 5 个 name，标注"共 N 个"
7. 用 `<!-- CAPABILITY_LIST_START -->` 和 `<!-- CAPABILITY_LIST_END -->` 包裹

### 第三步：更新 CLAUDE.md

1. 用 Read 读取 `--target` 指定的文件（默认 `~/.claude/CLAUDE.md`）
2. 检查是否已有 `<!-- CAPABILITY_LIST_START -->` 和 `<!-- CAPABILITY_LIST_END -->` 标记
3. 如果有：用 Edit 工具精准替换两个标记之间的内容，其余不动
4. 如果没有：在文件末尾追加能力清单章节
5. 输出 diff 预览：
   ```
   [预览] 将在 ~/.claude/CLAUDE.md 中更新能力清单章节：
   + 新能力清单（前 10 行）...
   总计 N 行。确认写入？
   ```
6. 等待用户确认后用 Write 或 Edit 写入
7. 写入后提示："✅ 能力清单已更新。新开会话时生效（当前会话不会热重载）。"

### 第四步：人设推荐（条件触发）

判断是否触发人设推荐：
- 如果 `--no-suggest` → 跳过
- 如果 `--suggest-agents` → 强制触发
- 否则：统计所有工具的 agent 数量总和，< 5 个则触发
  - `~/.claude/agents/*.md`
  - `~/.config/opencode/agents/*.md`（如存在）
  - `~/.codex/agents/*.toml`（如存在）
  - `~/.trae/user_rules/agent-*.md`（如存在）
  - `~/.workbuddy/SOUL.md`（如存在且非模板）
  - `~/.openclaw/skills/` 中含 agent 角色的 skill

触发后执行：

1. **推断用户领域**
   - 用 Read 读取 `~/.claude/CLAUDE.md`，提取工作相关关键词
   - 综合已装 skills 的 description 词频
   - **扫描对话历史**：用 Glob 扫描 `~/.claude/projects/`、`~/.claude/sessions/` 下的会话记录，提取高频任务主题（如金融分析、代码开发、文档写作等）
   - **扫描记忆系统**：读取 `~/.claude/.remember/now.md`、`recent.md`、`core-memories.md`（如存在），提取反复出现的工作模式
   - **扫描全局偏好**：读取 `~/.claude/settings.json`，了解配置了哪些 MCP 和 plugin，反推使用场景
   - 综合以上多源信号，列出 3 个最可能的工作领域（附推断依据）

2. **交互式询问**
   - 输出推断依据和选项：
     ```
     综合你的 CLAUDE.md、已装 skills（如 china-stock-analysis, meeting-prep-brief）、
     对话记录高频主题（如财经数据、路演报告）和记忆系统中的工作模式，
     推测你的工作领域可能是：
       A) 金融投资
       B) 内容创作与营销
       C) 软件工程
       D) 以上都不是，我来手动选择
     ```
   - 等待用户选择

3. **展示推荐清单**
   - 各工具的 agent 安装路径和格式差异见 `references/agent-install.md`
   - 从 agency-agents-zh 仓库 GitHub Raw URL 获取对应部门的 agent 列表
   - 仓库结构：`https://raw.githubusercontent.com/ikevss/agency-agents-zh/main/{部门目录}/{agent名}.md`
   - 展示推荐（每个部门 3-5 个，总数上限 15 个）：
     ```
     推荐安装以下 agent 人设（来自 ikevss/agency-agents-zh）：
     
     金融部：
       1. 金融数据分析师 - 专业财务建模与数据分析
       2. 投资研究员 - 深度研究与投资建议
     
     营销部：
       3. 小红书运营 - 内容运营与社区管理
       4. 公众号编辑 - 深度内容创作
     
     是否确认安装以上 agent？[全部安装 / 逐个选择 / 取消]
     ```

4. **安装**
   - 用 Bash `curl` 或 WebFetch 从 GitHub Raw 下载对应的 .md 文件内容
   - 下载 URL 格式：`https://raw.githubusercontent.com/ikevss/agency-agents-zh/main/{path}`
   - 校验下载内容的 frontmatter 至少包含 `name` 和 `description`
   - 检测本机可用的安装目标，按优先级逐个安装：

   | 工具 | 安装路径 | 格式 | 检测方式 |
   |------|---------|------|---------|
   | Claude Code | `~/.claude/agents/{name}.md` | Markdown（直接写入） | `test -d ~/.claude/agents/` |
   | TRAE Work | `~/.trae/user_rules/agent-{name}.md` | Markdown（写入 user_rules） | `test -d ~/.trae/user_rules/` |
   | Codex | `~/.codex/agents/{name}.toml` | TOML（需转换，见 agent-install.md） | `test -d ~/.codex/agents/` |
   | OpenCode | `~/.config/opencode/agents/{name}.md` | Markdown + `mode: subagent` | `test -d ~/.config/opencode/agents/` |
   | WorkBuddy | `~/.workbuddy/` 下的 `SOUL.md` + `IDENTITY.md` | 人设写入顶层文件（见 agent-install.md） | `test -f ~/.workbuddy/SOUL.md` |
   | OpenClaw | `~/.openclaw/skills/{name}/SKILL.md` | 内嵌为 skill（sessions_spawn 机制，见 agent-install.md） | `test -d ~/.openclaw/skills/` |

   - 每个目标检查同名文件是否已存在（存在则跳过并提示）
   - WorkBuddy 的 `SOUL.md` 是全局人格文件，覆盖前必须备份原文件
   - OpenCode：在 frontmatter 中额外加 `mode: subagent` 字段
   - Codex：TOML 格式转换规则见 `references/agent-install.md`
   - TRAE Work：写入 `user_rules/` 目录（TRAE 没有独立 agents 目录，user_rules 是最接近的替代）
   - 安装完成后提示每个工具安装了多少个 agent

5. **更新能力清单**
   - 重跑第二步和第三步，将新安装的 agent 纳入能力清单
   - 提示："可运行 /ikevss-zh-friendly zh 将新 agent 的 description 汉化"

### 第五步：输出变更摘要

```
scan 完成。

[能力统计]
- Claude Code: Skills N / Agents N / Commands N / MCP N
- Codex: Skills N / Agents N
- OpenCode: Skills N / Agents N / MCP N / Providers N
- Antigravity: Extensions N / MCP N
- TRAE Work: Skills N / Agents(user_rules) N / MCP N / Extensions N
- OpenClaw / 变体: Skills N / Agents(skill内嵌) N
- 其他平台: Cursor N / Windsurf N / WorkBuddy N
- CLI 工具: N 个（{name1}, {name2}, ...）

[能力清单更新]
- 写入：~/.claude/CLAUDE.md
- 行数：N 行
- 下次新会话生效

[新安装的 agent]（如有）
- ~/.claude/agents/xxx.md
- ~/.claude/agents/yyy.md
```
