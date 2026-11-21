# Knowledge in files, short chats - a guide for Claude

Talk to the user in the language they write in. Keep role files, templates and file contents in English unless the user asks otherwise.
Change nothing before the user has said yes to that change.

**For:** the Claude instance that works with the user in Claude Code (terminal, IDE extension, or the Code tab
of Claude Desktop) on a Pro or Max plan.
**How to use it:** read this file fully first. Then go through it with the user step by step.
**Part of:** guide 2 of 2. It follows guide 1 (`01-rollen-und-modelle.md`, roles and models), but also works on its
own. Templates are in `vorlagen/`.

## 0. Why a full context window costs so much - and what follows

Run `/context` in the user's working chats and look at the history share. An example of what you may find
(made-up numbers, for illustration only):

| Chat | Context used | History share | State |
|---|---|---|---|
| main project chat, running for a week (1M window) | 92 % | 86 % | in use, close to auto-compaction |
| side chat | 29 % | 20 % | idle |
| older chat | 55 % | 46 % | idle |

- If tools are small (MCP, skills and memory files a few percent each), **the history is the cost.** Every request
  in the big chat sends the whole history again, even a one-line question (costs).
- The way out: one chat per section of work, the rest in a new session.
- The gap: a new session only works if the knowledge of the old one survives. Today it often lives only in the chat
  history, so chats grow. **The fix: knowledge lives in files, the chat only works.** Then any chat can end at any
  time, and a new one picks up in minutes.

## 1. If a chat is already very large: rescue it, then stop using it

1. In the big chat, send one last request before auto-compaction runs. Auto-compaction loses detail, and compaction
   is itself a large request (costs). The request:
   > "Write everything that is not yet in files: the current state into `STAND.md`, every decision into
   > `decisions.md`, every lesson into `memory.md` - by the templates in `vorlagen/`. Paths, not contents. Then stop."
2. That one request costs one read of the whole history. Every further request there would cost the same, so it pays
   off at once.
3. Check the files with the user in a new chat ("read STAND.md"), fill gaps, and then leave the old chat alone: no
   more requests. Close or archive it.
   - An idle chat costs little. It can still use some tokens in the background (session recap, idle compaction;
     costs, "Background token usage").
   - Coming back to it later is expensive: after a break longer than the cache lifetime (one hour on a
     subscription), the next message reprocesses the whole history uncached (prompt-caching).
4. Other large chats: the same, at the end of their current section.

## 2. The principle

| | Chat history | Files |
|---|---|---|
| What it is | working memory | long-term memory |
| Cost | sent again with every request | read only when needed |
| Lifetime | until `/clear`, compaction or the end of the chat | stays; versioned with git if the project uses it |

- Whatever is settled goes into a file **at the moment it is settled**, not at the end of the day. A decision, a
  measured value, a working setting, a mistake and its fix: one line, right away.
- **Only the main session can do this.** It alone knows the conversation; a subagent sees only its brief. Notes are
  short, so this stays cheap.

## 3. Where knowledge lives: CLAUDE.md, STAND.md, decisions.md, memory.md

A small, fixed set per project folder:

| File | Holds | Rule |
|---|---|---|
| `CLAUDE.md` | what the project is, where things are, the working rules, pointers to the files below | short. The docs target under 200 lines per file; shorter is cheaper. It loads into every session. No history, no states. |
| `STAND.md` | the handover: current section, done, next steps, open questions, key paths | at most one page, overwritten at each section end. The first file a new chat reads. |
| `decisions.md` | every decision, dated, with the user's own words when they decide | append only; a changed decision gets a new line that names the old one |
| `memory.md` | facts (settings, formats, versions that work) and lessons (what went wrong and the fix) | one line each, dated; remove what is no longer true |
| `plan/<section>.md` | for a bigger job: the steps with their status | one file per job, status updated as it goes |

("Stand" is German for "current state". Rename the file if the user prefers, for example `STATE.md`; keep the name
the same in all projects.)

**What applies across all projects** (the user's preferences, naming rules, how a tool is set up on this machine)
goes into the user-level `~/.claude/CLAUDE.md`. It loads in every project.

**Claude's auto memory** is a separate thing. It is stored per project (per git repository) in
`~/.claude/projects/<project>/memory/`, with `MEMORY.md` as its index. The first 200 lines or 25 KB of `MEMORY.md`
load at session start. It is not loaded into subagents. Turn it on or off with `/memory` (memory). It does not
replace the files above: those are visible, reviewable and can be versioned with the project.

What does **not** go into files: chat text, raw logs, images, intermediate attempts.

Templates: `vorlagen/STAND.md`, `vorlagen/decisions.md`, `vorlagen/memory.md`.

## 4. The rhythm

- **Start of a chat:**
  - `CLAUDE.md` loads by itself.
  - Read `STAND.md`, nothing more.
  - Open other files only when the step needs them, or let a cheap `scout` role look them up (guide 1).
- **During a chat:**
  - A decision -> one line in `decisions.md`.
  - A lesson or a working setting -> `memory.md`.
  - A step done -> its status in the plan.
- **End of a section, or when the context window gets full (whichever comes first):**
  1. Update `STAND.md`, check that decisions and lessons are in.
  2. Commit, if the project uses git.
  3. New chat: `/clear` (it costs nothing) or a new session; first message "read STAND.md".
- **When is "full"?** A rule of thumb, not from the docs: hand over at about 40-50 % of a 200,000-token window, or
  at about 100,000-200,000 tokens. With a 1M window, percentages mislead: 40 % is already 400,000 tokens sent with
  every request.
- **Do not let a chat run into auto-compaction.** The summary loses detail, and the compaction request reads the
  whole conversation. Your own handover is better and cheaper. `/compact` with a hint is the second-best option, for
  the middle of a task (context-window).
- **Parallel chats in one project:** each writes its own `STAND-<section>.md`, so they do not overwrite each other.
  `decisions.md` and `memory.md` stay shared (append only).

## 5. Guard rails in the setup (with the user's yes)

1. **Know the context window size.** On Pro and Max, the default model is Opus 5.5. Opus 4.7 and later, Sonnet 5 and
   later and Haiku 5.5 run with a 1 million token window by default. Without a setting, those sessions auto-compact
   at about 967,000 tokens (model-config). That is late: every request before that point carries the whole history.
2. **Set auto-compaction earlier, as a safety net.**
   - `/autocompact 200k` sets how full the context gets before auto-compaction runs, for the current model. Claude
     Code saves it to the user settings. Values of 100k or more are accepted; `/autocompact auto` goes back to the
     default (model-config, commands).
   - For every model: `"autoCompactWindow": 200000` in `~/.claude/settings.json`.
   - Alternative: `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` (in the shell or in the `env` block of `settings.json`) keeps
     sessions at a 200K window. Environment variables are read at start, so it applies to new sessions.
   - This is only the net. The handover in section 4 comes first.
3. **Watch the context fill.** In the terminal: `/context`. In the desktop app: the Context card in the session
   details, or the usage ring popover in the earlier layout (desktop). When it gets full, hand over.
4. **One line in the user-level `~/.claude/CLAUDE.md`**, so it holds in every project (`vorlagen/CLAUDE-md-lines.md`):
   > "Knowledge goes into files when it is settled (STAND.md, decisions.md, memory.md). When the context gets full or
   > at the end of a section: update STAND.md, then start a new chat."

## 6. Check it with the user after a week

- **`/context` in the working chats:** none close to the hand-over point, the history share clearly lower than
  before.
- **`/usage`:** compare with the baseline from guide 1.
- **A cold start test:** a new chat with only "read STAND.md". Can it continue without asking? If not, the handover
  lacks something: add it to the template.
- **A lesson about the process:** note it in `memory.md` (for example "STAND.md lacked the export settings -> now in
  Key paths").

## Not part of this

- No automation, no hooks that write files by themselves.
- Nothing gets deleted. Old chats are left alone or archived; files are trimmed only after the user's yes.

## Sources (checked 2026-11-21)

Claude Code docs:
- https://code.claude.com/docs/en/costs - why usage climbs in a long session, compaction cost, `/clear`, background
  token usage
- https://code.claude.com/docs/en/context-window - `/context`, `/compact`, auto-compaction, `/autocompact`
- https://code.claude.com/docs/en/model-config - 1M window, default auto-compact thresholds, `autoCompactWindow`,
  `CLAUDE_CODE_DISABLE_1M_CONTEXT`
- https://code.claude.com/docs/en/prompt-caching - cache lifetime, model switches, compaction
- https://code.claude.com/docs/en/memory - CLAUDE.md locations and size, auto memory, `MEMORY.md`
- https://code.claude.com/docs/en/commands - `/autocompact`, `/clear`, `/compact`, `/memory`
- https://code.claude.com/docs/en/desktop - context and usage display in the desktop app

Not from the docs: the 40-50 % / 100,000-200,000 token hand-over point is a rule of thumb.

The docs change. If a command or setting behaves differently, trust the current docs over this file.
