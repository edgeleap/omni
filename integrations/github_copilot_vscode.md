### GitHub Copilot (Coding Agent)

Copilot's coding agent supports **`preToolUse`** hooks, configured in `.github/hooks/hooks.json` on your default branch. [youtube](https://www.youtube.com/watch?v=8T0kFSseB58)

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

# Intercept gh pr create → omni pr
if echo "$COMMAND" | grep -qE '(gh\s+pr\s+create|git\s+push)'; then
  echo "Use 'omni pr --yes --json' instead of gh pr create / git push." >&2
  exit 2
fi

# Allow everything else
exit 0
```

Then make it executable:

```bash
chmod +x .github/hooks/omni-intercept.sh
```

**How it works:**

- The `preToolUse` hook fires before the agent executes any tool. [youtube](https://www.youtube.com/watch?v=8T0kFSseB58)
- The stdin JSON contains `toolName` (e.g. `"bash"`) and `toolArgs` as a **JSON string** that needs a second parse to extract `.command`. [youtube](https://www.youtube.com/watch?v=8T0kFSseB58)
- **Exit code 0** allows the command; **exit code 2** blocks it and feeds `stderr` back to the agent as feedback. [youtube](https://www.youtube.com/watch?v=8T0kFSseB58)
- Note: Copilot's coding agent already **cannot run `git push` directly** — it's restricted to pushing only to `copilot/` branches through an internal mechanism. The main command to intercept is `git commit`. [developers.openai](https://developers.openai.com/blog/openai-for-developers-2025/)

**(Optional) Add custom instructions for reinforcement:**

Create `.github/copilot-instructions.md`:

```markdown
## Git Policy
- Never run `git commit` or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
```

This gives you the same two-layer setup as Claude Code: custom instructions guide the agent, the `preToolUse` hook enforces it. [developers.openai](https://developers.openai.com/blog/openai-for-developers-2025/)
 