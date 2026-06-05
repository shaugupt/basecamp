# basecamp — Feature Reference

Tiers: **Core** (always) · **Recommended** (on by default) · **Optional** (by archetype / opt-in).
"Install" = the file or `AGENTS.md` section that carries the feature.

## Core
| Feature | Install |
|---|---|
| Entry point | `AGENTS.md` |
| Read-first rule | `.cursor/rules/read-agents-first.mdc` |
| Folder map | `AGENTS.md` §3 |
| KB index (read-first) | `kb/INDEX.md` |
| Source index + citations | `kb/SOURCE-INDEX.md` + `AGENTS.md` §5 |
| Verified/Guess labeling | `AGENTS.md` §5 |
| Session handoff | `agent-sessions/` |

## Recommended (on by default)
| Feature | Install |
|---|---|
| Do-Not-Load list | `AGENTS.md` §4 |
| Start-Here sequence | `AGENTS.md` §1 |
| Claude bridge | symlink `CLAUDE.md`→`AGENTS.md` (pointer fallback) — see below |
| Definition of Done | `AGENTS.md` §7 |
| Trackers | `project-management/{CURRENT_FOCUS,DECISIONS,OPEN_QUESTIONS,ACTION_ITEMS}.md` |
| Conventions | `AGENTS.md` §6 |
| Capability matrix | `CAPABILITY-MATRIX.md` |
| Q&A→KB promotion | `AGENTS.md` §6 |
| Drift check | `AGENTS.md` §6 |
| Jira/Confluence approval gates | `AGENTS.md` §5 |

## Claude Code / Cowork bridge (why & how)

`AGENTS.md` is read natively by Codex, Cursor, Copilot, Gemini CLI, Windsurf, Aider, Zed, Warp, etc. **Claude Code is the holdout** — it reads `CLAUDE.md`, not `AGENTS.md` (anthropics/claude-code#6235, ~3.6k upvotes; no native support as of 2026). A `CLAUDE.md` that only *points to* or `@AGENTS.md`-imports is followed **less reliably** than inline content (#35295).

- **Default:** symlink so Claude loads the identical file, zero drift — `ln -s AGENTS.md CLAUDE.md`, then verify it resolves.
- **Fallback** (Windows without symlink rights, zip/copy flows, git not preserving links): copy `templates/CLAUDE.md.template` — a strong imperative pointer.
- **Monorepo:** repeat the bridge next to each nested `AGENTS.md`.
- **Cowork:** Claude Cowork (Claude Desktop) runs the same engine and works from the folder; the root bridge covers it. Nested-file discovery is `[Unverified]` — verify if you depend on it.
- **Other agents:** no mirror — keep `AGENTS.md` canonical.

## Optional — by archetype
| Archetype | Adds | Notes |
|---|---|---|
| Knowledge / Q&A | maturity grounding, source router, issue capture | answer reliability |
| Strategy / PM | meeting notes, action-tracker MCP | tracker-heavy |
| Engineering / code | CHANGELOG, no-milestone framing, git hooks | end-state framing |
| Research / RAG | RAG cache, raw→wiki compile | heavy; see provenance |

Auto-detect (any archetype): Obsidian (`.obsidian/` → wikilinks/graph); nested `AGENTS.md` (monorepo subtrees, per the agents.md standard).

## Review before enabling (provenance)
NOT installed by default. Enable only after you confirm you understand and own the pattern.

| Feature | Origin | Guidance |
|---|---|---|
| Maturity-tier grounding + confidence scoring | internal workspace pattern | Adopt the concept (tag facts shipped / in-dev / planned, caveat the rest). Rebuild a simple version; do not copy bespoke scoring formulas. |
| Agentic RAG cache (cache-first, TTL, SSO fetch) | internal workspace pattern | Heaviest feature. Prefer a separate opt-in sub-skill over baking it in. |
| CHANGELOG (Keep a Changelog) | public standard | Safe to adopt as your own. |
| Subagent personas | generic agent pattern | Safe to adopt the pattern. |
| Source router (local vs MCP) | internal workspace pattern | Confirm ownership; concept is generic and safe to rebuild. |
| No-Milestone-Leakage | internal workspace pattern | Confirm ownership; generic engineering practice. |

## Sub-skill candidates
Heavy features better shipped as their own skills that basecamp installs on request, rather than bloating one skill: `maturity-grounding`, `agentic-rag-cache`, `wiki-compiler` (raw→wiki + drift).

## Lineage (where each pattern was observed)
- Public: agents.md (entry point, native AGENTS.md + Claude bridge, nested AGENTS.md, Definition of Done), Keep a Changelog, Karpathy LLM-wiki (read-first index, source provenance, drift, Q&A→KB, raw→wiki).
- Quarantined (others' / unconfirmed): internal workspace patterns — see provenance table above.
