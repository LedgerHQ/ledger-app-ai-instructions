# Ledger App AI Instructions

Reusable AI instruction files for Ledger embedded application repositories.

## Usage

Add this repository as a git submodule of your application. Below, `<submodule>` is its path from the application root, where all commands run.

### Instructions

Create an `AGENTS.md` at the root of your application that imports the entry point for its language, `C.md` or `RUST.md`:

```markdown
@<submodule>/C.md
```

Add your application's own instructions to the same file. Do not symlink `AGENTS.md` into the submodule: Copilot CLI then loads no rules.

Claude Code (v2.1.277 or later), Copilot CLI and VS Code read `AGENTS.md`. Do not add a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md`, in the application or above it: Claude Code then ignores `AGENTS.md`. Your personal `~/.claude/CLAUDE.md` does not count. For personal notes, use a gitignored `.claude/rules/local.md`.

### Migrating from earlier versions

- Replace your `CLAUDE.md` with the `AGENTS.md` above, moving its other content there: its import of `<submodule>/CLAUDE.md` now loads no rules.
- Remove a `.github/instructions` symlink into the submodule: `AGENTS.md` loads the same rules.

## Files

| File | Purpose |
|---|---|
| `C.md` | Entry point for C applications |
| `RUST.md` | Entry point for Rust applications |
| `instructions/EMBEDDED.instructions.md` | Cross-language embedded constraints (hardware, security, UI) |
| `instructions/C.instructions.md` | C rules, toolchain, build workflow |
| `instructions/RUST.instructions.md` | Rust rules, toolchain, build workflow |
| `instructions/TEST.instructions.md` | Test writing rules (Ragger, Speculos, snapshots) |
| `instructions/REVIEW.instructions.md` | Code review checklist (security, coherence, quality gate) |
| `skills/SECURITY.REVIEW.md` | Full-depth security audit methodology |

Import a new instruction file from each entry point it applies to.
