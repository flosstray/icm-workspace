---
name: icm-workspace
description: Interpretable Context Methodology (ICM / Model Workspace Protocol) — filesystem-based agent orchestration via a 5-layer markdown hierarchy and numbered stage folders with Inputs/Process/Outputs contracts. Use when (1) a project contains a `workspace/CONTEXT.md` file — always read it first, (2) the user says any of: "set up ICM", "let's use ICM", "use ICM here", "scaffold a workspace", "scaffold this project", "organize this project with stages", "structure this project", "set up stages", "I want a workspace here", "add a stage", "add an instance / brand / tenant / client", "onboard a brand", (3) you're entering a multi-stage or multi-instance project that needs structured agent navigation, (4) the user asks for "consistent project structure" or "the same shape across projects". Plugs into the existing skill-router preflight without replacing it. Reference paper arxiv 2603.16021 by Jake Van Clief.
---

# ICM Workspace — Interpretable Context Methodology

A filesystem-based orchestration pattern. Files *are* the orchestration: a 5-layer markdown hierarchy and numbered stage folders carry the navigation discipline. Plain text, framework-agnostic, inspectable, git-friendly.

## When this skill applies

- You opened a session in a directory and notice a `workspace/CONTEXT.md` exists at the project root. **Read it first**, then continue per the read-order rule below.
- The user asks to scaffold ICM structure, add a stage, add an instance (brand/tenant/client/model), or navigate an existing workspace.
- A project has clear sequenced work (research → draft → publish, or onboard → fingerprint → recon → verify, etc.) or runs the same pipeline across multiple instances.

## The 5 layers

| Layer | Location | Purpose | Token budget |
|---|---|---|---|
| **L0** | `CLAUDE.md` at project root | "Where am I?" — cardinal rules, conventions, do-not-touch list | ~800 |
| **L1** | `workspace/CONTEXT.md` | "Where do I go?" — entry router; points at stages, instances, agents | ~300 |
| **L2** | `workspace/stages/NN-<verb>/CONTEXT.md` | "What do I do?" — stage contract: Inputs / Process / Outputs | ~300-500 |
| **L3** | `workspace/_config/`, `workspace/shared/` | "What rules apply?" — declarative reference material (symlinks OK) | unbounded |
| **L4** | `workspace/stages/NN-<verb>/output/` | Working artifacts — produced by stages, consumed by later stages | unbounded |

## Read-order invariant (every agent obeys)

```
L0 (root CLAUDE.md)
  → nearest L1 (workspace/CONTEXT.md, or instance CONTEXT.md if inside one)
    → current stage L2 (workspace/stages/NN-<verb>/CONTEXT.md)
      → only THEN load L3 references and L4 prior outputs — and only those listed in the stage's Inputs table
```

Cross-stage L4 reads are allowed only when the current stage's Inputs table lists them. Never grep across other stages' outputs to "see what's there." If you need data not in your Inputs table, the contract is wrong — update it instead of working around it.

## Stage contract — the L2 standard

Every `workspace/stages/NN-<verb>/CONTEXT.md` has these three sections, in this order:

```markdown
# Stage NN — <verb> (L2)

## Inputs
| File | Layer | Required |
|---|---|---|
| ../../shared/<ref>.md | L3 | yes |
| ../<prev-stage>/output/<artifact> | L4 | yes |

## Process
1. <step> — verify: <check>
2. <step> — verify: <check>
Stop conditions: <when to halt and ask>
Cardinal-Rule reminders: <project-specific guardrails from L0>

## Outputs
| File | Consumed by |
|---|---|
| output/<artifact>.<ext> | stages NN+1, NN+2 (or `terminal`) |
```

**Contract integrity invariant:** every Output listed in stage N must appear as an Input of some later stage, or be explicitly marked `terminal`. Run a one-shot grep to verify.

## Canonical layout

```
<project-root>/
├── CLAUDE.md                              # L0 (already exists in most Justin projects)
├── workspace/
│   ├── CONTEXT.md                         # L1 — entry router
│   ├── stages/                            # L2 — numbered + verb folders
│   │   ├── 01-<verb>/
│   │   │   ├── CONTEXT.md                 # stage contract
│   │   │   └── output/                    # L4 artifacts (gitkeep if empty)
│   │   ├── 02-<verb>/
│   │   └── ...
│   ├── _config/                           # L3 — symlinks to existing config/
│   ├── shared/                            # L3 — symlinks to existing docs/
│   ├── agents/                            # role agents + master orchestrator
│   │   ├── master-orchestrator.md
│   │   ├── <role-1>.md
│   │   └── ...
│   └── instances/                         # OPTIONAL — only for multi-tenant projects
│       ├── _template/
│       │   ├── CONTEXT.md
│       │   └── orchestrator.md            # per-instance sub-orchestrator
│       └── <id>/
│           ├── CONTEXT.md                 # lean instance index
│           └── orchestrator.md
└── .claude/                               # untouched — harness config only
```

`workspace/instances/` may use a project-specific noun (`brands/`, `clients/`, `tenants/`, `models/`). The pattern is the same.

## Orchestrator hierarchy

Three tiers stack on top of L2 stages:

```
Master orchestrator (workspace/agents/master-orchestrator.md)
└── Cross-instance. Knows all instances, all stages, the status board.
    Dispatches to per-instance sub-orchestrators.
    Source of truth: workspace/shared/<status-board>.md

Per-instance sub-orchestrator (workspace/instances/<id>/orchestrator.md)
└── One instance, full lifecycle. Dispatches role agents in stage order,
    passing instance identity. Reports status in instance CONTEXT.md.

Role agents (workspace/agents/<role>.md)
└── One capability, instance-parameterized (extractor, analyst, uploader,
    monitor, etc.). Sub-orchestrator passes instance context at invocation.
    Always obeys the read-order invariant above.
```

**Single-instance projects** collapse the sub-orchestrator tier away — only master + role agents.

All three tiers use the standard agent frontmatter (`name`, `description`, `model`, `maxTurns`, `tools`).

## Naming conventions (locked)

- **Stage folders**: `NN-<verb>/` — numbered + verb. Sub-phases use letter suffix (`00c-egress/`). Numbers encode order; verb keeps `ls` output readable.
- **Instance folders**: project-specific noun (`brands/`, `clients/`, `tenants/`). Each instance is a single `<id>/` directory with `CONTEXT.md` + `orchestrator.md`.
- **Role agents**: kebab-case capability name (`extractor.md`, `data-puller.md`, `fni-lens.md`).
- **L0 location**: `CLAUDE.md` stays at project root — never inside `workspace/`. It's the discovery anchor.
- **L3 references**: `_config/` for declarative (YAML, JSON, env templates); `shared/` for prose references (playbooks, glossaries, infra docs). Use **symlinks** to existing in-project paths — don't duplicate.
- **L4 outputs**: always under `workspace/stages/NN-<verb>/output/`. Never write artifacts elsewhere from a stage agent.

## Onboarding conversation (when the user uses natural language)

If the user invokes ICM with natural language ("let's use ICM here", "set up ICM for this project", "I want to organize this with stages") and the current project does **not** have `workspace/CONTEXT.md` yet, **do not** run `/icm-init` immediately. Conduct a brief intake first — three short questions, max — then propose the scaffold, get a one-word "go", and run the command.

### Intake script

> "Got it. Before I scaffold, three quick questions so the stages match the actual work:
>
> **1. In 1–2 sentences, what does this project do?** *(I'll use this to propose stage names.)*
>
> **2. Does the work decompose into clear sequenced phases?** If yes, list them (rough is fine — "research, draft, publish" is enough). If you're not sure, describe a typical end-to-end task and I'll propose stages.
>
> **3. Will this project run the same pipeline across multiple instances** (brands, clients, tenants, models, environments)? If yes, what noun? If you only ever have one, skip this.
>
> Once you answer, I'll propose the full scaffold (stage names + agents + optional `--instances`). You say "go" and I run `/icm-init`."

### Proposing stages from the intake

After the user answers, propose 3–7 numbered stages with verb names. Examples by project type:

- **Content / writing project** → `01-research`, `02-outline`, `03-draft`, `04-edit`, `05-publish`
- **Web app / feature build** → `01-design`, `02-implement`, `03-test`, `04-deploy`
- **ML / data pipeline** → `01-ingest`, `02-clean`, `03-train`, `04-evaluate`, `05-serve`
- **Scraper / extraction** → `00-onboard`, `00c-egress`, `01-fingerprint`, `02-recon`, `03-extract`, `04-verify`, `05-long-tail` (see sg-dealer-scraper)
- **Marketing / launch** → `01-positioning`, `02-creative`, `03-channels`, `04-launch`, `05-iterate`
- **Research / analysis** → `01-scope`, `02-gather`, `03-analyze`, `04-synthesize`, `05-report`

Don't force a project into one of these templates — propose stages that fit the user's actual answer. Anchor stages to **verbs**; if you can't articulate the verb, it's not a stage yet.

### Confirmation step (before scaffolding)

Present a compact summary:

> "Here's what I'll scaffold:
> - Stages: `01-x`, `02-y`, `03-z` ...
> - Role agents: <list — propose based on stages, e.g. researcher, drafter, reviewer>
> - Instances: <none | `brands` | `clients` | ...>
>
> Say `go` and I'll run `/icm-init` + write the stage contract templates. Say `change` and tell me what to adjust."

### After the user says "go"

1. Run `/icm-init --stages "..."` (with `--instances <noun>` if applicable).
2. Help the user fill in each stage's Inputs/Process/Outputs (work through them one at a time — don't dump 7 empty contracts and walk away).
3. Write the role agents based on the project's actual work.
4. If multi-instance: scaffold ONE concrete instance from `_template` as a pilot — not all of them.
5. Suggest a verification: open a fresh session and ask it to trace a typical task; confirm the read-order invariant holds.

## Scaffolding a new workspace

Use the companion slash command:

```
/icm-init --stages "01-research,02-draft,03-publish"
/icm-init --stages "00-onboard,01-fingerprint,02-recon,03-verify" --instances brands
```

The command is idempotent (skip-if-exists) and detects existing `config/` + `docs/` directories, offering to symlink them into `_config/` and `shared/`. See `~/.claude/commands/icm-init.md` for arguments.

## Adoption decision tree

| Project shape | Action |
|---|---|
| New project being created | Run `/icm-init` from day one |
| Existing project with a clear staged playbook (multi-phase prose docs) | Schedule a ~2-hour migration: scaffold + extract stage contracts from existing prose + symlink refs |
| Existing project with no obvious stages | Leave alone — ICM is for sequenced or multi-instance work |
| Project with multi-tenant/multi-brand variation running the same pipeline | Highest leverage. Add `--instances <noun>` flag. Onboarding a new instance becomes `cp -r _template <new>` + fill in |

## Integration with the skill-router preflight

This skill is indexed in `~/.claude/orchestration/skill-index.json`. The mandatory preflight in `~/.claude/CLAUDE.md` surfaces it automatically whenever the inbound task matches by name, keyword (`workspace`, `stage`, `instance`, `ICM`), or intent (multi-stage navigation, multi-instance scaling). No per-project configuration needed once a project has `workspace/CONTEXT.md`.

## Reference

- Paper: Van Clief, J. & McDermott, D. *Interpretable Context Methodology: Folder Structure as Agentic Architecture*. arxiv 2603.16021.
- Reference repo: github.com/RinDig/Interpreted-Context-Methdology (MIT).

## Future ICM skills/commands (namespace convention)

Anything ICM-related lives under the `icm-` prefix:
- Skills: `~/.claude/skills/icm-<topic>/SKILL.md`
- Commands: `~/.claude/commands/icm-<verb>.md`

This keeps the global Claude config navigable as the ICM toolkit grows.
