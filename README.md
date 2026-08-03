# hive-board — moved

> **This repository is archived. The hive-board viewer now ships inside the HIVE plugin package.**
>
> **→ [lohnn/evolutional_agent_structure](https://github.com/lohnn/evolutional_agent_structure)**

The code here is frozen at the point of the move (2026-08-03) and is kept only as a historical
record. Do not open issues or PRs against it, and do not run it — the container entrypoint that
used to launch the viewer from this repo now launches it from the plugin package.

## What this was

A kanban board layer over [HIVE](https://github.com/lohnn/evolutional_agent_structure)'s on-disk
state — visualising and driving **work items** through Backlog → Todo → In Progress → Done, bound
to real session, dream, and message state rather than a parallel bookkeeping file.

It still is that. It just lives somewhere else now.

## Why it moved

This repo was `private: true` and depended on the plugin via `file:../evolutional_agent_structure`,
which is unpublishable — meaning the viewer could not be installed on any other machine at all.
Absorbing it into the plugin package was the enabling change, not an optimisation: HIVE and the
board now install as **one dependency**.

It cost nothing to do. The viewer's entire runtime surface was the plugin's own modules, Node
built-ins, and `bun:sqlite` — zero third-party runtime dependencies, so nothing was added to what
a HIVE user installs.

## Where everything went

| Was (here) | Now (plugin package) |
|---|---|
| `src/` | `src/board-viewer/` |
| `src/server.ts` | `src/board-viewer/server.ts` (bin: `hive-board`) — plus `index.ts`, a side-effect-free programmatic entry |
| `test/` | `test/board-viewer/` |
| `fixtures/` | `fixtures/board/` |
| `docs/` (DESIGN, SCHEMA, OPEN-QUESTIONS) | `docs/board-viewer/` |
| `AGENTS.md` | `docs/board-viewer/OVERVIEW.md` |

Run it from the plugin repo root with `bun run board`, or as the `hive-board` bin.

## The history is not lost

All 23 commits of this repo were imported with `git subtree add`, so they are genuine ancestors of
the plugin repo's HEAD — not a flattened copy. From a checkout of the plugin repo:

```sh
git merge-base --is-ancestor 5231176 HEAD && echo "full history present"

# Paths are the PRE-MOVE ones (src/config.ts, not src/board-viewer/config.ts):
git log 12ae0c5^2
git log 12ae0c5^2 -- src/config.ts
```

Note that `git log --follow src/board-viewer/<file>` returns **nothing**, and that is not history
loss — history simplification will not traverse a subtree merge. Use the `12ae0c5^2` form above.

The same instructions live in `docs/board-viewer/OVERVIEW.md` in the plugin repo, which is the
maintained copy.
