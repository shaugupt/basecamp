---
name: basecamp
description: Bootstraps a project into an agent-ready knowledge base — scaffolds AGENTS.md, a read-first KB index with cited sources, project trackers, and a session handoff log so new agents onboard fast and answer reliably without re-discovery. Use when setting up a new or existing repo for AI agents, or when the user runs /basecamp.
disable-model-invocation: true
---

# basecamp

Set up a project once so every future agent arrives oriented, grounded, and aware of recent work — without re-discovery and without bloating context.

## When to run
- Once, at the start of working a project (new or existing).
- Re-run only to add features or re-survey after a major change.

## Rules
- Only the entry layer (`AGENTS.md` + `kb/INDEX.md`) is meant to auto-load. Reference everything else on demand.
- Copy files from `templates/` into the target repo, fill `{{PLACEHOLDERS}}`, delete unused rows/sections.
- Write nothing you can't ground. Tag facts `[Verified: <source>]` or `[Guess: needs check]`.
- Additive by default; never rewrite history. Newer source wins on conflict.
- Keep every generated file terse and imperative — agents read structure, not prose.

## Workflow

### Phase 0 — Survey (never assume greenfield)
List the repo's top 2 levels. Note existing docs, code, data, `.obsidian/`, monorepo subprojects, and wired MCPs. Extend what exists; don't duplicate it.

### Phase 1 — Interview (ask, don't assume)
Confirm in one batch: project name + one-line purpose + owner; archetype (map below); authoritative sources vs raw/scratch; wired tools/MCPs.

### Phase 2 — Scaffold
Install **Core + Recommended** always. Add the **archetype bundle**. Auto-enable detected extras (Obsidian, nested `AGENTS.md`). Copy each template, fill placeholders, and link it from `AGENTS.md`.

**Claude bridge:** other agents (Codex, Cursor, Copilot, Gemini CLI, Windsurf…) read `AGENTS.md` natively — no mirror. Only Claude Code / Cowork need it: symlink `CLAUDE.md → AGENTS.md` (`ln -s AGENTS.md CLAUDE.md`) and verify it resolves; if symlinks aren't supported, copy `templates/CLAUDE.md.template` (strong pointer) instead. Repeat next to each nested `AGENTS.md`.

### Phase 3 — Ground + hand off
Verify `AGENTS.md` carries the grounding + handoff rules (citations, Verified/Guess, newer-wins, write-approval gates). Write the first `agent-sessions/` entry.

### Phase 4 — Verify
- `AGENTS.md` ≤ ~400 lines; links resolve; no leftover placeholders.
- `kb/INDEX.md` lists every KB doc; `kb/SOURCE-INDEX.md` maps each topic → source + confidence.
- One handoff entry exists.

## Feature tiers

**Core — always install**

| Install | Carries |
|---|---|
| `AGENTS.md` | Entry point, folder map, grounding + handoff rules |
| `.cursor/rules/read-agents-first.mdc` | Forces the read every session |
| `kb/INDEX.md` | Read-first map of the KB |
| `kb/SOURCE-INDEX.md` | Topic → source + confidence |
| `agent-sessions/` | Per-session handoff log + rolling index |

**Recommended — on by default**
Do-Not-Load list · Start-Here sequence · Claude bridge (`CLAUDE.md`↔`AGENTS.md`) · Definition of Done · `project-management/` trackers · conventions · `CAPABILITY-MATRIX.md` · Q&A→KB promotion · drift check · Jira/Confluence approval gates.

**Optional — by archetype**

| Archetype | Adds |
|---|---|
| Knowledge / Q&A | maturity grounding, source router, issue capture |
| Strategy / PM | meeting notes, action-tracker MCP |
| Engineering / code | CHANGELOG, no-milestone framing, git hooks |
| Research / RAG | RAG cache, raw→wiki compile (heavy) |

Auto-detect: Obsidian (`.obsidian/`), nested `AGENTS.md` (monorepo).

## Details
Full feature list, archetype bundles, sub-skill candidates, and provenance/review notes: [reference-features.md](reference-features.md).
