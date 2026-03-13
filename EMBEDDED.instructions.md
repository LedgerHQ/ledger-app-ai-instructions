---
description: "Ledger embedded platform constraints shared by C and Rust applications"
applyTo: "**/*"
---

# Ledger Embedded Platform Rules

These rules apply to all embedded code (C and Rust) running on Ledger devices.

## Application Privileges

- Application flags must abide by the principle of least privilege.
- Derivation paths must be restricted to coin-specific BIP32 prefixes.
- Curves usage must be restricted to only those required by the application.

## User Interface and Clear Signing

- The UI is the fundamental part of the embedded application, NOT a cosmetic side. Ensure all sensitive operations (signing, public key export) are preceded by an explicit user validation screen. Flag any "blind signing" patterns or flows where the screen doesn't accurately represent the buffer being signed.
- If blind signing is implemented, it must be disabled by default behind a settings flag.
- Critical and important information must be clear signed using a user-friendly format. The user must not be confused or tricked by the application workflow or displayed information.

## Memory and Runtime

- The RAM is limited to around 24 kilobytes. Ensure that the code is optimized for low memory usage and does not contain unnecessary allocations or unnecessarily large data structures.
- Remember that the RAM is reset on every power cycle.

## Secrets and Cryptography

- Ensure sensitive data such as private keys are explicitly cleaned from memory as soon as possible after usage.
- Secrets derived from the seed must not be stored, exported, or shown to the user.
- Cryptographic calls must be made through the SDK's functions, not implemented in application code. Ensure that all cryptographic operations are performed using these functions and that they are used correctly to maintain security and performance.
- Structure integrity of messages must be verified before signature. Never allow signing of attacker-controlled messages.

## APDU Handling

- APDUs are the sole entry point of the application. Ensure the code treats the incoming APDUs as untrusted input and implements proper validation and error handling to prevent potential security vulnerabilities. Look for robust parsing of APDU commands, validation of input data, and appropriate responses to invalid or malicious requests.
