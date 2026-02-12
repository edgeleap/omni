### Gemini CLI

Gemini CLI (v0.26.0+) has a **`BeforeTool`** hook configured in `.gemini/settings.json`: [reddit](https://www.reddit.com/r/ClaudeAI/comments/1loodjn/claude_code_now_supports_hooks/)

```json
{
  "hooks": {
    "BeforeTool": [
      {
        "matcher": "shell|run_shell_command",
        "hooks": [
          {
            "name": "omni-intercept",
            "type": "command",
            "command": "$GEMINI_PROJECT_DIR/.gemini/hooks/omni-intercept.sh",
            "description": "Redirects git commands to omni"
          }
        ]
      }
    ]
  }
}
```

Create `.gemini/hooks/omni-intercept.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

# Intercept git commit → omni commit
if echo "$COMMAND" | grep -qE '^git\s+commit'; then
  cat <<EOF
{"decision":"deny","reason":"Use omni commit --yes --json instead of git commit.","systemMessage":"git commit is blocked by project policy."}
EOF
  exit 0
fi

# Intercept gh pr create / git push → omni pr
if echo "$COMMAND" | grep -qE '(gh\s+pr\s+create|git\s+push)'; then
  cat <<EOF
{"decision":"deny","reason":"Use omni pr --yes --json instead of gh pr create / git push.","systemMessage":"git push / gh pr create is blocked by project policy."}
EOF
  exit 0
fi

# Allow everything else
echo '{"decision":"allow"}'
exit 0
```

Then make it executable:

```bash
chmod +x .gemini/hooks/omni-intercept.sh
```

**How it works:**

- The `matcher: "shell|run_shell_command"` filters so the hook only fires on shell commands, not file writes or other tools  [reddit](https://www.reddit.com/r/ClaudeAI/comments/1loodjn/claude_code_now_supports_hooks/).
- The stdin format uses `.tool_input.command` (same path as Claude Code). [reddit](https://www.reddit.com/r/ClaudeAI/comments/1loodjn/claude_code_now_supports_hooks/)
- Returning `{"decision": "deny"}` blocks the command; `reason` is shown to the agent so it can self-correct, and `systemMessage` provides a system-level note. [reddit](https://www.reddit.com/r/ClaudeAI/comments/1loodjn/claude_code_now_supports_hooks/)
- The optional `description` field in the config helps when inspecting active hooks with the `/hooks` command. [reddit](https://www.reddit.com/r/ClaudeAI/comments/1loodjn/claude_code_now_supports_hooks/)

**(Optional) Add a `GEMINI.md` for reinforcement:**

```markdown
## Git Policy
- Never run `git commit`, `git push`, or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
```

This gives you the same two-layer setup: `GEMINI.md` guides the agent, the `BeforeTool` hook enforces it. [datacamp](https://www.datacamp.com/tutorial/claude-code-hooks)
 