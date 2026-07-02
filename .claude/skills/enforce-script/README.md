# Claude Code Skill: enforce-script

A [Claude Code](https://claude.com/claude-code) skill that turns this wiki into
an always-on reference for AI-assisted DayZ modding. It inlines the
compiler-critical Enforce Script rules (no ternary, `g_Game` instead of
`GetGame()`, `ref` rules, script-layer hierarchy, RPC enums ≥ 10000, …) and
routes every modding task to the exact wiki page to read first.

## Install

**Option A — automatic (working inside this repository):**
Nothing to do. Claude Code discovers `.claude/skills/` in the repo and loads
the skill whenever a task involves Enforce Script.

**Option B — user-level (available in all your projects):**
Copy the `enforce-script/` folder to your user skills directory:

```
Windows:      %USERPROFILE%\.claude\skills\enforce-script\SKILL.md
Linux/macOS:  ~/.claude/skills/enforce-script/SKILL.md
```

Then edit the *Overview* section of `SKILL.md` and prefix the doc paths with
the absolute path of your local clone of this repository.

## Use

- **Automatic:** the skill loads whenever Claude writes, reviews, or debugs
  Enforce Script code, or hits errors like `Broken expression (missing ';'?)`.
- **Manual:** type `/enforce-script` in a Claude Code session.
