---
description: "Ledger application test writing rules using Ragger, Pytest, and Speculos"
applyTo: "**/*"
---

# Ledger Test Writing Rules (Python)

Rules for writing functional tests for Ledger embedded applications. Python is used
**only** for testing — it is not part of the embedded application.

## Stack

- **Framework:** [Ragger](https://github.com/LedgerHQ/ragger) (Python + Pytest)
- **Emulator:** [Speculos](https://github.com/LedgerHQ/speculos)
- **Backend:** `SpeculosBackend` to load the application ELF and exchange APDUs

## Test Directory Discovery

Read `[pytest.standalone].directory` from `ledger_app.toml` to find the test directory.
Do not hardcode paths. The `requirements.txt` is at `<test_dir>/requirements.txt`.
The `application_client/` subfolder contains the Python command sender and response
unpacker.

## APDU Construction — No Magic Hex

**Never** use raw hex strings for complex payloads:

```python
# ❌ BAD — unreadable, unmaintainable
payload = bytes.fromhex("050012ab...")

# ✅ GOOD — semantic, self-documenting
amount = 1000
recipient = bytes.fromhex("de0b295669a9fd93d5f28d9ec85e40f4cb697bae")
payload = struct.pack(">I", amount) + recipient
```

Name variables explicitly (`amount`, `derivation_path`, `fee`, `nonce`) before packing
them. Every field in the APDU payload should be traceable to a named value.

## Coverage Requirements

Every tested feature must include:

| Scenario | Expected SW | Purpose |
|---|---|---|
| **Happy path** | `0x9000` | Valid inputs produce correct output |
| **Error paths** | `0x6A80`, `0x6B00`, etc. | Invalid P1/P2, bad data length, malformed data |
| **User rejection** | `0x6985` | User denies the operation on-device |
| **Edge cases** | varies | Empty data, max-length data, boundary values |

## UI Verification — Golden Snapshots

For any operation that displays information on-screen (signing, key export):

1. Use the Ragger `navigator` to simulate button presses (Nano) or touch events
   (Stax/Flex/Apex).
2. Use `navigator.navigate_and_compare()` to verify screen content against reference
   images (Golden Snapshots).
3. **Verify what the screen displays matches what is being signed.** A mismatch between
   displayed data and signed data is a critical security issue.

## Test Pattern

Use the `CommandSender` abstraction — do not call `backend.exchange()` directly with
raw bytes:

```python
def test_sign_transaction_nominal(backend, scenario_navigator):
    client = CommandSender(backend)

    # 1. Build a semantic transaction
    transaction = Transaction(
        nonce=1,
        to="0xde0b295669a9fd93d5f28d9ec85e40f4cb697bae",
        value=666,
        memo="Test"
    ).serialize()

    # 2. Send APDU (UI interaction expected)
    with client.sign_tx(path="m/44'/1'/0'/0/0", transaction=transaction):
        # 3. Simulate user review & approval
        scenario_navigator.review_approve()

    # 4. Verify response
    response = client.get_async_response()
    assert response.status == 0x9000
    assert len(response.data) > 0  # Signature present
```

## Device Matrix

All devices listed in `[app].devices` in `ledger_app.toml` must be tested. Do not
skip any device.

| `ledger_app.toml` device | Ragger `--device` value |
|---|---|
| `nanos+` | `nanosp` |
| `nanox` | `nanox` |
| `stax` | `stax` |
| `flex` | `flex` |
| `apex_p` | `apex_p` |

## Test Execution Workflow

Run tests inside Docker using the `ledger-app-dev-tools` image (verify with
`docker images | grep ledger` before running).

1. **Setup:** Activate venv and install `<test_dir>/requirements.txt`.
2. **Golden Run:** Run `pytest <test_dir>/ --tb=short -v --device <device> --golden_run`
   for each supported device to generate reference snapshots.
3. **Verification:** Run `pytest <test_dir>/ --tb=short -v --device <device>` (without
   `--golden_run`) for each device to confirm stability.
4. **Iterate:** If tests fail, read the pytest output. Fix test bugs in Python; report
   application bugs for the embedded code to be fixed in `src/`. Re-execute until all
   tests pass on all devices.
