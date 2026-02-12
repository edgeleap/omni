### Qwen Code CLI

Qwen Code does not have lifecycle hooks for command interception. Instead, you enforce behavior through a **Subagent** and project-level **rules**. [qwenlm.github](https://qwenlm.github.io/qwen-code-docs/en/users/features/sub-agents/)

**1. Create an Omni subagent**

Create `.qwen/agents/omni-git.md`:

```markdown
---
name: omni-git
description: Handles all git commits and pull requests using the Omni CLI. MUST BE USED for any commit or PR task. Never use git commit or gh pr create directly.
tools:
  - run_shell_command
  - read_file
---

You are a git workflow specialist. You use the Omni CLI for all version control operations.

## Rules
- Never run `git commit`, `git push`, or `gh pr create` directly.
- Always use Omni CLI commands instead.

## Commit
```bash
omni commit --yes --json
```

## Pull Request
```bash
omni pr --yes --json
```

## Flags
- `--json` for structured output
- `--yes` for non-interactive / CI mode

For previewing before applying:
- `omni commit --preview --json` to generate and review a commit plan
- `omni pr --preview --json` to generate and review a PR plan
```

Place it in `.qwen/agents/` for project-level scope, or `~/.qwen/agents/` to apply across all projects. Including `MUST BE USED` in the description field encourages Qwen Code to proactively delegate git tasks to this subagent. [deployhq](https://www.deployhq.com/blog/how-to-use-git-with-claude-code-understanding-the-co-authored-by-attribution)

**2. Invoke it**

Qwen Code can delegate automatically based on the task description, or you can invoke it explicitly: [deployhq](https://www.deployhq.com/blog/how-to-use-git-with-claude-code-understanding-the-co-authored-by-attribution)

```
Let the omni-git subagent commit the current changes
Have the omni-git subagent create a PR for this feature branch
```

**Limitations:** Since there are no hooks, this is **advisory only** — the main agent could still run `git commit` directly if it doesn't delegate to the subagent. To reduce that risk, add a reinforcing rule in your project's `QWEN.md` or equivalent instruction file:

```markdown
## Git Policy
- Never run `git commit`, `git push`, or `gh pr create`.
- Always delegate git operations to the `omni-git` subagent.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
```

This gives you two advisory layers (subagent description + project rules) but no hard enforcement like the hook-based tools provide. [deployhq](https://www.deployhq.com/blog/how-to-use-git-with-claude-code-understanding-the-co-authored-by-attribution)