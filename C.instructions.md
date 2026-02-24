---
description: "Ledger embedded C application development rules and build workflow"
applyTo: "**/*"
---

# Ledger Embedded C Rules

- Ledger C embedded applications use the Ledger SDK, which has its own set of APIs and conventions. Ensure that the code follows the SDK guidelines and makes efficient use of its features. The SDK code is available at https://github.com/LedgerHQ/ledger-secure-sdk/
- The SDK exposes a deprecated API for custom exceptions. Ensure the PR does not introduce new THROW calls.
- Usage of dynamic allocation is impossible and forbidden.
