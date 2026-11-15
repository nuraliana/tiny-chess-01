# tiny-chess-01 — 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/nuraliana/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/nuraliana/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A single-file (`main.py`, 4,083 bytes) Python 3.13 stdlib-only engine
for the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## How it plays

0x88 board, negamax + alpha-beta with iterative deepening on a fixed
30,000-node budget per move, PVS at the root, transposition table,
killer moves, MVV-LVA ordering, quiescence search (captures/promotions,
all evasions in check), null-move pruning, repetition = draw.
Evaluation: PeSTO tapered middlegame/endgame piece-square tables
(quantized Q8, max error 4 cp), maintained incrementally in make/unmake
— the evaluator is O(1), never a full-board rescan. Fully deterministic:
fixed node budget, no clock, no randomness, no model calls. Talks to the
env at `MAKEFUN_ENV_URL` via `POST /game`, `POST /move` (UCI) and
`POST /resign`.

## Run it

```
MAKEFUN_ENV_URL=http://127.0.0.1:18080 timeout 1700 python3 main.py
```

## Practice results (local, official kit scorer)

Official-set trial-0 at 30k nodes: 3-1-4 / Elo 1,326, above the 1,154
floor — wins vs depth-2, depth-3 and depth-4, draw vs nodes-100k;
losses vs nodes-250k, nodes-500k, nodes-1M and nodes-2M. The readable
source is `engine_v16.py` (9,776 bytes); `main.py` is what the kit's
`makefun compress` makes from it, verified byte for byte after every
change. Move generation covers all pieces and both castling directions;
legality is checked by making the move and testing whether the mover's
king is attacked. The node budget is a hard cap: the search raises an
internal abort once 30,000 nodes are spent, the root catches it, and
the best move found at the last completed depth is played.

## License

MIT — see `LICENSE`.
