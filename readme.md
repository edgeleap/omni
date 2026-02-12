# Omni

> The Ultimate Git Experience in the Agentic Era

<a href="https://github.com/edgeleap/omni/releases" target="_blank" rel="noopener noreferrer"><img src="https://img.shields.io/github/v/release/edgeleap/omni?color=4724AB&style=for-the-badge" alt="latest release" /></a>
<a href="https://github.com/edgeleap/omni/releases" target="_blank" rel="noopener noreferrer"><img src="https://img.shields.io/github/downloads/edgeleap/omni/total?color=5832E2&style=for-the-badge" alt="total downloads" /></a>

### About

Omni is a copilot for git. It automatically enriches your commit history into a knowledge graph. Every git action you or your agent makes compounds that graph. The more you use it, the sharper your AI's reasoning becomes.

### Why Omni

1. Omni runs in its own context window — a dedicated sub-agent — so your main agent never wastes tokens on git housekeeping and enrichment. Works across CLI, IDE, Agent, and CI/CD.
2. Omni outperforms any skill + MCP rig you can build yourself — backed by proprietary research and thousands of hours of A/B testing. Because enrichment compounds, your codebase improves automatically over time.
3. Prune your PRs: collapse 100s of commits into 10 razor-sharp commits, or split a messy refactor into clean, logical chunks. (Git worktree support coming soon.)


### Supported Operating Systems

- macOS Apple Silicon / Intel
- Linux x64 / ARM
- Windows x64 / ARM

### Install

**macOS / Linux:**

`curl -fsSL [https://edgeleap.github.io/omni.sh](https://edgeleap.github.io/omni.sh) | bash`

**Windows (PowerShell):**

`irm [https://edgeleap.github.io/omni.ps1](https://edgeleap.github.io/omni.ps1) | iex`


### CLI Commands

See [CLI_Command_Reference.md](CLI_Command_Reference.md) for full command references.

**Commit**

- `omni commit` (quick commit)
- `omni commit --preview` (generate plan + print, do not apply)
- `omni commit --edit` (open plan file for editing / or launch editor)
- `omni commit --apply` (apply plan)

Flags:

- `--json` (agent output)
- `--yes` (CI/non-interactive apply)

**PR**

- `omni pr --preview`
- `omni pr --edit`
- `omni pr --apply`

Flags:
- `--json` (agent output)
- `--yes` (CI/non-interactive apply)

**Release**

- `omni release --preview`
- `omni release --edit`
- `omni release --apply`

Flags:
- `--json` (agent output)
- `--yes` (CI/non-interactive apply)
- `--draft` (GitHub release draft)
- `--tag v1.2.3` (use explicit tag)

### Agents

Omni integrates with coding agents at two levels: **hooks** (system-triggered, hard enforcement) and **instructions** (advisory rules the agent follows). Some agents support both; some support only instructions.

| Agent | Hooks | Instructions | Enforcement | Integration |
| --- | --- | --- | --- | --- |
| [Claude Code](integrations/claude_cli.md) | `PreToolUse` | `CLAUDE.md` | Hard block | Intercept script |
| [Cursor](integrations/cursor_app.md) | `beforeShellExecution` | `.cursor/rules/*.mdc` | Hard block | Intercept script |
| [Cline](integrations/cline_vscode.md) | `PreToolUse` | `.clinerules` | Hard block | Intercept script |
| [Gemini CLI](integrations/gemini_cli.md) | `BeforeTool` | `GEMINI.md` | Hard block | Intercept script |
| [Copilot CLI](integrations/github_copilot_cli.md) | `preToolUse` | `.github/copilot-instructions.md` | Hard block | Intercept script |
| [Copilot Agent](integrations/github_copilot_vscode.md) | `preToolUse` | `.github/copilot-instructions.md` | Hard block | Intercept script |
| [Codex CLI](integrations/codex_cli.md) | — | `AGENTS.md` + `execpolicy` | Partial (sandbox + deny rules) | Policy rules |
| [Codex App](integrations/codex_app.md) | — | Skills + `AGENTS.md` | Advisory (agent doesn't auto-commit) | Skill |
| [Antigravity](integrations/antigravity.md) | — | Rules + Workflows + Skills | Advisory only | Rule + Workflow |
| [Kimi CLI](integrations/kimi_cli.md) | — | `AGENTS.md` | Advisory only | Instruction file |
| [Qwen Code](integrations/qwen_cli.md) | — | Subagents + project config | Advisory only | Subagent |

**How to pick your setup:**

- **Agent supports hooks?** → Use the intercept script to hard-block `git commit` / `gh pr create`, plus an instruction file as backup. The agent literally cannot bypass `omni`.
- **Agent has sandbox/policy rules?** (Codex CLI) → Use `execpolicy` deny rules to block git commands, plus `AGENTS.md` to teach the agent what to use instead.
- **Agent doesn't auto-commit?** (Codex App) → No interception needed. Teach the agent about `omni` through a Skill, and it uses `omni` when you ask it to commit.
- **Advisory only?** (Antigravity, Kimi, Qwen) → Instruction files are your only option. The agent *should* follow them, but there's no system-level guarantee.

### CI/CD

**CI (two-step):**

```bash
omni release --preview --tag v1.2.3 --json > /tmp/release-plan.json
omni release --apply --json
```

> Note: GitHub release creation requires `GITHUB_TOKEN`/`GH_TOKEN` (GitHub Actions provides `GITHUB_TOKEN` automatically).

 
