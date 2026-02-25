---
description: "Ledger-specific Rust rules and custom test harness for embedded targets"
applyTo: "**/*"
---

# Ledger Embedded Rust Rules

Ledger-specific deviations from standard Rust development. For cross-language embedded
constraints (memory, security, UI), see `EMBEDDED.instructions.md`.

Standard Rust conventions (formatting with `rustfmt`, linting with `cargo clippy`,
idiomatic error handling with `Result<T, E>`, ownership and borrowing best practices)
are expected but not repeated here — the tooling enforces them.

## `no_std` Environment

This codebase targets ARM Cortex-M devices with no operating system. Key implications:

- **No standard library.** The code is `#![no_std]`. You cannot use `std::` types —
  use `core::` and `alloc::` equivalents where available.
- **No heap by default.** Avoid `Box`, `Vec`, `String`, and other heap-allocated types
  unless a global allocator is explicitly configured for the target.
- **No `println!` / `eprintln!`.** For debug output, use semihosting (only available
  under the Speculos emulator with the `speculos` + `debug` cargo features enabled).

## Patterns to Prefer

- Prefer `&str` over `String`, fixed-size arrays over `Vec`.
- Use `Option<T>` and `Result<T, E>` for all fallible operations — never `panic!`
  or `unwrap()` in application code.
- Prefer borrowing and zero-copy parsing to minimize RAM usage.
- Use enums for state machines and protocol variants instead of integer flags.

## Custom Test Harness

Standard `cargo test` does **not** work — tests must run inside the Speculos emulator
against a device target.

### Writing Tests

Use `testmacro::test_item` as a drop-in replacement for `#[test]`:

```rust
#[cfg(test)]
mod tests {
    use crate::testing::TestType;
    use testmacro::test_item as test;

    #[test]
    fn test_example() {
        // Return Ok(()) on success
    }
}
```

- Use `assert_eq_err!` (from `testing.rs`) instead of `assert_eq!`. It returns
  `Err(())` instead of panicking, so the harness reports all failures rather than
  aborting on the first one.
- Do **not** create a `tests/` directory for integration tests. Examples in
  `ledger_device_sdk/examples/` serve as integration-level verification on Speculos.

### Running Tests

```bash
# Unit tests — requires Speculos on PATH
cargo test --target nanosplus --features speculos,debug --tests

# Run an example on Speculos
cargo run --example nbgl_home_and_settings --target stax --release \
  --config ledger_device_sdk/examples/config.toml
```

Always enable the `speculos` and `debug` cargo features so that emulator-specific
code paths and semihosting output are active.
