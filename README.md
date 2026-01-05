<p align="center">
  <h1 align="center">Axiom</h1>
  <p align="center">
    <strong>AI-First Programming Language Specification</strong>
  </p>
  <p align="center">
    Deterministic. Capability-Safe. Zero Ambiguity.
  </p>
</p>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#design-principles">Design Principles</a> •
  <a href="#key-features">Key Features</a> •
  <a href="#syntax-preview">Syntax</a> •
  <a href="#documentation">Documentation</a> •
  <a href="#contributing">Contributing</a>
</p>

---

## Overview

**Axiom** is a programming language specification designed from the ground up for the AI era. Its core philosophy: *sloppy code does not compile.*

Unlike traditional languages that accumulate ambiguity for backwards compatibility, Axiom enforces deterministic semantics at every level—from evaluation order to error handling. The result is a language where both humans and AI systems can reason precisely about program behavior.

**Current Version:** v0.2.1.1 (Final, Normative)

## Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Determinism** | No unspecified evaluation order, implicit conversions, or default behaviors |
| **Explicit Effects** | All side effects declared via effect system (`!IO`, `!Db`, `!Async`) |
| **Capability Safety** | File and network access requires explicit capability tokens |
| **Closed Sets** | Keywords, effects, and stdlib surface are fixed and complete |
| **Anti-Ambiguity** | Semicolon-free syntax with mandatory newline separation |

## Key Features

### Effect System
```axiom
fn fetch_data(url: Url) -> Result<String, HttpError> !IO + Async {
    let net: Network<"*"> = caps.network::<"*">()
    http::get(net, url).await
}
```

### Capability-Based Security
```axiom
fn read_config() -> Result<String, IoError> !IO {
    let caps: AllCaps = caps
    let read: FileRead<"config/**"> = caps.file_read::<"config/**">()
    fs::read_to_string(read, Path::new("config/app.toml"))
}
```

### Refinement Types
```axiom
type Email = String where len > 3
type Port = u16 where 1..=65535
type NonEmpty<T> = Vec<T> where len > 0
```

### Algebraic Data Types
```axiom
type Result<T, E> =
    | Result::Ok { value: T }
    | Result::Err { error: E }

type Option<T> =
    | Option::Some { value: T }
    | Option::None
```

### Exhaustive Pattern Matching
```axiom
fn handle(result: Result<i32, IoError>) -> () {
    match result {
        Result::Ok { value: v } => io::println(fmt::to_string(v))
        Result::Err { error: e } => io::println("Error occurred")
    }
}
```

## Syntax Preview

```axiom
// Main entry point with IO effect
fn main() -> () !IO {
    let caps: AllCaps = caps

    // Type-safe string validation
    let raw: Raw<String> = io::read_line()?
    let email: Result<Email, ValidationError<Email>> = validate::<Email>(raw)

    match email {
        Result::Ok { value: e } => io::println(fmt::format1("Valid: {}", e))
        Result::Err { error: _ } => io::println("Invalid email format")
    }
}
```

## Documentation

| Document | Description |
|----------|-------------|
| [`axiom_language_spec_v0.2.1.1_SSOT.md`](axiom_language_spec_v0.2.1.1_SSOT.md) | Complete language specification (Single Source of Truth) |
| [`axiom_ebnf_v0.2.1.1.ebnf`](axiom_ebnf_v0.2.1.1.ebnf) | Formal grammar in Extended Backus-Naur Form |
| [`stdlib_stubs_v0.2.1.1.ax`](stdlib_stubs_v0.2.1.1.ax) | Standard library type signatures |
| [`axiom_error_codes_v0.2.1.1.json`](axiom_error_codes_v0.2.1.1.json) | Compiler diagnostic codes and messages |
| [`axiom_conformance_tests_v0.2.1.1.md`](axiom_conformance_tests_v0.2.1.1.md) | Conformance test suite for implementations |

## Repository Structure

```
axiom/
├── axiom_language_spec_v0.2.1.1_SSOT.md   # Language specification
├── axiom_ebnf_v0.2.1.1.ebnf               # Formal grammar
├── stdlib_stubs_v0.2.1.1.ax               # Standard library signatures
├── axiom_error_codes_v0.2.1.1.json        # Error code catalog
├── axiom_conformance_tests_v0.2.1.1.md    # Conformance tests
├── axiom_agents_bootstrap_handbook_v0.1.md # AI agent implementation guide
└── axiom_ai_mastery_masterplan_v0.1.md    # AI integration roadmap
```

## Conformance

A conforming Axiom compiler **MUST**:

- Reject programs with unspecified evaluation order or implicit conversions
- Implement all closed sets exactly (keywords, effects, builtin lints)
- Produce diagnostics using the stable schema defined in the specification
- Pass all conformance tests without additional diagnostics

## Contributing

Contributions to the Axiom specification are welcome. Please ensure any proposals:

1. Maintain deterministic semantics
2. Do not introduce implicit behavior
3. Include conformance test cases
4. Update the EBNF grammar if syntax changes

## License

This specification is provided for implementation and research purposes.

---

<p align="center">
  <sub>Axiom: Where precision meets possibility.</sub>
</p>
