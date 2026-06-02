# zotero-polish

A [Claude Code](https://claude.ai/code) skill that polishes English academic paragraphs **strictly according to your own Zotero writing notes** — not generic academic-English advice.

一个 [Claude Code](https://claude.ai/code) 技能：**仅依据你 Zotero 写作笔记 collection 下的 note** 来润色英文学术段落，不引入其他文献或通用润色经验。

---

## What it does · 功能简介

**EN** — Most "polish my writing" tools impose a generic house style. This skill does the opposite: it pulls **your** writing rules from a dedicated Zotero collection of notes and applies only those rules. Every edit it makes is traced back to a specific rule or example in your notes — if a change has no support in your notes, it won't make it.

This is useful when you've spent time distilling your own (or your advisor's / target journal's) writing conventions into Zotero notes and want an assistant that follows *your* conventions consistently across abstract, introduction, methods, results, and discussion.

**中文** — 大多数"润色"工具会强加一套通用文风。本技能正相反：它从你专门建立的 Zotero 笔记 collection 中读取**你自己的**写作规则，并且只应用这些规则。每一处修改都会溯源到笔记中的具体规则或示例——如果某处改动在笔记里找不到依据，它就不改。

当你已经把自己（或导师 / 目标期刊）的写作规范沉淀成 Zotero 笔记，并希望有个助手在摘要、引言、方法、结果、讨论各部分都一致地遵循*你的*规范时，这个技能尤其有用。

---

## How it works · 工作原理

**EN**

1. **Detects the section** of the paragraph you give it (abstract / intro / methods / results / discussion) and selects the matching note.
2. **Fetches the note body** from the Zotero Web API at runtime (notes are not bundled — they stay in your library).
3. **Rewrites the paragraph** following only the rules and examples in those notes.
4. **Returns a three-part bilingual output**: the polished English, a Chinese change-log where every edit cites the specific note rule it follows, and the list of notes referenced.

The skill always additionally loads a general "expressions & conventions" note that applies to every section (terminology, citation style, etc.).

**中文**

1. **识别段落归属**：判断你给的段落属于哪一部分（摘要／引言／方法／结果／讨论），选定对应的 note。
2. **运行时拉取 note 正文**：通过 Zotero Web API 实时读取（笔记不打包进仓库，始终留在你的文库里）。
3. **按规则改写段落**：只依据这些 note 中的规则与示例进行润色。
4. **输出三段式中英双语结果**：润色后的英文、逐条引用 note 规则的中文修改说明、以及所参考的 note 列表。

技能还会额外加载一条通用的"特定表达与规范"note（术语、引用格式等），它对所有章节都适用。

---

## Requirements · 环境要求

- [Claude Code](https://docs.claude.com/en/docs/claude-code) with skills enabled · 已启用 skills 的 Claude Code
- A [Zotero](https://www.zotero.org/) account and a Zotero Web API key · 一个 Zotero 账号及 Web API key
- `curl` and (optionally) `python` or `jq` on your PATH · PATH 中可用的 `curl`，以及可选的 `python` 或 `jq`

---

## Setup · 安装配置

### 1. Create a Zotero collection of writing notes · 建立写作笔记 collection

**EN** — This skill expects standalone notes organized by paper section (abstract, introduction, methods, results, discussion, plus a general expressions note).

**中文** — 技能期望你按论文章节建立独立的 note（摘要、引言、方法、结果、讨论，外加一条通用表达 note）。

### 2. Get a Zotero API key · 获取 API key

Get a key at https://www.zotero.org/settings/security and set it as an environment variable named `zotero_api_key`.

在 https://www.zotero.org/settings/security 申请 key，并设为名为 `zotero_api_key` 的环境变量。

```powershell
# Windows (PowerShell) — persist for current user · 为当前用户持久化
setx zotero_api_key "your_key_here"
# restart the shell afterwards so the variable is visible · 之后重启 shell 才能读到
```

```bash
# macOS / Linux
export zotero_api_key="your_key_here"   # add to ~/.bashrc or ~/.zshrc to persist · 写入 rc 文件以持久化
```

### 3. Configure your own IDs · 替换为你自己的 ID

**EN** — `SKILL.md` ships with placeholders. Replace each with your own values:

**中文** — `SKILL.md` 中是占位符，使用前请替换为你自己的值：

- `<YOUR_ZOTERO_USER_ID>` — your Zotero User ID (find it at https://www.zotero.org/settings/security) · 你的 Zotero User ID
- `<YOUR_COLLECTION_KEY>` — the collection key of your writing-notes collection · 写作笔记 collection 的 key
- `<NOTE_KEY_*>` — the note keys and their section mapping in the index table · 索引表中各 note 的 key 及章节映射

### 4. Install the skill · 安装技能

Place this folder in your Claude Code skills directory · 把本文件夹放入 Claude Code 的 skills 目录：

```
~/.claude/skills/zotero-polish/
```

---

## Usage · 使用方法

**EN** — In Claude Code, paste an English paragraph and explicitly reference your notes. The skill triggers only when you both ask for polishing **and** name your Zotero notes:

**中文** — 在 Claude Code 中贴入英文段落，并**明确点名**你的笔记。只有当你同时"要求润色"**且**"点名 Zotero note"时，技能才会触发：

- `按我的 zotero note 润色这段 abstract：<paragraph>`
- `polish this with my zotero notes: <paragraph>`
- `用我的写作笔记改一下这段方法描述：<paragraph>`

If you don't reference your notes, the skill stays out of the way. · 如果你没点名笔记，技能不会介入。

---

## Why "notes only" · 为什么"只看笔记"

**EN** — The skill deliberately refuses to mix in advice from generic writing skills (`humanizer`, `research-paper-writing`, etc.). Your notes represent your personal style preferences and **always take precedence**, even where they conflict with conventional academic-English guidance. If your notes don't cover a situation, the skill says so and makes only minimal, non-conflicting edits rather than improvising.

**中文** — 技能刻意拒绝混入其他通用写作技能（`humanizer`、`research-paper-writing` 等）的建议。你的笔记代表你的个人风格偏好，**始终优先**，即便与通用学术英语建议冲突也以笔记为准。若笔记未覆盖某情形，技能会如实说明，并只做最小的、不冲突的润色，而不擅自发挥。

---

## Files · 文件说明

- `SKILL.md` — the skill definition: trigger conditions, note index, fetch commands, workflow, and output format · 技能定义：触发条件、note 索引、拉取命令、工作流程、输出格式
- `README.md` — this file · 本文件

---

## Troubleshooting · 故障排查

| Symptom · 现象 | Fix · 处理 |
|---|---|
| `curl` returns empty · curl 返回空 | Check `echo $zotero_api_key` is non-empty; on Windows restart the shell after `setx`. · 检查环境变量非空；Windows 用 `setx` 后需重启 shell。 |
| HTTP 403 | API key invalid or lacks permission — regenerate at https://www.zotero.org/settings/security. · key 失效或权限不足，重新生成。 |
| HTTP 404 | Wrong note key — re-verify the keys in your collection. · note key 错误，核对你 collection 下的 key。 |
| Garbled HTML output · 输出乱码 | Terminal encoding issue; adjust your shell encoding or use another API client. · 终端编码问题，调整编码或换用其他 API 客户端。 |

---

## License · 许可

MIT
