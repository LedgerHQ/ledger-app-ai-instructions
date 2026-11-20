# Ledger App AI Instructions

Reusable AI instruction files for Ledger embedded application repositories.

## Usage

Add this repository as a git submodule of your application. Below, `<submodule>` is its path from the application root, where all commands run.

### Instructions

Create an `AGENTS.md` at the root of your application that imports the entry point for its language, `C.md` or `RUST.md`, followed by this check:

```markdown
@<submodule>/C.md

Ledger rules are mandatory. If the "Ledger Embedded Platform Rules" section is not in your context, read the file imported above and every file it imports, relative to the importing file, before any work. If they are missing, stop and tell the user to run `git submodule update --init`.
```

Add your application's own instructions to the same file. Do not symlink `AGENTS.md` into the submodule: Copilot CLI then loads no rules.

Claude Code (v2.1.277 or later), Copilot CLI and VS Code read `AGENTS.md`. Do not add a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md`, in the application or above it: Claude Code then ignores `AGENTS.md`. Your personal `~/.claude/CLAUDE.md` does not count. For personal notes, use a gitignored `.claude/rules/local.md`.

### Skills

Link each skill into `.agents/skills/`, and make `.claude/skills` a symlink to that directory. If `.claude/skills/` already exists, first move its content into `.agents/skills/` and delete it:

```bash
mkdir -p .agents/skills .claude
ln -sfn ../../<submodule>/skills/ledger-security-review .agents/skills/ledger-security-review
ln -sfn ../.agents/skills .claude/skills
```

Codex, Copilot CLI and VS Code read `.agents/skills/`; Claude Code does not (anthropics/claude-code#31005), so remove the `.claude/skills` symlink once it does. Stage skills under `.agents/skills/`: git rejects paths through the symlink. In Claude Code and Copilot CLI, invoke the skill with `/ledger-security-review`, or ask for a security review.

### Migrating from earlier versions

- Replace your `CLAUDE.md` with the `AGENTS.md` above, moving its other content there: its import of `<submodule>/CLAUDE.md` now loads no rules.
- Remove a `.github/instructions` symlink into the submodule: `AGENTS.md` loads the same rules.
- Replace references to `skills/SECURITY.REVIEW.md` with `skills/ledger-security-review/SKILL.md`.

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
| `skills/ledger-security-review/` | Full-depth security audit methodology |

Import a new instruction file from each entry point it applies to. Do not rename the instruction files or the "Ledger Embedded Platform Rules" heading: `ledger-vscode-extension` fetches the files by name, and the applications' check looks for the heading.
