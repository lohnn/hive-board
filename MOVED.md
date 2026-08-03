# ⚠️ MOVED — this repo is no longer the source of truth

**Date:** 2026-08-03
**New home:** `projects/evolutional_agent_structure/src/board-viewer/`

The hive-board viewer was **absorbed into the HIVE plugin package** so that HIVE and the board
install on another machine as **one npm dependency**. That was impossible from here: this repo is
`private: true` and depends on the plugin via `file:../evolutional_agent_structure`, which cannot
be published.

## Do not edit anything in this directory

The code here is a **frozen copy**. Edits made here will not ship, will not be tested by CI, and
will silently diverge from the real viewer — the exact "stale thing that actively lies" failure
mode the board itself was designed to avoid (W-030, I-214).

| You want | Go to (in `projects/evolutional_agent_structure/`) |
|---|---|
| Viewer source | `src/board-viewer/` |
| Programmatic entry | `src/board-viewer/index.ts` (export `./board-viewer`) |
| Executable entry | `src/board-viewer/server.ts` (bin: `hive-board`) |
| Tests | `test/board-viewer/` — run `bun test` from the plugin repo root |
| Fixtures | `fixtures/board/` |
| DESIGN / SCHEMA / OPEN-QUESTIONS | `docs/board-viewer/` |
| Deploy guide | `docs/board-viewer/deploy/` |

## Nothing was lost

All 23 commits of this repo's history were imported with `git subtree add`, so they are genuine
ancestors of the plugin repo's HEAD — not a flattened copy. In the plugin repo:

```sh
git merge-base --is-ancestor 5231176 HEAD && echo "full history present"

# Browse the imported history (paths are the PRE-MOVE ones, e.g. src/config.ts):
git log 12ae0c5^2
git log 12ae0c5^2 -- src/config.ts
```

`git log --follow <new-path>` stops at the reshape commit — that is a known limit of history
simplification across a subtree merge, not missing data. Use the `12ae0c5^2` form above.

The GitHub remote `lohnn/hive-board` still exists and is untouched by this move.

## Why this directory still exists at all

The container entrypoint currently launches the board from
`/workspace/projects/hive-board/src/server.ts`, and that entrypoint lives in **host-side container
config, outside `/workspace`** — it cannot be edited from inside this container. Deleting this
directory now would break the running board at the next container restart with no in-container fix.

**To finish the retirement** (user action, in this order):

1. Replace the hive-board block in your entrypoint with the updated fragment at
   `projects/evolutional_agent_structure/docs/board-viewer/deploy/entrypoint-fragment.sh`
   (it points at the plugin repo and keeps `--host 0.0.0.0`, which is now required —
   the viewer defaults to loopback).
2. Rebuild / restart the container and confirm the board answers on `:4400`.
3. Then delete `projects/hive-board/` entirely.
