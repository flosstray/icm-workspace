---
description: Scaffold an Interpretable Context Methodology (ICM) workspace in the current project — creates workspace/CONTEXT.md, numbered stage folders with contracts, _config/, shared/, agents/, optional instances/, and a 0-START-HERE.md visibility marker at root. Idempotent. Pairs with the icm-workspace skill.
argument-hint: --stages "01-name,02-name,..." [--instances <noun>] [--auto-symlink] [--no-marker]
allowed-tools: Bash, Read, Write, Edit
---

# /icm-init — Scaffold an ICM workspace

You are scaffolding an ICM workspace in the current working directory. Read the `icm-workspace` skill at `~/.claude/skills/icm-workspace/SKILL.md` first to ground yourself in the methodology (5 layers, read-order invariant, stage contract format).

## Arguments

`$ARGUMENTS` contains the user's flags. Parse:

- `--stages "01-research,02-draft,03-publish"` — comma-separated stage list. **Required.** Each entry MUST be `NN-<verb>` or `NNX-<verb>` (sub-phase). Reject otherwise.
- `--instances <noun>` — optional. Creates `workspace/instances/` (or noun-named dir: `brands/`, `clients/`, `tenants/`, `models/`) with a `_template/` subdir containing `CONTEXT.md` + `orchestrator.md`.
- `--auto-symlink` — optional. If set, automatically symlink existing `./config/` and `./docs/` into `workspace/_config/` and `workspace/shared/`. If not set, detect them and ASK the user before linking.
- `--no-marker` — optional. Skip creating the `0-START-HERE.md` visibility marker at root. Default: marker is created.

If `--stages` is missing, print usage and stop.

## Pre-flight checks

1. Confirm working directory contains `CLAUDE.md` at root (L0). If missing, print a warning — the user can still proceed but L0 anchoring is recommended.
2. Confirm `workspace/` doesn't already exist. If it does, switch to **additive mode** — only create missing pieces, never overwrite existing files. Report every file as `[created]` or `[skipped — exists]`.
3. Run `git status` if in a git repo. If there are staged or unstaged changes affecting `workspace/`, stop and ask the user to commit or stash first.

## Files to create

### `workspace/CONTEXT.md` (L1 entry router)

```markdown
# Workspace Entry Point (L1)

> Read order: L0 (`/CLAUDE.md`) → this file → stage `CONTEXT.md` → only-then L3/L4.
> See `~/.claude/skills/icm-workspace/SKILL.md` for the full methodology.

## Stages
| # | Stage | Purpose | Contract |
|---|---|---|---|
{one row per stage, with link to its CONTEXT.md}

## Instances
{if --instances given, list instances/ tree; else write: "Single-instance project — instances/ collapsed away."}

## Agents
- Master orchestrator: `agents/master-orchestrator.md` (to be written)
- Role agents: `agents/<role>.md` (to be written)
- Per-instance sub-orchestrators: `instances/<id>/orchestrator.md` (to be written)

## References
- L3 declarative: `_config/`
- L3 prose: `shared/`
- L4 stage outputs: `stages/<NN-verb>/output/`
```

### `workspace/stages/<NN-verb>/CONTEXT.md` (L2 stage contract — one per stage)

```markdown
# Stage <NN> — <verb> (L2)

> Read this stage's full Inputs table BEFORE reading any L3 or L4 file.

## Inputs
| File | Layer | Required | Notes |
|---|---|---|---|
| <fill in> | L3 | yes | <fill in> |

## Process
1. <fill in> — verify: <check>
2. <fill in> — verify: <check>

**Stop conditions:** <when to halt and ask the user>
**Cardinal-Rule reminders:** <pull from /CLAUDE.md L0 — anything stage-relevant>

## Outputs
| File | Consumed by |
|---|---|
| output/<artifact>.<ext> | <next stage> or `terminal` |
```

### `workspace/stages/<NN-verb>/output/.gitkeep` (empty file so the dir survives in git)

### `workspace/_config/README.md`

```markdown
# L3 — Declarative configuration

Symlink in here any YAML, JSON, or env templates that stages reference declaratively.
Don't duplicate — `ln -s ../../config/<file>.yaml .` instead.
```

### `workspace/shared/README.md`

```markdown
# L3 — Prose reference material

Symlink in here any playbooks, glossaries, infrastructure docs, or pattern catalogs
that stages reference. Don't duplicate — `ln -s ../../docs/<file>.md .` instead.
```

### `workspace/agents/README.md`

```markdown
# Role agents + master orchestrator

Each `<role>.md` uses the standard agent frontmatter (`name`, `description`, `model`,
`maxTurns`, `tools`) and obeys the read-order invariant. The body declares which
stages this role operates in.

For a project with instances: write `master-orchestrator.md` here; per-instance
sub-orchestrators live in `../instances/<id>/orchestrator.md`.
```

### `0-START-HERE.md` (visibility marker at project root) — UNLESS `--no-marker` was passed

The `0-` prefix forces this file to sort above all letters in case-insensitive Finder sort, so it appears immediately after dotfiles — making the ICM layer the first visible thing in any file browser.

```markdown
# 📍 START HERE

This project uses **ICM (Interpretable Context Methodology)** — a filesystem-based agent-orchestration pattern. The repo has two layers:

\`\`\`
🧭 NAVIGATION + ORCHESTRATION layer
   workspace/CONTEXT.md       ← read this FIRST
   workspace/stages/          ← the sequenced pipeline
   workspace/<instances>/     ← per-instance status + sub-orchestrators  (if applicable)
   workspace/agents/          ← master + role agents
   workspace/_config/         ← declarative refs (symlinks recommended)
   workspace/shared/          ← prose refs (symlinks recommended)

🛠️ IMPLEMENTATION layer
   <project's existing src/, lib/, apps/, packages/, config/, docs/, etc.>
\`\`\`

## For new sessions — read order

1. **`CLAUDE.md`** (this file's neighbor) — cardinal rules.
2. **`workspace/CONTEXT.md`** — entry router.
3. **`workspace/<instances>/<id>/CONTEXT.md`** — for instance-specific work (if applicable).
4. **`workspace/stages/<NN-verb>/CONTEXT.md`** — for stage-specific work.

## ICM methodology reference

- Local skill: `~/.claude/skills/icm-workspace/SKILL.md`
- Shared bundle: https://github.com/flosstray/icm-workspace
- Paper: Van Clief, J. & McDermott, D. *Interpretable Context Methodology: Folder Structure as Agentic Architecture* — arxiv 2603.16021

## Project-specific docs

For project context (what this codebase does, how to build/deploy, etc.) see [`README.md`](README.md). For non-ICM operational rules see [`CLAUDE.md`](CLAUDE.md).
```

When scaffolding for a real project, replace the `<project's existing ...>` placeholder with the actual top-level directories you observed during pre-flight, and fill in instance-specific routing if applicable.

### If `--instances <noun>` is given:

Rename `instances/` to `<noun>/` if the noun isn't `instances`. Then create:

#### `workspace/<noun>/_template/CONTEXT.md`

```markdown
# <Instance> — Index (L1 for this instance)

## Status snapshot
- <field>: <value>
- Last verify: <date>

## Run order
Execute stages in `/workspace/stages/` in numeric order. Instance-specific notes below.

## Stage notes
{one bullet per stage from --stages — left blank for the user to fill}

## Sub-orchestrator
See `./orchestrator.md`.
```

#### `workspace/<noun>/_template/orchestrator.md`

```markdown
---
name: <instance>-orchestrator
description: Sub-orchestrator for the <instance> instance. Owns the full stage lifecycle. Dispatches role agents in stage order, passing instance identity.
model: sonnet
maxTurns: 60
tools: Read, Bash, Edit, Write
---

You are the sub-orchestrator for the `<instance>` instance. Read this instance's
`CONTEXT.md` first, then drive stages in `/workspace/stages/` in numeric order.
For each stage:

1. Read the stage's `CONTEXT.md` (L2) — pay attention to Inputs / Process / Outputs.
2. Invoke the appropriate role agent from `/workspace/agents/<role>.md`, passing
   the current instance identity.
3. Verify the role agent wrote the declared Outputs before advancing.
4. Update this instance's `CONTEXT.md` status snapshot on completion.

Read-order invariant (NEVER violate): L0 (`/CLAUDE.md`) → this instance's CONTEXT.md
→ current stage L2 → only then L3/L4 files in that stage's Inputs table.
```

## Auto-symlink detection

After scaffolding, check for `./config/` and `./docs/`:

```bash
[ -d ./config ] && echo "Found ./config/ — symlink into workspace/_config/?"
[ -d ./docs ] && echo "Found ./docs/ — symlink into workspace/shared/?"
```

If `--auto-symlink` is set, perform the symlinks automatically (use `ln -s` with relative paths from inside `workspace/_config/` and `workspace/shared/`). If not, ask the user before linking.

**Never copy or move files** during symlink. Always `ln -s ../../config/<file>` so the original source location remains canonical.

## Report

Print a summary tree of what was created. Example:

```
0-START-HERE.md                             [created — unless --no-marker]
workspace/                                  [created]
├── CONTEXT.md                              [created]
├── stages/
│   ├── 01-research/CONTEXT.md              [created]
│   ├── 01-research/output/.gitkeep         [created]
│   ├── 02-draft/...
├── _config/README.md                       [created]
├── shared/README.md                        [created]
├── agents/README.md                        [created]
└── <noun>/_template/                       [created — only if --instances <noun> was passed;
    ├── CONTEXT.md                          [created]   <noun> is whatever the user chose:
    └── orchestrator.md                     [created]   clients, tenants, brands, models, etc.]
```

Then suggest next steps:
1. Populate the stage `CONTEXT.md` Inputs/Process/Outputs tables.
2. Add role agents under `workspace/agents/` per the `icm-workspace` skill.
3. If multi-instance: copy `_template/` to create each instance.
4. Open a fresh Claude session in the project root to verify the skill-router preflight surfaces `icm-workspace`.

## Failure handling

- Invalid `--stages` format → print expected format, stop. No partial scaffold.
- `workspace/` exists with conflicting files → switch to additive mode, never overwrite.
- Filesystem error → roll back any files created in this invocation.
