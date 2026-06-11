# Agent Transcript Archive Companion Skill Design

Date: 2026-06-10
Last updated: 2026-06-11
Status: Companion skill built; basecamp integration added
Project: agent-transcript-archive companion skill with basecamp integration

## Summary

Create a standalone **agent-transcript-archive** companion skill. The skill creates a project-local archive of normalized agent transcripts from multiple agent applications so a project has one history source regardless of whether prior work happened in OpenCode, Claude Code, Cursor, Codex, or VSCode GitHub Copilot Chat.

The archive is local-only by default. It is meant to preserve project-relevant interactions, help users find prior conversations, and give future agents a searchable history source without forcing full transcripts into the normal read-first context.

Basecamp should not own transcript sync logic, app adapters, storage discovery, or normalization. The companion skill is published at `https://github.com/shaugupt/agent-transcript-archive`. Basecamp can ask: "Enable transcript archive?" If the user opts in, basecamp points to the companion skill, offers to install it from GitHub, and adds only the minimal project instructions needed to use it.

## Goals

- Keep all project-relevant agent transcripts in one project-local location.
- Keep basecamp lightweight by moving transcript archival into a separate companion skill.
- Use the published `agent-transcript-archive` GitHub-backed skill as the transcript archive implementation.
- Support OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat in the first version.
- Store normalized Markdown transcripts, not raw source exports.
- Include useful metadata: date, time, app, model when detectable, source session identifier when available, project root, and match confidence.
- Keep full transcripts out of git by default.
- Preserve `agent-sessions/` as the project history area while separating curated handoffs from full transcripts.
- Let future agents discover transcript history through a lightweight index and open full transcripts only when the task needs that history.
- Define a small integration contract so basecamp can later offer to install and enable the companion skill.

## Non-Goals

- No background daemon or scheduled sync in v1.
- No raw transcript backup in v1.
- No redaction or secret filtering in v1.
- No support for every VSCode agent extension; v1 targets GitHub Copilot Chat in VSCode only.
- No import of ambiguous sessions that cannot be confidently tied to the current project.
- No external writes or cloud sync.
- No transcript adapter implementation inside basecamp.
- No transcript sync implementation inside basecamp; basecamp integration stays limited to opt-in installation guidance and generated project instructions.

## Skill Boundary

`agent-transcript-archive` owns the full transcript archival capability:

- Transcript sync command.
- App-specific adapters.
- macOS source discovery.
- Project matching.
- Markdown normalization.
- Transcript index updates.
- Local-only gitignore rules for full transcript archives.
- User-facing docs and warnings for transcript privacy.

`basecamp` owns only optional discovery and integration:

- Ask whether the user wants transcript archiving during basecamp setup.
- If yes, offer to pull/install `agent-transcript-archive` from GitHub.
- Add a short pointer in generated `AGENTS.md` telling agents where the transcript archive index lives and when to run the companion sync command.
- Avoid embedding adapter details or transcript sync internals.

## User Flow

The standalone companion skill can be used directly, and basecamp can offer it during setup.

The companion skill flow is:

1. User installs or invokes `agent-transcript-archive` for a project.
2. The skill creates the transcript archive structure, gitignore rules, and usage instructions.
3. A user or agent works in one of the supported apps.
4. At session end, project instructions tell the agent to run the transcript sync command from the project root.
5. The user can also run the sync command on demand.
6. The sync command checks each supported app adapter for sessions confidently associated with the current project.
7. Matching sessions are normalized into Markdown files under `agent-sessions/transcripts/`.
8. `agent-sessions/transcripts/INDEX.md` is updated with one row per imported transcript.
9. Future agents read only the lightweight indexes by default and open full transcripts only when historical detail is needed.

The basecamp integration flow is:

1. Basecamp asks whether to enable transcript archiving.
2. If the user opts in, basecamp offers to pull/install `agent-transcript-archive` from GitHub.
3. Basecamp adds only a small integration section to `AGENTS.md` and delegates setup/sync details to the companion skill.

## File Layout

The feature extends the existing `agent-sessions/` area:

```text
agent-sessions/
  INDEX.md
  _TEMPLATE.md
  YYYY-MM-DD-handoff-slug.md
  transcripts/
    INDEX.md
    YYYY/
      YYYY-MM-DD-HHMM-app-session-slug.md
```

`agent-sessions/INDEX.md` remains the normal read-first handoff index. It may link to `agent-sessions/transcripts/INDEX.md`, but it must warn agents not to open full transcripts unless the task requires historical detail.

`agent-sessions/transcripts/INDEX.md` is the transcript catalog. Its table includes:

- Date/time
- App
- Model
- Project match confidence
- Summary or title
- Transcript path

Full transcript files are ignored by git by default. Curated handoff files remain normal project files and may be committed.

## Transcript Format

Each transcript is a Markdown file with metadata at the top followed by normalized conversation content.

Required metadata fields:

```yaml
app: opencode | claude-code | cursor | codex | vscode-copilot
model: detected-or-unknown
started_at: timestamp-or-unknown
ended_at: timestamp-or-unknown
source_session_id: detected-or-unknown
project_root: path
project_match: confident
```

If an app does not expose a value, the field uses `unknown`. The sync must not guess missing metadata.

The transcript body should be readable Markdown. It should preserve speaker roles, message order, timestamps when available, and tool/action summaries when available from the source. The format should be stable so repeated syncs do not create noisy diffs.

## Read-First Boundary

The archive must not make full transcripts part of the normal first-load context.

Generated instructions should say:

- Read `agent-sessions/INDEX.md` during normal onboarding.
- Use `agent-sessions/transcripts/INDEX.md` to find relevant transcript history.
- Open full transcript files only when the task needs prior conversation details or when searching project history.

This keeps onboarding lightweight while preserving a deeper history source.

## Sync Architecture

The sync command is an adapter-based local tool.

Core pieces:

- **Sync runner:** runs from the project root, loads supported adapters, coordinates import, writes transcripts, and updates indexes.
- **App adapters:** isolate source discovery and parsing for one app each.
- **Project matcher:** accepts only sessions confidently associated with the current project.
- **Markdown normalizer:** converts app-specific message records into the canonical Markdown transcript format.
- **Index updater:** updates `agent-sessions/transcripts/INDEX.md` after successful transcript writes.

The runner must continue when one adapter fails or is unavailable. It should print a per-adapter report rather than failing the whole sync.

Example report:

```text
OpenCode: imported 2
Claude Code: no confident sessions found
Cursor: adapter unavailable; source path not found
Codex: imported 1
VSCode Copilot: skipped; unsupported local format
```

## Project Matching

The user selected current-project-only behavior with confident scanning across supported apps.

For v1, a session is eligible only when the adapter can confidently associate it with the current project. Confidence may come from:

- Explicit workspace root metadata.
- Current working directory metadata.
- Repo path metadata.
- App-specific workspace fields.
- A direct source path that matches the current project root.

Ambiguous sessions are skipped. The archive should be incomplete rather than polluted with unrelated project conversations.

## Adapter Requirements

All v1 adapters target macOS discovery paths first, while the design should leave room for Linux and Windows paths later.

Adapters:

- **OpenCode:** discover local session history, import sessions tied to the current project root when metadata allows it, and detect model/date when available.
- **Claude Code:** discover local project/session history and import sessions confidently tied to the current repo path.
- **Cursor:** discover accessible Cursor workspace or chat history. If Cursor's local format is opaque or unstable, report limited support instead of guessing.
- **Codex:** import local Codex sessions associated with the current project when session metadata exposes working directory or repo information.
- **VSCode Copilot Chat:** target GitHub Copilot Chat in VSCode. Do not treat this as support for all VSCode agent extensions.

Each adapter should make its support level explicit in sync output.

## Error Handling

The sync should favor partial success and clear reporting.

Rules:

- Missing app store: report adapter unavailable.
- Unknown schema: report unsupported format.
- Ambiguous project match: skip the session.
- Duplicate transcript: do not create a second transcript entry.
- Changed source session: update or replace the existing normalized transcript deterministically.
- Unknown model/date: write `unknown`.
- Index update failure: avoid leaving transcript files and the index inconsistent where practical.

The command should exit successfully when at least one adapter completes normally, even if other adapters are unavailable. It should exit with failure only when the core runner cannot operate.

## Privacy And Safety

The default is local-only archival.

- Full transcript files are gitignored.
- No redaction is applied in v1.
- Documentation must warn that transcripts may contain secrets, personal data, proprietary code, prompts, tool outputs, and private reasoning context.
- The feature performs no external writes.
- Users who want to commit transcripts later must opt in manually by changing ignore rules.

## Testing Strategy

Use fixture-driven tests for predictable behavior.

Required coverage:

- Adapter tests with sample session records for each supported app.
- Project matcher tests for confident and ambiguous project association.
- Markdown normalizer tests for required metadata and stable formatting.
- Index updater tests for creating and updating `agent-sessions/transcripts/INDEX.md`.
- Idempotency tests proving repeated sync does not duplicate transcript entries.
- Failure tests for missing app stores, unknown schemas, and unavailable adapters.

## Companion Skill Deliverables

The standalone `agent-transcript-archive` skill includes:

- `SKILL.md` describing when to use the companion skill.
- Setup instructions for adding transcript archiving to an existing basecamp-managed or non-basecamp project.
- `agent-sessions/transcripts/INDEX.md` template.
- Instructions that can be inserted into `AGENTS.md` explaining when to run the sync and how to read transcript history.
- `.gitignore` entries for full transcript files.
- Documentation that distinguishes curated handoffs from full transcripts.
- A sync command or script installed in a predictable project-local location.
- Adapter implementation and fixtures for OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat.

## Basecamp Integration

Basecamp integration is limited to:

- Ask "Enable transcript archive?" during setup.
- If the user opts in, offer to pull/install `agent-transcript-archive` from GitHub as a companion skill.
- Add the companion skill's short AGENTS integration snippet.
- Keep the basecamp skill itself free of transcript adapter code, sync code, and source discovery logic.

## Acceptance Criteria

- `agent-transcript-archive` can describe and install the optional transcript archive feature in a project.
- Generated project instructions tell agents to run transcript sync at session end when transcript archiving is enabled.
- Transcript archives live under `agent-sessions/transcripts/`.
- Transcript files are normalized Markdown with required metadata.
- The transcript index is lightweight and read-first safe.
- Full transcript files are gitignored by default.
- The sync command includes adapter slots for OpenCode, Claude Code, Cursor, Codex, and VSCode GitHub Copilot Chat, with each adapter reporting imported, unavailable, unsupported, or no confident sessions found.
- Ambiguous sessions are skipped rather than imported.
- Re-running sync is idempotent.
- Basecamp integration stays limited to opt-in install guidance and generated AGENTS instructions; sync logic and adapters remain in the companion skill.
