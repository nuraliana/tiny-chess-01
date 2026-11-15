# tiny-chess-01 — 4 KB chess engine

A single-file (`main.py`, 3,830 bytes) Python 3.13 stdlib-only chess engine
for the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## How it plays

0x88 board, negamax + alpha-beta with iterative deepening on a fixed
30,000-node budget per move, PVS at the root, transposition table,
killer moves, MVV-LVA ordering, quiescence search (captures/promotions,
all evasions in check), null-move pruning, repetition = draw.
Evaluation: material + piece-square tables + bishop pair + king table.
Fully deterministic: fixed node budget, no clock, no randomness, no
model calls. Talks to the env at `MAKEFUN_ENV_URL` via `POST /game`,
`POST /move` (UCI) and `POST /resign`.

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
| trial-0 | `main.py` (packed), 30k nodes, round 3 | 3-1-4 | **1,326** |

Official-set trial-0 at 30k nodes: 3-1-4 / Elo 1,326, above the 1,154
floor — wins vs depth-2, depth-3 and depth-4, draw vs nodes-100k;
losses vs nodes-250k, nodes-500k, nodes-1M and nodes-2M. The 25k run
scored 3-0-5 / Elo 1,253 in 403 s; the 30k run finishes in 408 s.
Mean of the three trials (1,326/1,087/989) is 1,134; best (1,326)
is just above the 1,325 sunfish baseline mean, so a prize is not
assured — the engine qualifies for the final.

## Inside the 4 KB

The readable source is `engine_slim.py` (12,876 bytes, 394 lines);
`main.py` is what the kit's `makefun compress` makes from it, verified
byte for byte after every change.
Move generation covers pawns (single, double, capture, en passant,
promotion to any piece), knights, kings, bishops, rooks, queens and
both castling directions; legality is checked by making the move and
testing whether the mover's king is attacked.
The search never trusts a pseudo-legal move: every candidate is made
and unmade again, so an illegal move can never be sent. Repetition
is a draw once a position key has appeared twice. The node budget is a hard cap: the search raises an
internal abort once 30,000 nodes are spent, the root catches it,
and the best move found at the last completed depth is played.
Round 2 cut the budget from 50,000 to 25,000 nodes and sped up the
search: ordering scores without a helper call per move, the attack
test splits knights from kings, and the evaluation reads one folded
piece-square table. A local move takes about 0.6 s on average (1.2 s
in the slowest spots); trial-0 finishes in 408 s of wall, down
from 833 s.
A passed-pawn bonus (+5 per rank) was tried and reverted on
14 November after a practice trial fell to Elo 989; the plain
evaluation won the trial back to 3-0-5 and Elo 1,253.

## License

MIT — see `LICENSE`.
