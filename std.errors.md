# Prism Standard Error Set
## Version 0.1

---

## Overview

The Prism Standard Error Set is a universal set of error codes built into every Prism program. These codes are always available without any `$use` directive — they are part of the language itself.

Standard errors live under the `STD` namespace:

```prism
on result begin:
    STD::E001 => print("Type mismatch!");
    ok        => print(result);
end;
```

Each error code follows the full Prism error format when raised by the compiler or runtime:

```
E0x######/ED0x################################ at line:char — description
-> https://prism-lang.github.io/errors/E/0x######/ED/0x################################
```

---

## Error Classes

### Class 1 — Type Errors (`E001`–`E005`)

Errors caused by invalid or incompatible type usage.

---

#### `STD::E001` — Mismatched Types

Raised when a value of one type is used where a different, incompatible type is expected and the compiler cannot safely coerce it.

**Example:**
```prism
fn add(a: int, b: int): int begin:
    add::return << a + b;
end;

add("hello", 5); -- STD::E001: str passed where int expected
```

**Notes:**
- If coercion is safe (e.g. `int` to `float`), the compiler emits `W0x001` instead and coerces automatically.
- `STD::E001` is only raised when coercion is genuinely unsafe or impossible.

---

#### `STD::E002` — Invalid Cast

Raised when an explicit cast between two types is not permitted.

**Example:**
```prism
let x: str = "hello";
let y: int = x as int; -- STD::E002: str cannot be cast to int
```

**Notes:**
- Distinct from `E001` — this is an explicit cast the developer requested, not an implicit coercion.

---

#### `STD::E003` — Overflow

Raised when an arithmetic operation produces a value exceeding the maximum for its type.

**Example:**
```prism
let x: int8 = 127;
let y: int8 = x + 1; -- STD::E003: int8 overflow
```

**Notes:**
- Does not apply to arbitrary precision types (`int`, `float`) as they have no fixed ceiling.
- Only raised for explicitly sized types (`int8`, `int16`, `int32`, `int64`, etc.).

---

#### `STD::E004` — Underflow

Raised when an arithmetic operation produces a value below the minimum for its type.

**Example:**
```prism
let x: int8 = -128;
let y: int8 = x - 1; -- STD::E004: int8 underflow
```

**Notes:**
- Same scoping rules as `E003` — only applies to explicitly sized types.

---

#### `STD::E005` — Division by Zero

Raised when any value is divided by zero.

**Example:**
```prism
let x: int = 10;
let y: int = 0;
let z: int = x / y; -- STD::E005: division by zero
```

**Notes:**
- Always a hard error. There is no safe coercion path for division by zero.

---

### Class 2 — Variable Errors (`E006`–`E008`)

Errors caused by incorrect variable declaration or usage.

---

#### `STD::E006` — Undefined Variable

Raised when a variable is referenced before it has been declared.

**Example:**
```prism
print(x); -- STD::E006: x is not defined
```

---

#### `STD::E007` — Immutable Assignment

Raised when an attempt is made to assign a new value to an immutable variable declared with `let`.

**Example:**
```prism
let x: int = 5;
x << 10; -- STD::E007: x is immutable, use `let mut`
```

**Notes:**
- The compiler message suggests using `let mut` to fix the issue.

---

#### `STD::E008` — Uninitialized Variable

Raised when a variable is declared but used before being assigned a value.

**Example:**
```prism
let mut x: int;
print(x); -- STD::E008: x has not been initialized
```

---

### Class 3 — Function Errors (`E009`–`E011`)

Errors caused by incorrect function calls or definitions.

---

#### `STD::E009` — Invalid Argument Count

Raised when a function is called with the wrong number of arguments.

**Example:**
```prism
fn greet(name: str, age: int): str begin:
    greet::return << name;
end;

greet("Lucas"); -- STD::E009: expected 2 arguments, got 1
```

---

#### `STD::E010` — Invalid Argument Type

Raised when a function is called with an argument of the wrong type that cannot be safely coerced.

**Example:**
```prism
fn greet(name: str): str begin:
    greet::return << name;
end;

greet(42); -- STD::E010: expected str, got int
```

**Notes:**
- If safe coercion is possible, the compiler emits `W0x002` instead.

---

#### `STD::E011` — Missing Return Value

Raised when a function declares a non-void return type but does not always return a value.

**Example:**
```prism
fn add(a: int, b: int): int begin:
    if a is 0 begin:
        add::return << 0;
    end;
    -- STD::E011: function does not always return a value
end;
```

---

### Class 4 — I/O Errors (`E012`–`E015`)

Errors caused by input/output failures.

---

#### `STD::E012` — File Not Found

Raised when an attempt is made to access a file that does not exist.

**Example:**
```prism
let content = read_file("missing.txt"); -- STD::E012
```

---

#### `STD::E013` — Permission Denied

Raised when the program does not have sufficient permissions to access a file or resource.

**Example:**
```prism
let content = read_file("/etc/shadow"); -- STD::E013
```

---

#### `STD::E014` — Read Failure

Raised when a file or stream exists and is accessible but cannot be read due to an unexpected failure.

**Example:**
```prism
let content = read_file("corrupted.txt"); -- STD::E014
```

**Notes:**
- Distinct from `E012` — the file exists. The read itself failed.

---

#### `STD::E015` — Write Failure

Raised when a write operation to a file or stream fails unexpectedly.

**Example:**
```prism
write_file("output.txt", "data"); -- STD::E015
```

**Notes:**
- Common causes include full disks, broken pipes, or revoked write permissions mid-operation.

---

### Class 5 — Runtime Errors (`E016`–`E020`)

Errors caused by unexpected conditions during program execution.

---

#### `STD::E016` — Stack Overflow

Raised when the call stack exceeds its limit, typically caused by unbounded recursion.

**Example:**
```prism
fn recurse(): void begin:
    recurse();
    recurse::return << void;
end;

recurse(); -- STD::E016: stack overflow
```

---

#### `STD::E017` — Out of Memory

Raised when the runtime cannot allocate enough memory to continue execution.

**Notes:**
- This error cannot always be caught gracefully. The runtime makes a best-effort attempt to surface it before halting.

---

#### `STD::E018` — Null Reference

Raised when a variable or field is accessed but holds no value.

**Example:**
```prism
let mut a: Animal;
print(a.name); -- STD::E018: null reference on `a`
```

---

#### `STD::E019` — Index Out of Bounds

Raised when an array is accessed with an index outside its valid range.

**Example:**
```prism
let scores: [int] = [1, 2, 3];
print(scores[5]); -- STD::E019: index 5 out of bounds for array of length 3
```

---

#### `STD::E020` — Module Not Found

Raised when a `$use` directive references a module that cannot be located.

**Example:**
```prism
$use "NonExistent" -- STD::E020: module NonExistent not found
```

---

## Quick Reference

| Code       | Class    | Description              |
|------------|----------|--------------------------|
| `STD::E001`| Type     | Mismatched types         |
| `STD::E002`| Type     | Invalid cast             |
| `STD::E003`| Type     | Overflow                 |
| `STD::E004`| Type     | Underflow                |
| `STD::E005`| Type     | Division by zero         |
| `STD::E006`| Variable | Undefined variable       |
| `STD::E007`| Variable | Immutable assignment     |
| `STD::E008`| Variable | Uninitialized variable   |
| `STD::E009`| Function | Invalid argument count   |
| `STD::E010`| Function | Invalid argument type    |
| `STD::E011`| Function | Missing return value     |
| `STD::E012`| I/O      | File not found           |
| `STD::E013`| I/O      | Permission denied        |
| `STD::E014`| I/O      | Read failure             |
| `STD::E015`| I/O      | Write failure            |
| `STD::E016`| Runtime  | Stack overflow           |
| `STD::E017`| Runtime  | Out of memory            |
| `STD::E018`| Runtime  | Null reference           |
| `STD::E019`| Runtime  | Index out of bounds      |
| `STD::E020`| Runtime  | Module not found         |

---

*Prism Standard Error Set v0.1 — subject to change.*
