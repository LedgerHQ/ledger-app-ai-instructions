---
description: "Ledger application test writing rules using Ragger, Pytest, and Speculos"
applyTo: "**/*"
---

# Ledger Test Writing Rules (Python)

Python is used for testing Ledger device applications, it is not part of the embedded application.

## Testing Framework and Tools

The tests are written using the Ragger framework, which provides a bridge between Pytest and the Speculos emulator through pytest fixtures.

- **Framework:** [Ragger](https://github.com/LedgerHQ/ragger) (Python + Pytest)
- **Emulator:** [Speculos](https://github.com/LedgerHQ/speculos)

## Coverage Requirements

Every tested feature must include:
- Happy path
- Error paths (invalid inputs, edge cases, malicious inputs)
- User rejection (where applicable)
- Edge cases (empty data, max-length data, boundary values)
