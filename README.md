# ikevss-zh-friendly

> 装了一堆英文技能，看名字不知道是干嘛的。调用前还得先猜。
> 这个技能把它们全翻译成中文，一眼看懂。

<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License">
  <a href="https://www.skills.sh/"><img src="https://img.shields.io/badge/skills.sh-ikevss_zh_friendly-purple" alt="skills.sh"></a>
</p>

---

## 为什么会有这个项目

我装了 352 个 skills、27 个 agents、无数 MCP 和 CLI 工具。

但每次打开斜杠菜单，一排 `Generate mermaid diagrams`、`Deploy the application`、`Executive summary generator`……

我不知道哪个是干什么的。AI 自己也不知道自己有多少能力。

所以我写了这个技能。它做的事很简单：扫一遍你电脑上所有的 AI 工具，把英文描述翻译成中文，生成一张能力清单告诉 AI"你有这些本事"。

踩过的坑都在 `references/` 里写成了规则：哪些字段不能碰、符号链接怎么处理、Agent 正文为什么一个字都不能改。

---

## 30 秒开始

```bash
npx skills add ikevss/ikevss-zh-friendly
```

装完直接跟 AI 说：

```
帮我把这些英文技能翻译成中文
```

或者精确一点：

```
/ikevss-zh-friendly zh
```

**默认只看不改。** 确认没问题再 `--confirm`。

---

## 效果

| 改之前 | 改之后 |
|--------|--------|
| `Generate mermaid diagrams for documentation` | `生成 Mermaid 图表 \| 画流程图、架构图、mermaid、diagram` |
| `Deploy the application to production` | `部署应用 \| 帮我发布、上线、deploy、推代码` |
| `Code review skill for pull requests` | `代码审查 \| review、检查代码、帮我看看代码、code review` |

不只是翻译。**每个描述里加了 3-5 个你可能会说的中文口语表达**，让 AI 更容易听懂你。

还有：
- 🏷 自动生成中文 `display_name`，斜杠菜单里不再只有英文名
- 📋 扫出所有缺失 description 的技能，根据内容补充
- 🔍 发现你环境里的 CLI 工具（git、node、docker……）
- 🧠 连 Agent 人都帮你算进去了（Claude Code 27 个、Codex 7 个、OpenCode 7 个……）

---

## 它还能干什么

🤖 **推荐 Agent 人设** — 你的 agents 太少？扫一遍你的 CLAUDE.md、对话记录、已装 skills、记忆系统和工作习惯，猜出你大概做什么的，然后从 [268 个中文 agent 人设库](https://github.com/ikevss/agency-agents-zh)里推荐适合你的。

🔒 **三种汉化安全模式**：
- 默认模式 — 直接翻译（改前备份）
- `--overlay` 模式 — 不改原文，新加 `description_zh` 字段（不怕 marketplace 更新覆盖）
- 符号链接选项 C — 断开链接创建独立副本再汉化（完全不碰源文件）

🛡 **Agent 正文保护** — 翻译完逐字符对比 System Prompt，哪怕多了一个空格也会从备份恢复。

---

## 适合谁

| ✅ 适合 | ❌ 不适合 |
|---------|---------|
| 英语一般但想用好 AI 工具的人 | 所有技能描述已经是中文了 |
| 装了几十个 skills 记不住哪个是干什么的 | 你只用一两个技能 |
| 想让技能描述一眼看懂 | 你喜欢看英文界面 |
| 同时用好几个 AI 工具（Claude Code / TRAE / Codex） | — |

---

## 触发方式

以下任何一句都能触发这个技能：

> "帮我把这些英文技能翻译成中文"
> "我装了哪些技能？AI 环境里有什么工具？"
> "agent 太少了，推荐几个"
> "有些技能连描述都没有，帮我补上"
> "更新一下能力清单"

不用说 `/ikevss-zh-friendly`，直接说人话就行。

---

## 支持哪些 AI 工具

| 工具 | 出品方 | 汉化 | 扫描 |
|------|--------|:--:|:--:|
| Claude Code | Anthropic | ✅ | ✅ |
| TRAE Work | 字节跳动 | ✅ | ✅ |
| Codex | OpenAI | ✅ | ✅ |
| Cursor | Cursor Inc | ✅ | ✅ |
| OpenCode | 开源 | ✅ | ✅ |
| Windsurf | Codeium | ✅ | ✅ |
| WorkBuddy | 腾讯 | ✅ | ✅ |
| Antigravity | Google | — | ✅ |
| OpenClaw | 开源 | ✅ | ✅ |

---

## 它是怎么工作的

不是什么大语言模型翻译 API，就是一个 Claude Code 技能——**一组写好的指令和规则**。

它读取你电脑上各工具的 SKILL.md 文件，找到 `description` 字段，按规则翻译成中文，再写回去。翻译时会参考一个 60+ 词的术语表，保证"commit"不被翻译成"提交"、"frontmatter"不被翻译成"前置物质"。

扫描部分的 adapter 设计是在五位工程师（Claude / Codex / OpenCode / OpenClaw / Harness）的对抗性评审下改出来的——每个人都在挑"你这个路径不对""这个格式不兼容""这个字段不能动"。

---

## 安装

```bash
npx skills add ikevss/ikevss-zh-friendly
```

或手动：

```bash
git clone https://github.com/ikevss/ikevss-zh-friendly.git ~/.claude/skills/ikevss-zh-friendly
```

---

## 常见问题

**汉化后技能还能触发吗？**

能。翻译时保留了英文关键词和口语表达。实测在新电脑上跑通了 14 个技能汉化，`/skills` 列表正常显示。

**会把我的 Agent 改坏吗？**

不会。Agent 文件的正文（System Prompt）一个字不动。翻译完会逐字符对比校验——多一个空格都会从备份恢复。

**只翻译 Claude Code？还是其他工具也管？**

都管。TRAE Work、Codex、OpenCode、Cursor、Windsurf、WorkBuddy、OpenClaw——只要装了就能扫。Antigravity 因为格式不同只扫描不修改。

**翻译完以后更新工具会被覆盖吗？**

marketplace 来源的技能会。`--overlay` 模式可以避免——不改原 description，新加 `description_zh` 字段。

---

## License

MIT © 2026 ikevss
