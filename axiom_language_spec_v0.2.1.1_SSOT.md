# AXIOM LANGUAGE SPEC v0.2.1.1 (SSOT FINAL)

**Status:** Final (normative)
**Spec Version:** v0.2.1.1
**Scope:** Language + locked stdlib surface + tooling/diagnostics contracts  
**Design Goal:** AI-first, deterministic, anti-slop — *sloppy code does not compile.*

---

## CHANGELOG v0.2.1.1
- **P0-1:** EBNF erweitert, so dass Appendix A (locked stdlib stubs) syntaktisch parsebar ist, inkl. `impl Type { fn ... }` (inherent impls) und signatures-only `fn`-Deklarationen im Stub-Kontext.
- **P0-1:** Normativ festgelegt: Inherent `impl Type { ... }` Blöcke sind **stdlib-only**; User-Code MUSS sie ablehnen (E0211).
- **P0-2:** Neue normative Sektion: Reserved Prelude/Stdlib Identifiers; Shadowing/Hijacking ist verboten (E0210).
- Error Code Catalog um E0210/E0211 ergänzt (closed set bleibt gültig).
- Appendix A stubs: `impl Copy for FileRead<P>`/`FileWrite<P>`/`Network<H>` formalisiert als `impl<P>`/`impl<H>`; opaque-record Platzhalter in parsebare Form gebracht.

## Patch Units
- **P-001:** Version/Status bump auf v0.2.1.1; Status → Final (normative).
- **P-002:** Reserved Prelude Identifiers + E0210.
- **P-003:** Inherent impl blocks (stdlib-only) + E0211; EBNF + Appendix A Parse-Closure.

---

## 0. Normativity and Terminology

This document is the **Single Source of Truth (SSOT)** for Axiom v0.2.1.1. It is **self-contained** and MUST NOT rely on earlier spec versions.

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirements.

Unless explicitly marked **(Non‑Normative)**, all rules in this document are normative.

### 0.1 Determinism

A conforming compiler:

- MUST reject any program whose meaning depends on unspecified evaluation order, unspecified implicit conversions, unspecified imports, or unspecified defaults.
- MUST implement all “closed sets” in this spec exactly (keywords, effects, builtin lints, diagnostics schema, stdlib surface).
- MUST produce diagnostics using the stable schema in §22.

### 0.2 Example Policy

All Axiom code examples in this spec are explicitly labeled:

- **COMPILE-PASS**: Must parse and type-check against the locked stdlib stubs in Appendix A.
- **COMPILE-FAIL**: Must be rejected with the given error code and a short reason.

A compiler MAY produce additional notes, but MUST include the referenced error code.

---

## 1. Source Files, Crates, and Modules

### 1.1 Source Encoding

- Source files MUST be UTF‑8.
- A file MUST NOT contain invalid UTF‑8 sequences. (**E0001**)

### 1.2 File Extensions

- Axiom source files MUST use the extension `.ax`.

### 1.3 Crate Roots

A crate is a compilation unit.

- A binary crate root file is `src/main.ax`.
- A library crate root file is `src/lib.ax`.

The crate root defines the root module `crate`.

### 1.4 Module File Mapping

A module path `crate::a::b` resolves to exactly one of:

1) `src/a/b.ax`  
2) `src/a/b/mod.ax`

If both exist, it is an error. (**E0702**)

If neither exists, it is an error. (**E0703**)

Inline modules (`mod name { ... }`) do not use file mapping.

---

## 2. Lexical Structure

### 2.1 Tokens

The lexer produces the following token classes:

- Identifiers: `ident`
- Keywords: listed in §2.2
- Literals: §2.5
- Operators and delimiters: §2.6
- Newline token: `NL`
- End-of-file token: `EOF`

### 2.2 Keywords (Closed Set)

The following are the **only** keywords in Axiom v0.2.1.1:

`let, mut, fn, type, trait, impl, pub, mod, use, return, if, else, match, loop, for, in, break, continue, as, where, async, await`

Any other word is an identifier.

### 2.3 Identifiers

- An identifier is `/[A-Za-z_][A-Za-z0-9_]*/` excluding keywords.
- Identifiers are case-sensitive.

### 2.4 Comments

- Line comments start with `//` and extend to the end of the line.
- Block comments do not exist.

### 2.5 Literals

Supported literals:

- Integer literals: decimal digits, optionally `_` separators  
  Examples: `0`, `42`, `1_000_000`
- Float literals: digits `.` digits, optionally `_` separators  
  Example: `3.14`
- Boolean literals: `true`, `false`
- String literals: double-quoted, with escapes `\\n`, `\\t`, `\\"`, `\\\\`
- Char literals: single-quoted, single Unicode scalar value, with the same escapes as strings

Integer literal default type is `i32`. Float literal default type is `f64`.

A compiler MUST NOT guess other numeric types via context unless the context uniquely constrains it.

### 2.6 Operators and Delimiters (Tokens)

Delimiters: `(` `)` `{` `}` `[` `]` `,` `:` `::` `->` `=>`

Operators: `=` `+` `-` `*` `/` `%` `!` `+` (effects separator)  
Comparison: `==` `!=` `<` `<=` `>` `>=`  
Boolean: `&&` `||`  
Ranges: `..` `..=`  
Postfix: `?` `.await`

### 2.7 Newlines and Statement Separation (NL Semantics)

Axiom is **semicolon-free**.

- The lexer emits an `NL` token for each physical line break **outside** string/char literals.
- The parser treats `NL` as **required statement separator** in statement position.
- The parser treats `NL` as **whitespace** inside:
  - `(...)`, `[...]`, `{...}` when parsing expressions
  - after an operator token (infix/prefix)
  - after `,` in argument lists / tuple / record literals

#### 2.7.1 One Statement or Item per Line

A file MUST NOT contain two statements or two items on the same physical line. (**E0008**)

A “statement” here means:
- a `let` binding statement
- an assignment statement
- a `return` statement
- a `break` / `continue` statement
- an expression statement

An “item” here means a module-level declaration:
- `use` item
- `mod` item
- `type` item
- `trait` item
- `impl` item
- `fn` item
- an attribute line `#[...]`

---

## 3. Expressions

### 3.1 Evaluation Order

Unless stated otherwise:

- Function call arguments are evaluated left-to-right.
- Tuple/record/array literal elements are evaluated left-to-right.
- Binary operators evaluate left operand then right operand.
- `match` evaluates its scrutinee first; then evaluates only the selected arm.

### 3.2 Blocks

A block has the form:

`{ <stmt_or_expr> (NL <stmt_or_expr>)* (NL)? }`

- The value of a block is the value of its final expression if present, otherwise `()`.

### 3.3 Variables and Paths

A name resolves to the nearest binding in lexical scope, else to an imported item, else to a prelude name.

#### 3.3.1 Reserved Prelude Identifiers (P0)

Axiom defines a **reserved identifier set** to prevent shadowing/hijacking of the prelude/stdlib surface.

**Rule (normative):** In **user code** (i.e., any non-stdlib source file), a program MUST NOT introduce a binding with any reserved identifier in any namespace (type/module/value/pattern).

This prohibition applies to **all** binding sites, including but not limited to:
- `type`, `trait`, `mod`, `fn` item names
- `use ... as <ident>` aliases
- generic parameter names (`<T>`, `<P>`, ...)
- function parameter names
- `let` bindings and all pattern-bound identifiers

Reserved identifiers (minimum closed set for v0.2.1.1):
- Types/values/intrinsics: `Raw`, `Result`, `Option`, `String`, `AllCaps`, `FileRead`, `FileWrite`, `Network`, `Future`, `Base`, `validate`
- Stdlib module names: `io`, `fmt`, `fs`, `http`, `iter`, `time`, `rand`, `task`, `chan`, `assert`

**Diagnostic (normative):** Violations MUST be rejected with `E0210 reserved_name_redefinition`.

**Note (normative):** This rule does **not** apply inside the locked stdlib stub file (Appendix A), which defines these identifiers.

**COMPILE-FAIL** (E0210 reserved_name_redefinition):

```axiom
fn main() -> () {
  let Raw: i32 = 0
}
```


Paths use `::` segments: `io::println`, `crate::math::add`.

### 3.4 Function Calls

`f(arg1, arg2, ...)`

Generic instantiation uses **turbofish**:

`f::<T>(arg)`

If a call supplies explicit generic arguments without `::<...>`, it is an error. (**E0010**)

### 3.5 Field Access

- Record fields are accessed with `expr.field`.
- Field access on an opaque type is an error unless the field exists in its definition. (**E0201**)

### 3.6 Operators and Precedence

From highest to lowest:

1. Postfix: `.await`, `?`, function call `(...)`, field access `.field`
2. Prefix: `-`, `!`, `&`, `&mut`
3. Multiplicative: `*`, `/`, `%`
4. Additive: `+`, `-`
5. Range: `..`, `..=`
6. Comparison: `<`, `<=`, `>`, `>=`, `==`, `!=`
7. Boolean AND: `&&`
8. Boolean OR: `||`
9. Assignment: `=` (statement-only; not an expression)

Chained comparisons are forbidden. (**E0009**)

### 3.7 If Expressions

`if cond { ... } else { ... }`

- `cond` MUST have type `bool`.
- Both branches MUST have the same type (or both be `()`).

### 3.8 Match Expressions

`match expr { pattern (if guard)? => expr NL ... }`

- `match` MUST be exhaustive.
- For scrutinee types with infinite domain (`i*`, `u*`, `f*`, `char`, `str`, `String`), exhaustiveness requires a final wildcard arm `_ => <expr>`.
- For finite domain scrutinee types (`bool`, `Option<T>`, `Result<T, E>`, and user-defined sum types), arms MUST cover all variants; a final wildcard arm is permitted as a shorthand for remaining cases.
- Wildcard `_` is permitted only as the **final** arm. (**E0109**)

---

## 4. Statements

### 4.1 Let Bindings

`let name: Type = expr`  
`let mut name: Type = expr`  
Type annotation MAY be omitted only if the compiler can infer a unique type **without defaults**, except the numeric literal defaults in §2.5.

Binding `let _ = expr` is permitted only if `expr` is **not** a MUST-HANDLE type (§11). Otherwise error **E0904**.

### 4.2 Assignment

`name = expr`

- Assignment requires that `name` was declared `mut`.
- Assignment is a statement and does not yield a value.

### 4.3 Return / Break / Continue

`return expr`  
`break`  
`continue`  
Labels are written as `'label:` and referenced as `break 'label` or `continue 'label`.

---

## 5. Types

### 5.1 Primitive Types (Closed Set)

Signed integers: `i8 i16 i32 i64`  
Unsigned integers: `u8 u16 u32 u64`  
Floats: `f32 f64`  
Other: `bool char`

Unsized string slice: `str` (may only appear behind a reference as `&str`).

`()` is the unit type.

### 5.2 References

`&T` immutable reference  
`&mut T` mutable reference

Borrow rules are defined in §13.

### 5.3 Tuples

Tuple type: `(T1, T2, ...)`  
Tuple value: `(e1, e2, ...)`

A 1-tuple uses a trailing comma: `(e,)`.

### 5.4 Arrays and Slices

#### 5.4.1 Arrays (fixed size)

Array type uses **comma** separator:

`[T, N]` where `N` is a positive integer literal.

Array literal: `[e1, e2, ...]` with exactly `N` elements.

A compiler MUST reject `[T; N]` syntax. (**E0102**)

#### 5.4.2 Slices

Slice type: `&[T]` or `&mut [T]`.

### 5.5 Records

Record type definition uses `type` (see §6.1).

Record literal:

`User { id: 1, name: String::from("A") }`

Field order in literals is irrelevant, but duplicates are forbidden. (**E0207**)

### 5.6 Sum Types (Enums)

Sum type definition uses `type` (see §6.2).

Variants are referenced as `Type::Variant`.

### 5.7 Nominal Newtypes (CRITICAL)

#### 5.7.1 Definition

`type UserId = u64`

This defines a **nominal** type distinct from `u64`. It is NOT an alias.

A compiler MUST NOT treat `UserId` as interchangeable with `u64`. (**E0205**)

#### 5.7.2 Construction and Destructuring

Newtype constructor:

`UserId(42)`

Pattern destructuring:

`match id { UserId(x) => ... }`

There is no field access or implicit conversion for newtypes.

### 5.8 Refinement Types (where DSL)

#### 5.8.1 Definition

`type Port = u16 where 1..=65535`

A refinement type is a nominal type with an attached predicate over its base type.

The refinement DSL is **closed**:

- Numeric range: `LO ..= HI` where LO and HI are integer/float literals.
- Length predicate: `len OP N` where:
  - `len` means `len(self)` (length of the base value)
  - OP ∈ `{>, >=, ==, <=, <}`
  - N is a non-negative integer literal
  - Valid only for base types `String` and `Vec<T>`

Regex or general string predicates do not exist. (**E0800**)

#### 5.8.2 Construction Rule

A value of refinement type `R` MUST NOT be constructed from a non-literal base value via `R(x)`.

The only allowed constructions are:

1) **Compile-time proven literals**:
   - `let p: Port = Port(80)` is permitted because `80` is a literal and satisfies the predicate.
2) `validate::<R>(x)` which returns `Result<R, ValidationError<R>>` (see §7).

Attempting `Port(x)` where `x` is not a literal MUST be rejected. (**E0801**)

---

## 6. Type Declarations

Axiom uses a single keyword `type` to declare all user-defined types.

### 6.1 Record Type

`type User = { id: UserId, name: String }`

- Record fields are named.
- Field names MUST be unique within the type. (**E0207**)

### 6.2 Sum Type

Sum types use `|`-separated variants:

`type Status = | Status::Pending | Status::Active { since: u64 }`

Variant forms:

- Unit variant: `Type::Variant`
- Payload variant: `Type::Variant { field: Type, ... }` (record payload)

### 6.3 Newtype and Refinement Type

See §5.7 and §5.8.

### 6.4 Type Aliases

Type aliases do not exist in v0.2.1.1.

Any syntax resembling an alias MUST be interpreted as a newtype or rejected if it would create ambiguity. (**E0205**)

---

## 7. Raw<T> and Validation (CRITICAL)

### 7.1 Raw<T> (Taint)

`Raw<T>` represents tainted/untrusted data.

#### 7.1.1 Opaqueness

`Raw<T>` is **opaque**:

- It has no accessible fields.
- It has no accessible constructors.
- It MUST NOT be pattern-matched.

Any attempt to access or destructure a `Raw<T>` value MUST be rejected. (**E0501**)

### 7.2 validate::<R>(...) Intrinsic

Axiom provides an intrinsic `validate` in the prelude.

`Base<R>` is a compiler intrinsic type operator that yields the base type of `R`:

- If `R` is a newtype, `Base<R>` is its underlying type.
- If `R` is a refinement, `Base<R>` is its underlying type.
- If `R` is neither, `Base<R> = R`.

`Base<R>` is available only in type positions.

Two overloads exist:

- `validate::<R>(x: Raw<Base<R>>) -> Result<R, ValidationError<R>>`
- `validate::<R>(x: Base<R>) -> Result<R, ValidationError<R>>`

`validate` is **pure** (no effects).

### 7.3 ValidationError<R>

`ValidationError<R>` is a stdlib type (Appendix A) with a stable, closed structure.

A compiler MUST treat `ValidationError<R>` as MUST-HANDLE (§11) because it is wrapped in `Result`.

### 7.4 Example

**COMPILE-PASS:**

```axiom
type Email = String where len > 3

fn main() -> Result<(), IoError> ! IO {
  let raw: Raw<String> = io::read_line()?
  let v: Result<Email, ValidationError<Email>> = validate::<Email>(raw)
  match v {
    Result::Ok { value: email } => {
      match email {
        Email(s) => io::println(s)
      }
      Result::Ok { value: () }
    }
    Result::Err { error: ve } => {
      match ve {
        ValidationError::Failed { message: msg } => Result::Err { error: IoError::Other { message: msg } }
      }
    }
  }
}
```

---

## 8. Traits and impl (Minimal)

Traits are nominal interfaces for ad-hoc polymorphism.

### 8.1 Trait Declarations

`trait Display { }`

A trait declaration in v0.2.1.1 contains **no methods**. It is a marker used only for constraints.

### 8.2 impl Blocks

`impl Display for String { }`

- `impl` MAY only implement marker traits (no methods).
- Overlapping impls are forbidden. (**E0601**)

(Reason: method-based traits and coherence rules are out of scope for v0.2.1.1.)

---


### 8.3 Inherent `impl Type { ... }` Blocks (stdlib-only, P0)

Appendix A (locked stdlib stubs) contains **inherent impl blocks** of the form `impl <Type> { <fn_decl>* }`.

**Rule (normative):**
- Inherent impl blocks (`impl <Type> { ... }` without `for`) MAY appear **only** in the locked stdlib stub file (Appendix A / `stdlib_stubs_v0.2.1.1_FINAL.ax`).
- User code MUST NOT declare inherent impl blocks. If a user source file contains an inherent impl block, the compiler MUST reject it with `E0211 inherent_impl_forbidden_in_user_code`.

**Semantics (minimal, normative):**
- Functions declared inside an inherent impl block are **associated functions** of the type.
- Calls to inherent functions MUST use qualified paths: `Type::fn(...)` or `Type::fn::<...>(...)`.
- The language defines **no general method-call syntax** for inherent functions (i.e., `value.fn(...)` is not a feature in v0.2.1.1).

**COMPILE-FAIL** (E0211 inherent_impl_forbidden_in_user_code):

```axiom
type X = { }

impl X {
  fn bad(self: &X) -> ()
}
```

## 9. Functions

### 9.1 Function Declarations

`fn name(param: Type, ...) -> ReturnType { ... }`

Parameters MUST be typed.

Return type MAY be omitted only if it is `()`.

### 9.2 Async Functions

`async fn fetch(url: Url) -> Result<String, HttpError> ! Async + IO { ... }`

Rules:

- An `async fn` MUST include `Async` in its effect set. Otherwise error **E0308**.
- Calling an `async fn` produces a `Future<T>` value, where `T` is the declared return type.
- `.await` may only be used inside an `async fn`. Otherwise error **E0302**.

### 9.3 Closures (Restricted)

Closure literal syntax:

`|x: i32| x + 1`  
`|| 42`

In v0.2.1.1, closure values are **pure-only**:

- A closure body MUST NOT call effectful functions. Otherwise error **E0307**.
- A closure value has type `fn(...) -> R`.

---

## 10. Control Flow

### 10.1 loop

`loop { ... }`

`break` exits the nearest loop unless labeled.

### 10.2 for (Restricted, No Iterator Traits) (CRITICAL)

A `for` loop has the form:

`for pat in iterable { body }`

The iterable MUST be one of:

1) `lo .. hi` where `lo` and `hi` are `i32`
2) `lo ..= hi` where `lo` and `hi` are `i32`
3) `&Vec<T>`
4) `&[T]`

If not, the compiler MUST reject with **E0110**.

Iteration yields:

- For ranges: `i32` values.
- For `&Vec<T>` and `&[T]`: `&T` values.

### 10.3 Example

**COMPILE-FAIL (E0110 invalid_for_iterable):**

```axiom
fn main() -> () ! IO {
  let x: i32 = 1
  for n in x {
    io::println(n)
  }
}
```

---

## 11. MUST-HANDLE (CRITICAL)

### 11.1 MUST-HANDLE Types (Closed Set)

The following types are MUST-HANDLE:

- `Result<T, E>`
- `Option<T>`
- `Raw<T>`
- `Future<T>`

### 11.2 Forbidden Ignoring (Expression Statement Rule)

An expression statement whose type is MUST-HANDLE is forbidden. (**E0903**)

### 11.3 Forbidden Underscore Bindings

Binding a MUST-HANDLE value to `_` or to any name starting with `_` is forbidden. (**E0904**)

### 11.4 Bound-but-Unused at Scope End (CRITICAL)

At the end of any block scope, any local variable of MUST-HANDLE type that is still live (not moved) MUST be reported as an error **E0911**.

A MUST-HANDLE value is considered **used** if it is moved by value in one of these ways:

- as the operand of `?`
- as the operand of `.await`
- as the scrutinee of a by-value `match`
- as the operand of `return`
- as an argument passed to a parameter of by-value type (including method receiver `self` by value)

Borrowing (`&x`) does not count as using.

### 11.5 Example

**COMPILE-FAIL (E0911 must_handle_bound_but_unused):**

```axiom
fn main() -> () ! IO {
  let u: Result<Url, UrlError> = Url::parse("https://example.com")
  io::println("done")
}
```

---

## 12. Effects

### 12.1 Effect Set (Closed Set)

The only effects in v0.2.1.1 are:

`IO, Db, Async, Random, Time, Log, Spawn`

Any other effect name is an error. (**E0308**)

### 12.2 Declaring Effects

A function declares effects with:

`fn f() -> () ! IO + Time { ... }`

- A function with no `!` clause is pure.
- Effects are unordered; duplicates are forbidden. (**E0309**)

### 12.3 Effect Checking

If function `f` calls function `g`, then `effects(g) ⊆ effects(f)` MUST hold. Otherwise error **E0301**.

---

## 13. Ownership and Borrowing (Minimal)

### 13.1 Copy Types

The following are Copy:

- All primitive types (§5.1)
- All references (`&T`, `&mut T`)
- All capability token types (§16)

All other types are move-only unless the stdlib marks them Copy (not allowed in v0.2.1.1).

### 13.2 Moves

Moving a non-Copy value transfers ownership and invalidates the source binding. Using a moved value is an error. (**E0401**)

### 13.3 Borrow Rules

- Any number of immutable borrows `&T` MAY coexist.
- At most one mutable borrow `&mut T` MAY exist at a time.
- A mutable borrow is exclusive: while active, no other borrows are allowed.

Violations are errors. (**E0404**)

---

## 14. Capabilities (CRITICAL)

Capabilities are unforgeable tokens representing authority, tracked by the type system.

### 14.1 Capability Types

Capability types are parameterized by a **policy literal** (a compile-time string literal):

- `FileRead<"glob">`
- `FileWrite<"glob">`
- `Network<"host">`

The generic argument MUST be a string literal token. Otherwise error **E0403**.

### 14.2 AllCaps Minting

The environment supplies `AllCaps` only to the crate entry point `main`.

Minting methods:

- `caps.file_read::<"glob">() -> FileRead<"glob">`
- `caps.file_write::<"glob">() -> FileWrite<"glob">`
- `caps.network::<"host">() -> Network<"host">`

A call to a minting method MUST use turbofish `::<"...">`. Otherwise error **E0402**.

### 14.3 Using Capabilities

Stdlib functions require explicit capability parameters (Appendix A). There is no ambient authority.

### 14.4 Example

**COMPILE-PASS:**

```axiom
fn main(caps: AllCaps) -> Result<(), IoError> ! IO {
  let r: FileRead<"data/**"> = caps.file_read::<"data/**">()
  let p: Path = Path::new("data/input.txt")
  let s: String = fs::read_text(r, p)?
  io::println(s)
  Result::Ok { value: () }
}
```

**COMPILE-FAIL (E0402 missing_capability_turbofish):**

```axiom
fn main(caps: AllCaps) -> () {
  let r: FileRead<"data/**"> = caps.file_read()
}
```

---

## 15. Modules, Imports, and Layers

### 15.1 mod

Inline module:

`mod name { ... }`

File modules are discovered via §1.4 and do not require a `mod name;` item.

### 15.2 use (Absolute Only)

Import syntax:

`use crate::path::to::Item`  
`use crate::path::to::Item as Alias`

Relative imports are forbidden. (**E0704**)

### 15.3 Visibility

Visibility modifiers:

- `pub` — exported from the module
- `pub(crate)` — exported within crate
- `pub(super)` — exported to parent module

`crate` and `super` are **contextual** keywords only inside `pub(...)` and `use` roots.

### 15.4 Layers (Enforced)

A module MAY declare its layer:

`#[layer(domain, depth=0)]`

Rules:

- A crate MUST define exactly these layer names (closed): `domain, app, infra, ui, test`.
- `depth` MUST be an integer literal.
- Imports MUST NOT go from lower layers to higher layers (e.g., `domain` MUST NOT import `infra`). (**E0710**)

If a module has no `#[layer(...)]`, it defaults to `app` depth 0.

---

## 16. Budgets (Anti-Slop)

Budgets are enforced by attributes. Missing attributes use defaults.

### 16.1 Budget Attributes (Closed Set)

- `#[max_lines(N)]`
- `#[max_params(N)]`
- `#[max_nesting(N)]`
- `#[max_complexity(N)]`

All `N` MUST be non-negative integer literals.

### 16.2 Defaults

If not specified:

- `max_lines = 60`
- `max_params = 4`
- `max_nesting = 3`
- `max_complexity = 10`

### 16.3 Counting Rules

All counting is performed on the **formatted** output (§21 formatter contract).

- **Lines**: count of non-empty, non-comment lines in the function body, excluding the outer `{` and `}`.
- **Params**: number of parameters in the function signature.
- **Nesting**: maximum nesting depth of these constructs: `if`, `match`, `loop`, `for`.
  - Each occurrence increases depth by 1 within its body.
- **Complexity**: defined as:

  `1 + (#if) + (#for) + (#loop) + Σ(match_arms - 1)`

### 16.4 Violations

Budget violations are errors:

- `E0802 max_lines_exceeded`
- `E0803 max_params_exceeded`
- `E0804 max_nesting_exceeded`
- `E0805 max_complexity_exceeded`

---

## 17. Standard Library (Locked Surface)

The Axiom standard library surface is **locked** in Appendix A. A compiler MUST ship exactly that surface for v0.2.1.1.

- Programs MUST NOT assume any additional stdlib APIs exist.
- Tools and agents MUST treat Appendix A as the only available stdlib.

### 17.1 Opaque Stdlib Types (Closed Set)

The following stdlib types are **opaque** in v0.2.1.1:

- `String`
- `Vec<T>`, `Map<K, V>`, `Set<T>`
- `Path`, `Url`
- `Future<T>`
- `AllCaps`
- `FileRead<P>`, `FileWrite<P>`, `Network<H>`
- `Raw<T>` (special rules in §7)

User code MUST NOT:

- construct values of these types via record literals or enum variant literals
- destructure or pattern-match values of these types

Violation MUST be rejected:

- for `Raw<T>` with **E0501**
- for all other opaque stdlib types with **E0202**

---

## 18. Tooling Contracts (Summary)

### 18.1 Formatter (Idempotent)

- Formatting MUST be deterministic and idempotent: formatting the output again produces byte-identical output. (**E1201**)

### 18.2 Diagnostics JSON

Compilers MUST emit diagnostics in the stable schema defined in §22.

---

## 19. Examples (Conformance)

### 19.1 Hello World

**COMPILE-PASS:**

```axiom
fn main() -> () ! IO {
  io::println("Hello, World!")
}
```

### 19.2 FizzBuzz

**COMPILE-PASS:**

```axiom
fn fizzbuzz(n: i32) -> String {
  match (n % 3, n % 5) {
    (0, 0) => String::from("FizzBuzz")
    (0, _) => String::from("Fizz")
    (_, 0) => String::from("Buzz")
    _ => fmt::to_string(n)
  }
}

fn main() -> () ! IO {
  for n in 1..=20 {
    io::println(fizzbuzz(n))
  }
}
```

### 19.3 MUST-HANDLE via `?`

**COMPILE-PASS:**

```axiom
fn main() -> Result<(), UrlError> ! IO {
  let u: Url = Url::parse("https://example.com")?
  io::println(u)
  Result::Ok { value: () }
}
```
### 19.4 format1 Contract Violations

**COMPILE-FAIL (E0204 format_string_not_literal):**

```axiom
fn main() -> () ! IO {
  let s: &str = "x={}"
  let out: String = fmt::format1(s, 1)
  io::println(out)
}
```

**COMPILE-FAIL (E0203 invalid_format_string):**

```axiom
fn main() -> () ! IO {
  let out: String = fmt::format1("a={} b={}", 1)
  io::println(out)
}
```

---

## 20. Diagnostics (Normative)

### 20.1 Error Code Requirement

Every diagnostic MUST have a `code` field that is present in the Error Code Catalog (Appendix B).

### 20.2 Stable JSON Shape (axiom.diagnostics.v1)

Diagnostics MUST be emitted as UTF‑8 JSON objects with:

Required top-level fields:

- `schema`: MUST equal `"axiom.diagnostics.v1"`
- `compiler`: string (compiler name)
- `version`: string (compiler version)
- `diagnostics`: array of Diagnostic objects

A Diagnostic object has required fields:

- `code`: string, e.g. `"E0903"`
- `severity`: one of `"error" | "warning" | "note"`
- `message`: string (human-readable, single line)
- `primary_span`: Span object
- `spans`: array of Span objects (MAY be empty)
- `notes`: array of strings (MAY be empty)
- `help`: array of HelpItem objects (MAY be empty)

Span object:

- `file`: string (path)
- `start`: integer byte offset (0-based, inclusive)
- `end`: integer byte offset (0-based, exclusive)
- `line`: integer (1-based)
- `col`: integer (1-based)

HelpItem object:

- `kind`: `"suggestion" | "fix-it"`
- `message`: string
- `replacement`: optional object:
  - `file`: string
  - `start`: integer
  - `end`: integer
  - `text`: string

All offsets MUST be in UTF‑8 bytes.

---

## Appendix A: stdlib_stubs_v0.2.1.1_FINAL.ax (Locked, Signatures-Only)

**COMPILE-PASS (stdlib stubs):**

```axiom
// Axiom Standard Library Stubs v0.2.1.1
// Locked surface. Signatures only.

/////////////////////////
// Marker Traits
/////////////////////////

trait Copy { }
trait Eq { }
trait Ord { }
trait Hash { }
trait Display { }
trait Debug { }

/////////////////////////
// Core Sum Types
/////////////////////////

type Option<T> =
  | Option::Some { value: T }
  | Option::None

type Result<T, E> =
  | Result::Ok { value: T }
  | Result::Err { error: E }

/////////////////////////
// Core Opaque Types
/////////////////////////

type String = { }
type Vec<T> = { }
type Map<K, V> = { }
type Set<T> = { }

type Raw<T> = { }

type Path = { }
type Url = { }

type Future<T> = { }

/////////////////////////
// Core Errors
/////////////////////////

type IoError =
  | IoError::NotFound
  | IoError::PermissionDenied
  | IoError::InvalidInput
  | IoError::Other { message: String }

type UrlError =
  | UrlError::InvalidUrl

type HttpError =
  | HttpError::Network
  | HttpError::BadStatus { status: u16 }

type ValidationError<R> =
  | ValidationError::Failed { message: String }

/////////////////////////
// Capabilities
/////////////////////////

type AllCaps = { }

type FileRead<P> = { }
type FileWrite<P> = { }
type Network<H> = { }

/////////////////////////
// Trait Impls (Marker Only)
/////////////////////////

impl Copy for i8 { }
impl Copy for i16 { }
impl Copy for i32 { }
impl Copy for i64 { }
impl Copy for u8 { }
impl Copy for u16 { }
impl Copy for u32 { }
impl Copy for u64 { }
impl Copy for f32 { }
impl Copy for f64 { }
impl Copy for bool { }
impl Copy for char { }

impl Display for i8 { }
impl Display for i16 { }
impl Display for i32 { }
impl Display for i64 { }
impl Display for u8 { }
impl Display for u16 { }
impl Display for u32 { }
impl Display for u64 { }
impl Display for f32 { }
impl Display for f64 { }
impl Display for bool { }
impl Display for char { }
impl Display for String { }
impl Display for Url { }
impl Display for Path { }
impl Display for &str { }

impl Copy for AllCaps { }
impl<P> Copy for FileRead<P> { }
impl<P> Copy for FileWrite<P> { }
impl<H> Copy for Network<H> { }

/////////////////////////
// String
/////////////////////////

impl String {
  fn from(s: &str) -> String
  fn len(self: &String) -> u64
}

/////////////////////////
// Path, Url
/////////////////////////

impl Path {
  fn new(s: &str) -> Path
}

impl Url {
  fn parse(s: &str) -> Result<Url, UrlError>
}

/////////////////////////
// Prelude Intrinsics
/////////////////////////

fn validate<R>(x: Raw<Base<R>>) -> Result<R, ValidationError<R>>
fn validate<R>(x: Base<R>) -> Result<R, ValidationError<R>>

/////////////////////////
// fmt
/////////////////////////

mod fmt {
  fn to_string<T: Display>(v: T) -> String
  fn format1<T: Display>(fmt_literal: &str, a0: T) -> String
  fn count_lines(s: String) -> u64
}

/////////////////////////
// io
/////////////////////////

mod io {
  fn println<T: Display>(v: T) -> () ! IO
  fn read_line() -> Result<Raw<String>, IoError> ! IO
}

/////////////////////////
// fs
/////////////////////////

mod fs {
  fn read_text<P>(cap: FileRead<P>, path: Path) -> Result<String, IoError> ! IO
  fn write_text<P>(cap: FileWrite<P>, path: Path, contents: String) -> Result<(), IoError> ! IO
}

/////////////////////////
// http
/////////////////////////

mod http {
  async fn get<H>(cap: Network<H>, url: Url) -> Result<String, HttpError> ! IO + Async
}
```


---

## Appendix B: Error Code Catalog (Added/Changed in v0.2.1.1)

This catalog is closed for v0.2.1.1: every emitted `E####` MUST appear here.

| Code | Summary (template) | Primary span rule | Suggestion / Fix-it |
|---|---|---|---|
| E0001 | invalid_utf8: source file is not valid UTF-8 | whole file | re-save file as UTF-8 |
| E0008 | multiple_statements_one_line: only one item or statement per line | start token of the second item/statement | split into multiple lines |
| E0009 | chained_compare_forbidden: chained comparisons are not allowed | comparison operator token | rewrite using `&&` |
| E0010 | generic_args_require_turbofish: use `::<...>` for generics | `<` token after callee | insert `::` before `<` |
| E0102 | array_type_semicolon_forbidden: use `[T, N]` | `;` token inside array type | replace `;` with `,` |
| E0109 | match_wildcard_must_be_last: `_` must be final arm | `_` pattern | move wildcard arm to end |
| E0110 | invalid_for_iterable: for-loop iterable not supported | `in <expr>` span | use `i32` range or `&Vec`/`&[T]` |
| E0201 | unknown_field: no such field on type | `.field` token | fix field name or type |
| E0202 | opaque_type_use_forbidden: cannot construct/destructure opaque stdlib type | construction/pattern span | use stdlib constructor/function |
| E0203 | invalid_format_string: format string shape not supported | format string literal | use exactly one `{}` |
| E0204 | format_string_not_literal: fmt argument must be a string literal | fmt argument span | inline the literal |
| E0205 | nominal_type_not_alias: type is nominal, not interchangeable | the type name in assignment/call | use explicit newtype constructor or pattern-match |
| E0207 | duplicate_field: field listed more than once | duplicate field name | remove duplicates |
| E0210 | reserved_name_redefinition: `{name}` is a reserved prelude/stdlib identifier | the identifier token that introduces the forbidden binding |  |
| E0211 | inherent_impl_forbidden_in_user_code: inherent impl blocks are stdlib-only | the `impl` keyword of the inherent impl item |  |
| E0301 | missing_effect: callee requires effect not declared by caller | call expression span | add missing effect(s) to caller signature |
| E0302 | await_outside_async: `.await` used outside `async fn` | `.await` token | make function `async` |
| E0307 | effectful_closure_forbidden: closures must be pure in v0.2.1.1 | closure body span | extract effectful code into a named function |
| E0308 | unknown_or_invalid_effect: effect not in closed set or async mismatch | effect identifier token | use only `IO, Db, Async, Random, Time, Log, Spawn` |
| E0309 | duplicate_effect: effect listed more than once | duplicate effect token | remove duplicates |
| E0401 | use_after_move: moved value used again | use site span | clone not available; restructure ownership |
| E0402 | missing_capability_turbofish: mint requires `::<"...">` | call expression span | add `::<"policy">` |
| E0403 | capability_policy_not_literal: capability policy must be string literal | generic arg span | replace with literal like `::<"data/**">` |
| E0404 | borrow_violation: violates &/&mut exclusivity | borrow site span | adjust borrows/lifetimes |
| E0501 | raw_access_forbidden: cannot access/destructure Raw<T> | offending pattern/field access span | use `validate::<R>(raw)` |
| E0601 | overlapping_impl_forbidden: overlapping trait impls are not allowed | second impl span | remove or narrow one impl |
| E0702 | module_file_ambiguous: both file and dir module exist | module path span | remove one of the two files |
| E0703 | module_file_missing: module path has no file | module path span | create `*.ax` or `mod.ax` |
| E0704 | relative_use_forbidden: `use` must start with `crate::` | `use` path span | rewrite as `use crate::...` |
| E0710 | layer_violation: illegal import across layers | `use` item span | move code or invert dependency |
| E0800 | invalid_refinement_predicate: refinement predicate not in closed DSL | predicate span | use range `lo..=hi` or `len OP N` |
| E0801 | refinement_constructor_forbidden: use validate for non-literals | constructor call span | replace with `validate::<R>(x)?` |
| E0802 | max_lines_exceeded | function name span | split function |
| E0803 | max_params_exceeded | function name span | group params into record |
| E0804 | max_nesting_exceeded | deepest construct span | extract helpers |
| E0805 | max_complexity_exceeded | function name span | refactor conditionals |
| E0903 | must_handle_expression_statement: cannot ignore must-handle | expression statement span | handle via `?`, `match`, or `return` |
| E0904 | must_handle_underscore_binding_forbidden | binding pattern span | bind to a name and handle |
| E0911 | must_handle_bound_but_unused: must-handle value not consumed | variable name span | add `?`, `match`, `return`, or pass by value |
| E1201 | formatter_not_idempotent: formatting is not idempotent | whole file | run formatter until stable; report bug |

---

## Appendix C: Conformance Checklist (v0.2.1.1)

A v0.2.1.1 implementation MUST pass all items:

1. **Lexer NL:** emits `NL` tokens per §2.7 and never inside string/char literals.  
2. **No semicolons:** rejects statement semicolons and `[T; N]` array syntax (E0102).  
3. **One statement per line:** enforces E0008.  
4. **Precedence:** implements operator precedence exactly as §3.6 (including `.await`/`?` highest).  
5. **Turbofish:** enforces E0010 for explicit generic args.  
6. **for restriction:** enforces §10.2 (E0110) and iteration yields correct element types.  
7. **No implicit conversions:** rejects implicit base↔newtype/refinement conversions (E0205).  
8. **Opaque stdlib types:** forbids user construction/destructuring of opaque stdlib types (E0202).  
9. **Raw opacity:** rejects any Raw field access, constructor, or pattern match (E0501); `validate` is the only untaint path.  
10. **Must-handle:** enforces expression-statement rule (E0903), underscore binding rule (E0904), and scope-drop rule (E0911).  
11. **Effects closed set:** rejects unknown effects (E0308) and missing effects on calls (E0301).  
12. **Capabilities:** requires policy literal (E0403) and turbofish minting (E0402).  
13. **Layers:** enforces layer ordering (E0710) and absolute imports (E0704).  
14. **Budgets:** computes counts per §16.3 on formatted output and emits E0802–E0805.  
15. **Formatter:** is deterministic + idempotent (byte-identical on second run) and reports E1201 if not.  
16. **Diagnostics JSON:** emits schema `axiom.diagnostics.v1` with required fields and valid UTF‑8.  
17. **Stdlib lock:** ships exactly Appendix A surface; examples in §19 compile/fail as labeled.

---

## Appendix D: Core Grammar (EBNF, Normative)

This grammar describes the surface syntax. Semantic restrictions (effects, must-handle, capability literals, etc.) are specified in the main text.

```ebnf
file            = NL* item (NL+ item)* NL* EOF ;
stdlib_stub_file = NL* stub_item (NL+ stub_item)* NL* EOF ;

item            = attr_line* visibility? (use_item | mod_item | type_item | trait_item | impl_item | fn_item) ;
stub_item       = use_item | stub_mod_item | type_item | trait_item | impl_item | fn_decl ;
stub_mod_item   = "mod" ident stub_item_block ;
stub_item_block = "{" NL* stub_item (NL+ stub_item)* NL* "}" ;


attr_line       = "#[" attr_name attr_args? "]" NL ;
attr_name       = ident ( "::" ident )* ;
attr_args       = "(" attr_arg ( "," attr_arg )* ( "," )? ")" ;
attr_arg        = ident
                | int_lit
                | string_lit
                | ident "=" ( ident | int_lit | string_lit ) ;

visibility      = "pub" ( "(" ("crate" | "super") ")" )? ;

use_item        = "use" "crate" "::" path_tail ( "as" ident )? ;
path_tail       = ident ( "::" ident )* ;

mod_item        = "mod" ident item_block ;
item_block      = "{" NL* item (NL+ item)* NL* "}" ;

fn_sig         = async_kw? "fn" ident generic_params? "(" params? ")" return_type? effects? ;
fn_item         = fn_sig block ;
fn_decl         = fn_sig ;
async_kw        = "async" ;

generic_params  = "<" ws_nl* generic_param ( ws_nl* "," ws_nl* generic_param )* ws_nl* ( "," ws_nl* )? ">" ;
generic_param   = ident ( ws_nl* ":" ws_nl* bound_list )? ;
bound_list      = path ( ws_nl* "+" ws_nl* path )* ;

params          = param ( "," param )* ( "," )? ;
param           = ident ":" type ;

return_type     = "->" type ;

effects         = "!" effect ( "+" effect )* ;
effect          = "IO" | "Db" | "Async" | "Random" | "Time" | "Log" | "Spawn" ;

trait_item      = "trait" ident generic_params? "{" NL* "}" ;

impl_item       = "impl" ws_nl* generic_params? ws_nl* ( trait_impl | inherent_impl ) ;
trait_impl      = trait_ref ws_nl* "for" ws_nl* type ws_nl* item_block_empty ;
trait_ref       = path ;
inherent_impl   = type ws_nl* inherent_block ;
inherent_block  = "{" NL* fn_decl (NL+ fn_decl)* NL* "}" ;


type_item       = "type" ident generic_params? "=" type_def ;

type_def        = record_def
                | sum_def
                | newtype_def ;

record_def       = "{" ws_nl* ( field_decl ( ws_nl* "," ws_nl* field_decl )* ws_nl* ( "," ws_nl* )? )? "}" ;
field_decl      = ident ":" type ;

sum_def         = "|" ws_nl* sum_variant ( ws_nl* "|" ws_nl* sum_variant )* ;
sum_variant     = path ( ws_nl* record_payload )? ;
record_payload   = "{" ws_nl* ( field_init ( ws_nl* "," ws_nl* field_init )* ws_nl* ( "," ws_nl* )? )? "}" ;

newtype_def     = type ( ws_nl* where_clause )? ;
where_clause    = "where" ws_nl* refinement_pred ;
refinement_pred = range_pred | len_pred ;
range_pred      = number_lit ws_nl* "..=" ws_nl* number_lit ;
len_pred        = "len" ws_nl* rel_op ws_nl* int_lit ;
rel_op          = ">" | ">=" | "==" | "<=" | "<" ;

type            = ref_type | tuple_type | array_type | path_type ;

ref_type        = "&" ws_nl* "mut"? ws_nl* type ;
tuple_type      = "(" ws_nl* type ( ws_nl* "," ws_nl* type )+ ws_nl* ( "," ws_nl* )? ")"
                | "(" ws_nl* type ws_nl* "," ws_nl* ")" ;
array_type      = "[" ws_nl* type ws_nl* "," ws_nl* int_lit ws_nl* "]" ;
path_type       = path generic_args? ;

path            = ident ( "::" ident )* ;
generic_args    = "<" ws_nl* generic_arg ( ws_nl* "," ws_nl* generic_arg )* ws_nl* ( "," ws_nl* )? ">" ;
generic_arg     = type | string_lit | int_lit ;

block           = "{" NL* stmt_list? NL* "}" ;
stmt_list       = stmt_or_expr ( NL+ stmt_or_expr )* ;

stmt_or_expr    = stmt | expr ;

stmt            = let_stmt | assign_stmt | return_stmt | break_stmt | continue_stmt | expr_stmt ;
let_stmt        = "let" ws_nl* "mut"? ws_nl* pattern ( ws_nl* ":" ws_nl* type )? ws_nl* "=" ws_nl* expr ;
assign_stmt     = lvalue ws_nl* "=" ws_nl* expr ;
lvalue          = ident ( "." ident )* ;

return_stmt     = "return" ( ws_nl* expr )? ;
break_stmt      = "break" ( ws_nl* label_ref )? ( ws_nl* expr )? ;
continue_stmt   = "continue" ( ws_nl* label_ref )? ;

expr_stmt       = expr ;

label_ref       = "'" ident ;

expr            = if_expr | match_expr | loop_expr | for_expr | logical_or ;

if_expr         = "if" ws_nl* expr ws_nl* block ws_nl* "else" ws_nl* ( if_expr | block ) ;
match_expr      = "match" ws_nl* expr ws_nl* "{" NL* match_arm ( NL+ match_arm )* NL* "}" ;
match_arm       = pattern ( ws_nl* "if" ws_nl* expr )? ws_nl* "=>" ws_nl* expr ;

loop_expr       = "loop" ws_nl* block ;
for_expr        = "for" ws_nl* pattern ws_nl* "in" ws_nl* expr ws_nl* block ;

logical_or      = logical_and ( ws_nl* "||" ws_nl* logical_and )* ;
logical_and     = comparison ( ws_nl* "&&" ws_nl* comparison )* ;
comparison      = range ( ws_nl* comp_op ws_nl* range )? ;
comp_op         = "==" | "!=" | "<" | "<=" | ">" | ">=" ;

range           = additive ( ws_nl* ( ".." | "..=" ) ws_nl* additive )? ;
additive        = multiplicative ( ws_nl* ( "+" | "-" ) ws_nl* multiplicative )* ;
multiplicative  = prefix ( ws_nl* ( "*" | "/" | "%" ) ws_nl* prefix )* ;

prefix          = ( "-" | "!" | "&" | "&mut" )* ws_nl* postfix ;

postfix         = primary ( ws_nl* postfix_op )* ;
postfix_op      = turbofish_op | call_op | field_op | await_op | question_op ;
turbofish_op    = "::" generic_args ;
call_op         = "(" ws_nl* args? ws_nl* ")" ;
args            = expr ( ws_nl* "," ws_nl* expr )* ws_nl* ( "," ws_nl* )? ;
field_op        = "." ident ;
await_op        = "." "await" ;
question_op     = "?" ;

primary         = literal
                | closure
                | path
                | tuple_or_paren_expr
                | array_lit
                | record_lit
                | block ;

tuple_or_paren_expr = "(" ws_nl* expr ( ws_nl* "," ws_nl* expr )* ws_nl* ( "," ws_nl* )? ")" ;
array_lit        = "[" ws_nl* expr_list? ws_nl* "]" ;
expr_list        = expr ( ws_nl* "," ws_nl* expr )* ws_nl* ( "," ws_nl* )? ;

record_lit       = path ws_nl* "{" ws_nl* field_inits? ws_nl* "}" ;
field_inits      = field_init ( ws_nl* "," ws_nl* field_init )* ws_nl* ( "," ws_nl* )? ;
field_init       = ident ws_nl* ":" ws_nl* expr ;

closure          = "|" ws_nl* closure_params? ws_nl* "|" ws_nl* expr ;
closure_params   = closure_param ( ws_nl* "," ws_nl* closure_param )* ws_nl* ( "," ws_nl* )? ;
closure_param    = ident ( ws_nl* ":" ws_nl* type )? ;

pattern          = wildcard_pat
                | literal
                | ident
                | tuple_pat
                | record_pat
                | newtype_pat ;

wildcard_pat     = "_" ;
tuple_pat        = "(" ws_nl* pattern ( ws_nl* "," ws_nl* pattern )* ws_nl* ( "," ws_nl* )? ")" ;
record_pat       = path ws_nl* "{" ws_nl* pat_fields? ws_nl* "}" ;
pat_fields       = pat_field ( ws_nl* "," ws_nl* pat_field )* ws_nl* ( "," ws_nl* )? ;
pat_field        = ident ws_nl* ":" ws_nl* pattern ;

newtype_pat      = path ws_nl* "(" ws_nl* pattern ws_nl* ")" ;

literal          = int_lit | float_lit | string_lit | char_lit | "true" | "false" ;
number_lit       = int_lit | float_lit ;

ws_nl            = ( NL )? ;
```