### Codex CLI

Codex CLI doesn't have `PreToolUse`-style hooks that intercept and redirect commands mid-execution. Instead, it uses **`AGENTS.md`** (instruction layer), **sandbox policies** (filesystem enforcement), and **`execpolicy` rules** (command-level enforcement). [blott](https://www.blott.com/blog/post/openai-codex-cli-build-faster-code-right-from-your-terminal)

**1. `AGENTS.md` (instruction layer)**

Create an `AGENTS.md` at your repo root so Codex knows to use `omni` instead of `git`:

```markdown
# AGENTS.md

## Git Policy
- Never run `git commit`, `git push`, or `gh pr create` directly.
- For commits, run `omni commit --yes --json`.
- For pull requests, run `omni pr --yes --json`.
- See the command expectation matrix in the project docs for all available flags.
```

Codex reads `AGENTS.md` files at the start of every session and merges global (`~/.codex/AGENTS.md`) with project-level files, so this guidance is always present. [mlearning.substack](https://mlearning.substack.com/p/100-openai-codex-cli-tricks-and-tips)

**2. `execpolicy` rules (command enforcement)**

Codex supports `execpolicy` rule files that control whether a command is allowed, prompted, or blocked. Create `.codex/rules/omni-git.toml`:

```toml
[[rule]]
pattern = "git commit*"
decision = "deny"
reason = "Use 'omni commit --yes --json' instead of git commit."

[[rule]]
pattern = "git push*"
decision = "deny"
reason = "Use 'omni pr --yes --json' instead of git push."

[[rule]]
pattern = "gh pr create*"
decision = "deny"
reason = "Use 'omni pr --yes --json' instead of gh pr create."

[[rule]]
pattern = "omni *"
decision = "allow"
```

You can verify your rules with: [blott](https://www.blott.com/blog/post/openai-codex-cli-build-faster-code-right-from-your-terminal)

```bash
codex execpolicy check --rules .codex/rules/omni-git.toml -- git commit -m "test"
```

**3. Sandbox + approval policy (filesystem enforcement)**

In `workspace-write` sandbox mode, Codex keeps `.git/` and `.codex/` read-only, which means `git commit` will fail unless explicitly approved: [mlearning.substack](https://mlearning.substack.com/p/100-openai-codex-cli-tricks-and-tips)

```toml
# .codex/config.toml
approval_policy = "untrusted"
sandbox_mode    = "workspace-write"
```

With `approval_policy = "untrusted"`, Codex prompts before running *any* shell command, giving you a manual gate to reject raw `git` calls. For CI/automated flows, use `--full-auto` but keep the `AGENTS.md` and `execpolicy` rules tight. [mlearning.substack](https://mlearning.substack.com/p/100-openai-codex-cli-tricks-and-tips)

**How it compares to Claude Code:**

| Aspect | Claude Code | Codex CLI |
| --- | --- | --- |
| Hard block on commands | `PreToolUse` hook exits with code 2 [code.claude](https://code.claude.com/docs/en/hooks) | `execpolicy` rules deny specific commands  [blott](https://www.blott.com/blog/post/openai-codex-cli-build-faster-code-right-from-your-terminal) |
| Instruction layer | `CLAUDE.md` rules (advisory) [code.claude](https://code.claude.com/docs/en/hooks) | `AGENTS.md` files, merged per directory (advisory)  [mlearning.substack](https://mlearning.substack.com/p/100-openai-codex-cli-tricks-and-tips) |
| Filesystem enforcement | N/A | Sandbox keeps `.git/` read-only  [mlearning.substack](https://mlearning.substack.com/p/100-openai-codex-cli-tricks-and-tips) |
| CI mode | `--yes --json` flags in omni | `--full-auto` or `codex exec` with `--json`  [blott](https://www.blott.com/blog/post/openai-codex-cli-build-faster-code-right-from-your-terminal) |

**Bottom line:** Codex can't *redirect* a `git commit` call to `omni commit` mid-execution the way Claude Code hooks can. But with `execpolicy` rules blocking `git commit` and `AGENTS.md` instructing the agent to use `omni` instead, you get a comparable two-layer safety net. [blott](https://www.blott.com/blog/post/openai-codex-cli-build-faster-code-right-from-your-terminal)