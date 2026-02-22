# MissedClick Chess Engine – Project Reference & Notes

Use this document to answer questions about the project (interviews, college, resume). You can also paste sections into other AI tools to learn more about specific topics.

---

## 1. Project Overview

| Item | Description |
|------|-------------|
| **Name** | MissedClick |
| **Type** | UCI chess engine |
| **Language** | C++ (C++17) |
| **Purpose** | Plays chess via the UCI protocol; can be used in Arena, ChessBase, etc. |
| **Output** | Executable: `MissedClick.exe` (Windows) or `MissedClick` (Linux/Mac) |

---

## 2. Tech Stack & Concepts

- **Board representation:** Bitboards (64-bit integers, one bit per square)
- **Move representation:** Encoded as 32-bit integers (source, target, piece, flags)
- **Search:** Negamax, alpha–beta pruning, iterative deepening
- **Evaluation:** Stockfish NNUE (neural network) + 50-move rule handling
- **Protocol:** UCI (Universal Chess Interface)
- **Hashing:** Zobrist hashing for position identity and transposition table
- **Attack generation:** Magic bitboards for rooks and bishops; lookup tables for pawns, knights, king

---

## 3. Repository Structure

```
bbc-master/
├── README.md                 # Main project readme
├── .gitignore                # Ignores .exe, .o, backups, etc.
├── PROJECT_NOTES.md          # This file
├── bin/
│   ├── windows/              # MissedClick.exe after build.bat
│   └── linux/                # MissedClick after make install
├── pgn/                      # Optional: sample game files
└── src/
    └── MissedClick/
        └── cpp/              # Engine source root
            ├── main.cpp      # Entry point, init, UCI loop
            ├── Makefile      # Build (Linux/Mac)
            ├── build.bat     # Build + install (Windows)
            ├── missedclick_arena.ini  # Arena config
            ├── include/      # Headers (.hpp)
            │   ├── types.hpp, bitboard.hpp, board.hpp
            │   ├── attacks.hpp, movegen.hpp, makemove.hpp
            │   ├── hash.hpp, tt.hpp, moveorder.hpp
            │   ├── evaluate.hpp, search.hpp, uci.hpp
            │   ├── utils.hpp, evaluation_enhanced.hpp, search_enhanced.hpp
            │   └── ...
            └── src/          # Implementation (.cpp)
                ├── board.cpp, attacks.cpp, movegen.cpp, makemove.cpp
                ├── hash.cpp, tt.cpp, moveorder.cpp
                ├── evaluate.cpp, search.cpp, uci.cpp
                ├── evaluation_enhanced.cpp, search_enhanced.cpp
                ├── missing_implementations.cpp
                ├── utils.cpp, types.cpp
                └── ...
```

---

## 4. Module Summary (What Each Part Does)

| Module | Role |
|--------|------|
| **types.hpp/cpp** | Enums (Square, Piece, Side, MoveType), constants (INFINITY, MAX_PLY), move encoding macros, MoveList |
| **bitboard.hpp** | Bit operations: set/clear bit, count bits, LS1B/MS1B, etc. |
| **board.hpp/cpp** | 12 piece bitboards, occupancies, FEN parse, reset, copy/make state |
| **attacks.hpp/cpp** | Pawn/knight/king attack tables; magic bitboards for bishop/rook; init_sliders_attacks() |
| **movegen.hpp/cpp** | generate_moves() – generates all legal moves into a MoveList |
| **makemove.hpp/cpp** | make_move(), copy_board(), take_back() – copy/make style |
| **hash.hpp/cpp** | Zobrist keys, init_random_keys(), generate_hash_key(), repetition |
| **tt.hpp/cpp** | Transposition table (TT): read_tt(), write_tt(), TTEntry, TT_SIZE |
| **moveorder.hpp/cpp** | Move ordering: score_move(), sort_moves(), killers, history, PV move |
| **evaluate.hpp/cpp** | evaluate(), NNUE refresh, is_draw(), material, piece-square |
| **search.hpp/cpp** | negamax(), quiescence(), iterative_deepening(), PVS, LMR, NMP |
| **evaluation_enhanced.hpp/cpp** | Caching and extra evaluation logic |
| **search_enhanced.hpp/cpp** | Enhanced search (LMR/NMP limits, multi-cut, stats) |
| **uci.hpp/cpp** | uci_loop(), parse_uci_command(), position, go, stop, quit, id name |
| **utils.hpp/cpp** | get_time_ms(), input_waiting(), read_input(), random numbers for Zobrist |
| **main.cpp** | Init attacks, init keys, reset board, parse FEN, uci_loop() |

---

## 5. Data Structures (Important for Interviews)

- **Bitboard (U64):** One 64-bit word per piece set (e.g. all white pawns). Used for fast set operations.
- **Move (int):** 32-bit encoding:
  - Bits 0–5: source square
  - Bits 6–11: target square
  - Bits 12–15: piece
  - Bits 16–19: promoted piece
  - Bits 20–23: flags (capture, double push, en passant, castling)
- **MoveList:** Array of moves + scores, count. Used for move generation and ordering.
- **Board state:** 12 bitboards (P,N,B,R,Q,K and p,n,b,r,q,k), occupancies[3] (white, black, both), side, enpassant, castle, hash_key, fifty, ply, repetition_table.
- **TTEntry:** key, depth, flags (alpha/beta/exact), score, move. Used in transposition table.

---

## 6. Algorithms (Short Definitions for Q&A)

- **Negamax:** Single search function for both sides; score negated when switching side. Used with alpha–beta.
- **Alpha–beta pruning:** Cuts off branches that cannot affect the final result. Needs good move ordering.
- **Iterative deepening:** Search depth 1, then 2, then 3… until time runs out. Improves move ordering and time control.
- **Quiescence search:** Search only captures (and sometimes checks) at the horizon to avoid horizon effect.
- **PVS (Principal Variation Search):** Assume first move is best; search others with null window to fail high/low quickly.
- **LMR (Late Move Reduction):** Reduce search depth for moves that are likely bad (e.g. later moves, non-captures).
- **NMP (Null Move Pruning):** Try “pass” move; if score still above beta, prune. Saves time in midgame.
- **Transposition table:** Cache (position → score, move, depth). Reuse results when same position is reached.
- **Zobrist hashing:** XOR of random values per (piece, square) and side/castle/ep. Gives a position fingerprint.
- **Magic bitboards:** For rooks/bishops, use occupancy mask + magic multiply + shift to index precomputed attack table. Fast sliding attacks.

---

## 7. UCI (Universal Chess Interface)

- Engine talks to GUI (e.g. Arena) via **stdin/stdout**.
- **Commands engine receives:** `uci`, `isready`, `ucinewgame`, `position startpos [moves ...]`, `go [depth|movetime|wtime btime ...]`, `stop`, `quit`.
- **Responses engine sends:** `id name MissedClick ...`, `id author MissedClick`, `uciok`, `readyok`, `info ...`, `bestmove ...`.
- **Version string:** `VERSION = " - FINAL VERSION (1.4 + SF NNUE)"` in types.hpp.

---

## 8. Build & Run

- **Windows:** `cd src/MissedClick/cpp` then `build.bat` → produces `MissedClick.exe` in cpp and copies to `bin/windows/`.
- **Linux/Mac:** `cd src/MissedClick/cpp`, `make`, then `./MissedClick`; `make install` copies to `bin/linux/`.
- **Arena:** Add engine executable (e.g. `bin/windows/MissedClick.exe` or cpp folder exe). Protocol: UCI.

---

## 9. Constants (Good to Know)

- INFINITY = 50000, MATE_VALUE = 49000, MATE_SCORE = 48000
- MAX_PLY = 64
- TT_SIZE = 1048583 (transposition table entries)
- FULL_DEPTH_MOVES = 4, REDUCTION_LIMIT = 3 (search)
- START_POSITION FEN: `rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1`

---

## 10. Sample Q&A (Use for Practice or with AI)

**Q: How is the board represented?**  
A: With bitboards (U64). There are 12 piece bitboards (P through k) and 3 occupancy bitboards (white, black, both). Each bit is a square; operations use bitwise AND/OR and population count.

**Q: How are moves encoded?**  
A: As a 32-bit int: 6 bits source, 6 bits target, 4 bits piece, 4 bits promoted piece, and 4 bits for flags (capture, double push, en passant, castling). Macros like GET_MOVE_SOURCE(move) decode them.

**Q: What is the copy/make approach?**  
A: Before making a move we copy the current board state (bitboards, side, castle, ep, fifty, hash). After search we restore from that copy instead of explicitly undoing the move. Simplifies unmake and is cache-friendly.

**Q: What is the transposition table used for?**  
A: It stores (position hash → depth, score, bound type, best move). When we reach the same position again we can reuse the score/move instead of searching again, which speeds up the tree search.

**Q: How does the engine communicate with the GUI?**  
A: Via UCI over stdin/stdout. The GUI sends commands (e.g. `position`, `go`); the engine replies with `info` and `bestmove`. All in plain text.

**Q: What is NNUE?**  
A: Efficiently Updatable Neural Network (Stockfish). It evaluates the position using a small neural network. The engine calls an NNUE interface (e.g. nnue_refresh, evaluate) to get the score; the actual network is typically in a separate dependency (e.g. nnue_eval).

**Q: What optimizations are used?**  
A: Bitboards, magic bitboards, Zobrist hashing, transposition table, move ordering (PV/killer/history), LMR, NMP, iterative deepening, quiescence, copy/make, and compiler flags (-O3, -march=native, etc.). Optional: hardware popcount, prefetch, evaluation/search caches.

---

## 11. Keywords for Further Reading / AI Queries

Use these with search or other AI to go deeper:

- Bitboard chess programming  
- Magic bitboards  
- Negamax alpha-beta  
- UCI protocol  
- Zobrist hashing  
- Transposition table  
- Late move reduction (LMR)  
- Null move pruning  
- Quiescence search  
- Stockfish NNUE  
- FEN notation  
- Copy-make vs make-unmake  

---

*Last updated to match the current MissedClick codebase. Use PROJECT_NOTES.md as the single source of truth for project facts and as input for other AI tools.*
