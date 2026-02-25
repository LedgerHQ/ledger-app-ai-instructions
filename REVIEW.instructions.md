---
description: "Ledger application code review checklist — security, coherence, and quality gate"
applyTo: "**/*"
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
- **Secrets exported or shown** — Private keys or seeds are returned to the host,
  stored persistently beyond the operation, or displayed on screen: 🟥 CRITICAL.
- **Unsigned message without prefix** — User-supplied data signed without a
  domain-specific prefix (e.g., `\x19Ethereum Signed Message:\n`), enabling
  cross-context replay: 🟥 CRITICAL.
- **Message signed without structural validation** — A signing handler that does not
  fully verify the structure/integrity of the payload before signing: 🟥 CRITICAL.
- **Integer overflow / underflow** — Arithmetic on lengths, indices, or counters
  without overflow checks (e.g., loop counter type mismatch with bound type): 🟥 CRITICAL.
- **Format string vulnerability** — User-controlled data passed to `printf`-family
  functions as the format string: 🟥 CRITICAL.
- **Use of dangerous functions** — Calls to `strcpy`, `strlen` without bounds,
  `gets`, `sprintf` (unbounded), or similar functions that cannot guarantee safety: 🟭 HIGH.
- **Memory safety** — Memory leaks, use-after-free, or returning a
  pointer to stack-allocated memory: 🟥 CRITICAL.
- **Uninitialized memory returned** — Stack or heap memory that could contain
  sensitive data (keys, seeds) returned or sent to the host without zeroing: 🟥 CRITICAL.
- **Unchecked return values** — SDK or syscall return codes ignored, meaning errors
  silently lead to invalid or partial states: 🟧 HIGH.
- **State management** — Application state not initialized before use, not cleared
  on error/completion, or out-of-order steps accepted in a multi-step APDU flow: 🟧 HIGH.
- **Cryptographic oracle** — An attacker can learn secret-dependent information by
  observing the app's behaviour (timing, error messages, partial outputs): 🟥 CRITICAL.
- **Security restriction bypass** — The autolock, PIN entry, or any other OS access
  control can be circumvented through the application: 🟥 CRITICAL.
- **Keypair freely manipulated** — The host can request arbitrary derivation or signing
  without per-operation user approval: 🟥 CRITICAL.
- **No authenticated encryption for cached data** — Sensitive data cached on the
  client is not protected with an authenticated encryption scheme: 🟧 HIGH.

## 3. Quality & Testing Audit

- **Magic hex strings** — Raw `bytes.fromhex(...)` for complex payloads instead of
  `struct.pack` with named variables: 🟧 HIGH.
- **Missing Golden Snapshots** — Signing or key-export flows without UI verification
  via `navigator.navigate_and_compare()`: 🟧 HIGH.
- **Incomplete device matrix** — Tests not covering all devices listed in
  `[app].devices` in `ledger_app.toml`: 🟧 HIGH.
- **Missing error-path tests** — No tests for invalid inputs, wrong P1/P2, or user
  rejection: 🟧 HIGH.
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
