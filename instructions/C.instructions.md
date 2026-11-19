---
description: "Ledger embedded C application development rules and build workflow"
applyTo: "**/*"
---

# Ledger Embedded C Rules

- Use the Ledger SDK APIs and conventions. The SDK code is available at https://github.com/LedgerHQ/ledger-secure-sdk/
- The language standard is C (ISO C99), compiled with the Clang/LLVM toolchain.
- The project must use the Makefile template from the boilerplate application and the Makefile.standard_app from the ledger-secure-sdk.
- Do not add `THROW` calls: the SDK's exception API is deprecated. Do not use deprecated cryptographic functions that can throw; use their non-throwing SDK alternatives (`*_no_throw`).
- Usage of dynamic allocation is impossible and forbidden. Prefer static global buffers over heavy stack usage.
- Prefer `memmove`/`memset` over manual byte-by-byte loops.
- Use `strlcpy` or explicit bounds checking when manipulating strings. Always validate `dataLength` against expected sizes before any memory copy.

## How to build

- Run `BOLOS_SDK=<SDK_VAR> make -j` in the Docker environment, with `<SDK_VAR>` the C SDK variable of the target device.
- The application must compile without errors or warnings. Warnings must not be silenced through compiler flags or `#pragma` without a clear, documented reason.
