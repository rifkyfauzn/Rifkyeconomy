# tiny-chess-01 — 4 KB chess engine

An original 0x88-board chess engine written from scratch in Python 3.13
stdlib only. `main.py` is a 4,092-byte loader: it LZMA-decompresses a
minified engine payload (base85) and executes it. No other file is read,
imported, opened or run; all data lives inside `main.py` itself.

## Design

- **Board:** 0x88 mailbox, white pieces 1–6, black −1..−6.
- **Move generation:** pseudo-legal generation with a legality filter
  (make + king-attack check). Handles castling, en passant, promotion.
- **Evaluation:** incremental material + piece-square tables (Zobrist
  hashing for the transposition table).
- **Search:** negamax + alpha-beta, iterative deepening with a fixed
  **60,000-node** budget (deterministic — no clock, no unseeded
  randomness), transposition table, quiescence search (captures and
  check evasions, merged into the search), PVS + LMR, null-move
  pruning, and move ordering by TT move, MVV-LVA captures, killer
  moves and history heuristic.
- **Protocol:** talks to the env at `MAKEFUN_ENV_URL` (`POST /game`,
  `POST /move`), rebuilds the game history for repetition detection.

## Measured results (practice trial 0, local harness)

| Version | main.py | Trial 0 (16 games) | Elo |
|---|---|---|---|
| v3 (full, before golfing) | 4,782 B (too big) | 6 won, 0 drawn, 10 lost | 1,253 |
| final (this entry) | **4,092 B** | 3 won, 0 drawn, 13 lost | **989** |

The house baseline (sunfish) is ~1,360 Elo on the practice trials.
**This entry does not beat the baseline** (989 < 1,360), so it would not
qualify for a prize. The gap is honest: fitting the engine into 4,096
bytes forced the removal of the bishop-pair bonus and the promotion
ordering bonus, and the simplified LMR re-search, which together cost
roughly 260 Elo against the ungolfed version.

## Files

- `main.py` — the entry (4,092 bytes, ≤ 4,096).
- `LICENSE` — MIT.
- `README.md` — this file.

Total repo size is well under the 8,192-byte limit; no file exceeds
4,096 bytes.

## Reproduce

From the competition kit directory, with the practice env:

```
node cli/makefun.mjs grade-local --kit . --bot <path to this dir> --driver process
```

(Docker is not required; the local harness in this workspace uses the
kit's own env server + scorer.)
