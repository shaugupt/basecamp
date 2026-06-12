# basecamp

An agent skill that bootstraps a project into an agent-ready knowledge base.

It scaffolds an `AGENTS.md` entry point, a read-first knowledge-base index, project trackers, and a session handoff log so future agents can onboard quickly without re-discovering the same context.

## Why basecamp?

Every fresh agent session starts from zero: it re-greps the repo, re-derives the architecture, and re-asks questions you already answered. basecamp pays that cost once and leaves a durable, grounded trail for everyone who follows.

- **No re-discovery tax.** New agents land oriented — entry point, folder map, and "start here" sequence are already written, so they skip the expensive re-exploration every session.
- **Grounded, not guessed.** Every KB fact is tagged `[Verified: <source>]` or `[Guess: needs check]` and mapped to a source, so answers are traceable instead of hallucinated.
- **Context-stays-lean.** Only the thin entry layer (`AGENTS.md` + `kb/INDEX.md`) auto-loads; everything else is referenced on demand, keeping the context window uncluttered.
- **Multi-agent by default.** `AGENTS.md` is read natively by Cursor, Codex, Copilot, Gemini CLI, and more, with a `CLAUDE.md` bridge for Claude Code — one source of truth, zero drift.
- **Continuity across sessions.** A per-session handoff log means the next agent (or you, next week) knows what just happened and what's open.
- **Set up once, benefit forever.** Run it a single time per project; re-run only to add features or re-survey after a major change.

## What it sets up

- **`AGENTS.md` entry point** — folder map, grounding rules, Definition of Done, and conventions in one read-first file.
- **Read-first rule** — `.cursor/rules/read-agents-first.mdc` forces the entry read at the start of every session.
- **Knowledge base** — `kb/INDEX.md` (read-first map) plus `kb/SOURCE-INDEX.md` mapping each topic to its source and confidence.
- **Project trackers** — `project-management/` files for current focus, decisions, open questions, and action items.
- **Session handoff log** — `agent-sessions/` with a per-session entry and rolling index.
- **Claude bridge** — `CLAUDE.md` symlinked to `AGENTS.md` (with a pointer-template fallback) so Claude Code stays in sync.
- **Archetype bundles** — optional extras tuned to your project type (Knowledge/Q&A, Strategy/PM, Engineering/code, Research/RAG).
- **Optional transcript archive** — opt-in `agent-transcript-archive` companion for project-local transcript history across OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat.

## Install

basecamp is a single `SKILL.md` (plus `templates/` and reference docs) that each agent surface loads from its global skills folder. Pick the home that matches your agent:

- Cursor → `~/.cursor/skills/`
- Claude Code → `~/.claude/skills/`
- Codex → `~/.codex/skills/`

### Option A — clone + symlink (recommended)

Keeps the repo somewhere you control and lets `git pull` update the skill in place. Symlink it into each surface you use.

```bash
git clone https://github.com/shaugupt/basecamp.git ~/code/basecamp

# Link into whichever agent surfaces you use:
mkdir -p ~/.cursor/skills && ln -s ~/code/basecamp ~/.cursor/skills/basecamp
mkdir -p ~/.claude/skills && ln -s ~/code/basecamp ~/.claude/skills/basecamp
```

### Option B — clone directly into a skills directory

Simplest, but the repo lives inside the agent's skills folder.

```bash
mkdir -p ~/.cursor/skills
git clone https://github.com/shaugupt/basecamp.git ~/.cursor/skills/basecamp
```

After installing, restart the agent (or reload the skills surface) so it picks up the new skill.

## Use

Ask an agent to run `basecamp` (or `/basecamp`) when setting up a new or existing repo for agent collaboration. The agent will:

1. Survey the repo — never assume greenfield; extend what already exists.
2. Interview you in one batch — project name, purpose, owner, archetype, authoritative sources, wired tools/MCPs, and whether to enable transcript archiving.
3. Scaffold the Core + Recommended files, plus the bundle for your archetype, and link each from `AGENTS.md`.
4. Ground and hand off — verify citation/handoff rules, optionally wire the transcript-archive companion, and write the first `agent-sessions/` entry.
5. Verify — links resolve, no leftover placeholders, and the KB and source indexes are complete.

During setup, basecamp can offer the optional `agent-transcript-archive` companion skill for project-local transcript history across OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat.

Companion repo: https://github.com/shaugupt/agent-transcript-archive

## License

MIT. See `LICENSE`.
