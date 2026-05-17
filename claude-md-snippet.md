# Snippet to paste into your global `~/.claude/CLAUDE.md`

Add this section near your skill router / preflight section. If you don't have a skill router, just paste it at the bottom — Claude will still find it.

```markdown
## ICM Workspaces (default for new projects with sequenced or multi-instance work)

When a project has clear staged work (research → draft → publish, etc.) or runs the same pipeline across multiple instances (brands, tenants, clients), use the **Interpretable Context Methodology (ICM)**:
- Skill: `~/.claude/skills/icm-workspace/SKILL.md` (5-layer hierarchy, stage contracts, orchestrator tiers)
- Scaffold command: `/icm-init --stages "01-x,02-y" [--instances <noun>]`

If you enter a directory containing `workspace/CONTEXT.md`, read it first — it's the project's L1 router. The skill is surfaced automatically by the preflight above (if you have one).

**Namespace convention:** all ICM-related skills and commands use the `icm-` prefix (`~/.claude/skills/icm-*/`, `~/.claude/commands/icm-*.md`).
```

That's it. Save the file and you're done.
