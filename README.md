# ICM Workspace — Filesystem-Based Agent Orchestration for Claude Code

## TL;DR

**ICM (Interpretable Context Methodology)** is a folder-structure pattern that makes Claude Code projects self-navigating. Every fresh Claude session knows what to read, in what order, what to produce, and where to put it — without rediscovering the project's mental model each time.

This bundle installs ICM into a Claude Code account (CLI, VS Code extension, Desktop app, web — works in all of them). After ~2 minutes of setup, you scaffold any project by typing a single natural-language sentence: *"Let's set up ICM for this project."*

Reference paper: Van Clief, J. & McDermott, D. — *Interpretable Context Methodology: Folder Structure as Agentic Architecture*, [arxiv 2603.16021](https://arxiv.org/abs/2603.16021).

---

## The problem ICM solves

You start a fresh Claude session in a project. It re-explores the codebase, asks where the playbook is, reads scattered docs, sometimes misses cardinal rules buried 800 lines into a `CLAUDE.md`. Multiply this across every session, every brand/tenant/client, every collaborator — and you pay the same rediscovery tax over and over.

ICM eliminates that tax by making the folder structure itself the orchestration. Five layers, strict read-order:

| Layer | Location | Answers |
|---|---|---|
| **L0** | `CLAUDE.md` (project root) | "Where am I?" — cardinal rules |
| **L1** | `workspace/CONTEXT.md` | "Where do I go?" — entry router |
| **L2** | `workspace/stages/NN-<verb>/CONTEXT.md` | "What do I do?" — Inputs / Process / Outputs contract |
| **L3** | `workspace/_config/`, `workspace/shared/` | "What rules apply?" — declarative + prose references |
| **L4** | `workspace/stages/NN/output/` | Working artifacts produced/consumed across stages |

Every agent obeys the read-order: **L0 → nearest L1 → current stage L2 → only then L3/L4** (and only the L3/L4 files listed in the stage's Inputs table). Fresh agents catch up in seconds, not minutes.

---

## What you get after installing

- A skill `icm-workspace` that Claude's skill-router surfaces automatically on natural-language triggers ("let's use ICM here", "scaffold this project", "set up stages") and whenever you enter a project containing `workspace/CONTEXT.md`.
- A slash command `/icm-init` that scaffolds the full workspace structure on demand. Idempotent — safe to rerun.
- A consistent project shape across every project you adopt it on.
- Zero changes to existing files. ICM is purely additive and fully reversible — `rm -rf workspace/` rolls back any project.

---

## Install (one-time, ~2 minutes)

```bash
# 1. Skill
mkdir -p ~/.claude/skills/icm-workspace
cp skills/icm-workspace/SKILL.md ~/.claude/skills/icm-workspace/

# 2. Slash command
mkdir -p ~/.claude/commands
cp commands/icm-init.md ~/.claude/commands/

# 3. Paste the contents of claude-md-snippet.md into ~/.claude/CLAUDE.md
#    (create the file if it doesn't exist)

# 4. (Optional) Rebuild your skill index, if you have one
[ -f ~/.claude/scripts/build-skill-index.py ] && python3 ~/.claude/scripts/build-skill-index.py
```

## Verify the install

Open a fresh Claude Code session anywhere and say:

> *"Let's set up ICM for this project."*

**Expected:** Claude conducts a 3-question intake (what does this project do, what are the stages, will there be multiple instances), proposes a scaffold, asks for "go", then runs `/icm-init` to create `workspace/`.

If Claude doesn't recognize the request, confirm the SKILL.md is at `~/.claude/skills/icm-workspace/SKILL.md` and that the ICM section is in your global `~/.claude/CLAUDE.md`.

---

## Using ICM on a NEW project

```bash
mkdir -p ~/Code/my-new-project
cd ~/Code/my-new-project
git init
# Open a Claude Code session here, then paste the prompt below.
```

**Copy-paste prompt:**

> *"Let's set up ICM for this project. It's a [content site / scraper / ML pipeline / web app / launch plan / etc.]. [If applicable: "It will have multiple instances — clients/brands/tenants/etc."]"*

Claude will:

1. **Ask 3 questions** — what the project does (1-2 sentences), what stages the work decomposes into, whether multiple instances apply.
2. **Propose a scaffold** — concrete stage names + role agents + optional `--instances <noun>`.
3. **Wait for "go"** — you say go (or "change ___" to adjust).
4. **Run `/icm-init`** with the agreed stages.
5. **Walk you through filling in** each stage's Inputs / Process / Outputs contract.
6. **Write role agents** based on the project's actual work (researcher, drafter, reviewer; or extractor, analyst, uploader; etc.).
7. **If multi-instance:** scaffold ONE concrete instance from `_template` as a pilot — not all of them at once.

### Naming conventions

- **Stages**: numbered + verb. `01-research/`, `02-design/`, `03-implement/`. Sub-phases use a letter suffix: `00c-egress/`. Numbers encode sequence; verbs keep `ls` output readable.
- **If you can't articulate the verb for a stage, it's not a stage yet.**
- **Instances**: only add if you'll have 3+ of the same thing (brands, clients, tenants, environments, models). Two isn't worth the template overhead; eight is.

### Discoverability marker

`/icm-init` scaffolds a **`0-START-HERE.md`** at the project root by default. Because filenames starting with `0-` sort above all letters in case-insensitive Finder/file-explorer views, this file appears immediately after dotfiles — making the ICM layer the first visible thing anyone sees when they open the project.

The marker explains the two-layer model (navigation in `workspace/` vs implementation everywhere else), points new sessions to the right read order, and lists common operations. Purely additive and non-breaking; pass `--no-marker` to skip it.

---

## Using ICM on an EXISTING project (safely)

ICM is additive and reversible. Adopting it on an existing project **does not move, modify, or remove any existing files.**

**Copy-paste prompt** (in a Claude Code session at the existing project root):

> *"This project has a natural workflow — let's adopt ICM safely. Read my existing playbook docs first to figure out the right stages, then scaffold workspace/ around them without touching existing files."*

Claude will:

1. **Read your existing docs** — `README.md`, any `playbook` / `runbook` / `docs/` directory, project-root `CLAUDE.md` — to identify natural phases.
2. **Propose stages** based on what the playbook actually says, not generic templates.
3. **Scaffold** `workspace/` next to your existing folders. No existing path is touched.
4. **Symlink** (not copy) existing `config/` and `docs/` into `workspace/_config/` and `workspace/shared/`. Originals stay canonical.
5. **Extract Inputs/Process/Outputs** into each stage CONTEXT.md from your existing playbook prose.
6. **Pilot one instance** if multi-instance — don't roll out to all at once.
7. **Verify** by opening a fresh session, asking it to trace a typical task without executing, and confirming the read-order works.

### Safety guarantees

| Rule | Why it matters |
|---|---|
| **Additive only** | `workspace/` is brand new. No existing files are moved or modified. |
| **Symlinks, not copies** | `workspace/_config/<file>` → `../../config/<file>`. Editing either path updates the same canonical file. Git history preserved. |
| **Reversible** | `rm -rf workspace/` returns the project to its exact prior state. No hidden side effects. |
| **Old paths still work** | Any tool, script, or agent that doesn't know about ICM continues to use the old paths. ICM is *new* navigation, not a replacement. |
| **Decision gate** | After step 7, you commit or roll back. No middle-state risk. |

### When NOT to adopt ICM

- **One-off scripts and throwaway utilities.** Overhead without payoff.
- **Projects you're about to deprecate.**
- **Projects in major architectural flux.** Wait for the dust to settle.
- **Solo-developer playgrounds without sequenced work.** ICM is for projects with phases or multiple instances.

---

## What's in this bundle

| File | What it does |
|---|---|
| `skills/icm-workspace/SKILL.md` | The methodology + natural-language onboarding flow. ~12 KB. |
| `commands/icm-init.md` | Slash command that scaffolds the workspace structure. Idempotent. |
| `claude-md-snippet.md` | Paragraph to paste into the recipient's global `~/.claude/CLAUDE.md`. |
| `README.md` | This file. |

---

## FAQ

**Q: Does this work with the Claude Code Desktop app and CLI, or only VS Code?**
All of them. ICM lives at `~/.claude/` which all Claude Code surfaces (CLI, VS Code extension, Desktop app, web) share. Same files, same behavior, same sessions.

**Q: My project already has CLAUDE.md and docs/. Do I have to consolidate everything?**
No. The existing `CLAUDE.md` stays as L0 (the cardinal-rules anchor). Existing docs stay where they are — they get symlinked into `workspace/shared/` for stage references. Nothing moves.

**Q: How is this different from just having a README?**
A README explains the project to humans. ICM structures the project so agents can navigate it programmatically. Stage contracts (Inputs/Process/Outputs) are machine-readable; READMEs are not. Both can coexist.

**Q: Can I roll this back if I don't like it?**
Yes. `rm -rf workspace/` in any project. The skill and command in `~/.claude/` stay installed but inert (they only act when a project has `workspace/`).

**Q: Can I extend the bundle with my own stage templates or role-agent presets?**
Yes — fork or clone, add files to `skills/icm-workspace/` or sibling directories. The bundle's structure is just markdown files; nothing magical.

**Q: What about concurrent agent sessions editing the same repo?**
ICM is additive only, so concurrent work doesn't conflict at the workspace level. But two agents both editing the same Python file or the same stage output will race. Standard advice: one active agent per repo at a time.

**Q: Where does session history live, and does it sync between Claude Code surfaces?**
Sessions are JSONL files under `~/.claude/projects/<encoded-path>/<uuid>.jsonl`. Since all Claude Code surfaces share `~/.claude/`, sessions from VS Code appear in the Desktop app's recents and vice versa.

---

## Reference

- Paper: Van Clief, J. & McDermott, D. *Interpretable Context Methodology: Folder Structure as Agentic Architecture*. [arxiv 2603.16021](https://arxiv.org/abs/2603.16021).
- Reference implementation (MIT-licensed): https://github.com/RinDig/Interpreted-Context-Methdology
