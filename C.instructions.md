---
description: "Ledger embedded C application development rules and build workflow"
applyTo: "**/*"
---

# Ledger Embedded C Rules

- Ledger C embedded applications use the Ledger SDK, which has its own set of APIs and conventions. Ensure that the code follows the SDK guidelines and makes efficient use of its features. The SDK code is available at https://github.com/LedgerHQ/ledger-secure-sdk/
- The language standard is C (ISO C11), compiled with the Clang/LLVM toolchain.
- The project must use the Makefile template from the boilerplate application and the Makefile.standard_app from the ledger-secure-sdk.
- The application must compile without errors or warnings. Compilation warnings must not be silenced through compiler flags or `#pragma` directives without a clear, documented reason.
- The SDK exposes a deprecated API for custom exceptions. Ensure the PR does not introduce new THROW calls. Deprecated cryptographic functions that can throw exceptions must not be used; prefer the non-throwing SDK alternatives.
- Usage of dynamic allocation is impossible and forbidden. Prefer static global buffers over heavy stack usage.
- Never use `float` or `double`. Use fixed-point arithmetic or SDK BigInt functions (`cx_math_...`) instead.
- Avoid recursion to prevent stack overflow on the constrained stack.
- Prefer `memmove`/`memset` over manual byte-by-byte loops.
- Use `strlcpy` or explicit bounds checking when manipulating strings. Always validate `dataLength` against expected sizes before any memory copy.
