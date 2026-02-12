### Claude Code Hooks

Claude Code **Hooks** let you intercept tool usage and run shell commands at specific lifecycle events—making them the ideal way to replace `git commit` and `gh pr create` with `omni`. Unlike Rules (advisory, in `CLAUDE.md`) or Skills (discovered by the model), hooks are **system-triggered** and fire automatically every time the matched action occurs. [code.claude](https://code.claude.com/docs/en/hooks)

Add the following to your project's `.claude/settings.json` (or `~/.claude/settings.json` for all projects): [code.claude](https://code.claude.com/docs/en/hooks)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/omni-intercept.sh"
          }
        ]
      }
    ]
  }
}
```

Then create `.claude/hooks/omni-intercept.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

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
chmod +x .claude/hooks/omni-intercept.sh
```

**How it works:**

- The `PreToolUse` event fires every time Claude is about to execute a Bash command. [code.claude](https://code.claude.com/docs/en/hooks)
- The `matcher: "Bash"` ensures only shell commands are inspected. [code.claude](https://code.claude.com/docs/en/hooks)
- **Exit code 2** is a blocking error—it prevents the tool call and sends your `stderr` message back to Claude as feedback, prompting it to use `omni commit` or `omni pr` instead. [code.claude](https://code.claude.com/docs/en/hooks)
- **Exit code 0** lets all other commands pass through untouched. [code.claude](https://code.claude.com/docs/en/hooks)

For the agent workflow, pair this with a `CLAUDE.md` rule so Claude knows the correct commands:

```markdown
## Git Policy
- Never use `git commit` or `gh pr create` directly.
- Use `omni commit --yes --json` for commits.
- Use `omni pr --yes --json` for pull requests.
- See the command expectation matrix in docs for all flags.
```

This gives you a two-layer safety net: the `CLAUDE.md` rule guides Claude's behavior, and the hook **enforces** it by blocking any `git commit` or `gh pr create` calls that slip through. [code.claude](https://code.claude.com/docs/en/hooks)