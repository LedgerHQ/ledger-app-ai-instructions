---
description: "Ledger application test writing rules using Ragger, Pytest, and Speculos"
applyTo: "**/*"
---

# Ledger Test Writing Rules

Python is used exclusively for testing Ledger device applications — it is not part of the embedded application.

## Framework

- **Ragger** (Python + Pytest): [github.com/LedgerHQ/ragger](https://github.com/LedgerHQ/ragger)
- **Speculos** emulator: [github.com/LedgerHQ/speculos](https://github.com/LedgerHQ/speculos)
- Test directory and supported devices come from `ledger_app.toml`.

## Test Readability

- **No magic hex:** Do not use raw hex strings (e.g., `bytes.fromhex("050012...")`) for complex APDU payloads.
- Use named variables (`amount`, `derivation_path`, `fee`) and `struct.pack` to construct payloads with clear semantic meaning.
- Use a `CommandSender` abstraction (typically in `application_client/`) to encapsulate APDU construction and response parsing.

## UI Verification

- For critical actions (signing, key export), verify that the device displays the correct information before the user approves.
- Use the Ragger `navigator` to simulate user interaction: button presses on Nano devices, touch events on Stax/Flex/Apex.
- Use `navigator.navigate_and_compare()` to check screen content against reference images (Golden Snapshots).
- Snapshots and tmp snapshots are handled by the framework, NEVER delete them manually, this is USELESS and error prone.

## Running Ragger Tests

### Configuration

- The test directory path is defined in `ledger_app.toml` under `[pytest.standalone].directory`. Do NOT hardcode it.
- The supported devices are listed in `[app].devices` inside `ledger_app.toml`. Every listed device MUST be tested — do NOT skip any.
- The `requirements.txt` is located at `<test_dir>/requirements.txt`.

### Device Mapping

Map `ledger_app.toml` device names to pytest `--device` values:

| `ledger_app.toml` | `--device` value |
| :--- | :--- |
| `nanos+` | `nanosp` |
| `nanox` | `nanox` |
| `stax` | `stax` |
| `flex` | `flex` |
| `apex_p` | `apex_p` |

### Execution Workflow

Tests MUST be executed in a two-step process:

1. **Golden Run** — Generate reference snapshots for each device:
   ```
   pytest <test_dir>/ --tb=short -v --device <device> --golden_run
   ```
   Run this once per device. Use `--golden_run` sparingly to avoid silencing involuntary UI changes.

2. **Verification Run** — Confirm snapshot stability and test correctness (no `--golden_run`):
   ```
   pytest <test_dir>/ --tb=short -v --device <device>
   ```
   Run this for every device. All tests must pass.

### Docker Environment

- **Image discovery:** Run `docker images | grep ledger` before any `docker run`. Do NOT assume a hardcoded image name.
- **Common image:** `ghcr.io/ledgerhq/ledger-app-builder/ledger-app-dev-tools:latest` (includes builder, Speculos, Python venv).
- **Volume mount:** Mount the project root to `/app` inside the container.
- **Venv activation:** Always start with `source /opt/venv/bin/activate && pip install -r <test_dir>/requirements.txt` before running pytest.
- **Host OS adaptation:** Adapt Docker commands to the host OS (e.g., shell syntax for current directory, variable escaping).
- Execute tests yourself in the terminal. Do NOT just display commands to the user.

### Fix Loop

- If tests fail, read the pytest output, identify whether it is a **test bug** (wrong assertion, missing navigation) or a **code bug** (app crash, wrong SW), fix accordingly, and re-execute.
- Do NOT ask the user to re-run. Re-execute the command yourself after fixing.
- Stop ONLY when all tests pass on all devices.

## Coverage Requirements

Every tested feature must include:
- Happy path
- Error paths (invalid inputs, edge cases, malicious inputs)
- User rejection (where applicable)
- Edge cases (empty data, max-length data, boundary values)
