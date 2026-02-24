---
description: "Ledger application code review checklist — security, coherence, and quality gate"
applyTo: "**/*"
excludeAgent: "coding-agent"
---

# Ledger Code Review Rules

Security-focused review checklist for Ledger embedded applications. These instructions
apply during code review only (excluded from the coding agent).

## Severity Scale

Classify every finding with a severity level:

| Level | Meaning | Blocks merge? |
|---|---|---|
| 🟥 CRITICAL | Security vulnerability, data corruption, crash, build failure | Yes |
| 🟧 HIGH | Logic error, missing coverage for key path, code/doc/test desync | Yes |
| 🟨 WARNING | Code style, naming, minor documentation gaps | No |
| ℹ️ INFO | Observation or suggestion | No |

A **FAIL** verdict requires at least one 🟥 CRITICAL or 🟧 HIGH finding.
🟨 WARNING and ℹ️ INFO alone result in **PASS with observations**.

## 1. Coherence — The Trinity Check

Verify that code, documentation, and tests tell the **same story**:

- **Code ↔ Documentation:** The INS codes, P1/P2 values, and payload formats in
  `doc/APDU.md` must match the `#define` macros and handler logic in `src/` exactly.
- **Code ↔ Tests:** The Python tests must cover the logic actually implemented in the
  embedded code — including error paths and edge cases.
- **Tests ↔ Documentation:** The tests must respect the protocol described in the
  documentation. A test that sends an APDU not listed in `doc/APDU.md` (or vice versa)
  is a desynchronization.

Any desynchronization between these three is at least 🟧 HIGH.

## 2. Security Audit

- **Signing without user approval** — Any handler that returns a cryptographic signature
  without prior on-screen user validation is 🟥 CRITICAL. No exceptions.
- **Missing key wiping** — Private keys or seeds not cleared with `explicit_bzero`
  (or Rust equivalent) immediately after use, including on error paths: 🟥 CRITICAL.
- **Buffer overflow risk** — `memcpy` / `memmove` without prior length validation of
  the source data: 🟥 CRITICAL.
- **APDU input not validated** — Missing length checks on incoming data fields before
  parsing: 🟧 HIGH.
- **Dynamic allocation** — Use of `malloc` / `free` or heap allocators in embedded
  code: 🟧 HIGH.
- **New `THROW` calls** — Introduction of the deprecated exception mechanism: 🟧 HIGH.
- **Custom crypto** — Any manual implementation of standard algorithms instead of
  using `cx_*` / `os_*` SDK functions: 🟥 CRITICAL.

## 3. Quality & Testing Audit

- **Magic hex strings** — Raw `bytes.fromhex(...)` for complex payloads instead of
  `struct.pack` with named variables: 🟧 HIGH.
- **Missing Golden Snapshots** — Signing or key-export flows without UI verification
  via `navigator.navigate_and_compare()`: 🟧 HIGH.
- **Incomplete device matrix** — Tests not covering all devices listed in
  `[app].devices` in `ledger_app.toml`: 🟧 HIGH.
- **Missing error-path tests** — No tests for invalid inputs, wrong P1/P2, or user
  rejection (`0x6985`): 🟧 HIGH.
- **Test readability** — Unclear variable names, missing comments on non-obvious
  assertions: 🟨 WARNING.

## 4. Documentation Audit

- **INS code mismatch** — `doc/APDU.md` lists different INS/CLA/P1/P2 values than
  the C/Rust header files: 🟧 HIGH.
- **Missing Mermaid diagrams** — Complex flows (signing, multi-step APDU sequences)
  without visual documentation: 🟨 WARNING.
- **Blind signing not flagged** — If a flow allows signing without full on-screen
  display of the payload, the documentation must explicitly warn about it: 🟧 HIGH.

## 5. Embedded Best Practices

- **Floating point usage** — `float` or `double` in embedded code: 🟧 HIGH.
- **Recursion** — Recursive calls risking stack overflow: 🟧 HIGH.
- **Excessive stack usage** — Large local arrays or structs on the stack instead of
  static globals: 🟨 WARNING.
- **Code size** — Unnecessary code duplication inflating flash usage: 🟨 WARNING.

## Verdict Format

End every review with a clear verdict:

### FAIL Example
> **Verdict: ❌ FAIL**
>
> **Blockers:**
> 1. 🟥 CRITICAL — `sign_tx_handler` returns signature without user approval screen.
> 2. 🟧 HIGH — `doc/APDU.md` lists INS `0x04` but code defines `INS_SIGN_TX 0x05`.
> 3. 🟨 WARNING — Magic hex in `test_sign.py` line 42.

### PASS Example
> **Verdict: ✅ READY TO MERGE**
>
> Code is secure, tests are comprehensive, documentation matches implementation.
> ℹ️ INFO — Consider adding a Mermaid diagram for the multi-step message signing flow.
