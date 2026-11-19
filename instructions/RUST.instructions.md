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

- The toolchain is pinned in `rust-toolchain.toml`. Building `core`/`alloc` for the device target requires the unstable `build-std` feature, configured in `.cargo/config.toml`.
- `.cargo/config.toml` may declare a default cargo target; commands without an explicit target build for it.
- Run `cargo fmt --check` and `cargo clippy` before submitting.
- The application must compile without errors or warnings, including clippy's. Warnings must not be silenced through compiler flags or `#[allow]` without a clear, documented reason.
- Heap is 8192 bytes by default. Override via `HEAP_SIZE` env var; allowed values: 2048, 4096, 8192, 16384, 24576.
- `cargo build` on the host will not produce a runnable binary: there is no host target. In the Docker environment, run [`cargo ledger build`](https://github.com/LedgerHQ/cargo-ledger) with the Rust cargo target of the device (output: `target/<TARGET>/release/<app-name>`):

```bash
cargo ledger build nanosplus       # one of: nanox | nanosplus | stax | flex | apex_p
```

Enable debug logging with the `debug` feature:

```bash
cargo ledger build nanosplus -- --features debug
```
