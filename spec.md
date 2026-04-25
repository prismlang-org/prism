# Prism Language Specification
## Version 0.1

---

## Table of Contents

1. [Overview](#overview)
2. [Design Philosophy](#design-philosophy)
3. [Syntax](#syntax)
4. [Types](#types)
5. [Variables](#variables)
6. [Functions](#functions)
7. [Control Flow](#control-flow)
8. [Classes](#classes)
9. [Error Handling](#error-handling)
10. [Modules & Compiler Directives](#modules--compiler-directives)
11. [Operators](#operators)
12. [Raw Blocks](#raw-blocks)
13. [Compiler Behaviour](#compiler-behaviour)
14. [Bytecode Format (.pbc)](#bytecode-format-pbc)
15. [Instruction Set](#instruction-set)

---

## Overview

Prism is a statically typed, general-purpose scripting language designed to be as approachable as Python while easing the transition to intermediate languages like Go and Lua. It features a keyword-heavy syntax, explicit mutability, a module system, first-class error codes, and a garbage-collected runtime.

Prism compiles to `.pbc` (Prism Bytecode), which is executed by the Prism Virtual Machine (PVM).

---

## Design Philosophy

- **Helpful, not hostile.** The compiler warns and fixes where it can. Hard errors are reserved for situations where the compiler genuinely cannot proceed.
- **Explicit over implicit.** Mutability, visibility, and entry points are all declared explicitly.
- **Minimal keyword list.** The language does as much as possible with as few reserved words as possible.
- **Warnings over errors.** The compiler is a helpful colleague, not a gatekeeper.

---

## Syntax

### Statement Terminator

Every statement ends with a semicolon `;`.

```prism
let x: int = 5;
print("hello");
```

### Blocks

Blocks are opened with `begin:` and closed with `end;`. Prism is **not indentation-dependent** — indentation is purely cosmetic.

```prism
if x is 5 begin:
    print("five");
end;
```

One-liners are valid:

```prism
if x is 5 begin: print("five"); end;
```

### Comments

Single-line comments use `--`.

```prism
-- This is a comment
let x: int = 5; -- inline comment
```

---

## Types

### Primitive Types

| Type    | Description                                      |
|---------|--------------------------------------------------|
| `int`   | Signed integer, arbitrary precision              |
| `float` | Floating point, arbitrary precision              |
| `str`   | UTF-8 string (ASCII initially)                   |
| `bool`  | Boolean (`true` or `false`)                      |
| `void`  | Absence of a value                               |

Precision variants are supported:

```prism
let a: int64   = 100;
let b: int8192 = 99999999999999999999;
let c: float10000 = 3.14159; -- extremely discouraged but valid
```

### Custom Types

```prism
type Meters = float;
type UserId = int;
```

### Arrays

```prism
let scores: [int] = [1, 2, 3];
```

### Type Annotation Modifier

`#` is used to annotate compile-time constant types:

```prism
let prof: #str{"debug"}
let opt:  #int{1}
```

---

## Variables

### Immutable

```prism
let name: str = "Lucas";
```

### Mutable

```prism
let mut count: int = 0;
count << count + 1;
```

---

## Functions

Functions are declared with `fn`, take typed parameters, and declare a return type after `:`.

```prism
fn greet(name: str): str begin:
    greet::return << "Hello, " + name;
end;
```

### Return Values

There is no `return` keyword. Values are sent back via `fn-name::return << value`:

```prism
fn add(a: int, b: int): int begin:
    add::return << a + b;
end;
```

For `void` functions:

```prism
fn say_hello(): void begin:
    print("Hello!");
    say_hello::return << void;
end;
```

### Calling Functions

```prism
let result: str = greet("Lucas");
```

---

## Control Flow

### If / Else

```prism
if x is 5 begin:
    print("five");
else begin:
    print("not five");
end;
```

### For Loop

`for` is the only loop construct. It iterates over a collection or loops while a bool is true:

```prism
-- Iterate over collection
for item in [1, 2, 3] begin:
    print(item);
end;

-- Loop while bool is true
let mut running: bool = true;
for running begin:
    -- ...
end;
```

---

## Classes

Classes are Go-style: structs with methods attached. There is no inheritance — use composition instead.

```prism
class Animal begin:
    name: str;
    age:  int;

    fn speak(): str begin:
        speak::return << "...";
    end;
end;
```

### Instantiation

```prism
let a: Animal = Animal { name: "Rex", age: 3 };
```

### Composition

```prism
class Dog begin:
    base:  Animal;
    breed: str;

    fn speak(): str begin:
        speak::return << "Woof";
    end;
end;
```

---

## Error Handling

### Declaring Error Codes

Each module declares its own error codes. Codes are assigned string descriptions using `<<`:

```prism
errors begin:
    E001 << "File not found";
    E002 << "Permission denied";
end;
```

### Raising Errors

```prism
raise IO::E001;
```

### Handling Errors

```prism
let result = read_file("data.txt");

on result begin:
    IO::E001 => print("File not found!");
    IO::E002 => print("Permission denied!");
    ok       => print(result);
end;
```

### Namespacing

Error codes are namespaced to their module using `::`:

```prism
IO::E001
Calculator::E003
```

---

## Modules & Compiler Directives

All compiler-facing instructions use the `$` prefix. These are not runtime constructs.

| Directive       | Description                                      |
|-----------------|--------------------------------------------------|
| `$mod "Name"`   | Declare this file as a module                    |
| `$use "Name"`   | Import a module                                  |
| `$entry fn`     | Declare the entry point function (must be `$pub`)|
| `$pub`          | Mark the next declaration as publicly accessible |
| `$cfg`          | Conditional compilation flag                     |
| `$unsafe`       | Permit the following raw block                   |

### Example

```prism
$mod "Calculator"
$use "IO"
$entry main

$pub
fn main(): void begin:
    -- ...
    main::return << void;
end;
```

### Unused Imports

Unused `$use` directives produce a compiler warning, not an error.

---

## Operators

### Pipe Operators

| Operator | Description              |
|----------|--------------------------|
| `>>`     | Pipe right (send right)  |
| `<<`     | Pipe left (send left)    |

These are symmetric and equivalent — direction is a readability choice:

```prism
-- These are equivalent
"Hello" >> print();
print() << "Hello";

-- Chaining
read_file("data.txt") >> process() >> print();
```

### Comparison & Logic

| Operator  | Description       |
|-----------|-------------------|
| `is`      | Equal to          |
| `is not`  | Not equal to      |
| `>`       | Greater than      |
| `<`       | Less than         |
| `>=`      | Greater or equal  |
| `<=`      | Less or equal     |
| `and`     | Logical and       |
| `or`      | Logical or        |
| `not`     | Logical not       |

### Namespace

| Operator | Description        |
|----------|--------------------|
| `::`     | Namespace access   |

### Pattern Arms

| Operator | Description                        |
|----------|------------------------------------|
| `=>`     | Pattern arm in `on` blocks         |

---

## Raw Blocks

Raw blocks allow unsafe, low-level code. They require the `$unsafe` directive and always emit a compiler hint.

```prism
$unsafe
raw begin:
    -- low level operations
end;
```

The compiler scans raw blocks before bytecode generation. A catastrophic finding produces a `C` error and halts immediately.

---

## Compiler Behaviour

### Severity Levels

| Level | Prefix | Meaning                                                                 |
|-------|--------|-------------------------------------------------------------------------|
| Hint  | `H`    | Stylistic suggestion. Compiler continues normally.                      |
| Warn  | `W`    | Suspicious code. Compiler fixes it where possible and continues.        |
| Error | `E`    | Cannot compile. Terminal stays open so you can read all errors.         |
| Crit  | `C`    | Computer on fire. Catastrophic code in a raw block. Press enter — terminal closes. |

### Error Code Format

```
E0x083964/ED0x3F2A1B000000000000000000000001 at 24:3 — undefined variable
-> https://prism-lang.github.io/errors/E/0x083964/ED/0x3F2A1B000000000000000000000001
```

- Severity letter + hex category code before `/`
- Severity-prefixed D code after `/` — paste directly into the URL for full documentation
- Line and character position (`at line:char`)

### Auto-coercion

Type mismatches are warned and coerced where safe:

```
W0x001A2F/WD0x0000000000000000000000000000FF0042 at 12:7 — mismatched types
         passed int where float was expected. Coerced automatically.
```

### Optimisation Levels

| Level | Description                                              |
|-------|----------------------------------------------------------|
| 1     | Practically no optimisation                              |
| 2     | Basic optimisation                                       |
| 3     | Moderate optimisation                                    |
| 4     | Aggressive optimisation                                  |
| 5     | Maximum optimisation — lowest complexity, most instructions |

Higher levels produce more instructions with lower individual complexity.

---

## Bytecode Format (.pbc)

### Debug Profile

Human-readable hex instructions with full metadata.

### Release Profile

Raw binary. No metadata, no whitespace.

### File Structure

```
PRISM-SEVBREVSLVNUQVJU        (HEADERS-START)
    let prof: #str{"debug"}
    let opt:  #int{1}
    let mod:  #str{"MyApp"}
    let ver:  #str{"0.1"}
    let arch: #str{"x86_64"}
    let ts:   #str{"2026-04-25T00:00:00"}
PRISM-SEVBREVSLUVORA==         (HEADERS-END)

PRISM-RElSRUNUSVZFUy1TVEFSVA== (DIRECTIVES-START)
    -- compiled $directives
PRISM-RElSRUNUSVZFUy1FTkQ=     (DIRECTIVES-END)

PRISM-Q09OVEVOVC1TVEFSVA==     (CONTENT-START)  <- address 0
    -- instructions
PRISM-Q09OVEVOVC1FTkQ=         (CONTENT-END)
```

Section headers are Base64-encoded strings. The `PRISM-` prefix allows the VM to immediately validate a `.pbc` file.

### Foreign Raw Block Sections

Each `$unsafe raw` block targeting a foreign language gets its own tagged section:

```
PRISM-UlVTVA==          (RUST)
    -- raw rust
PRISM-UlVTVC1FTkQ=      (RUST-END)

PRISM-QQ==              (C)
    -- raw c
PRISM-QS1FTkQ=          (C-END)
```

---

## Instruction Set

### Operand Types

| Syntax      | Meaning                  |
|-------------|--------------------------|
| `#int{N}`   | Integer literal          |
| `#r{N}`     | Register reference (R_N) |

The VM only operates on integers. All types (strings, floats, bools) are resolved to integers by the compiler before bytecode generation.

Registers are unlimited and identified by number (`#r{0}`, `#r{1}`, ... `#r{999}`, etc.).

### Instruction Format

```
[width][0x][opcode+operands] {
    operands
}
```

Example — `ADD R0, R1 -> R2`:

```pbc
60x414444 {
    #r{0}
    #r{1}
    -> #r{2}
}
```

`->` denotes the destination operand.

### Function Layout

Functions are emitted at the top of CONTENT. A `JMP` at the end skips over them to the entry point:

```pbc
-- instruction 0: fn body
-- ...
-- instruction N: JMP to entry
50x4A4D50 {
    -> #int{N+1}
}
-- instruction N+1: entry point code starts here
```

### Instruction Reference

#### Data

| Opcode | Name  | Description                        |
|--------|-------|------------------------------------|
| `LET`  | Let   | Store a value into a register      |
| `CPY`  | Copy  | Copy one register to another       |
| `FRE`  | Free  | Free a register (GC hint)          |

#### Arithmetic

| Opcode | Name     | Description     |
|--------|----------|-----------------|
| `ADD`  | Add      | R0 + R1 -> R2   |
| `SUB`  | Subtract | R0 - R1 -> R2   |
| `MUL`  | Multiply | R0 * R1 -> R2   |
| `DIV`  | Divide   | R0 / R1 -> R2   |
| `MOD`  | Modulo   | R0 % R1 -> R2   |
| `POW`  | Power    | R0 ^ R1 -> R2   |

#### Comparison

| Opcode | Name                  |
|--------|-----------------------|
| `EQ`   | Equal                 |
| `NEQ`  | Not equal             |
| `GT`   | Greater than          |
| `LT`   | Less than             |
| `GTE`  | Greater or equal      |
| `LTE`  | Less or equal         |

All comparison instructions store `#int{1}` (true) or `#int{0}` (false) in the destination register.

#### Bitwise

| Opcode | Name         |
|--------|--------------|
| `AND`  | Bitwise and  |
| `OR`   | Bitwise or   |
| `NOT`  | Bitwise not  |
| `XOR`  | Bitwise xor  |
| `SHL`  | Shift left   |
| `SHR`  | Shift right  |

Bitwise ops take and return `#int{0}` or `#int{1}`.

```pbc
-- SHL R0 by 4 -> R1
60x53484C {
    #r{0}
    #int{4}
    -> #r{1}
}
```

#### Control Flow

| Opcode | Name                  | Description                              |
|--------|-----------------------|------------------------------------------|
| `JMP`  | Jump                  | Unconditional jump to address            |
| `JIF`  | Jump if true          | Jump if register is `#int{1}`            |
| `JNI`  | Jump if not           | Jump if register is `#int{0}`            |
| `CAL`  | Call                  | Call function at address                 |
| `RET`  | Return                | Return from function                     |
| `HLT`  | Halt                  | Halt the program                         |
| `NOP`  | No-op                 | Do nothing                               |

Jump addresses are absolute, scoped to the CONTENT section (starting at 0). Labels are resolved by the compiler — the VM only ever sees addresses.

```pbc
-- JMP to instruction 42
50x4A4D50 {
    -> #int{42} -- loop_start
}

-- JIF: if R0 is true, jump to instruction 24
60x4A4946 {
    #r{0}
    -> #int{24}
}
```

#### I/O

| Opcode | Name             | Description                          |
|--------|------------------|--------------------------------------|
| `PRT`  | Print            | Print a register value to stdout     |
| `PRM`  | Prompt           | Read a line from stdin into register |
| `PRF`  | Print formatted  | Print multiple registers combined    |

Newline is printed via `PRT #int{10}` (ASCII newline).

```pbc
-- Print R0
40x505254 {
    #r{0}
}

-- Read input into R0
40x50524D {
    -> #r{0}
}
```

---

## Keyword Reference

| Keyword  | Description                    |
|----------|--------------------------------|
| `let`    | Declare immutable variable     |
| `mut`    | Mutability modifier            |
| `fn`     | Declare function               |
| `begin`  | Open a block                   |
| `end`    | Close a block                  |
| `if`     | Conditional                    |
| `else`   | Alternate branch               |
| `for`    | Loop                           |
| `in`     | Iterator keyword               |
| `on`     | Error handler                  |
| `ok`     | Success arm in `on` block      |
| `raise`  | Raise an error                 |
| `errors` | Declare error codes            |
| `class`  | Declare a class                |
| `type`   | Declare a type alias           |
| `and`    | Logical and                    |
| `or`     | Logical or                     |
| `not`    | Logical not                    |
| `is`     | Equality operator              |
| `raw`    | Raw unsafe block               |

**Total: 20 keywords.**

---

*Prism Language Specification v0.1 — subject to change.*
