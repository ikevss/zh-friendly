# ikevss-zh-friendly

> 装了一堆 AI 技能，名字全是英文，不知道哪个是干嘛的？
> 一键翻译成中文，一眼看懂。

<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License">
  <a href="https://www.skills.sh/"><img src="https://img.shields.io/badge/skills.sh-ikevss_zh_friendly-purple" alt="skills.sh"></a>
</p>

---

## 先说说你可能的困惑

你装了 Claude Code、TRAE、Codex、Cursor……一堆 AI 工具。每个工具里又有几十个、上百个 skills、agents。

打开斜杠菜单——一排英文：`Generate mermaid diagrams`、`Deploy the application`、`Executive summary generator`……

**你不知道哪个是干嘛的。AI 自己也不知道自己有多少能力。**

更离谱的是，你问它"你能做什么"，它是靠记忆在猜的——因为它连一份清单都没有。

---

## 这个技能做什么

**两件事：翻译技能描述 + 生成能力清单。**

**翻译：** 把你所有 AI 技能的名字和描述，从英文翻译成中文。不只是机翻，每个描述里会加上 3-5 个你可能会说的中文口语表达。比如：

| 英文原版 | 翻译后 |
|---------|--------|
| Generate mermaid diagrams | 生成 Mermaid 图表 \| 画流程图、时序图、架构图、mermaid、diagram |
| Deploy the application to production | 部署应用 \| 帮我发布、上线、deploy、推代码 |
| Code review skill for pull requests | 代码审查 \| review、检查代码、帮我看看代码、code review |

**能力清单：** 扫一遍你所有 AI 工具，生成一份能力清单写入配置文件。这不只是给你看的——是给 AI 看的。开新会话时 AI 读到这份清单，就知道自己有哪些技能、Agent、MCP、CLI。如果启用了多智能体协作，每个 agent 都能看到全局能力，协作效率更高。

**Agent 引导：** 如果你的 agents 里还没设置人设或者人设太少，它会根据你已有的 skills 和工具，从 268 个中文人设库中推荐合适的，快速补齐。 |

翻译完之后，无论你说"帮我画个流程图"还是"draw a flowchart"，AI 都能听懂。

---

## 三个场景，你对号入座

**场景一：英文技能看不懂**
> "这些英文技能看不懂，帮我翻译一下"

扫描你所有 AI 工具的 skills/agents，把 description 翻译成中文。默认只预览，确认了才改。

**场景二：不知道自己有多少能力**
> "我装了哪些技能？AI 环境里有什么工具？"

扫描所有工具，生成一张能力清单写入配置文件。下次开新会话，AI 就知道自己有什么本事了。多智能体协作时，每个 agent 都能看清全局——知道有哪些技能、Agent、MCP、CLI 工具可用。

**场景三：Agent 太少，想推荐几个**
> "agent 太少了，推荐几个合适的"

分析你平时用 AI 干什么，从 268 个中文 agent 人设库里推荐适合你的。确认后自动安装。如果你的 agents 里还没设置人设，它会根据你已有的技能和工具，帮你快速创建。

---

## 安装

这个技能需要装在 **Claude Code** 里。如果你还没有 Claude Code，先装它：

```
# 在终端里执行这行命令
npm install -g @anthropic-ai/claude-code
```

装好 Claude Code 后，在终端里执行这一行：

```
npx skills add ikevss/zh-friendly
```

按回车等几秒钟，看到「Installed 1 skill」就是装好了。

然后打开 Claude Code，跟它说：

```
帮我把这些英文技能翻译成中文
```

或者直接输入斜杠命令：

```
/ikevss-zh-friendly zh
```

**第一次用建议先用 `zh`（默认只预览不改文件），看看效果没问题了再加 `--confirm`。**

---

## 还支持这些

- 🏷 自动生成中文 `display_name`，斜杠菜单里不再只有英文名
- 📋 扫出所有缺失 description 的技能，根据内容补充
- 🔍 发现你环境里的 CLI 工具（git、node、docker……）
- 🧠 连 Agent 人都帮你算进去了（Claude Code 27 个、Codex 7 个、OpenCode 7 个……）

---

## 三种安全模式

| 模式 | 说明 |
|------|------|
| 默认模式 | 直接翻译（改前备份） |
| `--overlay` 模式 | 不改原文，新加 `description_zh` 字段（不怕 marketplace 更新覆盖） |
| 符号链接选项 C | 断开链接创建独立副本再汉化（完全不碰源文件） |

**Agent 正文绝对不动。** 翻译完逐字符校验 System Prompt，多一个空格都会从备份恢复。

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

## 触发方式

以下任何一句都能触发这个技能：

> "帮我把这些英文技能翻译成中文"
> "我装了哪些技能？AI 环境里有什么工具？"
> "agent 太少了，推荐几个"
> "有些技能连描述都没有，帮我补上"
> "更新一下能力清单"

不用说 `/ikevss-zh-friendly`，直接说人话就行。

---

## 安装

```bash
npx skills add ikevss/zh-friendly
```

或手动：

```bash
git clone https://github.com/ikevss/zh-friendly.git ~/.claude/skills/ikevss-zh-friendly
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

marketplace 来源的技能会。`--overlay` 模式可以避免——不改原 description，新加一个 `description_zh` 字段。

---

## License

MIT © 2026 ikevss
