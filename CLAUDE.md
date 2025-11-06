# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains example programs written for **SectorLISP**, a 512-byte Lisp interpreter that fits inside the boot sector of a floppy disk and runs on bare metal. The original SectorLISP is available at https://github.com/jart/sectorlisp.

This is primarily a **showcase repository** of example programs demonstrating SectorLISP's capabilities, not an active development project with build systems or test suites.

## Repository Structure

- `lisp/` - All SectorLISP example programs (`.lisp` files)
- `ref/` - Reference implementations (e.g., `mandelbrot.py` shows Python reference for the Lisp implementation)
- `README.md` - Comprehensive documentation of all programs with descriptions and examples

## Key Architectural Concepts

### SectorLISP Language Constraints

All programs in `lisp/` are written for the extremely constrained SectorLISP interpreter:

- **No built-in numbers**: Integers are represented in unary as lists (e.g., `(1 1 1)` for 3)
- **Minimal special forms**: Only basic Lisp primitives (CAR, CDR, CONS, COND, QUOTE, LAMBDA, EQ, ATOM)
- **Pure symbolic computation**: Everything is built from S-expressions and atoms
- **I/O extension**: Some programs use I/O special forms (`READ`, `PRINT`) that were added to SectorLISP
  - `READ` accepts arbitrary S-expressions from user input
  - `PRINT` prints evaluated arguments to console; `(PRINT)` with no args prints newline
  - These features were merged into the original SectorLISP repo (previously at https://github.com/woodrush/sectorlisp/tree/io)
  - Added only 35 bytes to the binary (465-467 bytes total vs original 436 bytes)

### Program Categories

1. **Self-contained interpreters**: Programs that extend SectorLISP functionality
   - `basic.lisp`, `basic-2.lisp` - BASIC-subset interpreters supporting REM, LET, IF, GOTO, PRINT
   - `eval-macro.lisp`, `eval-macro-define.lisp` - Extended metacircular evaluators with MACRO support

2. **Interactive REPL programs**: Require I/O-enabled SectorLISP
   - `basic-repl.lisp` - BASIC interpreter with REPL (LIST, RUN, DISCARD commands)
   - `repl-macro.lisp`, `repl-macro-define.lisp` - Lisp REPLs with macro support
   - `number-guessing-game.lisp` - Interactive game

3. **Computational demonstrations**:
   - `fizzbuzz.lisp`, `fizzbuzz-decimal.lisp` - Algorithm demonstrations with unary numbers
   - `quine.lisp` - Self-replicating program
   - `mandelbrot.lisp` - Mandelbrot set using fixed-point arithmetic (uses numsectorlisp library from https://github.com/woodrush/numsectorlisp)

### Code Structure Pattern

Most programs follow this structure:
```lisp
((LAMBDA (MAIN-FUNCTION)
   (MAIN-FUNCTION (QUOTE <data>)))
 (QUOTE (LAMBDA (...) <implementation>)))
```

This pattern:
1. Defines the entire program as a self-contained lambda expression
2. Separates data (program input) from implementation
3. Allows the program to be evaluated in a single pass

### Number Representation

Since SectorLISP has no numeric types:
- **Unary integers**: `()` = 0, `(1)` = 1, `(1 1)` = 2, `(* * *)` = 3 (any atom works)
- **Fixed-point numbers**: `mandelbrot.lisp` implements a complete fixed-point library using symbolic expressions
- **Decimal display**: `fizzbuzz-decimal.lisp` shows how to convert unary to decimal representation

### Licensing Notes

Four programs (`eval-macro.lisp`, `eval-macro-define.lisp`, `repl-macro.lisp`, `repl-macro-define.lisp`) are based on `lisp.lisp` from the original SectorLISP repository and carry both ISC (from SectorLISP) and MIT (from this repo) licenses. All other programs are MIT-licensed only.

## Common Operations

**View program descriptions**: Read README.md which documents every program

**Test a program**: Programs are designed to be evaluated directly in the SectorLISP interpreter. Testing requires either:
- Running on bare metal (boot sector execution)
- Using Blinkenlights emulator (from SectorLISP repo)
- Using QEMU with test scripts from the SectorLISP repository under `./test`

**Reference implementations**: Check `ref/` directory for reference implementations in other languages (currently contains Python reference for mandelbrot)

## Important Programming Techniques for SectorLISP

### Sequential Execution with PROGN

Since lambda bodies can only have one expression, sequential execution is achieved by passing expressions as lambda arguments:

```lisp
((LAMBDA () NIL)
 (PRINT (QUOTE A))
 (PRINT (QUOTE B))
 (PRINT (QUOTE C)))
```

This works because `EVLIS` evaluates all arguments before binding them. You can define `PROGN` as `(LAMBDA () NIL)` for readability. Note: Returns `NIL`, not the last expression (unlike standard Lisp).

### Loops via Recursion

Since there are no loop constructs, loops are implemented by functions calling themselves. See `number-guessing-game.lisp` for examples of recursive `MAIN` and `GAMELOOP` functions.

### Print Debugging

Wrap expressions with a DEBUG helper to observe runtime values:

```lisp
((LAMBDA (DEBUG)
  ...
  (DEBUG EXPR)
  ...)
 (QUOTE (LAMBDA (X) (CDR (CONS (PRINT X) X)))))
```

This is essential for debugging large programs since `PRINT` has undefined return value (to save bytes).

### Comments in PROGN blocks

Use a dummy variable bound to NIL with quoted expressions:

```lisp
((LAMBDA (PROGN ;;)
  (PROGN ;; (QUOTE - This is a comment)
    (PRINT (QUOTE A))))
 (QUOTE (LAMBDA () NIL))
 NIL)
```

## Development Notes

- This is an examples/showcase repository - programs are complete demonstrations, not actively developed features
- All programs are single-file implementations in pure SectorLISP
- Programs cannot be easily unit tested in conventional ways - they must be run in the SectorLISP environment
- When modifying programs, maintain the single-file, self-contained structure
- Preserve the unary number representation and avoid assuming modern Lisp features
- Author: Hikaru Ikuta (woodrush) with contributions from Justine Tunney (@jart, creator of original SectorLISP)
