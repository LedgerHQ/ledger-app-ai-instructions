---
description: 'Ledger embedded Rust application development rules and build workflow'
applyTo: '**/*'
---

# Ledger Embedded Rust Rules

- Prefer borrowing and zero-copy operations to avoid unnecessary allocations.
- Dependencies must be checked with `cargo audit` to ensure no known impactful vulnerabilities are present.
- Avoid `unsafe` unless required and fully documented. All `unsafe` blocks must be justified in a comment.

## Error Handling

- A panic, including from `unwrap()` or `expect()`, exits the application: panic only on broken invariants, never on invalid input.
- Return errors to the host as APDU status words, such as an `AppSW` enum converted into a `Reply`, not as strings.

## Documentation

- Write item-level comments as `///` doc comments.
- Give every `unsafe fn` a `# Safety` section, and every function that can panic a `# Panics` section.

## How to build

### Configuration

- Supported devices are listed in `[app].devices` inside `ledger_app.toml`.
- Rust apps are built per device with [`cargo-ledger`](https://github.com/LedgerHQ/cargo-ledger). The target device is selected by its cargo target name:

  | `ledger_app.toml` device | cargo target |
  | :--- | :--- |
  | `nanos+` | `nanosplus` |
  | `nanox` | `nanox` |
  | `stax` | `stax` |
  | `flex` | `flex` |
  | `apex_p` | `apex_p` |

- The toolchain is pinned in `rust-toolchain.toml`. Building `core`/`alloc` for the device target requires the unstable `build-std` feature, configured in `.cargo/config.toml`.
- `.cargo/config.toml` may declare a default cargo target; commands without an explicit target build for it.
- The application must compile without errors or warnings. Run `cargo fmt --check` and `cargo clippy` before submitting; warnings must not be silenced without a documented reason.
- Heap is 8192 bytes by default. Override via `HEAP_SIZE` env var; allowed values: 2048, 4096, 8192, 16384, 24576.

### Docker Environment

- **Image discovery:** Run `docker images | grep ledger` before any `docker run`. Do NOT assume a hardcoded image name.
- **Common image:** `ghcr.io/ledgerhq/ledger-app-builder/ledger-app-dev-tools:latest` (includes the Rust toolchain, `cargo-ledger`, Speculos, enforcer). `cargo build` on the host will not produce a runnable binary — there is no host target.
- **Volume mount:** Mount the project root to `/app` inside the container.
- **Host OS adaptation:** Adapt Docker commands to the host OS (e.g., shell syntax for current directory, variable escaping).

### Build command

Run `cargo ledger build <TARGET>`, replacing `<TARGET>` with the cargo target for the device (output: `target/<TARGET>/release/<app-name>`):

```bash
cargo ledger build nanosplus       # one of: nanox | nanosplus | stax | flex | apex_p
```

Enable debug logging with the `debug` feature:

```bash
cargo ledger build nanosplus -- --features debug
```
