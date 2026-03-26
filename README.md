We build tools and frameworks that measure how well a test suite actually catches bugs, not just how much code it touches. Our work draws on mutation testing, fault simulation, and AI-assisted test generation to close the gap between "tests pass" and "tests work."


---

## Featured Projects

### [MetaTest](https://github.com/metatest/metatest-rest-java) — REST API Fault Injection Framework

MetaTest validates REST API test suites by systematically injecting faults into HTTP responses and measuring how many the tests catch.

It uses **AspectJ bytecode weaving** to intercept live HTTP responses at runtime — no test code changes required. For each field in each response, it applies mutations (null values, missing fields, empty collections, empty strings, invalid types) and re-runs the corresponding test. If the test still passes after the mutation, the fault "escaped" — a direct signal that an assertion is missing or too weak.

**Key capabilities:**
- Transparent interception via compile-time and load-time AspectJ weaving
- Thread-isolated test contexts for safe parallel execution
- **Invariants DSL** — define business rules in YAML that every response must satisfy, independent of test assertions
- Conditional invariants with cross-field comparison support (`if status == "FILLED" then quantity > 0`)
- Array field validation across all collection items
- HTML and JSON reports with per-endpoint fault detection rates, escaped fault details, and coverage gaps
- `stop_on_first_catch` mode for fast smoke runs in CI

The output is a **quantified defect detection score** per endpoint and per test — a metric that is directly interpretable and actionable.

---

### [Antigen](https://github.com/metatest/antigen) — AI-Powered Test Generation with Built-in Validation

Antigen generates JUnit 5 integration tests from OpenAPI specifications using Claude Code, then uses MetaTest to validate that the generated tests actually catch bugs before they're accepted.

The pipeline runs in four phases:

```
Generate → Build → Test → MetaTest
    ↑                          |
    └──── feedback loop ───────┘
```

If MetaTest reports escaped faults, Antigen feeds the fault report back to the LLM with targeted instructions to strengthen specific assertions. This continues until the fault detection threshold is met or the retry limit is reached.

Available as both a **standalone CLI tool** and a **Gradle plugin** (`antigen-lib`) for embedding directly into build pipelines.

**Key capabilities:**
- Zero-code setup — provide an OpenAPI spec and a config file
- Generates RestAssured + AssertJ + JUnit 5 tests
- Iterative LLM refinement loop driven by fault detection feedback
- Configurable quality thresholds (e.g., require ≥ 90% fault detection before accepting tests)
- Custom prompt templates and test requirements
- Cross-platform (Windows, Linux, macOS)

**[antigen-example](https://github.com/metatest/antigen-example)** — a complete working example using a trading/order execution API, including generated tests, MetaTest reports, and a GitHub Actions workflow.

---

## Research Direction

We're actively investigating how to **quantify test quality and defect detection capability** with more rigour and breadth.

Current threads:

**Fault simulation coverage** — MetaTest currently covers structural mutations on HTTP responses (field-level). We're extending this toward semantic fault types: boundary violations, ordering assumptions, timing-sensitive state, and protocol-level anomalies.

**Mutation testing integration** — Mapping source-level mutation operators (PIT/PITest) to the API response mutations MetaTest applies, building a unified defect detection score that spans both unit and integration layers.

**Property-based testing** — Rather than fixed example inputs, we're exploring shrinkable generators that derive test cases from OpenAPI schema constraints, with properties defined from the invariants DSL. This shifts assertion strength measurement from "does this test catch mutation X" to "what is the minimum input space this property covers."

**Test quality as a first-class metric** — The goal is a number (or a vector of numbers) that gives a team actionable signal about where their test suite is blind — broken down by endpoint, by field, by fault class — and a clear path to improving it.

---

## How the Ecosystem Fits Together

```
OpenAPI Spec
     │
     ▼
  Antigen ──── Claude Code ────► Generated Tests
     │                                  │
     │                          ┌───────▼────────┐
     │                          │   Build + Run   │
     │                          └───────┬────────┘
     │                                  │
     └──────── MetaTest ◄───────────────┘
                  │
                  ▼
         Fault Detection Report
         (escaped faults, scores,
          invariant violations,
          coverage gaps)
```

MetaTest can be used standalone against any existing JUnit 5 + RestAssured test suite — it does not require Antigen. Antigen uses MetaTest as its quality gate, but can also be configured to skip validation.

---

## Getting Started

- **MetaTest** — see [`metatest-rest-java`](https://github.com/metatest/metatest-rest-java) and the example project [`metatest-rest-java-example`](https://github.com/metatest/metatest-rest-java-example)
- **Antigen** — see [`antigen`](https://github.com/metatest/antigen) (CLI) or [`antigen-lib`](https://github.com/metatest/antigen-lib) (Gradle plugin), and [`antigen-example`](https://github.com/metatest/antigen-example) for a working reference

All projects require Java 17+ and Gradle 7.3+.

---

## Status

These projects are under active development. APIs and configuration formats may change between releases. Issues and feedback are welcome.
