[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/kkazushi-eng/tiny-chess-v3/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/kkazushi-eng/tiny-chess-v3/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 entry

A chess engine in a single `main.py` that fits the competition's 4,096-byte
entrypoint cap (this entry: 4,089 bytes). The readable engine source is
lzma-compressed into a payload string inside `main.py` and unpacked at
startup by a small loader; the engine itself is Python standard library
only and reads no other file.

## The engine

Written from scratch:

- 0x88 board representation with incremental material / piece-square
  evaluation and Zobrist hashing (repetition and fifty-move handling)
- Negamax alpha-beta search with PVS, a transposition table, null-move
  pruning, late-move reductions, futility pruning and check extensions
- Move ordering by transposition-table move, MVV-LVA captures, killers
  and a history heuristic
- Quiescence search over captures and promotions (all evasions when in
  check) with delta pruning
- Tapered evaluation for the king and for pawns (endgame pawn table
  rewards advanced pawns), interpolated by remaining material
- Deterministic fixed node budget of 75,000 nodes per move (no
  wall-clock dependence); worst measured move time locally was about
  3.5 s on a complex middlegame position

## Local validation

- Move generation verified by perft: startpos perft(3)=8,902 and
  perft(4)=197,281; kiwipete perft(3)=97,862; plus three further
  standard positions covering en passant, pins and promotions, all
  exact. Make/unmake state restoration checked over random games.
- Head-to-head against this author's previous entry (same protocol,
  book openings from the competition kit, both colours): at equal
  20,000-node budgets the new engine scored 8 wins, 1 draw, 3 losses
  in 12 games; at the real budgets (75,000 vs the previous entry's
  60,000 nodes) it scored 15 wins, 3 draws, 6 losses in 24 games.

## How it runs

The program speaks the competition gateway protocol: it connects to
`$MAKEFUN_ENV_URL`, registers, and plays its games move by move under
`python3 -P -s`, exiting when the environment signals completion.
