# basecamp

An agent skill that bootstraps a project into an agent-ready knowledge base.

It scaffolds an `AGENTS.md` entry point, a read-first knowledge-base index, project trackers, and a session handoff log so future agents can onboard quickly without re-discovering the same context.

## Install

Symlink this skill directory into the global skills folder for each agent surface that should use it.

```bash
ln -s /Users/shaugupt/Personal/Skills/basecamp ~/.cursor/skills/basecamp
```

## Use

Ask an agent to run `basecamp` when setting up a new or existing repo for agent collaboration.

During setup, basecamp can offer the optional `agent-transcript-archive` companion skill for project-local transcript history across OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat.

Companion repo: https://github.com/shaugupt/agent-transcript-archive

## License

MIT. See `LICENSE`.
