# tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/vayne01-hub/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/vayne01-hub/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in 4 KB of Python for the [tiny-chess-01 bounty](https://make.fun/bounty/tiny-chess-01) on make.fun. Stdlib only, no model calls, fully deterministic.

## Design

- **Board:** 0x88 array, pseudo-legal movegen with a legality filter.
- **Search:** negamax alpha-beta with iterative deepening to a fixed budget of 40,000 nodes per move (main search + quiescence), so no clock is ever read and replays are identical.
- **Quiescence:** captures and promotions, plus full evasions while in check; check extensions in the main search.
- **Ordering:** transposition-table move first, then MVV-LVA captures.
- **Transposition table:** up to 131,072 entries, keyed by Zobrist hash.
- **Eval:** one board scan per node — material plus tapered piece-square tables (Michniewski-style values, packed as strings), bishop-pair bonus, and a middlegame/endgame king table blended by game phase. Zobrist hash is maintained incrementally.
- **Repetition:** positions seen in the game or on the search line score as a draw inside the search.
- **Safety:** every move is validated against the env's `legal_moves`; if anything ever disagrees, the first legal move is played so a bug never loses on an illegal move.

## Local practice results

Graded locally against the kit env (packed `main.py`, Python 3.12):

| trial | Elo | W / D / L | wall |
| --- | --- | --- | --- |
| 0 | 1545 | 4 / 1 / 1 | 656 s |
| 1 | 1364 | 3 / 1 / 2 | 514 s |
| 2 | 1279 | 3 / 0 / 3 | 453 s |
| **mean** | **1396** | **10 / 2 / 6** | — |

Trial 0: wins vs depth-2, depth-3, depth-4 and nodes-100k; loss vs
nodes-250k; draw vs nodes-500k (adjudicated). Trial 1: wins vs depth-2,
nodes-100k and nodes-500k; draw vs nodes-250k; losses vs depth-3 and
depth-4. Trial 2: wins vs depth-2, depth-3 and depth-4; losses vs
nodes-100k, nodes-250k and nodes-500k. Worst single move across all
trials: 3,352 ms (limit 10,000 ms). Mean practice Elo 1396 clears the
1,154 floor by 242, just above the house baseline's ~1,365 mean.
Determinism check: trial 0 run twice (runs 4 and 5) — `state.json` and
`transcript.ndjson` byte-identical (sha256 `183eca27…`), same games
move-for-move, same Elo 1545.

## Build

`engine_src.py` is the readable source (kept out of the snapshot); `main.py` is its kit-compressed form (`makefun compress engine_src.py --out main.py`, 4,077 bytes).
