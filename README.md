# ICM Workspace — Shareable Bundle

This bundle installs the **Interpretable Context Methodology (ICM)** workflow into any Claude Code account.

ICM is a filesystem-based pattern for organizing Claude Code projects: a 5-layer markdown hierarchy (root `CLAUDE.md` → workspace router → stage contracts → references → working artifacts) with numbered stage folders that carry Inputs/Process/Outputs contracts. It works in Claude Code (CLI, VS Code extension, Desktop app, web) without any extension or plugin — files *are* the orchestration.

Reference paper: Van Clief, J. & McDermott, D. *Interpretable Context Methodology: Folder Structure as Agentic Architecture* (arxiv 2603.16021).

---

## Install (one-time, ~2 minutes)

### Step 1 — Copy the skill

```bash
mkdir -p ~/.claude/skills/icm-workspace
cp skills/icm-workspace/SKILL.md ~/.claude/skills/icm-workspace/SKILL.md
```

### Step 2 — Copy the slash command

```bash
mkdir -p ~/.claude/commands
cp commands/icm-init.md ~/.claude/commands/icm-init.md
```

### Step 3 — Append the snippet to your global `CLAUDE.md`

Open `~/.claude/CLAUDE.md` (create it if missing) and paste the contents of `claude-md-snippet.md` from this bundle. Place it near any other "skill router" or "preflight" instructions.

### Step 4 — Rebuild the skill index (if you use one)

If your `~/.claude/` has a `scripts/build-skill-index.py` (it's a common Claude Code setup), run it so the skill-router preflight picks up `icm-workspace`:

```bash
python3 ~/.claude/scripts/build-skill-index.py
```

If you don't have a skill index, the skill still works — Claude will find it by reading `~/.claude/skills/`.

---

## Verify the install

Open a fresh Claude Code session anywhere and type:

> *"Let's set up ICM for this project."*

Expected behavior: Claude conducts a brief intake (3 questions about the project), proposes stage names, asks for "go", then runs `/icm-init` to scaffold `workspace/`.

If Claude doesn't recognize the request, double-check that the SKILL.md is in `~/.claude/skills/icm-workspace/` and that the global CLAUDE.md preflight section references the skill router.

---

## What's in this bundle

| File | What it does |
|---|---|
| `skills/icm-workspace/SKILL.md` | The methodology + onboarding conversation. Surfaced by the skill-router preflight on natural-language triggers. |
| `commands/icm-init.md` | Slash command that scaffolds `workspace/CONTEXT.md` + numbered stage folders + agents/ + optional instances/. Idempotent. |
| `claude-md-snippet.md` | One paragraph to paste into your global `~/.claude/CLAUDE.md` so ICM becomes the default approach for new multi-stage projects. |
| `README.md` | This file. |

---

## Using ICM after install

### For a new project

```
mkdir -p ~/Code/my-new-project
cd ~/Code/my-new-project
git init
# Open Claude Code session here, then say:
# "Let's set up ICM for this project."
```

### For an existing project

ICM is additive — it doesn't move or modify existing files. The 6-step migration:

1. Open a session at the existing project root.
2. Say: *"This project has a natural workflow — let's adopt ICM safely."*
3. Claude reads your existing playbook/docs and proposes stages.
4. Confirms with you, then runs `/icm-init`.
5. Symlinks (doesn't copy) your existing `config/` and `docs/` into `workspace/_config/` and `workspace/shared/`.
6. Verify with a fresh session: ask it to trace a typical task without executing.

### Reversal

ICM is fully reversible: `rm -rf workspace/` returns the project to its prior state. The skill never modifies existing files outside `workspace/`.

---

## Recommended reading order if you want the deep version

1. `skills/icm-workspace/SKILL.md` — the methodology (covers the 5 layers, read-order invariant, stage contracts, orchestrator hierarchy, onboarding conversation).
2. `commands/icm-init.md` — what the scaffolder does, step by step.
3. The arxiv paper for theoretical grounding.

---

## License

This bundle is the user's local copy. The methodology itself (ICM) is described in the cited arxiv paper. The reference implementation at github.com/RinDig/Interpreted-Context-Methdology is MIT-licensed.
