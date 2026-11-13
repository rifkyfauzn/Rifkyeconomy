# tiny-chess-01 — 4 KB chess engine

An original 0x88-board chess engine in Python 3.13 stdlib only.
`main.py` is a 4,093-byte loader in two lines: the base85 payload is
split across two string literals joined by implicit concatenation, so
the entrypoint was written in the session in two messages. It
LZMA-decompresses a minified engine payload (base85) and executes it.
No other file is read, imported, opened or run; all data lives inside
`main.py` itself.

## Design (v4g, current)

- **Board:** 0x88 mailbox, white pieces 1–6, black −1..−6.
- **Move generation:** pseudo-legal generation with a legality filter
  (make + king-attack check). Castling, en passant, promotion.
- **Search:** negamax + alpha-beta, iterative deepening, fixed
  **85,000-node** budget (deterministic), transposition table,
  quiescence (captures and check evasions, inlined), PVS + 2-step
  LMR, null-move pruning, move ordering by TT move, MVV-LVA
  captures and promotion bonus (no killers, no bishop-pair bonus —
  dropped to fit the king taper). Repetition from game history.
- **Evaluation:** incremental material + piece-square tables
  (Zobrist hashing), plus a binary endgame king term: when fewer
  than 6 non-pawn pieces remain, the king is scored by an endgame
  centrality table (KEG − KMG difference) instead of the middlegame
  safety table.
- **Protocol:** talks to the env at `MAKEFUN_ENV_URL`.

## Measured results (local harness, practice trials)

| Version | main.py | T0 | T1 | T2 | Mean |
|---|---|---|---|---|---|
| v1 (submitted) | 4,092 B | 989 | — | — | — |
| v2 | 4,095 B | 1,253 | — | — | — |
| v4a (v2 + PST fix) | 4,096 B | 1,290 | 1,430 | 1,497 | 1,406 |
| v4b (v4a + king taper) | 4,254 B (too big) | 1,464 | — | — | — |
| v4f (v4a + taper, −bishop pair, −killers) | **4,080 B** | **1,362** | **1,564** | **1,598** | **1,508** |
| v4g (v4f, 85k nodes, two-line loader) | 4,093 B | 1,396 | — | — | — |

W/D/L: v4f 7-1-8 / 10-1-5 / 10-2-4. Sunfish baseline ~1,360 Elo
mean (1,326 / 1,464 / 1,290). **v4f's mean 1,508 beats the baseline
by ~148 Elo**, and v4a's mean by +102.

Rules changed 2026-11-12 (level-5 kit): a trial is now 8 games in
1,800 seconds, not 16 games in 3,600. v4g trial-0 wall on the new
8-game trial was 1,509 s, inside the limit. v4g T0: 4 won, 0 drawn,
4 lost — wins vs depth-2, depth-3, depth-4, nodes-250k; losses vs
nodes-100k, nodes-500k, nodes-1M, nodes-2M. Elo 1,396, above the
~1,360 baseline. The 85k node cut (from 100k) keeps the wall inside
the new limit; the 100k version would have needed ~1,750 s.

PST rows were flipped (zero bytes, +37 Elo T0); the king taper cost
158 packed bytes, paid for by dropping bishop-pair and killers.

## Files

- `main.py` — the entry (v4g, 4,093 bytes).
- `LICENSE` — MIT. `README.md` — this file.

Repo total: 4,093 + 1,062 + this file, under the 8,192-byte limit.
