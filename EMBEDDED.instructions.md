---
description: "Ledger embedded platform constraints shared by C and Rust applications"
applyTo: "**/*"
---

# Ledger Embedded Platform Rules

These rules apply to all embedded code (C and Rust) running on Ledger devices.
They reflect hardware constraints, security requirements, and SDK conventions that
the compiler and linters cannot enforce.

## Hardware Constraints

- **RAM is scarce.** Nano X has ~24 KB, Nano S+/Stax/Flex/Apex have ~40 KB. Design
  data structures accordingly. Avoid large stack-allocated buffers.
- **No floating point.** Never use `float` or `double`. Use fixed-point arithmetic
  or SDK big-integer functions (`cx_math_*`).
- **No recursion.** Stack space is extremely limited — prefer iterative algorithms
  to avoid stack overflow.
- **Watchdog.** Long-running operations (crypto, tight loops) can trigger a watchdog
  reset. Keep individual operations short.
- **Static allocation only.** Do not use dynamic memory allocation (`malloc`/`free`
  in C, heap allocators in Rust). Use statically-sized global or stack buffers.

## Security — Non-Negotiable Rules

- **User approval before signing.** Any handler that produces a cryptographic signature
  **must** display the transaction details on-screen and obtain explicit user approval
  **before** performing the signing operation. Returning a signature without prior
  on-screen validation is a critical security violation ("blind signing").
- **Wipe sensitive data immediately.** Private keys, seeds, and any derived secret
  must be cleared from memory with `explicit_bzero` (C) or equivalent zeroing (Rust)
  as soon as they are no longer needed — including on error paths.
- **No custom cryptography.** Never implement standard algorithms (SHA, AES, HMAC,
  ECC, BIP-32, etc.) manually. Use the SDK's hardware-accelerated `cx_*` (crypto)
  and `os_*` (system) functions. They provide side-channel protection that software
  implementations cannot match.
- **Secrets must not be exported or shown.** Private keys and seeds derived from the
  device seed must never be stored persistently, exported to the host, or displayed
  on screen. Only public material (public keys, addresses) may leave the device.
- **Verify message structure before signing.** The structural integrity of a message or
  transaction must be fully validated before performing any signing operation. Never
  sign attacker-controlled raw bytes.
- **User-supplied messages must use a prefix.** If arbitrary user-supplied data is to
  be signed, it must be prefixed with a domain-specific tag (e.g., `\x19Ethereum
  Signed Message:\n`) to prevent replaying signatures as transactions.
- **Blind signing must be disabled by default.** If blind signing is implemented, it
  must be opt-in (disabled in settings by default) and must not apply to basic coin
  transfers, which must always be clear-signed.
- **The client must not freely manipulate keypairs.** The host application must not be
  able to request arbitrary key derivations or signing operations without the user
  being fully informed and approving each one.
- **Authenticated encryption for cached data.** Any sensitive data cached on the client
  side must be protected with an authenticated encryption scheme.
- **State management.** Application state must be properly initialized before use and
  explicitly cleared (with `explicit_bzero` or equivalent) when finalized or when an
  error occurs. In multi-step operations (chunked APDU flows, signing workflows), the
  correct ordering of steps must be enforced — out-of-order messages must be rejected.

## APDU Handling

- **Treat all APDU input as untrusted.** Validate the length and content of every
  incoming data field before processing. Check `dataLength` against expected sizes
  before any copy or parse operation.
- **Return proper status words.** Use standard SW codes (`0x9000` for success,
  `0x6985` for user rejection, `0x6A80` for invalid data, etc.). Never silently
  swallow errors.

## UI Requirements

- **The UI is a security boundary**, not a cosmetic feature. Every sensitive operation
  (signing, public key export) must show the user exactly what is being approved.
- **Screen content must match the signed payload.** If the display shows different
  data than what is actually signed, it is a critical vulnerability.

## Device Configuration

- **`ledger_app.toml` is the source of truth** for supported devices, test directories,
  and build configuration. Read it — do not hardcode device lists or paths.
- Supported devices (as of this writing): Nano S+, Nano X, Stax, Flex, Apex.
  The legacy Nano S is excluded.
