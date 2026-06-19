# Nevada FORTRAN — CP/M Distribution Disk

This directory contains a complete distribution of **Nevada FORTRAN Version 3.0**, a FORTRAN compiler and runtime system for **CP/M-80** (the Z80/8080 operating system). All files are dated **26 June 1997** but originate from the early 1980s. The system is designed to run on Z80/8080-based microcomputers running CP/M, communicating with terminals such as the **Lear Siegler ADM-3A**.

---

## System Overview

Nevada FORTRAN is a two-phase system:

1. **Compile phase** — `FORT.COM` compiles `.FOR` source files into relocatable `.OBJ` or Intel HEX `.HEX` object files, and can optionally invoke the assembler for inline assembly.
2. **Run phase** — `FRUN.COM` loads and executes compiled FORTRAN programs. Alternatively, `RUNA.COM` can run a compiled object directly.

The compiler supports a dialect of FORTRAN 77 with CP/M-specific extensions (terminal I/O, CHAIN, LOAD, DUMP, TRACE, ERRSET, BIT manipulation, etc.).

---

## Binary Executables (.COM — CP/M Programs)

These are Z80/8080 machine code executables that run directly under CP/M. They begin with `C3` (the Z80 `JP` opcode), the standard CP/M entry point pattern.

### `FORT.COM` — The FORTRAN Compiler (22 KB)
The core compiler. Takes a `.FOR` source file and compiles it to a `.OBJ` (relocatable binary) or `.HEX` (Intel hex) object file. Accepts `$OPTIONS` directives embedded in source to control compiler behaviour (memory table sizes, debug output, cross-reference, etc.). Copyright notice visible in binary: "Copyright (C) 1980, 1981".

### `FRUN.COM` — The FORTRAN Runtime / Loader (16 KB)
Loads and executes a compiled FORTRAN `.OBJ` or `.HEX` file. This is the primary way to run Nevada FORTRAN programs. Provides the full runtime library including: file I/O, formatted output, `RAND()`, `CALL BIT()`, `CALL CHAIN()`, `CALL LOAD()`, `CALL SCREEN()`, `CALL PUT()`, `CALL DELAY()`, `CALL EXIT()`, error trapping (`ERRSET`), and execution tracing (`TRACE ON/OFF`).

### `ASSM.COM` — The Z80/8080 Assembler (11 KB)
An assembler for Z80/8080 assembly language (`.ASM` source). Used to assemble inline or standalone assembly routines that can be called from FORTRAN via the `CALL LOAD()` mechanism. Produces `.OBJ` or `.HEX` output. Referenced in the `ERRORS` file (`9C *FATAL* File "ASSM.COM" not found`), confirming tight integration with `FORT.COM`.

### `CONFIG.COM` — System Configuration Utility (5 KB)
Identified in its binary as **"NEVADA FORTRAN CONFIGURATION PROGRAM (27MAY83)"**. Used to configure the FORTRAN system for the specific CP/M machine and terminal in use — terminal escape sequences, I/O port assignments, screen dimensions, etc. Must be run before first use on a new system. Results are saved to the disk so `FORT.COM` and `FRUN.COM` pick up the correct settings.

### `PIP.COM` — Peripheral Interchange Program (7 KB)
The standard CP/M file copy and transfer utility. Included on the distribution disk as a convenience tool for copying files between drives/devices. Binary shows `(INP:/OUT:SPACE)` — the PIP command syntax prompt, confirming this is the standard CP/M PIP utility.

### `RUNA.COM` — Run Assembly / Alternate Runner (896 bytes)
A small loader (896 bytes — exactly one CP/M 128-byte block × 7). Used to directly execute a compiled program or assembly object, likely used when chaining programs or for testing assembled routines without the full FORTRAN runtime overhead.

---

## FORTRAN Source Files (.FOR — Example / Demo Programs)

These are Nevada FORTRAN source programs demonstrating runtime library features. They use the fixed-format FORTRAN source layout (columns 1–5: labels, column 6: continuation, columns 7–72: code). CP/M extensions are used throughout.

### `SAMPLE.FOR` — Terminal Screen Demo (Corrected Version)
Demonstrates CP/M terminal I/O via the custom `SCREEN` subroutine, which wraps `CALL PUT()` for character output to an ADM-3A terminal. Features:
- `CALL SCREEN(1,0,0)` — clear screen (sends ASCII 26 / Ctrl-Z)
- `CALL SCREEN(2,X,Y)` — position cursor (ADM-3A escape sequence: ESC `=` Y+32 X+32)
- `CALL SCREEN(5,0,0)` — ring terminal bell (sends ASCII 7)
- `CALL DELAY(500)` — 5-second pause (units are 10ms ticks)
- `RAND(0)` — generate a random float 0.0–1.0
- Plots 400 random `+` characters at random screen positions

This is the **corrected** version of the program from page 148 of the Nevada FORTRAN manual (the manual version had a bug); the correction is documented in `READ.ME`.

### `BITDEMO.FOR` — Bit Manipulation Demo
Demonstrates the `CALL BIT(var, bitnum, 'S')` runtime function. The BIT routine operates on a 48-bit floating-point accumulator (Nevada FORTRAN uses 48-bit reals). Accepts a bit number 0–47, sets that bit in variable `A`, and displays the result in hex using the `K12` format specifier (12-digit hex output — a Nevada FORTRAN extension).

### `CHAIN.FOR` — Program Chaining Demo
Demonstrates `CALL CHAIN(filename, ier)` — loading and transferring control to another FORTRAN `.COM` file from within a running program. This is the CP/M equivalent of `exec()`: it replaces the current program in memory with a new one, passing control to it. Useful for building multi-module applications that exceed the CP/M memory limit if compiled as one unit.

### `DUMP.FOR` — DUMP Statement / Error Debug Demo
Demonstrates two Nevada FORTRAN-specific features:
- `$OPTIONS X` — enables cross-reference / extended debug output from the compiler
- `DUMP /label/ var1,var2,...` — declares a set of variables to be automatically displayed if a runtime error occurs within the subroutine. The label (e.g., `/ROUTINE-X/`) appears in the debug output to identify the source location.
- Intentionally triggers a divide-by-zero (`Z=1/0`) to activate the DUMP output, showing values of `I`, `J`, `K` at the point of failure.

### `GRAPH.FOR` — Text-Mode Sine Wave Plotter
Plots `sin(angle)` for angle from −π to +π in 0.12-radian steps, using a 70-character wide text-mode graph on the console. Technique:
- Maps sin() range [−1, +1] to character columns [0, 70]
- Fills the line array with spaces, then places `*` characters from the zero line (column 35) out to the current value
- Always marks column 35 with `+` as the zero axis
- Uses `CALL OPEN(6,'CON:')` to open the console as a FORTRAN unit for `WRITE` statements
- Uses `$OPTIONS X` for debug/cross-reference

### `LOAD.FOR` — Mixed-Language (FORTRAN + Assembly) Demo
Demonstrates `CALL LOAD(name, type, ier)` — dynamically loading a Z80 assembly language routine into memory at runtime and calling it from FORTRAN. Workflow:
1. Prompts user to choose loading `LD.HEX` (Intel hex format) or `LD.OBJ` (binary object)
2. Calls `LOAD('LD', type, IER)` to read the file from disk into memory at `$8000`
3. Calls the routine at address `$8000` via `A = CALL($8000, 1)`, passing the integer `1` as argument
4. Expects the result `2` to be returned (because `LD.ASM` doubles its input)

This is a key interoperability demo: FORTRAN calling hand-written Z80 machine code.

### `RAND.FOR` — Random Number Distribution Test
Statistical test of the `RAND(0)` runtime function. Generates K random numbers, bins them into 10 equal intervals [0.0–1.0), counts how many fall into each bin, and prints the 10 counts. Loops continuously asking for new sample sizes. The `$OPTIONS X,Q` directive enables cross-reference and quiet (suppressed) compile-time output.

### `SORT.FOR` — Shell Sort Demo
Implements and demonstrates a **Shell sort** on a randomly-generated array of up to 2000 integers. Features:
- Uses `RAND(0)*NN+1` to fill the array with random integers
- Shell sort with gap sequence `D = (D+1)/2` (Knuth-style)
- Prints intermediate gap (`D`) values during sorting to show progress
- Displays the sorted result after completion
- Loops to allow repeated tests with different array sizes

### `TRACE.FOR` — Execution Trace and Error Trap Demo
Demonstrates two Nevada FORTRAN debugging features:
- `TRACE ON` / `TRACE OFF` — when ON, every FORTRAN statement executed prints its source line number to the console, giving a live execution trace
- `ERRSET label, var` — installs an error handler; on any runtime error, execution jumps to the specified label with the error code in `var` (instead of aborting)
- Demo counts from 1 to K; if K > 100 it enables tracing so the user can see the trace output; trapping Ctrl-C causes the ERRSET handler to fire and print the error code

---

## Assembly Language Source

### `LD.ASM` — Sample Z80 Subroutine (used by `LOAD.FOR`)
A tiny Z80 assembly routine, 4 instructions, assembled to load at address `$8000`:

```asm
ORG   8000H
PUSH  D      ; argument arrives in DE register pair (from FORTRAN)
POP   H      ; move to HL
DAD   H      ; HL = HL + HL  (i.e., double it)
RET          ; return result in HL to FORTRAN
```

Demonstrates the FORTRAN–assembly calling convention: arguments are passed in the `DE` register pair, integer results returned in `HL`. This is assembled by `ASSM.COM` to produce `LD.HEX` and/or `LD.OBJ`, which `LOAD.FOR` then loads at runtime.

---

## Documentation and Data Files

### `READ.ME` — Manual Errata / Corrections (Version 3.0)
Documents two bugs/corrections relative to the printed Nevada FORTRAN Version 3.0 manual:
1. **Page 148 sample program**: the manual's version is wrong; `SAMPLE.FOR` on the disk is correct. The full correct listing is reproduced here.
2. **Compiler bug**: using a subroutine name as a variable name within an expression confuses the compiler, causing it to emit a spurious temporary variable reference and pass one extra argument to the called routine. This should be avoided.

### `ERRORS` — Complete Compiler and Runtime Error Code Reference
A comprehensive hexadecimal error code table (codes `00`–`9A`) covering all compiler and runtime errors. Organized as:
- `00`–`7F`: Compiler errors (syntax, structure, overflow conditions). Includes both non-fatal (warning) and `*FATAL*` errors.
- `80`–`9A`: Fatal compiler/linker errors (missing files, table overflows, disk errors, missing `END` statement, etc.)

Error codes are hex values corresponding to messages printed by `FORT.COM` during compilation. Referenced by error number in `FRUN.COM` runtime output as well.

### `-KAY2.022` — Empty File (0 bytes)
A zero-byte file, likely a disk label, version marker, or format artifact from the original CP/M floppy disk image. The name suggests it may be a Kay Industries or similar CP/M system identifier, or a distribution version tag (`KAY2`, version `0.22`). Has no content.

---

## How the System Fits Together

```
Source (.FOR)  ──►  FORT.COM  ──►  Object (.OBJ / .HEX)
                        │
                    ASSM.COM  ◄──  Assembly (.ASM)
                        │
                    Object (.OBJ / .HEX)
                        │
                    FRUN.COM  ──►  Execution
                    RUNA.COM  ──►  (alternate runner)

CONFIG.COM  ──►  Terminal/system configuration (saved to disk)
PIP.COM     ──►  File management utility
```

**Typical workflow:**
1. Run `CONFIG.COM` once to set up terminal and system parameters.
2. Edit `.FOR` source (on the CP/M machine or transferred via `PIP.COM`).
3. Compile: `FORT filename` → produces `filename.OBJ`.
4. If using assembly routines: `ASSM LD` → produces `LD.OBJ` and/or `LD.HEX`.
5. Run: `FRUN filename` to execute.
6. Use `CHAIN`, `LOAD`, `DUMP`, `TRACE`, `ERRSET` at runtime for multi-module programs, mixed-language calls, and debugging.

---

## Historical Context

Nevada FORTRAN was a commercial CP/M FORTRAN product from the early 1980s, targeting Z80/8080 single-board computers and early microcomputers running the CP/M-80 operating system (Digital Research). It was notable for:
- Fitting a usable FORTRAN compiler and runtime in the ~60 KB of TPA (Transient Program Area) available under CP/M
- Providing an ADM-3A terminal driver model (the ADM-3A was extremely common in this era)
- Supporting direct Z80 assembly integration via `CALL LOAD()` and `CALL($addr, args)`
- Using 48-bit floating point (6 bytes) for real numbers, giving ~14 significant decimal digits

The file timestamps (`Jun 26 1997`) likely reflect when this disk image was extracted or last copied, not the original creation date. The software itself dates to 1980–1983 based on copyright notices in the binaries.
