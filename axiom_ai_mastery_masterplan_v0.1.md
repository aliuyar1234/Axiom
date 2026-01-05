# Masterplan: Wie AI Axiom **perfekt** beherrscht  
**Zielbild:** Ein AI‑Agent schreibt Axiom wie ein Senior‑Engineer mit 5 Jahren Erfahrung – korrekt, idiomatisch, architekturstark, skalierbar, zuverlässig.

> „Perfekt“ bedeutet hier: **praktisch perfekte Zuverlässigkeit** in der realen Entwicklung – nicht metaphysische Unfehlbarkeit. Der Plan zielt auf *nahezu null* Syntaxfehler, extrem hohe Compile‑Rate, hohe Test‑Pass‑Rate, konsistente Idiomatik über sehr große Codebasen, robuste Architekturentscheidungen und sichere Datenflüsse.

---

## Inhaltsverzeichnis

- [Teil 1: Was braucht AI um eine Sprache „perfekt“ zu können?](#teil-1-was-braucht-ai-um-eine-sprache-perfekt-zu-können)
- [Teil 2: Alle Wege zur Meisterschaft](#teil-2-alle-wege-zur-meisterschaft)
- [Teil 3: Die optimale Strategie (empfohlen)](#teil-3-die-optimale-strategie-empfohlen)
- [Teil 4: Synthetic Data Pipeline (Detail)](#teil-4-die-synthetic-data-pipeline-detail)
- [Teil 5: Fine-Tuning Architektur (Detail)](#teil-5-fine-tuning-architektur-detail)
- [Teil 6: Evaluation & Benchmarks](#teil-6-evaluation--benchmarks)
- [Teil 7: Kosten & Ressourcen](#teil-7-kosten--ressourcen)
- [Teil 8: Risiken & Mitigationen](#teil-8-risiken--mitigationen)
- [Teil 9: Ehrliche Einschätzung](#teil-9-die-ehrliche-einschätzung)
- [Appendix: Konkrete Deliverables & Checklisten](#appendix-konkrete-deliverables--checklisten)

---

# Teil 1: Was braucht AI um eine Sprache „perfekt“ zu können?

## 1.1 Was bedeutet „eine Sprache können“ für ein LLM?

Für ein LLM ist „eine Sprache können“ **kein** einzelner Skill, sondern ein Bündel aus vier Kompetenzebenen, die auf unterschiedlichen Signalen beruhen:

### Ebene A — Syntaxkompetenz (Form)
**Fähigkeit:** gültige Tokenfolgen erzeugen: korrekte Keywords, Klammern, Operatoren, Parsing‑Struktur.

**Wie ein LLM das lernt:**
- Aus *vielen* Beispielen, in denen Syntax häufig vorkommt.
- Aus „negative evidence“: Fehlermeldungen, Korrekturdiffs, Reviews, lint fixes.

**Warum Syntax in neuen Sprachen scheitert:**
- Ohne Beispiele gibt es kein robustes „Sprachgefühl“ für die Tokenfolge.
- Das Modell „fällt zurück“ auf nahe Sprachen (Rust/Go/TS), produziert Mischformen.

**Wichtig:** Syntax ist am einfachsten zu bootstrappen, weil sie sich mit Grammatik und Formatierung stark constrain lassen.

---

### Ebene B — Semantikkompetenz (Bedeutung)
**Fähigkeit:** Code schreiben, der korrekt ausführt: Typregeln, Ownership, Effekte, Error‑Handling, Concurrency, etc.

**Wie ein LLM das lernt:**
- Aus **(Code, Tests)** Paaren.
- Aus **(Code, Compilerfehler, Fix)** Trajektorien.
- Aus „konzeptioneller Literatur“: Spezifikationen, Lehrtexte, RFCs.
- Aus Feedbackschleifen mit einer echten Semantik‑Oracle (Compiler/Interpreter/VM).

**Warum Semantik ohne Tooling schwierig ist:**
- Semantik ist nicht rein lokal: eine kleine Entscheidung in einem Modul kann Typen/Effects in 20 anderen ändern.
- Semantikfehler sind oft erst in Randfällen sichtbar (Async, Race, error propagation).

---

### Ebene C — Idiomatische Kompetenz (Stil & „Axiom-native“)
**Fähigkeit:** „So schreibt man das in Axiom“ – kanonische Patterns, APIs, Strukturierung, Naming, Fehlerdesign, modulare Dekomposition.

**Wie ein LLM das lernt:**
- Aus großen Mengen **hochqualitativer, konsistenter** Codebases.
- Aus Community‑Konventionen (Guides, PR Reviews).
- Aus Formatter‑Output (ein Stil) + Pattern Libraries.

**Warum Idiomatik schwer zu bootstrappen ist:**
- Idiome entstehen sozial/historisch – am Anfang gibt es keine.
- Das Modell erzeugt sonst „Rust‑in‑Axiom‑Syntax“.

---

### Ebene D — Systemkompetenz (Architektur + Domain Modeling)
**Fähigkeit:** Große Systeme in Schichten strukturieren, Abhängigkeiten schneiden, Domains in Types/Enums/Refinements modellieren, Fehler- und Recovery‑Strategien entwerfen.

**Wie ein LLM das lernt:**
- Aus **Langhorizont‑Tasks** (multi‑file, multi‑module, monorepo).
- Aus Projekten mit klarer Architektur (DDD, layered, hexagonal) – plus deren Reviews.
- Aus Constraints: Layer checks, budgets, contracts, capability policies.
- Aus „plan‑execute‑verify“ Loops.

**Warum das schwer ist:**
- Architektur ist nicht „lokal optimierbar“.
- Gute Architektur ist stark abhängig von Produktanforderungen und Evolution (Change‑tolerant design).

---

### „Ist es Pattern‑Matching? Verstehen? Beides?“

**Praktische Antwort:**  
Ein LLM „kann“ eine Sprache perfekt, wenn es **drei Quellen** kombiniert:

1) **Statistische Verteilung** (pattern memory):  
   es hat genug Beispiele gesehen, um zu wissen, was „typisch“ ist.

2) **Regelbasierte Constraints** (spec/grammar/tooling):  
   es kann gar nicht erst syntaktisch/strukturell Unsinn erzeugen.

3) **Feedback‑Loops** (compiler/tests/reward):  
   es lernt und korrigiert systematisch, statt zu raten.

> Perfektion entsteht nicht durch *eine* Methode, sondern durch **Koordination** aus (Distribution + Constraints + Feedback).

---

## 1.2 Warum kann GPT‑4/Claude Python/Rust gut?

Es gibt drei Hauptgründe – und alle sind für Axiom replizierbar, aber nur mit Aufwand.

### 1) Massive Trainingsdaten (Breadth + Depth)
Modelle haben gesehen:
- Millionen Repos (verschiedene Stile, Libraries, Patterns).
- Dokumentation, Tutorials, Bücher, Blogposts.
- Q&A (Stack Overflow), Issue‑Tracker, Pull Request Diskussionen.
- Korrekturen: „Bug → Fix“ Diffs, Linters, formatter changes.

**Wichtig:** Es geht nicht nur um „Code“, sondern um das gesamte **Ökosystem‑Text‑Mesh** (Errors, Reviews, Docs, Diskussion).

**Für Axiom replizierbar?**  
Ja – aber du musst dieses Mesh bewusst erzeugen (siehe Pipeline).

---

### 2) Feedback als „Negative Beispiele“
Python/Rust Kompetenz kommt nicht nur aus richtigem Code, sondern aus:
- Fehlermeldungen (SyntaxError, type errors, borrow checker),
- „du solltest X nicht tun“ in Reviews,
- alternative Implementierungen.

**Für Axiom replizierbar?**  
Ja – sobald Compiler/Checker existieren. Vorher kannst du „pseudo‑negative examples“ erzeugen (siehe strukturelle Checks und pattern validator).

---

### 3) Tool‑Kompatibilität und „Ähnlichkeit zu existierenden Sprachen“
Rust‑ähnliche Syntax und Konzepte (Result/Option, match, ownership) sind in Modellen bereits präsent.  
Das verringert die Distanz, die Axiom überbrücken muss.

**Für Axiom replizierbar?**  
Ja – Axiom ist bewusst „nahe“ an Rust/ML‑Family, aber strenger.

---

## 1.3 Unterschied: Syntax kennen vs Semantik vs Idiomatik vs Architektur vs Domain Modeling

Hier ist die entscheidende Unterscheidung (und sie bestimmt den Trainingsplan):

| Kompetenz | „Kann“ bedeutet | Typische Fehler ohne diese Kompetenz | Wie man es trainiert/erzwingt |
|---|---|---|---|
| Syntax | parsebar, korrekte Token | erfundene Keywords, falsche Klammern | Grammar‑constrained decoding, formatter |
| Semantik | typ- und laufzeitkorrekt | falsche Ownership, fehlende Effects, unhandled Result | Compiler‑in‑loop, tests, static checks |
| Idiomatik | Axiom-native Stil | Rust‑Kopie, 5 Varianten für dasselbe | canonical patterns, formatter, curated repos |
| Architektur | gute Module/Layers | cyclic deps, layer violations, god modules | layer checker, contracts, repo-level tasks |
| Domain Modeling | richtige Types/Errors/Invariants | stringly typed, unvollständige error models | domain task corpora, contract-driven design |

**Zentrale Erkenntnis:**  
„Perfekt“ heißt nicht nur „kompiliert“. Es heißt „kompiliert + richtig + wartbar + sicher + konsistent“.

---

## 1.4 Minimale Voraussetzungen: Was ist das absolute Minimum?

Du brauchst einen **Minimal‑Stack**, der aus 5 Bausteinen besteht. Ohne diese wirst du „perfekt“ nicht erreichen.

### Minimum 1: Ein stabiler, kleiner Axiom‑Kernel (Spec‑Subset)
- definierte Syntax/Keywords,
- klare Regeln (forbidden constructs),
- minimaler stdlib‑Surface.

> Ohne stabilen Kernel kannst du keine Daten kuratieren, weil sich die Sprache ständig ändert.

### Minimum 2: Ein Formatter (oder pretty printer) als „Single Truth“
- canonical output,
- keine Style‑Konfiguration.

> Damit werden „100 Stile“ zu „1 Stil“. Das ist essenziell für Idiomatik und Konsistenz.

### Minimum 3: Ein Parser (mindestens) + AST
- um syntaktische Validität zu prüfen,
- um Code in strukturierter Form als Trainingssignal zu nutzen,
- um grammar‑constrained generation zu ermöglichen.

> Ein Parser ist viel leichter als ein kompletter Compiler – aber er ist die Eintrittskarte für Tooling.

### Minimum 4: Stdlib‑Stubs (Signaturen + docs)
- keine Halluzination von APIs,
- klare capability/effect signatures.

> Viele „unknown language“ Fehler sind eigentlich „unknown library“ Fehler.

### Minimum 5: Eine Evaluation Suite (AxiomBench v0)
- definierte Aufgaben,
- klare Metriken.

> Wenn du „perfekt“ willst, brauchst du eine Messfunktion, sonst optimierst du im Dunkeln.

---

### Gibt es einen Threshold, ab dem es „klickt“?

Ja, typischerweise gibt es einen **Qualitäts‑Phasenübergang** sobald:

1) der Agent nicht mehr frei Text generiert, sondern
2) **durch Constraints und Feedback** stabilisiert wird.

Für Axiom ist dieser „Klick“ meistens erreicht, wenn du:

- **Grammar/AST‑Constraints** +  
- **Formatter** +  
- **Compiler/Checker‑Feedback** +  
- **curated exemplars**  

kombinierst.

---

# Teil 2: Alle Wege zur Meisterschaft

Unten: alle genannten Ansätze (A–F) plus ein Ansatz G, der in der Praxis der beste Hebel ist.

## 2.0 Vergleichsmatrix (schneller Überblick)

Bewertungsskala:
- Qualität (1–10): 10 = „Senior‑Level zuverlässig in realen Projekten“
- Aufwand: grob (Low/Med/High)
- „Perfekt“ Zeitrahmen: realistische Bandbreite bei guter Ausführung

| Ansatz | Mechanik | Benötigt | Qualität (1–10) | Risiken | Zeit bis „perfekt“ |
|---|---|---|---:|---|---|
| A Synthetic Data + FT | generate→compile/test→train | Compiler + Pipeline + Compute | 8–10 | reward hacking, data drift, cost | 12–36 Monate |
| B Compiler-in-loop | generate→compile errors→repair | Compiler, Agent tools | 6–9 | langsam, local minima | 6–24 Monate |
| C Constrained / Spec→Code | structured spec→generator | AST/IR + generator | 7–10 | flexibility loss, spec gaps | 6–18 Monate |
| D Human bootstrapping | humans write→AI expands→curate | Humans + review infra | 6–9 | human bottleneck | 6–24 Monate |
| E Transfer Learning | learn deltas from Rust | curated delta set | 5–8 | hidden semantic differences | 6–18 Monate |
| F RL from compiler | reward = compile/tests | RL infra + compute | 7–10 | unstable, expensive | 12–36 Monate |
| G „Triad“ (AST-first + Co-evolution + Distillation) | constraints + tools + data + distill | Parser/formatter early, then compiler | 9–10 | engineering complexity | 9–30 Monate |

---

## Ansatz A: Massive Synthetic Data Generation (mit Fine‑Tuning)

### Mechanik (wie es funktioniert)
1) Task‑Generator erzeugt Aufgaben (divers, parametriert).  
2) Basismodell (z.B. code LLM) generiert Axiom‑Lösungen.  
3) Compiler + Test‑Harness validieren.  
4) Nur „beste“ Lösungen werden aufgenommen.  
5) Fine‑Tune/Instruction‑Tune auf dem Dataset.  
6) Iterate: Das bessere Modell erzeugt bessere Daten → neue Runde.

### Was braucht man?
- **Compiler** (mindestens Parser+Typecheck; ideal full).
- Test Framework (unit + property).
- Data pipeline (dedup, scoring, curation).
- Compute (Training + Inferenz).
- Governance: Versions/compatibility.

### Erwartete Qualität
- **8–10**, abhängig von:
  - Qualität der Validierung (Tests!),
  - Dataset‑Qualität (nicht nur „compiles“),
  - Curriculum,
  - Overfitting‑Kontrolle.

### Größte Risiken
- **Garbage‑in/garbage‑out**: Wenn „guter Code“ schlecht definiert ist, trainierst du Slop.
- **Reward hacking**: Modell lernt „Tests zu betrügen“ oder „Compiler zu befriedigen“.
- **Mode collapse**: zu wenige Stile/Patterns → Modell wird spröde.
- **Spec drift**: Sprache ändert sich; Dataset veraltet.

### Zeitrahmen bis „perfekt“
- Mit gutem Team + Tooling: **12–36 Monate**.

---

## Ansatz B: Compiler‑in‑the‑Loop (kein Fine‑Tuning)

### Mechanik
- Agent schreibt Axiom.
- Compiler gibt Fehler.
- Agent repariert.
- Repeat, bis compile/test passt.

### Was braucht man?
- Einen Compiler (mindestens robustes type/effect/cap checker).
- Gute Fehlermeldungen (AI‑readable JSON).
- Agent‑Orchestrator (automatisiert loops, patching).

### Erwartete Qualität
- **6–9**  
Sehr gut für Korrektheit, aber: Idiomatik/Architektur bleibt begrenzt, wenn der Compiler diese Dinge nicht misst.

### Grenzen
- **Langsam**: viele Iterationen.
- **Nur lokal optimiert**: Agent fixiert Compilerfehler, nicht unbedingt Systemdesign.
- **Kein „Absorbieren“**: Ohne Training bleibt Kompetenz transient; du brauchst immer Loop.

### Zeit bis „perfekt“
- **6–24 Monate**, aber: „perfekt“ ohne Fine‑Tuning ist schwer, weil Langzeit‑Idiomatik fehlt.

---

## Ansatz C: Structural / Constrained Generation (Spec → Generator → Axiom)

### Mechanik
- AI erzeugt **strukturierte Repräsentation**: AST/IR/JSON „Plan“.
- Ein Generator (deterministisch) erzeugt Axiom‑Code:
  - Syntax korrekt,
  - formatiert,
  - budgets enforced,
  - imports/layers automatisch.

### Was braucht man?
- AST Schema + generator.
- Library stubs.
- (Später) compiler feedback.

### Erwartete Qualität
- **7–10**  
Sehr hoch, wenn der IR expressiv genug ist.

### Verliert man Flexibilität?
**Ja – wenn der IR zu klein ist.**  
Die Lösung: IR muss nicht „einfach“ sein; es kann:
- freie Expressions erlauben,
- aber syntaktisch/semantisch typed sein,
- und generator übernimmt „mechanische“ Details.

### Risiken
- IR wird zu komplex (ähnlich einem Compiler).
- Agent muss IR lernen (aber IR ist stabiler und constraint‑friendly).

### Zeitrahmen
- **6–18 Monate** (oft schneller als full fine‑tuning, weil deterministic tooling viel abfängt).

---

## Ansatz D: Human‑in‑the‑Loop Bootstrapping

### Mechanik
1) Menschen schreiben ~1.000 hochwertige Beispiele (kleine libs, idioms).
2) AI lernt per RAG/prompting darauf.
3) AI generiert 10.000 weitere.
4) Menschen kuratieren (Sampling + active learning).
5) Repeat → Dataset wächst und wird besser.

### Was braucht man?
- 1–5 starke Engineers (Axiom‑Designer + system dev).
- Review tooling, scoring rubrics.
- Stable style (formatter) und pattern guidance.

### Erwartete Qualität
- **6–9**  
Kann sehr gut werden, wenn Menschen *wirklich* gute Vorbilder liefern.

### Wie viel menschliche Arbeit ist realistisch?
- 1.000 Beispiele sind machbar, wenn:
  - „Beispiele“ klein sind (50–200 LOC),
  - viele Variationen algorithmisch entstehen,
  - man „golden patterns“ baut statt jede App neu zu schreiben.

### Risiken
- Menschen werden zum Bottleneck.
- Bias: nur „der Stil der ersten 2 Entwickler“.
- Müdigkeit → Qualitätsabfall.

### Zeitrahmen
- **6–24 Monate**.

---

## Ansatz E: Transfer Learning (Rust‑Wissen + Axiom‑Deltas)

### Mechanik
- Nutze, dass Modelle Rust gut können.
- Trainiere/Prompt‑tune nur die Unterschiede:
  - Effects,
  - Capabilities,
  - Budgets,
  - Raw/Validated,
  - Layer/contracts.

### Was braucht man?
- Delta‑Spec (Mapping Rust → Axiom).
- „paired examples“: Rust snippet → Axiom idiomatic rewrite.
- Evaluation suite, die Axiom‑spezifische Dinge misst.

### Erwartete Qualität
- **5–8**  
Sehr effektiv für Syntax/Grundsemantik, aber:
- kann zu „Rust‑denken“ führen,
- Axiom‑native Idiomatik kann fehlen.

### Risiken
- Modelle „importieren“ Rust‑Konventionen, die Axiom bewusst verbietet.
- Ownership/Lifetimes: Axiom‑Vereinfachungen müssen klar.

### Zeitrahmen
- **6–18 Monate**.

---

## Ansatz F: Reinforcement Learning from Compiler Feedback

### Mechanik
- Treat compiler/tests as environment.
- Reward = compile success + test pass + budget/layer compliance + style.
- Agent explores, gets reward, updates policy.

### Was braucht man?
- Robust reward function (nicht nur „compiles“).
- Massive compute + infra.
- Curriculum (sonst lernt Agent wenig).

### Erwartete Qualität
- **7–10**, aber stark abhängig von reward design.

### Risiken
- **Reward hacking** ist hier besonders gefährlich.
- Instabilität: RL kann regressieren.
- Cost: RL ist teuer.

### Zeitrahmen
- **12–36 Monate**.

---

## Ansatz G (empfohlen): „The Triad“ – AST‑First + Co‑Evolution + Distillation

Das ist der Ansatz, der in der Praxis am ehesten zu „perfekt“ führt, weil er die Stärken aller anderen kombiniert.

### Mechanik (3 Säulen)

#### Säule 1: AST‑First Generation (Constrained Output)
- Agent generiert zuerst AST/IR oder grammar‑constrained code.
- Pretty printer erzeugt canonical Axiom.

**Ergebnis:** Syntaxfehler ≈ 0.

#### Säule 2: Co‑Evolution von Sprache, Tooling und Daten
- Früh: Parser/formatter/scanner.
- Dann: type/effect/cap checker.
- Dann: runtime/test harness.
- Jedes Tool erzeugt neue „negative examples“ und Trajektorien.

**Ergebnis:** Semantik + Architektur werden messbar.

#### Säule 3: Distillation (Training auf besten Trajektorien)
- Nutze compiler‑repair traces:
  - initial attempt,
  - error,
  - fix,
  - final.
- Trainiere das Modell, um „first pass correctness“ zu erhöhen.
- Wiederhole iterativ.

**Ergebnis:** Der Agent wird mit jeder Runde autonomer und braucht weniger loops.

### Was braucht man?
- Parser + AST + formatter (sehr früh).
- IR schema + generator (optional, aber stark).
- Compiler‑Feedback (später).
- Data pipeline.

### Erwartete Qualität
- **9–10** bei guter Umsetzung.

### Risiken
- Engineering‑Komplexität (Tooling + Data + Training parallel).
- Governance/Versioning.

### Zeitrahmen
- **9–30 Monate**.

---

# Teil 3: Die optimale Strategie (empfohlen)

## 3.1 Die empfohlene Kombination (Reihenfolge + Begründung)

Die beste Route zu „perfekt“ ist eine **mehrstufige Kombination**:

1) **Constrained Generation (C/G)** *so früh wie möglich*  
   - Parser/AST/formatter → Syntax‑Perfektion.
2) **Compiler-in-loop (B)** sobald ein typechecker existiert  
   - Semantik‑Feedback, Reparaturtraces.
3) **Human bootstrapping (D)** für Idiomatik und Architektur‑Vorbild  
   - kleine Anzahl, aber sehr hochwertig.
4) **Synthetic Data + Distillation (A/G)**  
   - skaliert das Wissen.
5) **RL (F)** nur am Ende, gezielt  
   - „polish“ und edge cases, nicht als Start.

**Warum diese Reihenfolge?**
- Ohne constraints trainierst du Chaos.
- Ohne feedback lernst du Semantik nicht.
- Ohne human exemplars bekommst du keine Idiomatik.
- Ohne scale (synthetic) kommst du nicht auf Senior‑Breite.
- RL ist teuer und riskant; nutze es, wenn du schon gute Rewards + Baseline hast.

---

## 3.2 Konkrete Roadmap

> Annahme: kleines Kernteam (3–6 Personen) mit PL/Compiler + ML + Systems.  
> Bei weniger Leuten verlängert sich die Timeline, aber die Sequenz bleibt.

### Monat 1–3: „Axiom Tooling Seed“ (Syntax + Style + Stubs)
**Ziele**
- Syntax‑Correctness und Stil‑Konsistenz als Fundament.
- Kein „perfekt“, aber *keine Syntax‑Slop mehr*.

**Deliverables**
- Parser (EBNF subset) + AST
- Formatter / pretty printer (canonical)
- `axiom-scan`:
  - forbidden constructs,
  - unused imports/vars (approx),
  - budget heuristics (lines/nesting/params),
  - no wildcard imports.
- Stdlib stubs (Signaturen) + docs JSON
- AxiomBench v0 (20–50 small tasks)
- Pattern Library v1 (30–60 patterns)

**Erfolgskriterien**
- >99% syntaktisch gültiger Output, wenn AST‑first genutzt wird.
- Stable formatting across samples.

---

### Monat 4–6: „Semantic Kernel“ (Typecheck + Result/Option must-handle)
**Ziele**
- Grundsemantik + zentrale Axiom‑Constraints.
- Erste echte compiler‑in‑loop work.

**Deliverables**
- Typechecker: primitives, records, sum types, generics basic
- `Result/Option must-handle` check
- Module system + import resolution
- Layer enforcement v1 + cycle detection
- Effect checker v1 (IO/Async minimal)
- Error messages als JSON (fix‑its)

**Erfolgskriterien**
- compile success rate (CSR) auf AxiomBench v1 > 70% in 1–3 loops
- first pass CSR > 30% (ohne loops)
- must-handle violations automatisch detektierbar

---

### Monat 7–12: „Full Axiom Constraints“ (Effects+Caps+Validation+Async)
**Ziele**
- Axiom‑Alleinstellungsmerkmale vollständig messbar.
- Agent kann echte Systeme bauen.

**Deliverables**
- Capability system checker + restriction semantics
- Raw/Validated flow + validate insertion rules
- Structured concurrency primitives (scope)
- Async/await semantics (IR + runtime)
- Budget checker (hard) + overrides policy
- Contract/spec parser + export JSON
- AxiomBench v2 (200–500 tasks, inkl. multi‑file)

**Erfolgskriterien**
- first pass CSR > 60%
- CSR after ≤2 loops > 90%
- test pass rate (TPR) auf Bench > 70%
- architecture score (Layer compliance) > 95%

---

### Jahr 2: „Data Scale + Fine‑Tuning“ (Distillation)
**Ziele**
- Agent wird „senior‑like“ ohne ständig compiler loops.
- Idiomatik konsistent über große Repos.

**Deliverables**
- Synthetic data pipeline (siehe Teil 4)
- Dataset v1 (50k–500k examples, je nach scope)
- SFT/Instruction tuning v1 (LoRA oder full, je nach budget)
- Repair trace dataset (error→fix trajectories)
- AxiomBench v3 (long-horizon tasks, 10k+ LOC)

**Erfolgskriterien**
- first pass CSR > 85%
- TPR > 80% auf Bench tasks
- style/idiom score: > 90% „canonical pattern adherence“
- large‑repo consistency: low drift (siehe Metriken)

---

### Jahr 3+: „Edge Cases + RL Polishing + Ecosystem“
**Ziele**
- „Perfekt“ nähert sich 95–99% reliability in real workflows.
- Ecosystem + libraries + docs generieren selbst.

**Deliverables**
- RL from compiler/test + architecture reward (sicher designed)
- Multi‑agent planning/execution pipelines
- Library generation with contract verification
- Ecosystem governance + versioning policy

**Erfolgskriterien**
- first pass CSR 95%+ (für in‑distribution tasks)
- long-horizon task success > 80% end‑to‑end
- regression slope: <1% quality loss across versions

---

## 3.3 Meilensteine und Metriken: Wie misst man „Axiom‑Kompetenz“?

### Kernmetriken (automatisch messbar)

1) **CSR – Compile Success Rate**
- Anteil Aufgaben, bei denen Code kompiliert.
- Zusätzlich: CSR@k (mit k Versuchen/loops).

2) **TPR – Test Pass Rate**
- Anteil kompilierten Codes, der alle Tests besteht.

3) **FPR – Forbidden Pattern Rate**
- Anteil Outputs mit verbotenen Konstrukten (`while`, `try/catch`, `null`, wildcard imports).

4) **MHR – Must‑Handle Compliance Rate**
- Anteil Outputs ohne unhandled `Result/Option`.

5) **ECR – Effect/Capability Correctness Rate**
- Anteil Outputs, in denen Effects und Capabilities korrekt deklariert/propagiert sind.

6) **ACS – Architecture Compliance Score**
- Layer rules eingehalten, keine cycles, contract requirements satisfied.

7) **BCS – Budget Compliance Score**
- Anteil Funktionen innerhalb budgets (lines/params/nesting/complexity).

8) **ICS – Idiomatic Consistency Score**
- Messbar durch:
  - formatter normalization + AST pattern matching,
  - Anteil an canonical patterns,
  - variance in naming/structure across repo.

### „Perfekt“ Operational Definition (für Axiom)

Ein Agent ist „praktisch perfekt“, wenn er über ein repräsentatives Benchmark‑Set:

- CSR (first pass) ≥ 95%  
- TPR (first pass) ≥ 90%  
- FPR ≤ 0.1%  
- MHR ≥ 99.9%  
- ECR ≥ 99%  
- ACS ≥ 98%  
- ICS ≥ 95%  
- Long‑horizon tasks: ≥ 80% end‑to‑end success ohne menschlichen Patch

> Wichtig: Diese Zahlen sind nur erreichbar, wenn Benchmarks „realistisch“ sind und nicht zu leicht.

---

# Teil 4: Die Synthetic Data Pipeline (Detail)

Wenn synthetische Daten Teil der Strategie sind (empfohlen), muss die Pipeline so gebaut sein, dass sie **Qualität** optimiert, nicht nur Menge.

## 4.0 Pipeline‑Prinzipien

1) **Compiler/test als Oracle** (so früh wie möglich).  
2) **Curriculum**: erst Syntax → dann Semantik → dann Idiome → dann Systeme.  
3) **Diversity by construction**: Variation muss systematisch erzeugt werden (nicht „random prompts“).  
4) **Negative examples sind Gold**: error→fix trajectories.  
5) **Dedup + leakage control**: keine „copy farms“.  
6) **Versioned datasets**: language version pinned.

---

## 4.1 INPUT: Was geht rein?

- Axiom Spec (versioned)
- Kernel rules + forbidden constructs
- Stdlib/API stubs (signatures + docs)
- Pattern library (canonical)
- Task templates (parametrierbar)
- Reference implementations (human goldens, wenn vorhanden)
- Policy config (budgets, layer rules)

---

## 4.2 Schritt 1: Task‑Generierung

### Wie entstehen diverse Programmieraufgaben?

#### Quelle A: Template‑Tasks (parametrisiert)
Beispiele:
- Parsing/validation tasks (Email, Port, Money, Date)
- CRUD domain services (users, orders, payments)
- Async fetch/aggregate
- Concurrency patterns (channels, mutex, scope)
- Capability propagation tasks (read config deep stack)
- Architecture tasks (create modules with layers)

Jede Aufgabe wird mit Parametern variiert:
- Anzahl error variants
- Anzahl module boundaries
- Kombination von effects (IO+Async, Db+Log, etc.)
- Variation von domain constraints (refinement ranges, regex)
- Variation von API usage

#### Quelle B: Program Synthesis „Micro‑specs“
Kleine Spezifikationen (pre/post/errors) → implementieren.

#### Quelle C: Refactoring Tasks
- gegeben „sloppy code“ (oder Rust‑like), zu idiomatic Axiom refactoren
- budget enforcement tasks

#### Quelle D: Multi‑file System Tasks
- „build a service skeleton“ (domain/app/infra)
- implement contracts + invariants

### Wer formuliert die Tasks?
- Zuerst Menschen (kleines Set).
- Dann AI‑Task‑Generator, aber mit Metaregeln:
  - „must include at least one effect“
  - „must require Raw->validate“
  - „must require at least 3 error variants“
  - „must be multi-module“

### Wie stellt man Vielfalt sicher?
- Coverage‑Matrix: Features × Difficulty × Domain.
- Jede Trainingswoche: Ziel‑Coverage definieren.
- Sampling: underrepresented buckets oversample.
- Adversarial tasks: gezielt tricky combos (async + caps + generics + bounds).

---

## 4.3 Schritt 2: Code‑Generierung

### Welches Modell?
- Anfang: best available code LLM + Kernel prompt + RAG (patterns + stubs).
- Später: eigenes fine‑tuned Axiom model + fallbacks.

### Wie viele Versuche pro Task?
- Start: k=8–32 Kandidaten (diverse decoding settings).
- Später: k=2–8 (modell besser, costs geringer).

### Wie Varianten erzeugen?
- Unterschiedliche Seed‑Prompts:
  - pattern-first
  - contract-first
  - architecture-first
- Unterschiedliche constraint levels:
  - AST-first vs raw text
- Cross-model generation:
  - Model A writes, Model B reviews, Model C repairs

### Output‑Formate
- Prefer: AST/IR + generator, um syntaktische Fehler zu eliminieren.
- Zusätzlich: raw Axiom text + formatter.

---

## 4.4 Schritt 3: Validierung

Je nach Reifegrad:

### Früh (ohne Vollcompiler)
- Parser check
- formatter roundtrip (parse → format → parse)
- static checks:
  - forbidden constructs
  - must-handle heuristic (AST-based)
  - budgets
  - layer graph analysis (imports)
- stub type checks (partial)

### Später (mit Compiler)
- full compile (type/effect/cap)
- unit tests (generated)
- property-based tests (PBT)
- fuzzing (inputs, concurrency)
- mutation testing (optional)

### Property‑Based Testing (PBT) in Axiom Kontext
- Ideal für refinements/invariants:
  - „withdraw never makes balance negative“
  - „parse then stringify is idempotent“

---

## 4.5 Schritt 4: Qualitäts‑Filterung

**Was macht Code „gut genug“ fürs Training?**  
Du brauchst eine harte Scoring‑Funktion.

### Harte Gates (muss)
- parses + formats cleanly
- no forbidden constructs
- must-handle ok
- budget compliance ok
- (wenn compiler) compile success
- (wenn tests) tests pass

### Soft Scores (rank)
- idiom score (pattern adherence)
- doc/spec completeness
- architecture score (layers, deps, cycles)
- complexity score (cyclomatic, nesting)
- security score (Raw flow, capability minimization)

**Sampling‑Strategie**
- Keep top N per task bucket.
- Keep diversity: do not keep 100 near-duplicates.

---

## 4.6 Schritt 5: Kuratierung (Human + AI)

### Wie viel menschliche Arbeit?
Du willst **nicht** alles reviewen. Du willst:
- 1–5% sampling per batch
- active learning: review nur dort, wo model unsicher/score low/novel pattern

### Wer macht das?
- Axiom core team + später community maintainers.
- Review‑Rubric:
  - correctness (semantics)
  - idiom (pattern match)
  - architecture (layers)
  - security (Raw/caps)

### Wie skaliert das?
- Multi‑stage filters reduzieren Kandidaten stark.
- Human reviews nur für „borderline“ oder „novel“ examples.

---

## 4.7 Schritt 6: Dataset‑Formatierung

### Welches Format?
Du brauchst mehrere Dataset‑Typen (multi‑task):

1) **Instruction → Code**  
   Prompt: Aufgabe + stubs + policies  
   Output: Axiom code

2) **Code → Fix (Compiler error repair)**  
   Prompt: code + error JSON  
   Output: patch/diff

3) **Contract/spec → Implementation**  
   Prompt: module contract + function spec  
   Output: code

4) **Refactor to budgets/idiom**  
   Prompt: non-idiomatic code + budgets  
   Output: idiomatic code

5) **Architectural plan → Repo**  
   Prompt: requirements + layer policy  
   Output: multi-file layout + code

### Wie groß sollten Beispiele sein?
- 50–300 LOC für SFT „core“
- 300–2k LOC für advanced modules
- 10k+ LOC als evaluation only (nicht fürs Training als single sample; stattdessen segmented).

### Balance Komplexität
- Curriculum:
  - 40% small
  - 40% medium
  - 20% advanced
- ensure every feature appears across sizes.

---

## 4.8 OUTPUT: Dataset Erwartung

**Größe:**  
- Für brauchbare Axiom‑Kompetenz: 50k–200k high‑quality examples (nicht raw).
- Für „Senior‑like“: 200k–1M examples + repair traces + long-horizon tasks.

**Qualität:**  
- wichtiger als Größe: jede sample muss „golden“ sein.

**Kosten:** siehe Teil 7.

---

# Teil 5: Fine-Tuning Architektur (Detail)

Wenn Fine‑Tuning Teil der Strategie ist (für „perfekt“ quasi notwendig), dann muss es mehrstufig sein.

## 5.1 Basis‑Modell Auswahl

### Kriterien (wichtig)
- **Code‑Kompetenz** (multi‑language, reasoning)
- **Long context** (für multi‑file)
- **Tool use / function calling** (für AST/IR constraints, compiler loops)
- **Fine‑tuning friendliness** (LoRA support, licensing)
- **Inference cost** (du brauchst viel generation für data)

### Open Source vs Closed

**Closed (API)**
- Vorteile: top raw ability, tool integrations.
- Nachteile: schwerer/teurer zu fine‑tunen, dataset privacy.

**Open Source**
- Vorteile: volle Kontrolle, LoRA/QLoRA, on‑prem, iterative training.
- Nachteile: raw ability geringer (je nach Modell), engineering overhead.

**Empfehlung:**  
- Anfang: nutze die besten verfügbaren Agentenmodelle via API für bootstrap + data generation.  
- Für „perfekt“: baue ein eigenes fine‑tuned Modell (open weights oder enterprise closed fine‑tuning).

---

## 5.2 Training‑Setup: Full FT vs LoRA vs QLoRA

### Empfehlung (praktisch)
- Start mit **LoRA/QLoRA**:
  - schnell,
  - billig,
  - iterativ.
- Wechsel zu **full fine‑tune** nur wenn:
  - du sehr großes Dataset hast,
  - du absolute Spitzenqualität brauchst,
  - du das Inferenzbudget hast.

### Hyperparameter (Richtwerte)
*(absichtlich als Richtwerte, nicht als Dogma; je nach Modellgröße)*

- Sequence length: so hoch wie möglich ohne Instabilität (z.B. 8k–32k)
- LR: klein (LoRA), warmup 1–3%, cosine decay
- Batch size: so groß wie möglich über grad accumulation
- Weight decay: moderat
- Mixed precision: BF16/FP16

### Hardware‑Anforderungen (Faustregel)
- LoRA: 1–8 GPUs je nach size und context
- full FT: 8–64 GPUs für große Modelle

---

## 5.3 Multi‑Stage Training (Curriculum)

Für „perfekt“ ist ein Curriculum fast immer besser als „alles auf einmal“.

### Stage 0: Syntax & Canonicalization
- Trainiere: AST‑to‑Axiom, format‑roundtrip.
- Ziel: near‑zero syntax errors.
- Daten: viele kleine examples, grammar‑tight.

### Stage 1: Semantik Core
- Type system, Result/Option must-handle, basic generics.
- Daten: compile‑verified samples.

### Stage 2: Axiom Differentiators
- Effects + Capabilities + Raw/Validated + Budgets + Layers.
- Daten: tasks, die diese Features zwingend brauchen.

### Stage 3: Idiomatik & Architektur
- Multi‑module design tasks.
- Contract/spec driven development.
- Daten: human goldens + top synthetic.

### Stage 4: Repair & Refactor
- Compiler error repair traces (high value).
- Budget refactoring tasks.
- Diff‑based training (patch prediction).

### Stage 5: Long Horizon
- Multi‑file sequences mit „plan → implement → test → fix“.

---

## 5.4 Evaluation während Training

Du brauchst ein *frozen* Validation Set, das nicht in Training leak’t.

- AxiomBench (frozen split)
- Hidden tests
- Architecture tasks
- Adversarial tasks (edge cases)

### Early Stopping Kriterien
- CSR / TPR plateau
- Overfitting: training CSR steigt, validation CSR fällt
- Idiom score sinkt (modells driftet zu shortcuts)

---

## 5.5 Post‑Training: RLHF nötig? DPO/PPO?

### Wann reicht SFT?
- Wenn Compiler/Test/Constraints stark sind
- Wenn Daten sehr sauber sind

### Wann zusätzlich RL (oder DPO)?
- Wenn du „policy compliance“ maximieren willst:
  - budgets,
  - capabilities minimization,
  - architecture style.

**Empfehlung:**  
- Erst DPO/Preference‑Training mit curated comparisons, weil stabiler als PPO.
- PPO/RL nur, wenn reward function sehr gut und robust ist.

---

# Teil 6: Evaluation & Benchmarks

„Perfekt“ ohne Benchmark ist Marketing. Du brauchst AxiomBench.

## 6.1 Syntax‑Korrektheit

### Metrik: Compile‑Success‑Rate (CSR)
- CSR(first pass)
- CSR@k (mit k repair loops)
- CSR@time (innerhalb 60s)

Zusatz: Formatter roundtrip rate (parse→format→parse).

---

## 6.2 Semantische Korrektheit

### Metrik: Test‑Pass‑Rate (TPR)
- TPR(first pass)
- Hidden tests (anti‑overfitting)
- Property‑based (PBT) pass rate
- Mutation score (optional)

---

## 6.3 Idiomatik

Idiomatik ist schwer, aber messbar genug für Fortschritt.

### Metriken
- **Pattern adherence score**: Anteil code regions, die einem canonical pattern matchen.
- **Style stability**: variance in naming/structure across repo.
- **Anti‑pattern rate**: z.B. „broad error enums“, „cap too wide“, „god modules“.

### Human Eval (gezielt)
- 50–200 samples pro release, double‑blind, rubric scoring.

---

## 6.4 Architektur

### Metriken
- Layer compliance: violations count
- Cycles: number of cycles (target 0)
- Dependency entropy: distribution of imports (avoid hubs)
- Contract compliance: requires/provides satisfied

---

## 6.5 Skalierung

### Metriken
- „Repo consistency over 10k+ LOC“
  - rename cohesion, repeated pattern uniformity
  - duplication rate
- „Long horizon success“
  - multi‑file tasks completion without manual patch
- Latency:
  - compile time / iteration time (important for loops)

---

## 6.6 Benchmark Suite: Konkrete Task‑Kategorien

AxiomBench sollte Versioniert sein.

### AxiomBench v1 (Core, 50–200 tasks)
- small functions: parsing/validation
- Result/Option handling
- small module tasks
- basic effects

### AxiomBench v2 (Full features, 200–500 tasks)
- async + caps + effects
- structured concurrency
- layered architecture tasks (3–10 modules)
- spec/contract compliance tasks

### AxiomBench v3 (System scale, 20–50 long tasks)
- build a service with 20–50 modules
- implement library with contracts
- refactor for budgets across repo
- capability propagation in deep stacks

### Scoring System (Beispiel)
Gesamtscore 0–100:

- 25: CSR
- 25: TPR
- 15: ECR (effects/caps)
- 10: MHR (must-handle)
- 10: Architecture (layers/cycles/contracts)
- 10: Budget compliance
- 5: Idiom score

---

# Teil 7: Kosten & Ressourcen

> Zahlen sind bewusst als **Bandbreiten** angegeben. Echte Kosten hängen von:
> - Modellgröße,
> - compute‑Preisen,
> - Ziel‑Qualität,
> - Scope der Stdlib/Ecosystem.

## 7.1 Realistische Phasen‑Schätzung (typisches Startup/Research Team)

| Phase | Dauer | Kosten (Band) | Personal (FTE) | Hauptkosten |
|---|---:|---:|---:|---|
| P1 Tooling Seed (parser/formatter/stubs/bench v0) | 1–3 Mon | $50k–$300k | 2–4 | Engineering |
| P2 Semantic Kernel Compiler | 2–4 Mon | $150k–$600k | 3–6 | Engineering |
| P3 Full Constraints (effects/caps/validation/async) | 6–12 Mon | $500k–$2.5M | 4–8 | Engineering + runtime |
| P4 Data Pipeline + Bench v2/v3 | 3–9 Mon (overlap) | $200k–$2M | 3–6 | Engineering + inference |
| P5 Fine‑Tuning (SFT/DPO) | 3–12 Mon | $300k–$5M | 2–6 | Compute + ML |
| P6 RL Polishing (optional) | 6–18 Mon | $1M–$10M | 3–8 | Compute |

**Total bis „praktisch perfekt“:** typischerweise **18–48 Monate**, **$1M–$20M**, je nach Ambition.

---

## 7.2 Szenario‑Pläne

### A) 1 Person, 1 Jahr, $10k
**Realistisch erreichbar:**
- Parser + formatter (rust/python implementation)
- Kernel prompt + pattern library + stubs
- AxiomScan checks
- AxiomBench v1
- Kein echtes fine‑tuning, kein full compiler

**Outcome:**  
Agent kann brauchbaren Axiom‑Subset schreiben, aber nicht „perfekt“.

**Plan:**
- 3 Monate: parser/formatter + stubs
- 3 Monate: scanner + bench
- 6 Monate: refine patterns + examples + docs, plus minimal typecheck if möglich

---

### B) 5 Personen, 2 Jahre, $500k
**Realistisch erreichbar:**
- MVP compiler (typecheck, must-handle, module/layer)
- effects/caps v1 (nicht komplett)
- AxiomBench v2
- kleines LoRA fine‑tune (wenn compute günstig)

**Outcome:**  
Sehr starke „Axiom competence“, nahe Senior für mid‑scale Projekte, aber edge cases bleiben.

---

### C) Startup mit $5M Funding
**Realistisch erreichbar:**
- full compiler constraints
- robust data pipeline
- high quality SFT + repair traces
- DPO for policies + optional RL
- LSP integration

**Outcome:**  
„Praktisch perfekt“ für in‑distribution tasks; sehr stark auch in großen Repos.

---

# Teil 8: Risiken & Mitigationen

| Risiko | Wahrscheinlichkeit | Impact | Mitigation |
|---|---:|---:|---|
| Spec/Language drift zerstört Dataset | hoch (early) | hoch | version pinning, migration tools, staged stabilization |
| Reward hacking (RL) | mittel | hoch | robust rewards, hidden tests, adversarial eval, use DPO first |
| Overfitting auf Bench | mittel | hoch | hidden splits, rotating tasks, real-world corpora |
| Library/API Halluzination | hoch (early) | mittel | stubs + RAG + strict “no invent” policy |
| Tooling fehlt → loops zu langsam | mittel | hoch | compile speed targets, incremental builds, caching |
| Idiomatik bleibt „Rust-ish“ | mittel | mittel | formatter + gold exemplars + pattern adherence metric |
| Architecture degeneriert (god modules) | mittel | hoch | layer checker + budgets + contract enforcement |
| Security regressions (caps too wide, Raw flow bugs) | mittel | hoch | policy checks + static analysis + training on negative examples |
| Dataset contamination mit Slop | hoch | hoch | strict gates + human sampling + dedup + rubric scoring |
| Team capacity (compiler+ml parallel) | hoch | mittel | phased approach + minimal viable tool loops |

---

# Teil 9: Die ehrliche Einschätzung

## 9.1 Ist „perfekt“ erreichbar?

**Absolut perfekt (mathematisch)**: nein – aus Gründen wie:
- unendliche Problemräume,
- unvollständige Spezifikationen,
- Anforderungen ändern sich,
- manche Eigenschaften sind unentscheidbar.

**Praktisch perfekt (engineering‑perfekt)**: ja, unter Bedingungen:
- Sprache stabil genug,
- Tooling liefert starke Oracles (compile/tests/analysis),
- Datenpipeline produziert große Mengen *hochqualitativer* exemplars,
- Agent nutzt constraints + retrieval + planning.

**Kurz:**  
„Perfekt“ ist erreichbar als: **Senior‑Level Zuverlässigkeit in den meisten realen Situationen**, mit sehr niedriger Fehlerquote.

---

## 9.2 Wahrscheinlichster Outcome

### Best Case
- Axiom toolchain reift schnell.
- Dataset ist sehr sauber.
- Agent erreicht 95%+ first pass compile + 90%+ first pass tests.
- Ecosystem wächst; docs sind AI‑lesbar.

### Realistic Case
- Gute Kompetenz in Axiom Kern + Differentiators.
- Edge cases (complex generics + deep async/caps) brauchen gelegentlich compiler loop.
- Architektur ist meist gut, manchmal menschlicher guidance bedarf.

### Worst Case
- Spec drift + Tooling Verzögerungen.
- Dataset wird noisy.
- Agent bleibt pattern‑bound und erreicht nie echte Systemkompetenz.

---

## 9.3 Was würde ich tun mit unbegrenzten Ressourcen?

1) **Tooling first**: parser/formatter → full compiler constraints → LSP.  
2) **AST‑first generation** als Standard.  
3) **Massive synthetic pipeline** mit robusten tests + hidden evaluation.  
4) **Multi‑stage training** (SFT → DPO → targeted RL).  
5) **Ecosystem**: contracts + docs + reference libs.  
6) **Governance**: versioning, migration, stability guarantees.

---

## 9.4 Was würde ich tun mit limitierten Ressourcen?

### Minimal (1 dev, 1 year)
- Parser + formatter + scan
- stubs + patterns + bench
- RAG‑based agent workflow
- Kein Fine‑Tuning, nur tool loops und constraints

### Medium (5 devs, 2 years)
- MVP compiler + effects/caps v1
- repair‑loop automation
- small LoRA tune on curated dataset (10k–100k)
- AxiomBench v2 + long tasks small

### Funded ($5M)
- full plan (Triad) + data pipeline + training + LSP

---

# Appendix: Konkrete Deliverables & Checklisten

## A. „Axiom Mastery Stack“ – Minimaler Toolchain‑Blueprint

1) `axiom-parse` (parser + AST)
2) `axiom-fmt` (pretty printer canonical)
3) `axiom-scan` (forbidden constructs, must-handle, budgets, imports)
4) `axiom-check` (type/effect/cap checker)
5) `axiom-test` (runner + PBT)
6) `axiom-spec` (export contracts/specs JSON)
7) `axiombench` (tasks + hidden tests + scoring)

## B. „Axiom Competence Levels“ (interne Milestones)

- **L0**: syntax‑safe output (AST/formatter)
- **L1**: must-handle + basic types correct
- **L2**: effects/caps correct for common IO
- **L3**: validation/Raw flow correct
- **L4**: architecture (layers/contracts) consistently correct
- **L5**: long-horizon repo tasks with high success

## C. Minimum Metrics Gate für „Production Agent“

- CSR(first pass) ≥ 90%
- TPR(first pass) ≥ 80%
- FPR ≤ 0.1%
- must-handle violations ≈ 0
- cap minimization within policy (no “*” caps by default)
- layer compliance 100%

---

## Schlusswort

Axiom hat einen entscheidenden Vorteil gegenüber „normalen neuen Sprachen“:  
Es wurde so entworfen, dass Qualität **messbar** und **erzwingbar** ist (Effects, Caps, Budgets, Layers, Raw/Validated, must-handle).

Der Weg zu „perfekt“ ist daher kein Wunder – es ist **Systemengineering**:

- Constraints reduzieren den Suchraum,
- Tooling liefert Oracles,
- Daten + Distillation geben dem Modell die Verteilung,
- Evaluation hält dich ehrlich.

Wenn du diese vier Dinge ernst nimmst, ist „Senior‑Level Axiom AI“ nicht nur möglich – es ist planbar.

