---
description: "Ledger embedded C application development rules and build workflow"
applyTo: "**/*"
---

# Ledger Embedded C Rules

C-specific rules for Ledger device applications. For cross-language embedded
constraints (memory, security, UI), see `EMBEDDED.instructions.md`.

## Language & Toolchain

- **Standard:** ISO C11, compiled with Clang/LLVM.
- **SDK:** Ledger C SDK ([ledger-secure-sdk](https://github.com/LedgerHQ/ledger-secure-sdk)).
  Always check SDK headers (`cx.h`, `os.h`) for existing functions before writing logic.
- **Deprecated API:** The SDK exposes a legacy exception mechanism via `THROW`. Do not
  introduce new `THROW` calls — use the modern error-return pattern instead.

## Project Structure

The application follows the [official C Boilerplate](https://github.com/LedgerHQ/app-boilerplate)
layout:

| Directory | Purpose |
|---|---|
| `src/` | Application source code |
| `icons/`, `glyphs/` | UI assets (icons go here, do not create new asset directories) |
| `doc/` | Documentation (`APDU.md`, `DESIGN.md`) |
| `tests/` | Test suites (exact path defined in `ledger_app.toml`) |

- **Makefile:** You may modify `SOURCE_FILES` or compiler flags, but do not alter the
  core include logic (`Makefile.defines`, `Makefile.rules`).
- **Naming:** The word "boilerplate" must not appear in final source code, macros, types,
  or user-facing strings. Replace generic names with the specific application name
  (e.g., `APP_SOLANA`). Exception: skip this rule when working on `app-boilerplate` itself.

## Memory & Buffer Safety

- `malloc` / `free` are **forbidden**. Use SDK allocation functions only if dynamic
  allocation is strictly necessary.
- Prefer static global buffers (e.g., `G_app` pattern) over heavy stack usage.
- Always validate `dataLength < bufferSize` before any `memcpy` / `memmove`.
- Use `memmove`, `memcpy`, `memset` from the SDK — not manual byte-by-byte loops.
- Use `strlcpy` or explicit bounds checking for string operations.

## Build Workflow

The build uses Docker with the `ghcr.io/ledgerhq/ledger-app-builder/ledger-app-dev-tools`
image (verify with `docker images | grep ledger` before running).

### Target → SDK Variable Mapping

Read `[app].devices` from `ledger_app.toml` and build only listed targets:

| Device in `ledger_app.toml` | Docker SDK variable |
|---|---|
| `nanos+` | `$NANOSP_SDK` |
| `nanox` | `$NANOX_SDK` |
| `stax` | `$STAX_SDK` |
| `flex` | `$FLEX_SDK` |
| `apex_p` | `$APEX_P_SDK` |

### Build Steps

1. **Build all targets:** Mount the project directory into Docker, run `make clean`,
   then chain `BOLOS_SDK=<SDK_VAR> make -j` for each target with `&&`.
2. **Run enforcer** (separate command, 900s+ timeout): Execute `/opt/enforcer.sh`
   inside Docker. This runs `scan-build` for every target and takes 10–15 minutes.
3. **Iterate:** If the build or enforcer fails, read the terminal output, fix the code,
   and re-execute. Do not stop until both pass.

On Windows/PowerShell, use `${PWD}` instead of `$(pwd)` for volume mounts.
