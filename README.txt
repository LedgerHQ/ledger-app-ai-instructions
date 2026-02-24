# Ledger App AI Instructions

Reusable AI instruction files for Ledger embedded application repositories.

## Usage

Clone this repository into your app's `.github/instructions/` directory:

```bash
cd your-app/.github/instructions/
git submodule add <repo-url> ledger-app-ai-instructions
```

VS Code (with GitHub Copilot) will automatically discover and apply `*.instructions.md`
files based on their `applyTo` glob patterns.

## Files

| File | Scope | Purpose |
|---|---|---|
| `EMBEDDED.instructions.md` | `*.c, *.h, *.rs` | Cross-language embedded constraints (hardware, security, UI) |
| `C.instructions.md` | `*.c, *.h` | C-specific rules, toolchain, build workflow |
| `RUST.instructions.md` | `*.rs` | Ledger-specific Rust deviations (custom test harness, `no_std`) |
| `PYTHON.instructions.md` | `*.py` | Test writing rules (Ragger, Speculos, snapshots) |
| `REVIEW.instructions.md` | `*` (review only) | Code review checklist, excluded from coding agent |

## How It Works

- Files with `applyTo` in their YAML frontmatter are **automatically included** when
  Copilot works on matching files.
- `REVIEW.instructions.md` uses `excludeAgent: "coding-agent"` so it only activates
  during code review, not when writing code.
- Instructions are additive — multiple files can apply to the same request.

## Customization

These instructions cover **Ledger-platform rules** shared across all apps. For
app-specific instructions (custom APDU definitions, specific parsing logic), create
additional `*.instructions.md` files in your app's own `.github/instructions/` directory.
