# Claude Code: reduce token usage and hit the weekly limit less often

Hit the Claude weekly limit too fast? Usually the cause is not too much work but how the work runs: long chats,
many images in the main chat, and Opus for every small step. This repository holds two guides that you give to your
own Claude. Claude then walks you through them step by step: measure first, change on purpose, measure again after
a week.

Free, for private users and solo freelancers on Claude Pro or Max who use Claude Code in the terminal, an IDE, or the
Code tab of Claude Desktop.

By René Hillers, [rhse.de](https://rhse.de/) - software development and AI automation.

## How to use it

1. Download the whole repository: **Code** → **Download ZIP**, then unzip it.
2. Give your Claude the path to `anleitungen/01-rollen-und-modelle.md` (or drag the file into the chat) and write:
   "Read it fully and walk me through it step by step."
3. Later, the same with `anleitungen/02-wissen-in-dateien.md`.

You don't need to touch the templates in `vorlagen/` yourself. The guides point to them and Claude uses them at the
right step. That is why you download the whole repository, not just one file.

Claude changes nothing before you have said yes to that change. Both guides say so at the top.

## Why does Claude Code hit the limit too fast?

Claude Code sends the whole conversation with every request, and every tool call is another request. The docs say
that even a one-line question in a session open all day draws usage for the whole conversation. Images add up: one
screenshot costs up to 4,784 tokens and rides along with every later request. And a subagent without its own model
runs on the main model, which is Opus by default on Pro and Max.
-> [Guide 1, section 1](anleitungen/01-rollen-und-modelle.md#1-why-claude-code-hits-the-limit-too-fast---the-mechanics)

## How do I save tokens in Claude Code?

Measure first: on Pro and Max, `/usage` shows the share of subagents, skills, plugins and single MCP servers, and
`/context` shows what fills the context window. Then three changes usually matter most: shorter chats with a
handover file, images checked in their own subagent instead of the main chat, and the right model per task.
Re-measure with `/usage` after a week.
-> [Guide 1, section 2](anleitungen/01-rollen-und-modelle.md#2-step-1---measure-token-usage-first-with-the-user-about-15-minutes-change-nothing)

## Opus vs Sonnet vs Haiku in Claude Code

Opus for open problems, planning and decisions where a mistake is expensive. Sonnet for well-defined work: code from
a spec, scripts, configuration, texts from an outline. Haiku for search, lookups and mechanical work with a clear
answer. The official docs also recommend Sonnet for most coding tasks and Haiku for simple subagent tasks.
`/model opusplan` uses Opus in plan mode and Sonnet to execute.
-> [Guide 1, section 3](anleitungen/01-rollen-und-modelle.md#3-step-2---opus-vs-sonnet-vs-haiku-which-model-for-which-work)

## How do I set the model for a Claude Code subagent?

Give every role file in `~/.claude/agents/` or `.claude/agents/` its own `model:` (for example `haiku`) and an
`effort:`. Without `model`, the subagent inherits the main model. Order of precedence: a model passed by the main
session at start, then the role file's `model`, then `CLAUDE_CODE_SUBAGENT_MODEL`, then the main model. So add a line
to CLAUDE.md: do not set a model when starting a subagent. Templates for three roles are in `vorlagen/agents/`.
-> [Guide 1, section 4](anleitungen/01-rollen-und-modelle.md#4-step-3---subagents-with-an-explicit-model-check-the-existing-ones-first-then-add-new-ones)

## CLAUDE.md best practices and memory files

CLAUDE.md holds what the project is, where things are, the working rules and pointers to other files. No history,
no states: it loads into every session, so every line costs on every request. The docs target under 200 lines per
file. Current state, decisions and lessons go into their own files (`STAND.md`, `decisions.md`, `memory.md`), read
only when needed.
-> [Guide 2, section 3](anleitungen/02-wissen-in-dateien.md#3-where-knowledge-lives-claudemd-standmd-decisionsmd-memorymd)

## Compact vs clear

`/clear` starts a new conversation and costs nothing. `/compact` summarizes the conversation and reads it once more
to do so; with a large context that is a large request. Rule of thumb: task done, write the state into `STAND.md`,
then `/clear`. Mid-task with a full context: `/compact` with a hint on what has to stay.
-> [Guide 2, section 4](anleitungen/02-wissen-in-dateien.md#4-the-rhythm)

## Claude Code context window full

Claude Code compacts automatically before the window is full. On Pro and Max, the default model runs with a
1 million token window and compacts at about 967,000 tokens. Until then, every request carries the whole history.
`/autocompact 200k` sets an earlier point as a safety net. Your own handover before that is better.
-> [Guide 2, section 5](anleitungen/02-wissen-in-dateien.md#5-guard-rails-in-the-setup-with-the-users-yes)

## Claude weekly limit reached

`/usage` shows your plan limits and recent usage. When a limit is reached, the message says when it resets. More
volume comes only from a bigger plan or paid usage credits (`/usage-credits`). This repository helps you use fewer
tokens; it does not get around limits.

## Notes

- Unofficial. Not affiliated with Anthropic. "Claude" is a trademark of Anthropic.
- All technical statements were checked against the official docs on 2026-11-21. Sources are listed at the end of
  each guide. The docs change; when in doubt, the current docs win.
- How much you save depends on how you work. No specific saving is promised.

## License

[CC BY 4.0](LICENSE).

## About the author

René Hillers - [rhse.de](https://rhse.de/), software development and AI automation. More than 25 years of software
development; today he builds with coding agents and his own multi-agent system.
[References](https://rhse.de/referenzen)

For teams in companies and self-employed professionals there is a Claude training, online and in German: [rhse.de/claude-schulung](https://rhse.de/claude-schulung)
