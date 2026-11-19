---
description: "Ledger application build and test environment: devices and Docker image"
applyTo: "**/*"
---

# Ledger Build and Test Environment

- Supported devices are listed in `[app].devices` of `ledger_app.toml`. Each one maps to a C SDK variable, a Rust cargo target and a pytest `--device` value:

  | `ledger_app.toml` | C SDK variable | Rust cargo target | pytest `--device` |
  | :--- | :--- | :--- | :--- |
  | `nanos+` | `$NANOSP_SDK` | `nanosplus` | `nanosp` |
  | `nanox` | `$NANOX_SDK` | `nanox` | `nanox` |
  | `stax` | `$STAX_SDK` | `stax` | `stax` |
  | `flex` | `$FLEX_SDK` | `flex` | `flex` |
  | `apex_p` | `$APEX_P_SDK` | `apex_p` | `apex_p` |

- Build and test in Docker:
  - **Image discovery:** Run `docker images | grep ledger` before any `docker run`. Do NOT assume a hardcoded image name.
  - **Common image:** `ghcr.io/ledgerhq/ledger-app-builder/ledger-app-dev-tools:latest` (includes the C and Rust toolchains, `cargo-ledger`, Speculos, the enforcer and a Python venv).
  - **Volume mount:** Mount the project root to `/app` inside the container.
  - **Host OS adaptation:** Adapt Docker commands to the host OS (e.g., shell syntax for current directory, variable escaping).
