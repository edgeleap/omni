### Antigravity

Antigravity does **not have a formal hooks system** yet — it's been [requested on the forum](https://discuss.ai.google.dev/t/hooks-in-antigravity/120458). Instead, you use two advisory mechanisms: **rules** and **workflows**. [smartscope](https://smartscope.blog/en/generative-ai/claude/claude-code-hooks-guide/)

**1. Add a rule**

Create `.agent/rules/omni-git.md`:

```markdown
---
name: omni-git
alwaysApply: true
---

# Git Policy — Use Omni

- Never run `git commit`, `git push`, or `gh pr create` directly.
- For commits, use `omni commit --yes --json`.
- For pull requests, use `omni pr --yes --json`.
- Always pass `--json` for structured output and `--yes` for non-interactive mode.
- If you catch yourself about to run a raw git command, stop and use the omni equivalent.
```

Rules in `.agent/rules/` act as behavioral constraints the agent must follow. [smartscope](https://smartscope.blog/en/generative-ai/claude/claude-code-hooks-guide/)

**2. Add a workflow**

Create `.agent/workflows/omni-commit.md`:

```markdown
---
name: omni-commit
description: Commit and PR workflow using the Omni CLI
---

# Omni Commit Workflow

## Steps

1. Stage changes as normal (`git add` is allowed).
2. Run `omni commit --preview --json` to generate and review the commit plan.
3. If the plan looks correct, run `omni commit --apply --json` to apply.
4. For pull requests, run `omni pr --preview --json` to preview, then `omni pr --apply --json` to create.

## Quick Reference

| Task | Command |
| --- | --- |
| Quick commit | `omni commit --yes --json` |
| Preview commit | `omni commit --preview --json` |
| Apply commit | `omni commit --apply --json` |
| Preview PR | `omni pr --preview --json` |
| Create PR | `omni pr --apply --json` |

## Rules
- Never use `git commit` directly.
- Never use `gh pr create` or `git push` directly.
- Always route through `omni`.
```

Workflows in `.agent/workflows/` define multi-step processes the agent follows when it encounters a matching task. [smartscope](https://smartscope.blog/en/generative-ai/claude/claude-code-hooks-guide/)

**3. (Optional) Add an Omni skill**

If you want the agent to have deeper knowledge of Omni's full command set, create `.agent/skills/omni.md`:

```markdown
---
name: omni
description: CLI tool that replaces git commit and gh pr create with AI-powered commits and PRs
---

# Omni CLI

Omni is a CLI tool used instead of git for commits and PRs.

## Commit Commands
- `omni commit` — quick commit
- `omni commit --preview` — generate plan, do not apply
- `omni commit --edit` — open plan for editing
- `omni commit --apply` — apply plan

## PR Commands
- `omni pr --preview` — preview PR
- `omni pr --edit` — edit PR plan
- `omni pr --apply` — create PR

## Flags
- `--json` — structured agent output
- `--yes` — CI/non-interactive apply
```

Skills in `.agent/skills/` are discovered automatically by the agent and teach it how to use specific tools. [github](https://github.com/gitbutlerapp/gitbutler-docs/blob/main/content/docs/features/ai-integration/claude-code-hooks.mdx)

**Limitations:** All three layers (rules, workflows, skills) are **advisory only**. The agent should follow them, but there's no system-level enforcement to hard-block a raw `git commit` the way hooks do in Claude Code, Cursor, or Gemini CLI. If Antigravity adds hooks in the future, you'd be able to add a hard intercept on top of this setup. [smartscope](https://smartscope.blog/en/generative-ai/claude/claude-code-hooks-guide/)