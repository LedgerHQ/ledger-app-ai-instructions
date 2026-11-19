# Ledger Embedded Platform Rules

These rules apply to all embedded code (C and Rust) running on Ledger devices.

## Application Privileges

- Application flags must abide by the principle of least privilege.
- Derivation paths must be restricted to the BIP32 prefixes the application needs, each justified by a protocol or standard it implements, such as its coin type. A broader prefix, such as a wildcard purpose, is acceptable only if the application checks each path when using it.
- Curves usage must be restricted to only those required by the application.

## User Interface and Clear Signing

- A signature must only be valid for what the user authorized:
  - By default, the user reviews each message on the device, and the screens must accurately represent what is signed. Parse and validate the whole message, and derive what is signed (the hash, or the message itself for schemes such as Ed25519) on the device from the reviewed bytes.
  - Signing what the user cannot review, such as a host-supplied hash, requires an explicit opt-in instead: the blind-signing setting, or an authorization the user gives as the protocol requires, such as an OpenPGP PIN, a FIDO user-presence check, a baking setup or a swap confirmed in the Exchange app. A request from the host alone is never an authorization.
  - Always validate what the device can check, and keep the protocol's domain separation.
- If blind signing is implemented, it must be disabled by default behind a settings flag.
- The UI is a security feature, NOT a cosmetic side: critical and important information must be clear signed using a user-friendly format, and the user must not be confused or tricked by the application workflow or displayed information.
- Public keys on the application's standard paths may be exported without confirmation, but addresses the user relies on, such as receive addresses, must be displayable on the device for verification.

## Security and Availability

- Security strictly overrides availability; the application MUST always "fail closed."
- When encountering an unexpected issue, the application must refuse to proceed with the standard flow and return an error.
- It is NEVER acceptable to silence a security issue to preserve availability. Even a crash is preferable to executing in an unverified or compromised state. This BANS eager patterns such as dubious fallbacks, default values, clamping, etc.
- In `switch` and `match`, give every valid value its own case; reserve `default:` and `_ =>` for errors and unexpected values.

## Memory and Runtime

- The application RAM, shared by globals, SDK buffers and the stack, is only a few tens of KB; the exact size is the `SRAM` length in the SDK's linker script, `target/<device>/script.ld`. Keep memory usage low: no unnecessary allocations or oversized data structures.
- RAM is reset on every power cycle: keep persistent state in NVM (`nvm_write` in C, `NVMData` in Rust), and write it only when the value changes, since flash wears out.
- Avoid recursion to prevent stack overflow on the constrained stack.
- Never use floating-point types (`float`/`double` in C, `f32`/`f64` in Rust). Use fixed-point arithmetic or SDK BigInt functions (`cx_math_...`) instead. This does not prevent you from displaying float-like strings.

## Secrets and Cryptography

- Ensure sensitive data such as private keys are explicitly cleaned from memory as soon as possible after usage.
- Secrets derived from the seed must not be stored, exported, or shown to the user.
- Cryptographic calls must be made through the SDK's functions, not implemented in application code.

## APDU Handling

- APDUs are the sole entry point of the application: treat them as untrusted input, validate every field, and answer invalid or malicious requests with an error status word.
- Check arithmetic on lengths, offsets and amounts for overflow and underflow before using the result; in Rust, use `checked_*` operations.

## Comments

- Comments are TIMELESS and must remain true and useful years later; do not use them to describe how things **were**, nor to forward information to the AI caller.
- Avoid comments that describe what the code is doing, unless it's doing something non-obvious; explain intent and rationale instead. Basic code understanding should come from proper variable and function names.
- Comments should stay concise and dense in information; multiline comments should be rare.
- Comments should respect separation of concerns; do not bring outside knowledge where it does not belong.
