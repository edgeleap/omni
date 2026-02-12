### Kimi CLI

Kimi CLI does not have lifecycle hooks yet — it's an [open feature request](https://github.com/MoonshotAI/kimi-cli/issues/785). Enforcement is advisory only, through `AGENTS.md` and MCP config. [github](https://github.com/MoonshotAI/kimi-cli/issues/785)

**1. Add an `AGENTS.md` file**

Kimi CLI reads `AGENTS.md` from the project root (you can generate one with `/init`). Add a git policy section: [stevekinney](https://stevekinney.com/courses/ai-development/claude-code-hook-examples)

```markdown
# AGENTS.md

## Git Policy
- Never run `git commit`, `git push`, or `gh pr create` directly.
- For commits, use `omni commit --yes --json`.
- For pull requests, use `omni pr --yes --json`.
- Always pass `--json` for structured output and `--yes` for non-interactive mode.

## Omni CLI Reference

| Task | Command |
| --- | --- |
| Quick commit | `omni commit` |
| Preview commit | `omni commit --preview --json` |
| Apply commit | `omni commit --apply --json` |
| Preview PR | `omni pr --preview --json` |
| Create PR | `omni pr --apply --json` |
| CI commit | `omni commit --yes --json` |
| CI PR | `omni pr --yes --json` |
```

**2. (Optional) Add Omni as an MCP tool**

If Omni exposes an MCP server, you can register it in your MCP config so Kimi discovers it as a first-class tool: [blog.gitbutler](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)

```json
{
  "mcpServers": {
    "omni": {
      "command": "omni",
      "args": ["mcp-server"]
    }
  }
}
```

Then run Kimi with:

```bash
kimi --mcp-config-file /path/to/mcp.json
```

This makes `omni` available as a tool Kimi can call directly, rather than shelling out via `run_shell_command`. [blog.gitbutler](https://blog.gitbutler.com/automate-your-ai-workflows-with-claude-code-hooks)

**Limitations:** Without hooks, there is no way to hard-block `git commit` at the system level. The agent *should* follow `AGENTS.md` instructions, but it's not guaranteed. Once the [hooks feature request](https://github.com/MoonshotAI/kimi-cli/issues/785) lands, you'll be able to intercept shell commands the same way Claude Code and Cursor do. [github](https://github.com/MoonshotAI/kimi-cli/issues/785)