# Ledger App AI Instructions

Reusable AI instruction files for Ledger embedded application repositories.

## Usage

Clone this repository into your embedded application as `.github/instructions/` directory:

```bash
cd app-example/.github/
git submodule add <repo-url> instructions
```

Create a symbolic link at the repository root pointing to the `AGENTS.md` file in the submodule:

```bash
cd app-example/
ln -sf .github/instructions/AGENTS.md AGENTS.md
```

This ensures AI tools that look for `AGENTS.md` at the repository root can find and follow the agent instructions.

## Files

| File | Scope | Purpose |
|---|---|---|
| `EMBEDDED.instructions.md` | `*.c, *.h, *.rs` | Cross-language embedded constraints (hardware, security, UI) |
| `C.instructions.md` | `*.c, *.h` | C-specific rules, toolchain, build workflow |
| `RUST.instructions.md` | `*.rs` | Ledger-specific Rust deviations (custom test harness, `no_std`) |
| `TEST.instructions.md` | `*.py` | Test writing rules (Ragger, Speculos, snapshots) |
| `REVIEW.instructions.md` | `*` | Code review checklist |

## Customization

These instructions cover rules generic across all Ledger embedded applications. For application specific instructions (custom APDU definitions, specific parsing logic), use the standard .github/copilot-instructions.md file
