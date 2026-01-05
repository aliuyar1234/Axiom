# AXIOM v0.2.1.1 Conformance Tests (Skeleton)

This file is a **tooling/CI skeleton**: it lists minimal compile-pass / compile-fail tests and deterministic properties
that MUST hold for an implementation claiming conformance to **AXIOM LANGUAGE SPEC v0.2.1.1 (SSOT FINAL)**.

## PASS compile tests (12)

Each PASS test MUST compile with **no diagnostics**.

### PASS-001: validate+match (from spec §7.2) — pure
```axiom
type Email = String where len > 3

fn main() -> () {
  let raw: Raw<String> = io::read_line()? 
  let email: Result<Email, ValidationError<Email>> = validate::<Email>(raw)

  match email {
    Result::Ok { value: e } => {
      match e { Email(s) => io::println(s) }
    }
    Result::Err { error: _ } => { io::println("bad") }
  }
}
```

### PASS-002: fmt::format1 — pure
```axiom
fn main() -> () {
  let out: String = fmt::format1("n={}", 1)
  io::println(out)
}
```

### PASS-003: file read with capability — !IO
```axiom
fn main() -> () ! IO {
  let caps: AllCaps = caps

  let read: FileRead<"data/**"> = caps.file_read::<"data/**">()
  let p: Path = Path::new("data/input.txt")
  let txt: Result<String, IoError> = fs::read_to_string(read, p)
  must_handle(txt)
}
```

### PASS-004: http get — !IO
```axiom
fn main() -> () ! IO {
  let caps: AllCaps = caps
  let net: Network<"*"> = caps.network::<"*">()
  let url: Result<Url, UrlError> = Url::parse("https://example.test/")
  let ok: Url = must_handle(url)
  let resp: Result<String, HttpError> = http::get(net, ok)
  must_handle(resp)
}
```

### PASS-005: Result must-handle propagation — pure
```axiom
fn f() -> Result<i32, IoError> { 
  // placeholder for compiler-internal stubbed bodies in tests
  // Implementation-specific.
  Result::Err { error: IoError { code: 1, msg: "x" } }
}

fn main() -> () {
  let x: Result<i32, IoError> = f()
  must_handle(x)
}
```

### PASS-006: Option match — pure
```axiom
fn main() -> () {
  let o: Option<i32> = Option::None
  match o {
    Option::Some { value: v } => { io::println(fmt::to_string(v)) }
    Option::None => { io::println("none") }
  }
}
```

### PASS-007: question mark on Result — effect is !IO because inner is !IO
```axiom
fn g() -> Result<String, IoError> ! IO {
  let caps: AllCaps = caps
  let read: FileRead<"data/**"> = caps.file_read::<"data/**">()
  let p: Path = Path::new("data/input.txt")
  let s: String = fs::read_to_string(read, p)?
  Result::Ok { value: s }
}
```

### PASS-008: layered file path (budget example shape) — !IO
```axiom
fn main() -> () ! IO {
  let caps: AllCaps = caps
  let read: FileRead<"data/**"> = caps.file_read::<"data/**">()
  let p: Path = Path::new("data/input.txt")
  let _ = fs::read_to_string(read, p)
}
```

### PASS-009: Url parse must-handle — pure
```axiom
fn main() -> () {
  let u: Result<Url, UrlError> = Url::parse("https://example.test/")
  let _ok: Url = must_handle(u)
}
```

### PASS-010: validate on Base literal — pure
```axiom
type Port = i32 where 0 <= self && self <= 65535

fn main() -> () {
  let p: Result<Port, ValidationError<Port>> = validate::<Port>(80)
  must_handle(p)
}
```

### PASS-011: Copy marker usage — pure
```axiom
fn main() -> () {
  let x: i32 = 1
  let y: i32 = x
  io::println(fmt::to_string(y))
}
```

### PASS-012: Reserved identifiers can be USED (not declared) — pure
```axiom
fn main() -> () {
  let x: Result<i32, IoError> = Result::Ok { value: 1 }
  must_handle(x)
}
```

## FAIL compile tests (12)

Each FAIL test MUST produce the **single primary error code** shown (implementations MAY emit additional notes, but MUST have the primary).

### FAIL-001: reserved name binding (E0210)
```axiom
fn main() -> () {
  let Raw: i32 = 0
}
```
Expected: `E0210`

### FAIL-002: reserved name function (E0210)
```axiom
fn validate() -> () { }
```
Expected: `E0210`

### FAIL-003: reserved name module (E0210)
```axiom
mod io { }
```
Expected: `E0210`

### FAIL-004: reserved name generic parameter (E0210)
```axiom
fn f<Raw>(x: Raw) -> Raw { x }
```
Expected: `E0210`

### FAIL-005: user-defined inherent impl (E0211)
```axiom
type X = { }
impl X {
  fn bad(self: &X) -> ()
}
```
Expected: `E0211`

### FAIL-006: must-handle ignored (E0401)
```axiom
fn main() -> () {
  let r: Result<i32, IoError> = Result::Ok { value: 1 }
  r
}
```
Expected: `E0401`

### FAIL-007: non-exhaustive match (E0302)
```axiom
fn main() -> () {
  let o: Option<i32> = Option::None
  match o {
    Option::Some { value: _ } => { }
  }
}
```
Expected: `E0302`

### FAIL-008: unknown field (E0201)
```axiom
type R = { a: i32 }
fn main() -> () {
  let r: R = R { a: 1 }
  let _ = r.b
}
```
Expected: `E0201`

### FAIL-009: construct opaque stdlib type (E0202)
```axiom
fn main() -> () {
  let _s: String = String { }
}
```
Expected: `E0202`

### FAIL-010: invalid effect call (E0502)
```axiom
fn main() -> () {
  io::println("hi")
}
```
Expected: `E0502`

### FAIL-011: type alias forbidden (E0102)
```axiom
type A = i32
type B = A
```
Expected: `E0102`

### FAIL-012: overlapping impls (E0205)
```axiom
trait T { }
type X = { }

impl T for X { }
impl T for X { }
```
Expected: `E0205`

## Deterministic properties (8)

1. **Formatter idempotence:** formatting a program twice yields identical bytes.
2. **Diagnostic schema stability:** diagnostic JSON conforms to the schema in spec §18 and is stable across runs.
3. **Error code determinism:** same program, same primary error code (no nondeterministic selection).
4. **Budget determinism:** budget counting yields same result independent of compilation order.
5. **Reserved names determinism:** E0210 triggers at the first forbidden binding introduction in source order.
6. **Inherent impl gating determinism:** E0211 triggers for any inherent impl in user code (even if other errors exist inside).
7. **EBNF conformance:** the implementation’s parser accepts all PASS programs and rejects all FAIL programs.
8. **Stdlib surface lock:** Appendix A / `stdlib_stubs_v0.2.1.1_FINAL.ax` is byte-identical to the shipped stub surface.

