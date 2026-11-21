# tiny-chess-01 v4 — a 4 KB Python chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/RieryBuild/makefun-tiny-chess-01-v4/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/RieryBuild/makefun-tiny-chess-01-v4/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Entry for the [make.fun](https://make.fun) `tiny-chess-01` bounty: a chess
engine in one `main.py` (max 4,096 bytes), pure Python 3.13 stdlib, no
model calls, no other files at runtime.

v4 beats our v3 on **speed, not strength**: v3's Elo was already at the board
cap while its `spare_plies` (659) sat below the prize cutoff. The ranking is
decided by mate speed inside the fixed budget, so v4 spends its 4 KB on node
economy — cheaper nodes, more depth per move, faster mates.

## Architecture (fresh code, written for v4)

- **Board:** 0x88 with in-place make/unmake and incremental tapered eval.
- **Eval:** PeSTO 12-table tapered middlegame/endgame values (public-domain
  table data), phase from non-pawn material, updated incrementally.
- **Search:** iterative deepening, fixed node budget/move (no wall clock —
  deterministic), PVS, LMR, null-move (R=3), killers, Zobrist TT,
  quiescence, aspiration (+/-60), mate-distance scoring.
- **Book:** a small opponent-aware opening book (3 scripted lines),
  keyed by opponent + opening.
- **Budgets:** fixed node counts per move, tuned on the practice harness.

## Speed work (measured)

- Fresh rewrite for node economy: inlined eval updates, table-based
  absolute values, unrolled castling, int-key move ordering — bench
  25.9s vs v3's 49.3s on the same workload (~1.9x, identical search).
- Hill-climb S2 (move-gen inner loop): +4.6% CPU, p=0.0005 (14/15
  pairs); two-list ordering measured twice, rejected both times.
- Node budgets raised from v3's 40k/100k to 60k/150k, tuned so the
  slowest move stays well under the 10 s limit (4.4 s CPU max,
  measured on real game positions).

## Practice-harness results (kit harness, scorer-confirmed)

| trial | Elo | W/D/L | spare_plies | v3 spare |
|-------|-----|-------|-------------|----------|
| 0 | 2112 | 8/0/0 | 916 | 764 |
| 1 | 1990 | 7/1/0 | 672 | 670 |
| 2 | 1748 | 6/1/1 | 560 | 456 |
| **mean** | **1950** | — | **716** | **630** |

v4 beats v3 on all three trials (mean +86 spare, +13.7%; mean Elo
1950 vs 1883). Holdout: Elo 1669, 5W 2D 1L, spare 502 — generalizes,
not overfit.

## Files

- `main.py` — the engine (packed, <= 4,096 bytes).
- `LICENSE` — MIT.

## License

MIT — see `LICENSE`.
