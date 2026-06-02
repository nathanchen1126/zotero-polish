# zotero-polish

A [Claude Code](https://claude.ai/code) skill that polishes English academic paragraphs **strictly according to your own Zotero writing notes** — not generic academic-English advice.

> 一个 Claude Code 技能：**仅依据你 Zotero「02写作笔记」collection 下的 note** 来润色英文学术段落，不引入其他文献或通用润色经验。

## What it does

Most "polish my writing" tools impose a generic house style. This skill does the opposite: it pulls **your** writing rules from a dedicated Zotero collection of notes and applies only those rules. Every edit it makes is traced back to a specific rule or example in your notes — if a change has no support in your notes, it won't make it.

This is useful when you've spent time distilling your own (or your advisor's / target journal's) writing conventions into Zotero notes and want an assistant that follows *your* conventions consistently across abstract, introduction, methods, results, and discussion.

## How it works

1. **Detects the section** of the paragraph you give it (abstract / intro / methods / results / discussion) and selects the matching note.
2. **Fetches the note body** from the Zotero Web API at runtime (notes are not bundled — they stay in your library).
3. **Rewrites the paragraph** following only the rules and examples in those notes.
4. **Returns a three-part bilingual output**: the polished English, a Chinese change-log where every edit cites the specific note rule it follows, and the list of notes referenced.

The skill always additionally loads a general "expressions & conventions" note that applies to every section (terminology, citation style, etc.).

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) with skills enabled.
- A [Zotero](https://www.zotero.org/) account and a Zotero Web API key.
- `curl` and (optionally) `python` or `jq` on your PATH for fetching note text.

## Setup

1. **Create a Zotero collection of writing notes.** This skill expects standalone notes organized by paper section (abstract, introduction, methods, results, discussion, plus a general expressions note).

2. **Get a Zotero API key** at https://www.zotero.org/settings/security and set it as an environment variable named `zotero_api_key`.

   ```powershell
   # Windows (PowerShell) — persist for current user
   setx zotero_api_key "your_key_here"
   # restart the shell afterwards so the variable is visible
   ```

   ```bash
   # macOS / Linux
   export zotero_api_key="your_key_here"   # add to ~/.bashrc or ~/.zshrc to persist
   ```

3. **Configure your own IDs.** `SKILL.md` ships with placeholders. Replace each with your own values:
   - `<YOUR_ZOTERO_USER_ID>` — your Zotero User ID (find it at https://www.zotero.org/settings/security)
   - `<YOUR_COLLECTION_KEY>` — the collection key of your writing-notes collection
   - `<NOTE_KEY_*>` — the note keys and their section mapping in the index table

4. **Install the skill** by placing this folder in your Claude Code skills directory:
   ```
   ~/.claude/skills/zotero-polish/
   ```

## Usage

In Claude Code, paste an English paragraph and explicitly reference your notes. The skill triggers only when you both ask for polishing **and** name your Zotero notes:

- `按我的 zotero note 润色这段 abstract：<paragraph>`
- `polish this with my zotero notes: <paragraph>`
- `用我的写作笔记改一下这段方法描述：<paragraph>`

If you don't reference your notes, the skill stays out of the way.

## Why "notes only"

The skill deliberately refuses to mix in advice from generic writing skills (`humanizer`, `research-paper-writing`, etc.). Your notes represent your personal style preferences and **always take precedence**, even where they conflict with conventional academic-English guidance. If your notes don't cover a situation, the skill says so and makes only minimal, non-conflicting edits rather than improvising.

## Files

- `SKILL.md` — the skill definition: trigger conditions, note index, fetch commands, workflow, and output format.
- `README.md` — this file.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `curl` returns empty | Check `echo $zotero_api_key` is non-empty; on Windows restart the shell after `setx`. |
| HTTP 403 | API key invalid or lacks permission — regenerate at https://www.zotero.org/settings/security. |
| HTTP 404 | Wrong note key — re-verify the keys in your collection. |
| Garbled HTML output | Terminal encoding issue; adjust your shell encoding or use the API client of your choice. |

## License

MIT
