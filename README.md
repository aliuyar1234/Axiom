# Axiom

Statically typed language spec with effect tracking and capability-based I/O. No implicit behavior, no semicolons, no guessing.

**Version:** v0.2.1.1 (normative)

## Why

Most languages accumulate ambiguity over time. Axiom doesn't. The compiler rejects anything with unspecified evaluation order, implicit conversions, or missing error handling. If it compiles, you know exactly what it does.

Built for codebases where AI agents and humans work on the same code.

## Quick Look

```axiom
type Email = String where len > 3

fn main() -> () !IO {
    let raw: Raw<String> = io::read_line()?
    let email: Result<Email, ValidationError<Email>> = validate::<Email>(raw)

    match email {
        Result::Ok { value: e } => io::println(fmt::format1("Valid: {}", e))
        Result::Err { error: _ } => io::println("Invalid email")
    }
}
```

## Core Concepts

**Effects** — Side effects are part of the type signature. `!IO`, `!Db`, `!Async`, etc. No hidden I/O.

```axiom
fn fetch(url: Url) -> Result<String, HttpError> !IO + Async {
    let net: Network<"*"> = caps.network::<"*">()
    http::get(net, url).await
}
```

**Capabilities** — File/network access requires tokens scoped to paths or hosts. You can't accidentally read `/etc/passwd` when you only have `FileRead<"config/**">`.

```axiom
fn read_config() -> Result<String, IoError> !IO {
    let read: FileRead<"config/**"> = caps.file_read::<"config/**">()
    fs::read_to_string(read, Path::new("config/app.toml"))
}
```

**Refinement Types** — Constraints checked at validation boundaries.

```axiom
type Port = u16 where 1..=65535
type NonEmpty<T> = Vec<T> where len > 0
```

**Sum Types** — Standard algebraic data types with exhaustive matching.

```axiom
type Result<T, E> =
    | Result::Ok { value: T }
    | Result::Err { error: E }
```

## Files

| File | What |
|------|------|
| `axiom_language_spec_v0.2.1.1_SSOT.md` | Full spec. Start here. |
| `axiom_ebnf_v0.2.1.1.ebnf` | Grammar |
| `stdlib_stubs_v0.2.1.1.ax` | Stdlib signatures |
| `axiom_error_codes_v0.2.1.1.json` | Error codes |
| `axiom_conformance_tests_v0.2.1.1.md` | Test cases for implementations |

## Compiler Requirements

A conforming implementation must:

- Reject unspecified evaluation order
- Reject implicit conversions
- Implement the exact keyword/effect sets (closed, no extensions)
- Use the diagnostic schema from the spec
- Pass all conformance tests with zero extra diagnostics

## Syntax Notes

- No semicolons. Newlines separate statements.
- Turbofish required for generics in expressions: `validate::<Email>(x)`
- One statement per line. No chained comparisons.
- Block comments don't exist. Use `//`.

## Keywords (complete set)

```
let, mut, fn, type, trait, impl, pub, mod, use, return,
if, else, match, loop, for, in, break, continue, as, where, async, await
```

That's it. Everything else is an identifier.

## Contributing

PRs welcome. Keep it deterministic, keep it explicit, include test cases.

## License

Research and implementation use.
