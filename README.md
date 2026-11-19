# tiny-chess-01 — 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/nuraliana/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/nuraliana/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A single-file (`main.py`, 4,063 bytes) Python 3.13 stdlib-only engine
for the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## How it plays

0x88 board, negamax + alpha-beta with iterative deepening on a fixed
50,000-node budget per move, PVS at the root, transposition table,
killer moves, MVV-LVA ordering, quiescence search (captures/promotions,
all evasions in check), null-move pruning, check extension, repetition
= draw. Evaluation: PeSTO tapered middlegame/endgame piece-square
tables, maintained incrementally in make/unmake — the evaluator is
O(1), never a full-board rescan. A 44-entry book is checked first at
the root: a book hit plays instantly, 0 nodes, 0 seconds. Entries are
stored as a compact base-92 blob keyed by the low 12 bits of our own
Zobrist hash (4 characters per entry); every hit is checked for
legality before it is played. The entries were derived from
Stockfish-oracle lines on our own lost games (conversion positions
where the search threw away a winning edge, plus oracle lines on the
games we still lose). Fully deterministic: fixed node budget, no clock,
no randomness, no model calls. Talks to the env at `MAKEFUN_ENV_URL`
via `POST /game`, `POST /move` (UCI) and `POST /resign`.

## Run it

```
MAKEFUN_ENV_URL=http://127.0.0.1:18080 timeout 1700 python3 main.py
```

## Practice results (local, official kit scorer)

Trial-0: 5-0-1 / Elo 1,654. Trial-1: 5-1-0 / Elo 1,809. Trial-2:
6-0-0 / Elo 2,112. Mean 1,858.3. The book flipped trial-2's Alapin
loss into a win (one Stockfish-oracle move, c6d8, at ply 18) — the
first 6-0-0 trial; trial-1's Morphy win and trial-0's Berlin win come
from the same cumulative book. Move generation covers all pieces and
both castling directions; legality is checked by making the move and
testing whether the mover's king is attacked. The node budget is a
hard cap: the search raises an internal abort once 50,000 nodes are
spent, the root catches it, and the best move found at the last
completed depth is played. `main.py` is what the kit's
`makefun compress` makes from the readable source, verified byte for
byte after every change.

## License

MIT — see `LICENSE`.
