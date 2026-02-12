### Codex App

The Codex app (macOS desktop) works differently from hook-based tools — it's a multi-agent orchestration layer with built-in Git worktree management. [openai](https://openai.com/index/introducing-the-codex-app/)

Each agent runs on an **isolated Git worktree** — a separate copy of your repo. The agent makes changes in its worktree, and you **review the diff** in the app's review pane before deciding to commit or merge. The model is *not* the one running `git commit` — it's a "do the work → human reviews the diff → human commits" flow. [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents)

This means there isn't a `git commit` call to intercept the way there is with Claude Code.

**1. Create an Omni skill**

The integration point in the Codex app is **Skills** — bundled instructions + scripts that Codex discovers and uses automatically or on demand. You can create skills through the app's built-in Skills UI, or commit them as markdown files in your repo (e.g. `.agents/skills/omni-commit.md`) so they're shared across threads, team members, and across the App, CLI, and IDE Extension. [developers.openai](https://developers.openai.com/codex/app/)

```markdown
---
name: omni-commit
description: Commit staged changes using the Omni CLI instead of git
---

# Omni Commit

When you need to commit changes, use the `omni` CLI instead of `git commit`.

## Commit
```bash
omni commit --yes --json
```

## Pull Request
```bash
omni pr --yes --json
```

## Rules
- Never use `git commit` directly.
- Never use `gh pr create` directly.
- Always pass `--json` for structured output.
- Always pass `--yes` for non-interactive execution.
```

**2. Pair it with `AGENTS.md` for reinforcement**

```markdown
# AGENTS.md

## Git Policy
- Use the `omni-commit` skill for all commits and PRs.
- Never run `git commit`, `git push`, or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
```

**3. Use Automations for scheduled flows**

The Codex app supports **Automations** — scheduled tasks that run agents on a cadence. You could set one up like: [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents)

> *"Every day at 5pm, review uncommitted work across all threads and run `omni commit --yes --json` for any completed tasks."*

The automation deposits results in the review queue for you to approve. [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents)

**How it differs from Claude Code:**

| Aspect | Claude Code | Codex App |
| --- | --- | --- |
| Who runs `git commit` | The agent, automatically | The human, after reviewing the diff [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents) |
| Interception mechanism | `PreToolUse` hooks block & redirect [code.claude](https://code.claude.com/docs/en/hooks) | Not needed — agent doesn't commit on its own [developers.openai](https://developers.openai.com/codex/app/) |
| How to teach `omni` | `CLAUDE.md` rules + hooks | Skills + `AGENTS.md` [developers.openai](https://developers.openai.com/codex/app/) |
| Multi-agent git safety | N/A (single agent) | Built-in worktrees isolate each agent [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents) |
| CI/scheduled work | Hooks + `--yes --json` | Automations with review queue [intuitionlabs](https://intuitionlabs.ai/articles/openai-codex-app-ai-coding-agents) |

The Codex app is a "do the body of work, then the human reviews and commits" tool. You don't need to intercept git calls — you teach Codex about `omni` through a **Skill**, and when you (or an Automation) decide it's time to commit, the agent runs `omni commit` instead of `git commit`. [developers.openai](https://developers.openai.com/codex/app/)