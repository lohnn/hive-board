---
description: "ARCHIVED / MOVED 2026-08-03 — the hive-board viewer now lives inside the HIVE plugin package at projects/evolutional_agent_structure/src/board-viewer/. This directory is a frozen copy; do not edit it. See MOVED.md."
---
# hive-board — ⚠️ MOVED

> **This is no longer the source of truth.** The viewer was absorbed into the HIVE plugin package
> on 2026-08-03 so HIVE + board ship as ONE npm dependency.
>
> **Source now lives at `projects/evolutional_agent_structure/src/board-viewer/`.**
> Tests: `test/board-viewer/`. Docs: `docs/board-viewer/`. Read **[MOVED.md](MOVED.md)** first —
> it explains where everything went, how to reach the preserved git history, and why this
> directory still exists (the container entrypoint still points here).
>
> **Do not edit code in this directory.** It is frozen and will silently diverge from the
> shipping viewer.

---

*The original charter is kept below for context only. Its paths are stale.*

## Status

**Build phase.** SCHEMA ratified v1.0 (2026-07-10). The canonical design lives in `docs/`:

- [`docs/DESIGN.md`](docs/DESIGN.md) — architecture, data model, lifecycle, write-authority
- [`docs/SCHEMA.md`](docs/SCHEMA.md) — the work-item contract (frontmatter fields, column↔status map, ID scheme)
- [`docs/OPEN-QUESTIONS.md`](docs/OPEN-QUESTIONS.md) — unresolved decisions to work through next

Read `DESIGN.md` first. It takes precedence over any assumptions.

## The one constraint that shapes everything

**A work item in `In Progress` (and onward) IS one full HIVE coordinator session** — the top-level
chat where you talk to HIVE, *not* a subagent. Subagent/capability dispatches are just activity
*within* that session.

- In Backlog/Todo, an item is a free-floating idea — no session exists yet; any chat can iterate on it.
- It reaches In Progress one of three ways: **bind** the current HIVE session to it, **auto-register**
  when a chat runs `/awaken`, or **create** a fresh top-level session for it.
- That owning coordinator session is the single canonical writer of the item's status and subtasks.
- **Only HIVE (awakened) sessions appear on the board.** Non-HIVE chats are never shown.
- **Every HIVE session auto-appears** on the board (In Progress) even if it never started as a card.
- Re-opening a Done card **re-attaches to the same coordinator session** (by id), preserving context.
- `Done` is reached when the owning session's **dream (DRM) reaches `COMPLETE`** (manual "done without
  dream" allowed, but badged). Once Done, the real session can be **archived without losing the record**.

The board's core purpose: make your opencode session list legible — which HIVE sessions are actively
being worked (or paused-but-resuming) vs. finished vs. mere ideas-not-yet-started.

## Core principle: derived from ground truth, never a second source

The board must **read HIVE's real state**, never maintain a parallel bookkeeping that can drift.
A stale hand-edited tracking file actively lies (dream W-030). The board avoids this by deriving
column position from real signals — session ownership, DRM completion — and using an append-only,
single-writer discipline for the state it *does* own (dream I-105).

## Tech (settled — see OPEN-QUESTIONS Q4/Q6)

TypeScript web app (runs under Bun; web layer TBD in Phase 1) that **renders read-only** from
`.opencode/` files and the work-item cache — but is **not a read-only tool**: board-side transitions
(create/start+auto-awaken, pause, demote, manual done) are executed directly by the app through a
**shared transition module** owned by hive-infra and exported from the plugin repo (locked storage,
SCHEMA §4a). In-session transitions (bind, awaken auto-register) live in the HIVE plugin as
`hive_board_*` **plugin custom tools** (built with `tool()` from `@opencode-ai/plugin`, same
extension point as the existing `hive_dream_*` tools — *not* MCP tools). No database.
Markdown + YAML on disk, matching HIVE conventions exactly.

## Relationship to the HIVE plugin repo

Transitions that spawn/resume/complete sessions belong in the HIVE plugin
(`projects/evolutional_agent_structure/`) as `hive_board_*` tools, owned by the **hive-infra**
capability. The viewer app lives here in `projects/hive-board/`. These are two deliverables with a
contract (the work-item schema in `docs/SCHEMA.md`) between them.
