# tiny-chess-01 — 4 KB chess engine

A single-file (`main.py`, 4,043 bytes) Python 3.13 stdlib-only chess engine
for the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## How it plays

0x88 board, negamax + alpha-beta with iterative deepening on a fixed
50,000-node budget per move, PVS at the root, transposition table,
killer moves, MVV-LVA ordering, quiescence search (captures/promotions,
all evasions in check), null-move pruning, repetition = draw.
Evaluation: material + piece-square tables + bishop pair + king table +
passed-pawn bonus (most-advanced pawn per file, blocked by enemy pawns
ahead on neighbouring files). Fully deterministic: fixed node budget, no clock, no randomness, no
model calls. Talks to the game env at `MAKEFUN_ENV_URL` via
`POST /game`, `POST /move` (UCI) and `POST /resign`.

## Local practice

```
# from the kit directory:
MAKEFUN_ENV_DATA_DIR=<kit>/practice/trial-0/input/env \
MAKEFUN_STATE_DIR=/tmp/chess-state-0 MAKEFUN_ENV_PORT=18080 \
node program/env/server.mjs &
MAKEFUN_ENV_URL=http://127.0.0.1:18080 timeout 1700 python3 main.py
```

## Practice results (local, official kit scorer)

| Trial | Engine | W-D-L | Elo |
|-------|--------|-------|-----|
| trial-0 | cut engine, unpacked, 50k nodes | 3-1-4 | **1,326** |
| trial-1 | `main.py` (packed), 50k nodes | 3-0-5 | **1,253** |
| trial-2 | `main.py` (packed), 75k nodes | 3-0-5 | **1,253** |

All trials beat the three depth anchors (depth-2/3/4) and are above the
1,154 final-entry floor (mean 1,277). Best single trial (1,326) is just
above the 1,325 sunfish baseline mean; the mean sits below it, so a
prize is not assured — the engine qualifies for the final.

## License

MIT — see `LICENSE`.
