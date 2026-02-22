# MissedClick Chess Engine

UCI chess engine - C++ conversion with Stockfish NNUE evaluation.

## Features
- Bitboard board representation
- Pre-calculated attack tables
- Magic bitboards for sliding pieces
- Negamax search with alpha-beta pruning
- PV/killer/history move ordering
- Iterative deepening, PVS, LMR, NMP
- Transposition table (up to 128MB)
- Stockfish NNUE evaluation
- UCI protocol

## Building

**Windows (Arena):**
```cmd
cd src/MissedClick/cpp
build.bat
```
Produces `MissedClick.exe` in the cpp folder and copies it to `bin/windows/`.

**Linux/Mac:**
```bash
cd src/MissedClick/cpp
make
./MissedClick
make install
```
`make install` copies the binary to `bin/linux/`.

