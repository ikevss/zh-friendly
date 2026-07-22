# ikevss-zh-friendly

**你的 AI 编程助手装了一堆英文技能，看不懂？不知道它能做什么？点一下，全变中文。**  

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Skills.sh](https://img.shields.io/badge/skills.sh-ikevss_zh_friendly-6B46C1)](https://www.skills.sh/)

---

## 一句话说清楚

你把 ChatGPT、Claude Code、TRAE 这些 AI 工具当副驾驶用，但它们装的那些"技能"都是英文的——
你看着一排 `Generate mermaid diagrams`、`Deploy the application`，不知道哪个干什么、什么时候用。

**这个技能帮你一键搞定：扫出所有英文描述 → 翻译成中文 → 告诉 AI 自己有多少能力。**

---

## 3 秒看效果

| 改之前 | 改之后 |
|--------|--------|
| `Generate mermaid diagrams for documentation` | `生成 Mermaid 图表 | 画流程图、时序图、架构图、mermaid、diagram、chart` |
| `Deploy the application to production` | `部署应用 | 帮我发布、上线、deploy、推代码` |
| `Code review skill for pull requests` | `代码审查 | review、检查代码、帮我看看代码、code review` |

改完之后，跟 AI 说"帮我画个流程图"，它也听得懂了。

---

## 我能做什么

两件事，都只要一句话：

| 我想…… | 我输入 | 效果 |
|---------|--------|------|
| 把英文技能翻译成中文 | `/ikevss-zh-friendly zh` | 扫描所有工具 → 翻译描述 → 不改任何代码 |
| 看看 AI 到底有哪些能力 | `/ikevss-zh-friendly scan` | 扫描所有工具 → 生成一张能力清单 |

### 还有这些场景也能用

- **有些技能连描述都没有** → 自动帮你生成一个
- **装了太多技能不知道哪个是干什么的** → 全部翻译成中文，一眼看明白
- **Agent 太少，想让它更聪明** → 自动推荐适合你的 Agent 人设（从 268 个中文人设库里挑）
- **怕翻译完之后更新又变回英文** → 支持"叠加模式"，新加一个中文字段，不动原文
- **只想看某个工具的翻译效果** → `/ikevss-zh-friendly zh --scope trae --dry-run` 先预览不改

---

## 怎么安装

打开终端，输一行命令：

```bash
npx skills add ikevss/ikevss-zh-friendly
```

或者直接从 [skills.sh](https://www.skills.sh/) 搜索 `ikevss-zh-friendly` 安装。

---

## 怎么用

安装完，在 Claude Code 里输入 `/ikevss` 就能看到这个技能。然后：

```bash
# 第一次用，先预览（不改任何文件）
/ikevss-zh-friendly zh

# 确认没问题，执行翻译
/ikevss-zh-friendly zh --confirm

# 扫描你的 AI 环境有哪些能力
/ikevss-zh-friendly scan
```

**不会改坏你的文件**：默认只预览不改东西，确认了才执行，修改前还会自动备份。

---

## 都支持哪些 AI 工具

装了这个技能，它会帮你扫描和翻译这些工具里的英文内容：

| 工具 | 谁出的 | 能翻译 | 能扫描 |
|------|--------|:------:|:------:|
| Claude Code | Anthropic | ✅ | ✅ |
| TRAE Work | 字节跳动 | ✅ | ✅ |
| Codex | OpenAI | ✅ | ✅ |
| Cursor | Cursor Inc | ✅ | ✅ |
| OpenCode | 开源社区 | ✅ | ✅ |
| Windsurf | Codeium | ✅ | ✅ |
| WorkBuddy | 腾讯 | ✅ | ✅ |
| Antigravity | Google | — | ✅ |
| OpenClaw | 开源社区 | ✅ | ✅ |

---

## 为什么放心用

- **默认只看不改** — 不加 `--confirm` 绝不碰你的文件
- **改前先备份** — 每个被改的文件都有备份，随时恢复
- **Agent 正文绝对不动** — 只翻译给人看的描述，AI 的内部指令一个字不改
- **不改文件名和目录** — 只动描述文字，不影响工具的正常运行
- **新机器试过才发布** — 已在真实电脑上跑通

---

## 谁在用、谁适合用

- 你用 Claude Code / TRAE / Codex / Cursor 但英语不太好
- 你装了十几个 AI 技能但记不住哪个是干什么的
- 你想让 AI 用起来更像"中文助理"而不是"英文工具"
- 你是金融/法律/教育/设计从业者，用 AI 但不想花时间学英语术语

---

## 关于作者

我是 ikevss，一个用 AI 解决实际问题的开发者。这个技能是为了让我自己——以及所有和我一样英语不够好、但想用好 AI 工具的中文用户——能更方便地用 AI。

如果你觉得有用，可以在 [skills.sh](https://www.skills.sh/) 上给个 ⭐，或者在 GitHub 上提 issue 告诉我哪里还可以更好。

---

**让 AI 工具说流利中文，从这一条命令开始。**

```bash
/ikevss-zh-friendly zh
```
