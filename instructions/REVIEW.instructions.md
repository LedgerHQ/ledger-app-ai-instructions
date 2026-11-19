# Ledger Code Review

Apply these rules when reviewing code.

## Severity Levels

Classify each reported issue with a severity:
- **🟥 CRITICAL** — Security vulnerability, data corruption, crash, or build failure.
- **🟧 HIGH** — Logic error, missing test coverage for a key path, or code/doc/test desynchronization.
- **🟨 WARNING** — Code style, naming, minor documentation gaps.
- **ℹ️ INFO** — Observation or suggestion

## Coherence Check

Verify that the three components tell the same story:
- **Code ↔ Documentation:** Confirm that the C or Rust code implements exactly the APDU commands (INS, P1, P2) described in documentation. CLA/INS codes must match their definitions in the code.
- **Code ↔ Tests:** Confirm that the Python tests cover the logic actually implemented in the embedded code, including error paths.
- **Tests ↔ Documentation:** Confirm that the tests respect the protocol defined in the documentation.

## C and Rust code review guidelines

Pay attention to the following code quality items that indirectly impact security:
- **Magic numbers**: Prefer enums, macros, or `sizeof`-based expressions over raw numeric literals.
- **State management**: Application state must be properly initialized, cleared/erased on finalization or error, and multi-step operations must enforce correct ordering.
- **Complexity**: Flag functions with high cyclomatic complexity or excessive length — they are harder to audit and more error-prone.

## Python test code review guidelines

Ensure new features are covered by functional tests, checking the valid expected behaviors, edge cases, and potential malicious inputs.
