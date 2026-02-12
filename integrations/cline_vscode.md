### Cline

Cline (v3.36+) supports **Hooks** — executable scripts that trigger at specific workflow events, including **`PreToolUse`** which fires before any tool execution. This gives you hard enforcement similar to Claude Code and Cursor. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)

**1. Enable hooks**

In VS Code, open Cline settings → Features → enable **Hooks**. Hooks are currently supported on macOS and Linux only. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)

**2. Create the intercept script**

Place it in `.clinerules/hooks/PreToolUse` (project-specific) or `~/Documents/Cline/Rules/Hooks/PreToolUse` (global). The file name must exactly match the hook type, with **no file extension**: [developers.openai](https://developers.openai.com/codex/guides/agents-md/)

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.toolName // ""')
COMMAND=$(echo "$INPUT" | jq -r '.parameters.command // ""')

# Only inspect shell/bash tool calls
if [ "$TOOL_NAME" != "execute_command" ]; then
  echo '{"cancel": false}'
  exit 0
fi

# Intercept git commit → omni commit
if echo "$COMMAND" | grep -qE '^git\s+commit'; then
  echo '{"cancel": true, "errorMessage": "Use omni commit --yes --json instead of git commit."}'
  exit 0
fi

# Intercept gh pr create / git push → omni pr
if echo "$COMMAND" | grep -qE '(gh\s+pr\s+create|git\s+push)'; then
  echo '{"cancel": true, "errorMessage": "Use omni pr --yes --json instead of gh pr create / git push."}'
  exit 0
fi

# Allow everything else
echo '{"cancel": false}'
exit 0
```

Make it executable:

```bash
chmod +x .clinerules/hooks/PreToolUse
```

**How it works:**

- `PreToolUse` fires after Cline has decided on an action but before executing it — your hook validates and can block it. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)
- The script receives JSON on stdin with `toolName`, `parameters`, and base fields like `taskId` and `workspaceRoots`. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)
- Returning `{"cancel": true, "errorMessage": "..."}` blocks the operation and shows the error to the agent. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)
- Returning `{"cancel": false}` allows it. You can optionally include `"contextModification": "..."` to inject text into the conversation that shapes future decisions. [developers.openai](https://developers.openai.com/codex/guides/agents-md/)

**3. Add `.clinerules` for reinforcement**

Create `.clinerules` in your project root:

```markdown
## Git Policy
- Never run `git commit`, `git push`, or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
- Always pass `--json` for structured output and `--yes` for non-interactive mode.
```

`.clinerules` are project-specific instructions injected into Cline's system prompt. [reddit](https://www.reddit.com/r/CLine/comments/1jk82br/getting_started_with_cline_questions_about_custom/)
 