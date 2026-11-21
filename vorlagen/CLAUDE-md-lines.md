# Lines for the user-level CLAUDE.md

Put the lines you choose into `~/.claude/CLAUDE.md` (on Windows: `%USERPROFILE%\.claude\CLAUDE.md`). That file loads
in every project. Keep it short: every line is sent with every request.

Add only the lines the user has said yes to.

## The lines

```markdown
## Working rules
- Answer in the language I write in. Keep role files, templates and file contents in English unless I ask otherwise.
- Do not set `model` when starting a subagent. The role file decides. Set it only on purpose, and say why.
- Hand well-defined work to the roles in ~/.claude/agents/ and .claude/agents/: lookups to `scout`, image checks to
  `visual-check`, prompts and briefs to `prompt-writer`.
- Images for checks go to `visual-check`, not into the main chat.
- Knowledge goes into files when it is settled (STAND.md, decisions.md, memory.md). When the context gets full or at
  the end of a section: update STAND.md, then start a new chat.
- Change nothing in my setup before I have said yes to that change.
```

## Alternative for the language: the `language` setting

Claude Code has a `language` setting. It makes Claude respond in that language by default. Claude Code passes the
value as written to Claude; it does not check the spelling. The same value also sets the language of auto-generated
session titles and of voice dictation. It can go into any settings file, for example `~/.claude/settings.json`:

```json
{
  "language": "german"
}
```

Use either the CLAUDE.md line or the setting:
- The CLAUDE.md line follows the language the user writes in, and keeps files in English.
- The `language` setting fixes one response language. If it is set, the "keep files in English" part still belongs in
  CLAUDE.md.

Source (checked 2026-11-21): https://code.claude.com/docs/en/settings-reference (entry `language`).
