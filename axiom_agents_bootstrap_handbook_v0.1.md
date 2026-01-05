# Axiom Agent Bootstrapping Handbook  
**Wie AI‑Coding‑Agents eine Sprache produktiv nutzen können, die sie nie gesehen haben**  
**Version:** v0.1 (für Start‑Phase ohne Compiler)  
**Ziel:** Morgen mit Axiom + AI starten – heute mit realistischen Workflows, Artefakten und klaren Grenzen.

---

## TL;DR (1 Seite)

Axiom „kennt“ kein Modell aus dem Training. Trotzdem kann ein Agent Axiom gut nutzen, wenn du ihm:

1) **Eine kleine, kanonische Kern‑Spezifikation** gibst (Axiom Kernel, < 2–3 Seiten).  
2) **Eine Pattern Library** mit 20–30 häufigen Bausteinen gibst (copy‑paste‑ready).  
3) **Ein Self‑Check‑Protokoll** gibst, das Slop‑Fehler systematisch verhindert.  
4) **Eine syntaktisch sichere Ausgabeform** nutzt, wenn kein Compiler existiert (optional, aber extrem hilfreich):  
   - Entweder *Pattern‑Instanzen (JSON)* → ein trivialer Generator erzeugt Axiom‑Code.  
   - Oder *Axiom‑Code direkt*, aber strikt entlang der Patterns + Checks.

Damit erreichst du: **Heute** brauchbarer, idiomatischer Axiom‑Code – ohne Fine‑Tuning und ohne Compiler. Sobald ein Parser/Formatter existiert, wird es deutlich schneller und zuverlässiger.

---

# Teil 1: Analyse des Problems

## 1. Warum ist das schwer?

### 1.1 Was LLMs ohne Training in einer Sprache **nicht** können

LLMs können erstaunlich viel generalisieren – aber neue Sprachen sind ein „Distribution Shift“. Typische Failure‑Modes:

1) **Syntax‑Halluzination**  
   Das Modell produziert Tokens, die „sprach‑ähnlich“ aussehen, aber nicht existieren (z.B. `while`, `;`, `+=`, `throws`, `try/catch`, `any`).

2) **Bibliotheks‑Halluzination**  
   Neue Sprache → keine gelernten Standard‑Library‑Namen. Das Modell erfindet Module/Methoden („`fs.readFile`“, „`String::format`“), die es nicht gibt.

3) **Semantik‑Lücken**  
   Selbst wenn die Syntax stimmt, sind die Regeln unbekannt: Ownership/Borrowing, `Result`‑Must‑Handle, Effects/Capabilities, Budget‑Limits, Layer‑Rules.

4) **Inkonsequente Idiome**  
   Ohne Beispiele wählt das Modell nicht *einen* Stil. Es driftet zwischen „Rust‑ish“, „Go‑ish“, „TS‑ish“. Ergebnis: ungleichmäßige Codebase.

5) **Fehlende „Negative Beispiele“**  
   Training lehrt auch: „Das ist falsch“ (Compiler Errors, PR reviews, conventions). Bei Axiom fehlen diese „Anti‑Patterns“ vollständig.

> Kurz: Ein LLM kann eine neue Syntax aus einer Spezifikation *nachbauen*, aber es fehlt die statistische Unterstützung für **Konsistenz, Library‑Genauigkeit und idiomatische Patterns**.

---

### 1.2 Grenzen von „Spec im Context Window“

„Spec in Prompt“ hilft, aber ist kein echtes Lernen.

**Probleme:**

- **Context ist begrenzt**: Spec + Projekt + Dialog + Code → Token‑Budget explodiert.  
- **Lesen ≠ Anwenden**: Modelle können eine Regel „sehen“, aber beim Generieren vergessen sie sie.  
- **Attention‑Decay**: Regeln am Anfang werden später weniger aktiv.  
- **Ambiguität bleibt**: Jede nicht explizit genannte Sache wird mit „bekannten Sprachen“ aufgefüllt.  
- **Refactoring‑Drift**: Bei langen Sessions bricht Style‑Kohärenz weg.

**Wichtige Erkenntnis:**  
Ein Prompt muss nicht *alles* erklären. Er muss vor allem **den Suchraum reduzieren** und **kanonische Bausteine** liefern.

---

### 1.3 Syntax kennen vs. idiomatisch programmieren

**Syntax kennen** heißt: „Ich kann `fn`, `match`, `type` korrekt schreiben.“  
**Idiomatisch programmieren** heißt:

- Datenmodellierung: *welche* Typen/Enums sind üblich?  
- Fehlerdesign: *wie* splitte ich Error‑Enums?  
- Architektur: *wo* liegen Module, welche Layer?  
- APIs: *wie* zwinge ich Validierung/Capabilities in Signaturen?  
- Budget‑Engineering: *wie* halte ich Limits ein, ohne Code zu zerhacken?

Idiome entstehen normalerweise durch:
- viele Beispiele,
- Community‑Konventionen,
- Compiler‑Feedback,
- Review‑Kultur.

Axiom hat das am Anfang nicht. Also müssen wir Idiome **als Artefakte** liefern: Pattern Library + Self‑Check + Starter‑Skeleton.

---

## 2. Was macht Axiom besonders schwer oder leicht?

### 2.1 Was ist für AI ohne Training schwer an Axiom?

1) **Effects + Capabilities gleichzeitig**  
   - Effects: „Welche Art Side‑Effect?“  
   - Capabilities: „Welche Berechtigung?“  
   Agent muss beides korrekt „verdrahten“.

2) **Must‑Handle (`Result`/`Option`)**  
   Modelle sind gewohnt: Return ignorieren. In Axiom ist das ein Compile‑Fehler. Ohne Compiler muss der Agent das konsequent selbst prüfen.

3) **Raw/Validated (Taint‑Flow)**  
   Externe Inputs dürfen nicht in sichere Kontexte. Agent muss überall `Raw<T>` denken und `validate` nutzen.

4) **Strukturelle Budgets (max_lines, max_nesting, etc.)**  
   Modelle erzeugen gerne große Funktionen. Axiom zwingt zu kleinen Units. Ohne Tooling ist es leicht, Budgets zu brechen.

5) **Layer/No‑Cycles Architektur**  
   Agents machen gerne „schnell importieren“. Axiom verbietet falsche Abhängigkeiten. Ohne Graph‑Check kann der Agent schnell in Widersprüche laufen.

6) **Doc‑Specs als Pflicht für public APIs**  
   Viele Modelle schreiben Doku nur optional. In Axiom ist es Teil des Systems.

---

### 2.2 Was macht Axiom ohne Training **leichter** als andere neue Sprachen?

Axiom wurde AI‑freundlich entworfen – das hilft massiv:

1) **Minimale Syntax‑Oberfläche**  
   Wenige Keywords, wenige Konstrukte, keine „10 Wege“.

2) **Rust‑ähnliche Grundform**  
   Viele Modelle kennen Rust‑Syntax: `fn`, `{}`, `match`, `Result`, `Option`.

3) **Explizitheit statt Magie**  
   Agent muss nicht raten: Effects/Capabilities/Validation sind sichtbar.

4) **Kanonische Patterns sind erzwingbar**  
   Axiom kann sich auf eine kleine Pattern Library stützen. Wenn du diese Patterns lieferst, kann der Agent sehr zuverlässig arbeiten.

---

### 2.3 Unterschied: „Axiom lernen“ vs „Rust lernen“ für AI

**Rust** ist groß, mit vielen API‑Varianten, Makros, Lifetimes, Ecosystem‑Eigenheiten.  
**Axiom** ist syntaktisch kleiner, aber hat zusätzliche Meta‑Constraints (Budgets, Layers, Capabilities).

Für AI bedeutet das:

- Rust‑Hürden: „Wie kompiliere ich diese Lifetime/trait bound/macro?“  
- Axiom‑Hürden: „Wie halte ich durchgängig die Qualitätsregeln ein, ohne Compiler‑Feedback?“

**Konsequenz:** Axiom braucht *weniger* „Syntax‑Wissen“, aber *mehr* „Prozess‑Wissen“.  
Genau deshalb sind die Artefakte (Prompt, Cheat Sheet, Pattern Library, Self‑Check) entscheidend.

---

# Teil 2: Die Lösung – eine praktikable Strategie

## 0. Design‑Ziel der Strategie

Die Strategie muss:

- **Heute funktionieren:** ohne Fine‑Tuning, ohne riesiges Beispiel‑Corpus.  
- **Skalieren:** echte Projekte, nicht nur Snippets.  
- **Ohne Compiler funktionieren (initial):** zumindest ohne „vollständigen Axiom‑Compiler“.  
- **Idiomatischen Code erzeugen:** nicht nur „irgendwie Axiom‑artig“.

Die zentrale Idee:

> **Reduziere den Suchraum** (Kernel + Regeln) und **baue Code aus kanonischen Bausteinen** (Patterns), statt „freie Kreativität“.

---

## 1. Baustein A: Axiom Kernel (Prompt‑kompatible Mini‑Spec)

### Was passiert?

Du gibst dem Agenten nicht die ganze Bibel, sondern eine **Kernel‑Spec**:

- 20–30 Regeln, die 95% des Codes bestimmen,
- ein paar Schlüssel‑Syntaxformen,
- „No‑Go‑Liste“ (Slop‑Syntax),
- Standard‑Patterns für Result/Option/Validation/Effects/Caps.

### Warum funktioniert das?

- Modelle folgen gut *kurzen, konkreten Regeln*.
- Wenn du die Sprache auf wenige Konstrukte beschränkst, kann das Modell sie „on‑the‑fly“ anwenden.
- Du minimierst Halluzination: „Wenn nicht im Kernel, existiert es nicht.“

### Limitationen

- Kernel deckt nicht alle Features ab (z.B. seltene generics/traits/async corners).
- Ohne Compiler kann Semantik trotzdem falsch sein.
- Du brauchst ergänzende Patterns für Libraries.

### Mini‑Demo

Kernel‑Regel: **kein `while`**.  
Stattdessen:

```ax
loop {
    if !condition { break () }
    body()
}
```

---

## 2. Baustein B: Pattern Library als „Idiom‑Träger“

### Was passiert?

Du gibst dem Agenten 20–30 Patterns, die:

- häufige Aufgaben abdecken (IO, validation, error handling, async, concurrency, modules, layering),
- idiomatisch sind (Axiom‑Style),
- copy‑paste‑bereit sind,
- IDs besitzen (`AX-PAT-...`), damit man sie referenzieren kann.

Der Agent arbeitet nicht „frei“, sondern:  
**Problem → Pattern auswählen → Parameter einsetzen → Self‑Check → fertiger Code**

### Warum funktioniert das?

LLMs sind extrem stark in:
- „Template instantiation“,
- „pattern completion“,
- „consistent repetition“.

Sie sind weniger stark in:
- „clean‑room design ohne Beispiele“.

Pattern Libraries geben dem Modell „Trainingsdaten im Kleinen“ – im Prompt/RAG.

### Limitationen

- Patterns müssen gepflegt werden.
- Neues Domain‑Problem → neues Pattern nötig.
- Zu viele Patterns → Auswahl wird wieder ambig. Deshalb: 20–30.

### Mini‑Demo

Pattern: „External input → Raw → validate → use“

```ax
fn handler(stdin: Stdin) -> Result<(), Error> ! IO {
    let raw: Raw<String> = io::read_line(stdin)?
    let email: Email = validate(raw)?
    send(email)?
    Ok(())
}
```

---

## 3. Baustein C: Self‑Check Protocol (ohne Compiler)

### Was passiert?

Der Agent muss vor dem Output eine Checkliste abarbeiten.  
Das klingt banal – ist aber der Unterschied zwischen „Slop“ und „verlässlich“.

### Warum funktioniert das?

- Checklisten kompensieren das größte LLM‑Problem: „Regeln vergessen beim Generieren“.
- Sie erzeugen deterministische Schritte: Effects/Caps/Raw/Result/Layer/Budgets.

### Limitationen

- Manuelle Checks sind langsamer als Compiler.
- Manche Dinge (z.B. Borrow‑Lifetime subtleties) sind schwer ohne Tool.

### Mini‑Demo

Self‑Check: „Habe ich jede `Result`‑Expression handled?“  
→ Agent scannt seinen eigenen Code und markiert jede `?`, jedes `match`, jedes `Ok/Err`.

---

## 4. Optionaler Turbo: Syntax‑sichere Ausgabe ohne Compiler  
*(empfohlen, weil billig und extrem wirksam)*

Die Frage verlangt „ohne Compiler“. Ein vollständiger Compiler ist schwer.  
Aber: **Ein Parser + Formatter + Pattern‑Stitcher** ist *kein* Compiler – und kann in Tagen entstehen.

### 4.1 Option 1: Pattern‑Instanzen (JSON) → Generator erzeugt Axiom

**Was passiert?**
- Der Agent outputtet JSON: „welches Pattern, welche Parameter“.
- Ein Tool `axiom-stitch` generiert formatierten Axiom‑Code.

**Warum funktioniert das?**
- JSON‑Ausgabe kann mit Schema/Function‑Calling hart constrained werden.
- Der Generator garantiert Syntax + Style + (teilweise) Budgets.
- Der Agent muss weniger „Syntax korrekt tippen“, mehr „Architektur korrekt denken“.

**Limitationen**
- Generator muss existieren.
- Bodies, die nicht in Patterns passen, brauchen „inline blocks“.

**Mini‑Demo (Pattern‑Instanz):**
```json
{
  "file": "src/main.ax",
  "patterns": [
    {"id": "AX-MAIN-IO", "vars": {"body": "call_app"}},
    {"id": "AX-READ-CONFIG", "vars": {"cap_pattern": "/app/*", "path": "/app/config.toml"}}
  ]
}
```

### 4.2 Option 2: CFG/EBNF‑Constrained Decoding (wenn Agent es unterstützt)

Einige Agenten/Frameworks können nach Grammatik decoden.  
Dann kann man Axiom‑Syntax „erzwingen“.

**Heute realistisch?**  
- In manchen Tools ja, in anderen nicht.  
- Pattern‑JSON ist universeller.

---

## 5. Konkreter End‑to‑End Workflow (heute, ohne Compiler)

### Schritt 1: Projekt‑Skeleton + Module‑Plan

**Was passiert?**
- Der Agent beginnt immer mit einem Skeleton: `domain/`, `app/`, `infra/`, `main`.
- Definiert Types + Errors zuerst.
- Definiert Capabilities & Effects explizit in Signaturen.

**Warum funktioniert das?**
- Erzwingt Architektur bevor „random code“ entsteht.
- Error‑Types werden früh stabil.

**Limitationen**
- Ohne Compiler werden Imports/Layers nicht automatisch validiert.

**Demo (Skeleton):**
```ax
#[layer(domain, depth = 0)]
mod domain {
    pub type UserId = u64
    pub type Email = String where matches(EMAIL_REGEX)

    pub type DomainError =
        | InvalidEmail
}

#[layer(application, depth = 1)]
mod app {
    use crate::domain::{Email, DomainError}

    pub fn register(raw: Raw<String>) -> Result<Email, DomainError> {
        let email: Email = validate(raw).map_err(|_| DomainError::InvalidEmail)?
        Ok(email)
    }
}

fn main(caps: AllCaps) -> () ! IO {
    io::println("ok")
}
```

---

### Schritt 2: Implementiere nur aus Patterns

**Was passiert?**
- Jede Funktion wählt ein Pattern‑ID.
- Kleine Anpassungen, keine freie Syntax.

**Warum funktioniert das?**
- Reduziert Drift.
- Erhöht Konsistenz.

**Limitationen**
- Manchmal braucht man „neues Pattern“. Dann: Pattern hinzufügen statt improvisieren.

---

### Schritt 3: Self‑Check durchführen (Pflicht)

**Was passiert?**
- Agent geht Checkliste durch.
- Fixiert systematische Fehler: `Result` handled? `Raw` validated? Effects/Caps? budgets? naming?

**Warum funktioniert das?**
- Ersetzt fehlenden Compiler in Phase 1.

---

### Schritt 4: (Optional) Axiom‑Scan (Parser‑only Tool)

**Was passiert?**
- Ein kleiner Parser/formatter prüft:
  - Syntax,
  - naming conventions,
  - unused,
  - budget approximation,
  - forbidden constructs.

**Warum funktioniert das?**
- 80% des Slops sind syntaktisch/strukturell.
- Parser‑only Tools sind schnell zu bauen.

---

## 6. Grenzen der Strategie (ehrlich)

Ohne Compiler bleibt schwer:

- **Borrow‑Checker**: Ownership‑Bugs sind ohne Tool schwer zu entdecken.
- **Effect‑Inference**: Ob `map` wirklich `! IO` braucht, hängt von callee ab.
- **Library Accuracy**: Ohne stdlib‑stubs kann der Agent API‑Namen erfinden.

**Mit dieser Strategie bekommst du trotzdem:**
- konsistente Syntax,
- idiomatische Struktur,
- sichere Datenflüsse (Raw/validate),
- explizite error handling,
- architektonische Ordnung (wenn Skeleton eingehalten wird).

---

# Teil 3: Artefakte (Copy‑Paste Ready)

## 3.1 System Prompt für AI‑Agents (≤ 4000 Tokens)

> **Copy‑paste** diesen Prompt als System Prompt / Agent Instructions.  
> Er ist bewusst kompakt und enthält nur Kernel + Regeln + Workflow.

```text
You are an Axiom coding agent.
Goal: produce idiomatic, spec-compliant Axiom code that prevents AI-slop.
You MUST follow the Axiom Kernel rules below. If something is not defined here or in provided stubs/patterns, do not invent it; create a minimal local type/function stub with TODO and clear signature.

=== Axiom Kernel (non-negotiable) ===
LANG CORE
1) Blocks use { } . Indentation has no meaning.
2) No semicolons. One statement per line.
3) Keywords are limited (fn, let, mut, type, trait, impl, if, else, match, for, in, loop, break, continue, return, mod, use, pub, async, await, where, true, false).
4) Forbidden constructs: while, do-while, switch, try/catch/throw/throws, null/undefined/nil, ++/--, +=/-=/*=/etc, ternary ?:, wildcard imports (*).
5) Naming: Types/Traits/Modules UpperCamelCase. Functions/vars/fields lower_snake_case. Unused is ERROR; prefix with _ only if intentionally unused.

TYPES & SAFETY
6) No null. Use Option<T> = Some(T) | None.
7) No exceptions. Errors are values: Result<T,E> = Ok(T) | Err(E).
8) Result/Option are MUST-HANDLE: you may not ignore them. Use one of: let-binding, match, ?, map/and_then/unwrap_or.
9) External/untrusted data is Raw<T>. Raw<T> must be validated before use in validated contexts.
10) Refinement/validated types are constructed only via validate(...) unless statically known safe.

CONTROL FLOW
11) if is an expression.
12) match must be exhaustive. For public API error enums, do NOT use '_' to hide variants.
13) Loops: use for or loop only. No while. Use break value for loop-as-expression.

FUNCTIONS
14) Function syntax is only:
   fn name(p1: T1, p2: T2) -> R [! Effects] { ... }
   All params AND return types are required.
15) return keyword is ONLY for early exit; do not end a function with 'return x' as the last line.
16) Pure by default. If a function performs side effects, declare them using ! Effect + Effect.
   Common effects: IO, Async, Db, Time, Random, Log, Spawn, Panic, Unsafe.
17) Capabilities: to read/write files/network/etc, functions must accept capability tokens explicitly.
   Capabilities come from main(caps: AllCaps) and can only be restricted, never widened.

ARCHITECTURE & BUDGETS
18) Prefer layered structure: domain (depth 0), application (depth 1), infrastructure/presentation (depth 2).
   Do not create import cycles. Do not import from higher/equal depth unless explicitly allowed by project policy.
19) Budgets (default targets):
   - max_lines per function: 30
   - max_params: 5
   - max_nesting: 3
   - max_complexity: 10
   If you exceed budgets, refactor into helper functions/modules.
20) Public functions (pub fn) MUST have doc-spec tags:
   @description, @param, @returns, @error (each variant), @effects, @pre/@post where relevant.

=== Output format ===
- If the user asks for code: output ONLY Axiom code in fenced blocks ```ax ... ```
- Keep code minimal and consistent with the Pattern Library.
- When unsure about stdlib API names, define a local stub with TODO and a precise signature instead of inventing.

=== Workflow you MUST follow ===
A) PLAN briefly (types, errors, effects, capabilities, modules).
B) SELECT patterns (use IDs if provided) and implement by instantiating patterns.
C) SELF-CHECK before output:
   - No forbidden constructs
   - All Result/Option handled
   - All Raw validated before use
   - Effects declared correctly
   - Capabilities passed explicitly
   - Budgets respected
   - match exhaustive; no '_' for public error enums
   - naming consistent; no unused vars/imports
D) Provide final code.
```

---

## 3.2 Cheat Sheet (1 Seite, scan‑bar)

```markdown
# Axiom Cheat Sheet (Agent Quick Scan)

## 1) Core Syntax
- Function: `fn name(p: T) -> R { ... }`
- Effects: `fn f(...) -> R ! IO + Async { ... }`
- Types (record): `type User = { id: u64, name: String }`
- Types (sum): `type E = A | B { x: i32 } | C(i32)`
- Option: `Option<T> = Some(T) | None`
- Result: `Result<T,E> = Ok(T) | Err(E)`
- if is expression: `let x = if cond { 1 } else { 2 }`
- match is exhaustive:
  `match v { Pattern => expr, ... }`
- Loops: `for x in xs { ... }` OR `loop { ... break value }`
- No `while`. No `;`. One statement per line.

## 2) MUST-HANDLE
You may not ignore:
- `Result<...>` and `Option<...>`
Handle by:
- `let x = ...`
- `match ... { ... }`
- `...?`
- combinators: `.map(...)`, `.and_then(...)`, `.unwrap_or(...)`

## 3) Validation / Taint
- External input is `Raw<T>`
- Convert: `let safe: T = validate(raw)?`
- Never pass `Raw<T>` into validated parameters.

## 4) Effects & Capabilities
- Side effects require `! Effect` on function.
- IO requires capability tokens:
  - `FileRead<"...">`, `FileWrite<"...">`, `Network<"...">`, `Env`, `Stdin`, ...
- Capabilities come from: `main(caps: AllCaps)`

## 5) Forbidden Slop
- `null`, `undefined`, `try/catch`, `throw`, `throws`
- `while`, `++`, `+=`, ternary `?:`
- wildcard import `use x::*`
- ignoring errors (`_ = ...` as pseudo-ignore)

## 6) Architecture
- Prefer layers:
  - domain depth 0
  - application depth 1
  - infrastructure/presentation depth 2
- No import cycles.

## 7) Budgets (targets)
- ≤30 lines / function
- ≤5 params
- ≤3 nesting depth
- complexity ≤10
Refactor early.

## 8) Public APIs
`pub fn` requires doc tags:
- `/// @description: ...`
- `/// @param ...`
- `/// @returns: ...`
- `/// @error ...`
- `/// @effects: ...`
```

---

## 3.3 Pattern Library (24 Patterns)

**Format:**  
- **ID**  
- **Problem**  
- **Axiom‑Lösung (Code)**  
- **Warum / Hinweise**

> Hinweis: Diese Patterns sind bewusst „kanonisch“. Wenn du etwas anders machen willst: erst neues Pattern designen, dann verwenden.

---

### Kategorie: Core / Types

#### AX-PAT-01: Record Type definieren
**Problem:** Datenmodell als Struct/Record.  
```ax
type User = {
    id: u64,
    name: String,
    email: Email,
}
```
**Warum:** Klar, explizit, keine Magic‑Constructor.

---

#### AX-PAT-02: Error Enum definieren (Sum Type)
**Problem:** Explizite Fehlerfälle.  
```ax
type FetchError =
    | NotFound
    | Timeout
    | InvalidInput { field: String, reason: String }
```
**Warum:** Exhaustive handling erzwingbar.

---

#### AX-PAT-03: Option auswerten (match)
**Problem:** Optionalen Wert nutzen.  
```ax
fn display(opt: Option<String>) -> String {
    match opt {
        Option::Some(s) => s,
        Option::None => "n/a".into(),
    }
}
```
**Warum:** Ein Idiom, keine 5 Varianten.

---

#### AX-PAT-04: Result auswerten (match)
**Problem:** Erfolg/Fehler explizit behandeln.  
```ax
fn handle(r: Result<i32, FetchError>) -> String {
    match r {
        Ok(v) => "ok {v}".into(),
        Err(FetchError::NotFound) => "not found".into(),
        Err(FetchError::Timeout) => "timeout".into(),
        Err(FetchError::InvalidInput { field, reason }) => "invalid {field}: {reason}".into(),
    }
}
```
**Warum:** Kein `_` für Error‑Enums in public APIs.

---

### Kategorie: Fehlerbehandlung / Propagation

#### AX-PAT-05: Fehler propagieren mit `?`
**Problem:** Funktion ruft fallible helper auf.  
```ax
fn load(path: Path, cap: FileRead<"/app/*">) -> Result<String, IoError> ! IO {
    let content: String = fs::read(cap, path)?
    Ok(content)
}
```
**Warum:** Standard‑Kette: `let x = f()?`.

---

#### AX-PAT-06: Error‑Mapping (domain error)
**Problem:** Low‑level Fehler in Domain‑Fehler umwandeln.  
```ax
type ConfigError = Missing | Invalid

fn parse_config(s: String) -> Result<Config, ConfigError> {
    parse(s).map_err(|_| ConfigError::Invalid)
}
```
**Warum:** API bleibt stabil, keine leaky errors.

---

#### AX-PAT-07: Best‑effort Operation (nicht ignorieren)
**Problem:** Fehler soll geloggt, nicht propagiert werden.  
```ax
fn cleanup(cap: FileWrite<"/tmp/*">, path: Path) -> () ! IO + Log {
    match fs::remove(cap, path) {
        Ok(()) => (),
        Err(e) => { log::warn("cleanup failed: {e}")? () },
    }
}
```
**Warum:** „Ignorieren“ ist explizit.

---

### Kategorie: Validation / Security

#### AX-PAT-08: Raw Input → validate → use
**Problem:** Input aus Außenwelt sicher machen.  
```ax
fn parse_email(raw: Raw<String>) -> Result<Email, DomainError> {
    let email: Email = validate(raw).map_err(|_| DomainError::InvalidEmail)?
    Ok(email)
}
```
**Warum:** Taint‑Flow ist sichtbar.

---

#### AX-PAT-09: Refinement Type + validate
**Problem:** Wertebereich erzwingen (z.B. Port).  
```ax
type Port = u16 where 1..=65535

fn mk_port(raw: Raw<String>) -> Result<Port, ParseError> {
    let n: u16 = parse_u16(raw)?
    let p: Port = validate(n)?
    Ok(p)
}
```
**Warum:** Domain‑Constraints als Typ.

---

#### AX-PAT-10: SQL Injection verhindern (typed query)
**Problem:** Keine SQL Strings konkatenieren.  
```ax
type SqlQuery = { /* opaque */ }
mod sql {
    fn select_user_by_id(id: UserId) -> SqlQuery { /* builder */ }
}

fn get_user(db: DbConn<"main">, id: UserId) -> Result<User, DbError> ! Db {
    let q: SqlQuery = sql::select_user_by_id(id)
    db::query_one(db, q)
}
```
**Warum:** „Unsichere Strings“ sind nicht API‑Pfad.

---

### Kategorie: Effects & Capabilities

#### AX-PAT-11: main mit Capabilities
**Problem:** Einstiegspunkt mit Permissions.  
```ax
fn main(caps: AllCaps) -> () ! IO {
    let read: FileRead<"/app/*"> = caps.file_read("/app/*")
    let cfg = fs::read(read, Path::new("/app/config.toml"))?
    io::println("cfg loaded")
}
```
**Warum:** Capabilities „minten“ nur in main.

---

#### AX-PAT-12: File read helper (cap in signature)
**Problem:** Funktion liest Datei.  
```ax
fn read_text(cap: FileRead<"/app/data/*">, path: Path) -> Result<String, IoError> ! IO {
    fs::read(cap, path)
}
```
**Warum:** Capability ist auditable.

---

#### AX-PAT-13: Network call helper
**Problem:** HTTP call braucht Network cap + Async + IO.  
```ax
async fn fetch(cap: Network<"api.example.com">, url: Url) -> Result<Response, HttpError> ! IO + Async {
    let r = http::get(cap, url).await?
    Ok(r)
}
```
**Warum:** Effects sind sichtbar und korrekt.

---

#### AX-PAT-14: Pure wrapper bleibt pure
**Problem:** Reine Funktion soll keine IO.  
```ax
fn normalize(s: String) -> String {
    s.trim().to_lower()
}
```
**Warum:** Compiler/Review erkennt sofort: keine hidden IO.

---

### Kategorie: Kontrollfluss / Loop‑Idiome

#### AX-PAT-15: loop-as-expression
**Problem:** Wiederholen bis Erfolg, dann Wert liefern.  
```ax
fn read_non_empty(stdin: Stdin) -> Result<String, IoError> ! IO {
    let s: String = loop {
        let raw: Raw<String> = io::read_line(stdin)?
        let line: String = validate(raw)? // assume: non-empty validated type in real code
        if line.len() > 0 {
            break line
        }
    }
    Ok(s)
}
```
**Warum:** Kein while, kein state spaghetti.

---

#### AX-PAT-16: for + continue/break
**Problem:** Filtern/Stoppen in Iteration.  
```ax
for item in items {
    if item.skip() { continue }
    if item.done() { break () }
    process(item)?
}
```
**Warum:** Kanonisch, flach.

---

### Kategorie: Modules & Architecture

#### AX-PAT-17: Layered modules skeleton
**Problem:** Standard Architektur.  
```ax
#[layer(domain, depth = 0)]
mod domain { }

#[layer(application, depth = 1)]
mod app { use crate::domain::* }

#[layer(infrastructure, depth = 2)]
mod infra { use crate::domain::* }

fn main(caps: AllCaps) -> () ! IO { ... }
```
**Warum:** verhindert Spaghetti.

---

#### AX-PAT-18: Module contract template
**Problem:** Modulgrenzen maschinenlesbar.  
```ax
#[contract{
    name: "User Service"
    version: "0.1.0"
    requires: { types: [User, UserId], effects: [Db] }
    provides: { functions: [create_user, get_user] }
    limits: { max_functions: 20, max_dependencies: 10 }
}]
#[layer(application, depth = 1)]
mod user_service { ... }
```
**Warum:** RAG/AI kann Contracts lesen.

---

### Kategorie: Concurrency / Async

#### AX-PAT-19: structured concurrency scope
**Problem:** Parallel work ohne zombie tasks.  
```ax
fn parallel() -> () ! Async {
    task::scope(|s| -> () {
        s.spawn(|| work1())
        s.spawn(|| work2())
        ()
    })
}
```
**Warum:** Scope join garantiert.

---

#### AX-PAT-20: channel producer/consumer
**Problem:** Message passing.  
```ax
let (tx, rx) = chan::new<Message>()

task::scope(|s| -> () {
    s.spawn(|| -> () { tx.send(Message::Hello)? () })
    s.spawn(|| -> () { for m in rx { handle(m)? } () })
})
```
**Warum:** Keine shared mutation.

---

#### AX-PAT-21: mutex update
**Problem:** Shared state controlled.  
```ax
let state = sync::Mutex::new(State::new())

task::scope(|s| -> () {
    s.spawn(|| -> () {
        let mut g = state.lock()?
        g.update()
        ()
    })
})
```
**Warum:** Mutability ist sichtbar.

---

### Kategorie: Budgets / Refactoring

#### AX-PAT-22: Split big function into helpers
**Problem:** Budget überschritten.  
```ax
fn handle(req: Request) -> Result<Response, ApiError> ! IO {
    let parsed = parse_request(req)?
    let validated = validate_request(parsed)?
    let result = execute(validated)?
    Ok(format_response(result))
}
```
**Warum:** Pipeline‑Style reduziert nesting/complexity.

---

#### AX-PAT-23: Avoid nesting depth > 3 (flatten)
**Problem:** Zu viele ifs.  
```ax
fn f(x: i32) -> Result<i32, Err> {
    if x < 0 { return Err(Err::Negative) }
    if x == 0 { return Err(Err::Zero) }
    Ok(x)
}
```
**Warum:** Early return ist erlaubt (aber nur early).

---

#### AX-PAT-24: Public API doc spec template
**Problem:** `pub fn` braucht machine‑readable spec.  
```ax
/// @description: Creates a user
/// @param raw_email: Untrusted email input
/// @returns: UserId on success
/// @error InvalidEmail: when validation fails
/// @effects: Db
pub fn create_user(raw_email: Raw<String>, db: DbConn<"main">) -> Result<UserId, CreateUserError> ! Db {
    ...
}
```
**Warum:** Stabilität + AI‑verständliche Constraints.

---

## 3.4 Self‑Check Protocol (vor/nach Codegen)

> Der Agent soll diese Schritte **explizit** durchführen.  
> Wenn du tooling hast, kannst du sie automatisieren, aber als Start sind sie manuell.

### Pre‑Generation (Plan)

1) **Was sind die Domain‑Typen?** (Records, Refinements, Validated Types)  
2) **Welche Error‑Enums brauche ich?** (keine String errors)  
3) **Welche Effects sind involviert?** (`IO`, `Async`, `Db`, …)  
4) **Welche Capabilities brauche ich?** (FileRead/Write, Network, …)  
5) **Welche Module/Layers?** (domain/app/infra)  
6) **Welche Budgets sind kritisch?** (max_nesting, max_params, max_lines)

### Post‑Generation (Code Check)

**Syntax/Verbote**
- [ ] Keine Semikolons
- [ ] Kein `while`, `try/catch`, `throw`, `null`, `+=`, `++`, `?:`
- [ ] Keine wildcard imports

**Result/Option**
- [ ] Jede `Result`/`Option` Expression ist handled (`?`, `match`, binding, combinator)
- [ ] Keine „silent ignore“

**Validation**
- [ ] Alle externen Inputs sind `Raw<T>`
- [ ] Kein `Raw<T>` fließt in validated Parameter
- [ ] `validate(...)` ist der einzige Übergang

**Effects & Capabilities**
- [ ] Jede effectful function deklariert `! Effects`
- [ ] IO/Network/Db Funktionen nehmen Capabilities explizit
- [ ] Capabilities kommen aus `main(caps: AllCaps)` und werden nur restricted

**match / exhaustiveness**
- [ ] match deckt alle Varianten ab
- [ ] In public API Fehler‑match: kein `_`

**Budgets**
- [ ] ≤30 Zeilen pro Funktion (target)
- [ ] ≤5 Parameter
- [ ] Nesting ≤3 (if/for/loop/match)
- [ ] Complexity ≤10 (keine condition chains)

**Naming / Hygiene**
- [ ] Naming conventions eingehalten
- [ ] Keine unused vars/imports (oder `_prefix` bewusst)

**Docs**
- [ ] Jede `pub fn` hat spec tags (description, params, returns, errors, effects)

---

## 3.5 Error Recovery Guide (häufige Agent‑Fehler)

**Format:** YAML‑Liste (AI‑parsbar).  
Du kannst das als interne Knowledge Base nutzen.

```yaml
- id: AXE001
  title: "Semikolon verwendet"
  symptom: "Zeilen enden mit ';'"
  detect: "search ';' outside strings"
  fix: "remove semicolons; ensure one statement per line"
  before: "let x: i32 = 1;"
  after: "let x: i32 = 1"

- id: AXE002
  title: "while verwendet"
  symptom: "keyword 'while'"
  detect: "token 'while' present"
  fix: "rewrite as loop + if + break"
  before: "while cond { work() }"
  after: |
    loop {
        if !cond { break () }
        work()
    }

- id: AXE003
  title: "Result/Option ignoriert"
  symptom: "call returns Result/Option but not bound/matched/?"
  detect: "line contains known fallible call without 'let', 'match', '?', '.map', '.and_then'"
  fix: "handle explicitly (let+match or use '?')"
  before: "parse_config(raw)"
  after: "let cfg = parse_config(raw)?"

- id: AXE004
  title: "Raw<T> in validated context genutzt"
  symptom: "passing Raw<T> to function expecting validated type"
  detect: "type mismatch in review OR code shows Raw passed to safe fn"
  fix: "validate raw first"
  before: "send_email(raw_email)?"
  after: |
    let email: Email = validate(raw_email)?
    send_email(email)?

- id: AXE005
  title: "Effect fehlt"
  symptom: "function uses IO/Async/Db but signature has no '! ...'"
  detect: "calls io::/fs::/http:: or await inside function without effect clause"
  fix: "add correct effect clause and propagate to callers"
  before: "fn f() -> i32 { io::println(\"x\") 1 }"
  after: "fn f() -> i32 ! IO { io::println(\"x\") 1 }"

- id: AXE006
  title: "Capability fehlt"
  symptom: "fs/http/db call without capability parameter"
  detect: "fs::read(path) style calls"
  fix: "thread capability from main into function signature"
  before: "fs::read(Path::new(\"/app/a\"))?"
  after: "fs::read(read_cap, Path::new(\"/app/a\"))?"

- id: AXE007
  title: "Wildcard import"
  symptom: "use x::*"
  detect: "pattern '::*' in use statements"
  fix: "import explicit names or use module path"
  before: "use crate::domain::*"
  after: "use crate::domain::{User, UserId, Email}"

- id: AXE008
  title: "Budget: zu tiefes nesting"
  symptom: "if/for/match nesting depth > 3"
  detect: "manual scan or nesting counter"
  fix: "extract helper, use early returns, pipeline style"
  before: "deep nested conditionals"
  after: "split into parse -> validate -> execute -> respond"

- id: AXE009
  title: "Public error match uses '_'"
  symptom: "match on public error enum contains '_' arm"
  detect: "underscore arm in match on Error type used in pub fn signatures"
  fix: "enumerate all variants explicitly"
  before: "Err(_) => ..."
  after: "Err(Error::NotFound) => ... etc"

- id: AXE010
  title: "Return style drift"
  symptom: "function ends with 'return x'"
  detect: "last non-empty line starts with 'return'"
  fix: "remove 'return' and keep last expression"
  before: "return x"
  after: "x"
```

---

# Teil 4: Bootstrapping‑Plan (von 0 → produktiv)

## Phase 1: Tag 1 (sofort starten)

### Ziel
Axiom‑Code generieren, der **konsistent, reviewbar, sicherheitsbewusst** ist – auch wenn er noch nicht kompiliert.

### Benötigt
- Axiom Kernel Prompt (oben)
- Cheat Sheet
- Pattern Library
- Projekt‑Skeleton (domain/app/infra/main)
- Stdlib‑Stub‑File (Signaturen als `.ax` – not executable)

### Was kann AI leisten?
- Architektur‑Plan erstellen
- Types/Errors modellieren
- Patterns instantiieren
- Code in kleinen Budgets halten
- Specs/Docs erzeugen

### Was müssen Menschen tun?
- Library‑APIs definieren (Stubs)
- Pattern Library kuratieren
- Review nach Self‑Check‑Protokoll

---

## Phase 2: Woche 1–4 (erste echte Nutzung)

### Ziel
„Echte“ Features entwickeln (CLI Tools, Services, Worker) mit stabilem Style.

### Benötigt
- **Axiom Starter Pack Repo**:
  - `/patterns/` (Pattern IDs + code)
  - `/stubs/std.ax` (stdlib signatures)
  - `/templates/` (project skeleton)
  - `/docs/kernel.md` (Kernel rules)
- Optional: `axiom-stitch` (Pattern‑JSON → Axiom code)
- Optional: `axiom-scan` (parser-only checks)

### Was kann AI leisten?
- Feature‑Implementierung in Patterns
- Tests entwerfen (auch wenn runner fehlt)
- Dokumentation + specs konsistent halten
- Refactoring für Budgets

### Was müssen Menschen tun?
- Parser/Formatter minimal bauen **oder** Generator (stitch)
- Entscheiden: welche Stdlib APIs existieren wirklich?
- Regression‑Suite: „golden files“ (formatter output)

---

## Phase 3: Monat 1–3 (Compiler existiert, Feedback‑Loop möglich)

### Ziel
Erster kompilerfähiger Axiom‑Subset (MVP), Compiler‑Feedback als Loop.

### Benötigt
- Lexer/Parser + AST
- Typechecking (primitives, records, sum types)
- Result/Option must-handle check
- Module system minimal
- Formatter canonical
- Error JSON format

### Was kann AI leisten?
- Compiler‑Error‑Driven iteration
- Implementierung von Standard‑Checks (budgets, unused, forbidden constructs)
- Generate tests for compiler

### Was müssen Menschen tun?
- Compiler‑Core implementieren
- Runtime Host (AllCaps injection)
- Build system

---

## Phase 4: Monat 3–12 (Ökosystem wächst)

### Ziel
Produktions‑Features: Effects, Capabilities, Validation, Async, structured concurrency.

### Benötigt
- Effect checker + propagation
- Capability checker
- Validation/Raw types + inference
- Package manager minimal
- LSP basic

### Was kann AI leisten?
- Library‑Code generieren aus Contracts
- Documentation + spec export
- Migration scripts wenn language evolves

### Was müssen Menschen tun?
- ABI/Std stabilisieren
- Security model finalisieren
- Governance: style + patterns versioning

---

## Phase 5: Jahr 1+ (Fine‑Tuning möglich)

### Ziel
Axiom‑spezifische Modelle oder adapters – aber jetzt mit realen Daten.

### Benötigt
- 10k+ qualitativ gute Axiom repos / snippets
- Compiler error logs
- Pattern usage telemetry (opt-in)
- Stable language versions

### Was kann AI leisten?
- Noch bessere idiomaticity
- Better repair from compiler errors
- Auto‑refactors across codebase

### Was müssen Menschen tun?
- Curate datasets (high-quality only)
- Define evaluation suite for “slop resistance”
- Maintain backward compatibility story

---

# Teil 5: Langfristige Vision

## 5.1 AI‑Native Development Workflow

Wenn Axiom erfolgreich ist, ändert sich der Alltag:

### Der Mensch
- schreibt Requirements (Specs, invariants, contracts)
- entscheidet Architektur & boundaries
- reviewt Changes auf „meaning“ & „risk“
- verantwortet Security‑Model (caps/effects policy)

### Die AI
- implementiert entlang Patterns/Contracts
- refactort automatisch für Budgets
- erzeugt exhaustive error handling
- erzeugt tests + property checks
- hält Doku/Spezifikation synchron

### Review wird anders
- Weniger „Style“ (Formatter + Patterns)
- Weniger „Fehlerpfade vergessen“ (Compiler erzwingt)
- Mehr „Domain correctness“ und „Policy compliance“

---

## 5.2 Feedback‑Loop: Sprache ↔ AI‑Nutzung

Axiom sollte sich anhand echter Agent‑Fehler weiterentwickeln.

**Mechanismus:**
- Sammle häufige Error Codes (anonymisiert).
- Sammle häufige Pattern deviations.
- Ergänze Sprache/stdlib dort, wo Agenten *immer* stolpern.

**Features, die später helfen könnten (AI‑freundlich):**
- „Effect inference hints“ (compile‑time)
- „Capability flow visualizer“
- „Contract-to-tests generator“
- „Budget-aware refactor suggestions“ im Compiler
- „Pattern IDs“ als first‑class Annotations (optional)

---

## 5.3 Ecosystem: Libraries, Docs, Wissensverteilung

Axiom‑Libraries sollten „AI‑lesbar“ entstehen:

- **Jede Library shippt Contracts + JSON spec export**
- Public APIs haben:
  - doc tags (pre/post/errors/effects)
  - strict error enums
  - capability‑guarded IO surfaces
- Knowledge sharing:
  - Pattern PRs (wie „Style Guides“, aber maschinenlesbar)
  - golden examples
  - integration tests als canonical usage

---

## Schluss: Der Trick ist nicht „AI lernt Axiom“  
…sondern:

> **Axiom wird so verpackt**, dass ein Agent *nicht frei erfinden muss*,  
> sondern aus **kleinen, richtigen Bausteinen** zusammensetzt –  
> und sich selbst mit einem Protokoll kontrolliert, bis Compiler‑Feedback existiert.

---

# Appendix: Minimal „Axiom Starter Pack“ Repo Struktur (empfohlen)

```text
axiom-starter/
  docs/
    kernel.md
    cheat_sheet.md
    patterns.md
    self_check.md
    error_recovery.yaml
  stubs/
    std.ax          # signatures only
    http.ax
    fs.ax
  templates/
    service_skeleton/
      src/
        domain.ax
        app.ax
        infra.ax
        main.ax
  patterns/
    AX-PAT-01.ax
    ...
```
