# Roles and models - a guide for Claude

Talk to the user in the language they write in. Keep role files, templates and file contents in English unless the user asks otherwise.
Change nothing before the user has said yes to that change.

**For:** the Claude instance that works with the user in Claude Code (terminal, IDE extension, or the Code tab
of Claude Desktop) on a Pro or Max plan.
**How to use it:** read this file fully first. Then go through it with the user step by step. You know the user's
projects; this guide knows the mechanics.
**Part of:** guide 1 of 2. Guide 2 (`02-wissen-in-dateien.md`) covers knowledge in files and short chats.
Templates are in `vorlagen/` next to this folder.

## 0. The situation and the goal

The usual complaint: the usage limit runs out fast. Typical patterns behind it:
- one strong model (Opus) does everything, including lookups and routine steps;
- many screenshots and reference images attached directly to the main chat;
- sessions that run for days;
- several chats in parallel;
- many MCP servers, subagents and skills, collected over time.

**Goal:** the main session directs and decides. Well-defined work runs in subagents ("roles") on the smallest
model that does it well. Images and long sessions get under control.
- This is not a multi-agent system: no orchestration, no hooks, no automation. It is a first step.
- **Success:** the same output for a clearly smaller share of the limit, measured with `/usage` before and after.

## 1. Why Claude Code hits the limit too fast - the mechanics

1. **Every request re-sends the whole conversation.**
   - Every tool call is another request.
   - Prompt caching makes the re-read cheaper, but it still counts. The docs say that even a one-line question in a
     session that has been open all day draws usage for the whole conversation (costs).
   - After a break longer than the cache lifetime, the next message reprocesses the full context uncached. On a
     subscription, the main conversation's cache lasts one hour (prompt-caching).
2. **Images are expensive, and they stay.**
   - Cost per image: `ceil(width / 28) x ceil(height / 28)` tokens.
   - On Claude 4.7 and later models, an image is scaled down when its long edge is above 2,576 px or it would cost
     more than 4,784 tokens. So one image costs at most 4,784 tokens. Older models use a lower limit
     (1,568 px, 1,568 tokens) (vision).
   - Examples, computed with the formula:
     - a 2560x1440 screenshot: 92 x 52 = 4,784 tokens;
     - the same image at 1280x720: 46 x 26 = 1,196 tokens (a quarter);
     - a 600x400 crop: 22 x 15 = 330 tokens.
   - Each image stays in the conversation and is sent again with every later request. Twenty full screenshots are
     about 95,000 tokens that ride along on every request until `/clear`.
   - When a request would carry too many images, Claude Code drops a batch of the oldest ones. The next turn then
     re-processes the conversation from that point (prompt-caching).
3. **A subagent without `model` runs on the main session's model.** On Pro and Max the default main model is Opus.
   - Order of precedence (sub-agents):
     1. a `model` that the main session passes when it starts the subagent;
     2. the `model` field in the role file (including `inherit`);
     3. the environment variable `CLAUDE_CODE_SUBAGENT_MODEL`;
     4. the main conversation's model.
   - So a role file with `model: haiku` still runs on Opus if the main session passes `model: opus`.
   - When the main session switches to Opus with `/model`, every subagent that inherits follows (model-config).
   - Subagents send their own requests. They count toward the same plan limit as the main session.
4. **Parallel chats each draw from the same limit.**
5. **A model switch starts the cache afresh.** `/model` mid-task, or toggling plan mode with `opusplan`, makes the
   next request read the whole history uncached (prompt-caching).
   - Changing the effort level keeps the cache on Opus 5.5, Sonnet 5.5 and Haiku 5.5 with a subscription. On most
     other models it does not.
6. **MCP, skills, CLAUDE.md.**
   - MCP tool definitions are deferred by default ("tool search"): only the tool names load until a tool is used.
     They load fully upfront if `ENABLE_TOOL_SEARCH=false` is set, or for a server with `"alwaysLoad": true` (mcp).
   - Skill descriptions load in every session. A subagent with a `skills:` list gets the full content of those
     skills at its start (sub-agents).
   - CLAUDE.md loads in every session, and into custom subagents unless the role sets `omitClaudeMd: true`.

## 2. Step 1 - measure token usage first (with the user, about 15 minutes, change nothing)

1. Run `/usage` (`/cost` and `/stats` are aliases). On Pro and Max it shows what counts against the plan limit:
   usage attributed to skills, subagents, plugins and single MCP servers, plus recommendations. Press `d` or `w`
   to switch between the last 24 hours and the last 7 days. Note the numbers: this is the baseline.
   - The figures come from local session history on this machine. Usage on other devices or on claude.ai is not
     included.
   - In the desktop app, the usage ring in the row under the prompt box shows plan usage.
2. Run `/context` in a typical long session. It shows what fills the context: history, images, MCP tools,
   CLAUDE.md, memory files, skills. In the desktop app, the context fill also shows in the session details
   (Context card), or in the usage ring popover in the earlier layout.
3. Make an inventory as one table:
   - **every subagent** in `~/.claude/agents/` and in each project's `.claude/agents/`: name, `model` (missing =
     inherits the main model), `effort`, `tools`, `skills:` preloads, and what it is actually used for;
   - **skills** (`~/.claude/skills/`, project skills): which ones were used in the last weeks;
   - **MCP servers** (`/mcp`): which ones each project really needs; whether `ENABLE_TOOL_SEARCH`, `alwaysLoad`,
     `CLAUDE_CODE_SUBAGENT_MODEL` or `model` are set in `settings.json` or its `env` block;
   - **the size of each CLAUDE.md** (user and project). It loads into the main session and into most subagents.
4. Show the user the table and the three biggest consumers. Only then propose changes.

## 3. Step 2 - Opus vs Sonnet vs Haiku: which model for which work

A rule of thumb to start from. The user decides.

| Model | For | Examples |
|---|---|---|
| `opus` | open problems, planning, judgement, work where a mistake is expensive | choosing an architecture, a design or style decision with the user, the final review before delivery, planning a larger job |
| `sonnet` | well-defined building: code with a clear spec, scripts, configuration, texts from a brief | a code change with tests, a build or export script (for example for Blender or an image editor), a blog draft from an outline, prompts for image or video tools |
| `haiku` | lookups, search, mechanical and repetitive work with a clear answer | "where is the file", renaming, checking a list, reading a log, checking sizes, counts or durations |

The docs say the same in short: Sonnet handles most coding tasks well and costs less than Opus; use Opus for
complex architecture and multi-step reasoning; for simple subagent tasks, use `model: haiku` (costs).

- **Every role names its model explicitly.** A missing `model` means the role inherits the main model, usually Opus.
- **Start low.** Raise a role's model only after it has failed twice on the lower one, and note why.
- **Effort:** `effort: low` for haiku roles, `medium` for most sonnet roles, higher only where a mistake is
  expensive. Valid levels are `low`, `medium`, `high`, `xhigh`, `max`; which ones work depends on the model. A
  role's `effort` overrides the session's effort, but not the `CLAUDE_CODE_EFFORT_LEVEL` variable (sub-agents).
- **The main session:** Opus is sensible for conversation and direction. If the user wants to cut further, two
  options. Both switch models, so switch only between tasks, never in the middle of one:
  - `/model opusplan` - Opus in plan mode, Sonnet when executing. Each plan-mode toggle is a model switch.
  - Sonnet as the default, and Opus on purpose for hard decisions.

## 4. Step 3 - subagents with an explicit model: check the existing ones first, then add new ones

If the user already has subagents: first give each one an explicit model according to section 3. Then merge
duplicates. Add new roles only where a gap remains. A proposal for a start (3-5 roles, adapted to what exists):

1. **`scout`** (haiku, effort low; tools `Read, Grep, Glob`) - template `vorlagen/agents/scout.md`
   - Job: finds things in large project folders and answers "where is X, what does file Y say".
   - Returns: paths and an answer of a few lines.
2. **`visual-check`** (sonnet; haiku for simple yes/no checks; tools `Read`) - template `vorlagen/agents/visual-check.md`
   - Job: gets image paths and a checklist (reference vs. result, errors, completeness). It looks at the images in
     its own context.
   - Returns: one line of verdict per item, never an image.
   - Why: large reads stay in the subagent's context window; only its summary comes back (context-window, costs).
     The docs do not mention images separately. Check it once with `/context` in the main session.
3. **Worker roles per tool, if useful** (sonnet, effort medium; tools limited to what the step needs)
   - Job: carry out defined steps, for example an export, a batch run, a build, a data import.
   - If the step runs through one MCP server, give the role only that server's tools, for example
     `tools: Read, mcp__<server>__*` (sub-agents).
   - Afterwards it reads the result back: file exists, size, counts.
   - Returns: what was done, the result, and what is unchecked.
4. **`prompt-writer`** (sonnet, or haiku for simple prompts) - template `vorlagen/agents/prompt-writer.md`
   - Job: writes prompts and task briefs (for image, video, audio or coding tools) from a short spec by the main
     session.
5. Optional **`researcher`** (sonnet; tools `WebSearch, WebFetch, Read`): tool documentation, versions, how-tos.

**The main session keeps:**
- the conversation with the user;
- direction and decisions;
- the briefs to the roles;
- the final judgement.

**Where:** roles for all projects in `~/.claude/agents/`; project-specific ones in `<project>/.claude/agents/`.
A project role with the same name replaces the user-level one. Claude Code watches both folders: a new or edited
role file is used from the next delegation on, without a restart (sub-agents).

**Role file fields** (sub-agents): `name`, `description`, `model`, `effort`, `tools`, `disallowedTools`, `skills`,
`mcpServers`, `maxTurns`, `omitClaudeMd`, and more.
- `omitClaudeMd: true` starts the role without the user, project and local CLAUDE.md files. Use it when the role
  gets everything it needs from the brief; otherwise leave it out. It needs Claude Code v2.1.271 or later.
- `skills:` loads the full content of each listed skill at the role's start. List a skill only when the role really
  needs it.

## 5. Step 4 - how the main session hands work to a role

- One goal per call:
  - **what** should exist;
  - **where** (paths, not contents);
  - **done when** (checkable);
  - **what comes back** (short, a fixed format).
- **Do not set `model` when starting a subagent.** The role file decides. Set it only on purpose, with a reason.
  Put this line into the user-level `~/.claude/CLAUDE.md` so every session keeps to it
  (`vorlagen/CLAUDE-md-lines.md`).
- Independent checks may run in parallel: one call per role, in one go.
- What a role returns is a summary. The main session asks back instead of reading the raw material itself.
- Not recommended here: `CLAUDE_CODE_SUBAGENT_MODEL_FORCE`. It forces one model on every subagent and ignores the
  `model` field in role files (sub-agents).

## 6. Step 5 - images, sessions, compact vs clear (often the bigger lever)

**Images:**
- **Size:** for checks, a long edge of 1280-1600 px is usually enough. When a detail matters, crop to the region
  instead of sending the full image. JPEG or WebP instead of large PNGs; check that text stays readable.
- **Images go to `visual-check`, not into the main chat.** Into the main chat only what the user and Claude must look
  at together.
- **Numbers instead of pictures:** check sizes, counts, resolutions, durations and similar values by script or tool,
  not by screenshot.

**Sessions:**
- One task = one session.
- At the end of a task: `/clear`. Use `/rename` first if the user wants to find the session again with `/resume`.
- Before a session would run for days: write a short handover file (state, decisions, next steps), `/clear`, then
  continue from that file. Guide 2 describes this in detail.
- Mid-task when the context is full: `/compact` with a hint on what has to stay, for example
  `/compact keep the open decisions and the file paths`.
- Run parallel chats only for really independent work.
- **Compact vs clear:** `/compact` reads the conversation it summarizes, so compacting a large context is itself a
  large request. `/clear` costs nothing. Use `/clear` when the next task does not need the old conversation (costs).

**MCP and skills:**
- **MCP:** with `/mcp`, switch off the servers a project does not need. Do it between sessions, not in the middle of
  a task. Leave `ENABLE_TOOL_SEARCH` unset (or `true`), and set `alwaysLoad: true` only where a server's tools are
  needed on every turn.
- **Skills:** remove what has not been used for weeks (after the user's yes, into a backup folder). Give subagents
  `skills:` only when the role really needs the skill.

## 7. Step 6 - decide with the user, build, measure

1. **Proposal to the user:**
   - the inventory;
   - a model per existing role;
   - 2-4 new roles;
   - the image and session rules;
   - the lines for CLAUDE.md.

   The user decides each point.
2. **Build:** create or edit the role files, put the lines into `~/.claude/CLAUDE.md`, switch off MCP servers per
   project.
3. **Check:**
   - Start each role once with a small real task. Run `/tasks`: it names the model (and effort) on each subagent's
     row (Claude Code v2.1.242 or later). Whether the desktop app shows the same is not confirmed in the docs; there,
     check the tasks pane.
   - Run `/context` in the main session after a `visual-check` round: do the images stay out?
4. **After one week:** run `/usage` again and compare with the baseline. Raise a role's model only where it failed
   twice; lower it where the role was clearly underused.

## Not part of this

- No orchestration, no hooks, no hierarchy of roles, no automation.
- Nothing gets deleted. Remove skills and roles only after the user's yes, and move them to a backup folder first.

## Sources (checked 2026-11-21)

Claude Code docs:
- https://code.claude.com/docs/en/sub-agents - role fields, model precedence, `omitClaudeMd`, `effort`, `skills`,
  MCP tools in roles, file watching, `/tasks`
- https://code.claude.com/docs/en/model-config - aliases, `opusplan`, effort, `CLAUDE_CODE_SUBAGENT_MODEL`
- https://code.claude.com/docs/en/costs - long sessions, `/usage`, model choice, subagents
- https://code.claude.com/docs/en/context-window - `/context`, what stays in a subagent
- https://code.claude.com/docs/en/prompt-caching - cache, model and effort switches, images, cache lifetime
- https://code.claude.com/docs/en/mcp - tool search, `ENABLE_TOOL_SEARCH`, `alwaysLoad`, `/mcp`
- https://code.claude.com/docs/en/commands - `/usage`, `/cost`, `/stats`, `/context`, `/clear`, `/compact`
- https://code.claude.com/docs/en/desktop - usage ring and context display in the desktop app

API docs:
- https://platform.claude.com/docs/en/build-with-claude/vision - image token formula and size limits

Not confirmed in the docs:
- that the desktop app's Code tab shows the model of each subagent the way `/tasks` does in the terminal.

The docs change. If a command or field behaves differently, trust the current docs over this file.
