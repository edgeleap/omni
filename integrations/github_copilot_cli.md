### GitHub Copilot CLI

GitHub Copilot CLI is an agentic terminal tool (separate from Copilot coding agent and IDE agent mode) that can autonomously navigate your codebase, run commands, commit changes, and open PRs. It shares the same hooks system as the Copilot coding agent — hooks are stored in `.github/hooks/*.json` and support the same lifecycle events. [github](https://github.blog/ai-and-ml/github-copilot/power-agentic-workflows-in-your-terminal-with-github-copilot-cli/)

**1. Add a hook**

Create `.github/hooks/hooks.json`:

```json
{
  "version": 1,
  "hooks": {
    "preToolUse": [
      {
        "type": "command",
        "bash": ".github/hooks/omni-intercept.sh",
        "timeoutSec": 10
      }
    ]
  }
}
```

**2. Create the intercept script**

Create `.github/hooks/omni-intercept.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.toolName // ""')
COMMAND=$(echo "$INPUT" | jq -r '.toolArgs' | jq -r '.command // ""')

# Only inspect shell/bash tool calls
if [ "$TOOL_NAME" != "bash" ] && [ "$TOOL_NAME" != "shell" ]; then
  exit 0
fi

# Intercept git commit → omni commit
if echo "$COMMAND" | grep -qE '^git\s+commit'; then
  echo "Use 'omni commit --yes --json' instead of git commit." >&2
  exit 2
fi

# Intercept gh pr create / git push → omni pr
if echo "$COMMAND" | grep -qE '(gh\s+pr\s+create|git\s+push)'; then
  echo "Use 'omni pr --yes --json' instead of gh pr create / git push." >&2
  exit 2
fi

# Allow everything else
exit 0
```

Make it executable:

```bash
chmod +x .github/hooks/omni-intercept.sh
```

**How it works:**

- The `preToolUse` hook fires before the agent executes any tool. [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-hooks)
- The stdin JSON contains `toolName` (e.g. `"bash"`) and `toolArgs` as a **JSON string** that needs a second parse to extract `.command`. [docs.github](https://docs.github.com/en/copilot/reference/hooks-configuration)
- **Exit code 0** allows the command; **exit code 2** blocks it and feeds `stderr` back to the agent. [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-hooks)
- Hooks must be committed to the default branch of your repository to take effect. [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-hooks)

**3. Add custom instructions for reinforcement**

Create `.github/copilot-instructions.md`:

```markdown
## Git Policy
- Never run `git commit`, `git push`, or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
```

Copilot CLI reads `.github/copilot-instructions.md` automatically for repository-wide instructions. [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions)

**Copilot CLI vs Copilot Coding Agent vs IDE Agent Mode:**

| Aspect | Copilot CLI | Copilot Coding Agent | IDE Agent Mode |
| --- | --- | --- | --- |
| Where it runs | Your terminal  [github](https://github.blog/ai-and-ml/github-copilot/power-agentic-workflows-in-your-terminal-with-github-copilot-cli/) | GitHub Actions (ephemeral VM)  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) | Your IDE (VS Code)  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) |
| Can auto-commit | Yes  [vladimirsiedykh](https://vladimirsiedykh.com/blog/github-copilot-cli-terminal-ai-agent-development-workflow-complete-guide-2025) | Yes (creates PRs)  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) | Yes (local edits)  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) |
| Hooks location | `.github/hooks/*.json`  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-hooks) | `.github/hooks/*.json`  [docs.github](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-hooks) | N/A (uses Cursor-style hooks if in Cursor, or VS Code settings) |
| Instruction file | `.github/copilot-instructions.md`  [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions) | `.github/copilot-instructions.md`  [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions) | `.github/copilot-instructions.md`  [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions) |

All three Copilot surfaces share the same instruction file, and the CLI and coding agent share the same hooks format — so your `.github/hooks/omni-intercept.sh` works for both. [docs.github](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions)