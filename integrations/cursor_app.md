### Cursor Hooks

Cursor's Agent Mode can run `git commit` and `git push` autonomously, so you need both a **rule** (instruction layer) and a **hook** (enforcement layer) to redirect it to `omni`. [blog.gitbutler](https://blog.gitbutler.com/cursor-hooks-deep-dive)

**1. Add a Cursor rule**

Create `.cursor/rules/omni-git.mdc`:

```markdown
---
alwaysApply: true
---

# Git Policy — Use Omni

- Never run `git commit`, `git push`, or `gh pr create` directly.
- For commits, use `omni commit --yes --json`.
- For pull requests, use `omni pr --yes --json`.
- Always pass `--json` for structured output and `--yes` for non-interactive mode.
```

Setting `alwaysApply: true` ensures the rule is injected into every agent session. [eastondev](https://eastondev.com/blog/en/posts/dev/20260110-cursor-rules-complete-guide/)

**2. Add a hook**

Create `.cursor/hooks.json` at the project root:

```json
{
  "version": 1,
  "hooks": {
    "beforeShellExecution": [
      {
        "command": ".cursor/hooks/omni-intercept.sh"
      }
    ]
  }
}
```

**3. Create the intercept script**

Create `.cursor/hooks/omni-intercept.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

COMMAND=$(jq -r '.command // ""')

# Intercept git commit → omni commit
if echo "$COMMAND" | grep -qE '^git\s+commit'; then
  jq -n '{permission:"deny", agentMessage:"Use omni commit --yes --json instead of git commit."}'
  exit 0
fi

# Intercept gh pr create / git push → omni pr
if echo "$COMMAND" | grep -qE '(gh\s+pr\s+create|git\s+push)'; then
  jq -n '{permission:"deny", agentMessage:"Use omni pr --yes --json instead of gh pr create / git push."}'
  exit 0
fi

# Allow everything else
jq -n '{permission:"allow"}'
```

Then make it executable:

```bash
chmod +x .cursor/hooks/omni-intercept.sh
```

**How it works:**

- `beforeShellExecution` fires every time Cursor's agent is about to run a terminal command. [blog.gitbutler](https://blog.gitbutler.com/cursor-hooks-deep-dive)
- The script reads the command from stdin (Cursor sends `{"command": "..."}`) and checks it against `git commit` / `git push` / `gh pr create`. [blog.gitbutler](https://blog.gitbutler.com/cursor-hooks-deep-dive)
- Returning `{permission: "deny"}` blocks the command, and the `agentMessage` tells the agent what to run instead. [blog.gitbutler](https://blog.gitbutler.com/cursor-hooks-deep-dive)
- Returning `{permission: "allow"}` lets all other commands pass through. [blog.gitbutler](https://blog.gitbutler.com/cursor-hooks-deep-dive)
 