<!--
Omni v2 CLI Command Reference

Audience
- Humans (interactive)
- AI agents (automation; prefers JSON)
- CI/CD (non-interactive; reproducible)

Scope
- Documents the behavior implemented in `src-tauri/src/bin/omni.rs`.
-->

# Omni v2 — CLI Command Reference

This guide is written so a new user can start from zero, but it also serves as an exact reference for agents and CI.

> Command name: in dev builds the executable may be `omni-cli`, but the command surface is `omni`.

---

## 0) TL;DR (quickstart)

```bash
cd /path/to/your/repo

# Pick a model once
omni model list
omni model use ollama:gemma3:1b

# Preview a plan (does not create commits)
omni commit --preview

# Apply the plan (creates real git commits)
omni commit --apply

# Optional: push the branch after applying
omni commit --apply --push
```

---

## 0.1) CI/CD (remote models) — install + environment

### Install (CLI-only)

For CI/CD you typically want the **standalone CLI-only** install (no GUI, no AFM sidecar).

```bash
curl -fsSL https://edgeleap.github.io/omni.sh | bash -s -- --cli
```

Why `bash -s -- --cli`?

- `bash -s` tells bash to read the script from stdin
- `--` ends bash’s own option parsing
- `--cli` becomes `$1` inside the installer script

### Required environment variables

**GitHub (required for `omni pr --apply` and `omni release --apply`):**

- `GITHUB_TOKEN` (preferred in GitHub Actions)
- or `GH_TOKEN`

**Remote model (bring-your-own-key; pick one):**

- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`
- Google/Gemini: `GOOGLE_API_KEY`

### Minimal CI example (bash)

```bash
export CI=true
export GITHUB_TOKEN="${GITHUB_TOKEN}"
export OPENAI_API_KEY="${OPENAI_API_KEY}"

omni model use remote:my-openai
omni commit --yes --split --push
omni pr --apply --json
```

### GitHub Actions example

```yaml
name: omni
on:
  workflow_dispatch:

jobs:
  omni:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4

      - name: Install omni (CLI-only)
        run: |
          curl -fsSL https://edgeleap.github.io/omni.sh | bash -s -- --cli
          echo "$HOME/.local/bin" >> $GITHUB_PATH

      - name: Run omni
        env:
          CI: "true"
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          omni model use remote:my-openai
          omni commit --yes --split --push
          omni pr --apply --json
```

---

## 1) Concepts (read once)

### 1.1 Files Omni writes

| File | Location | Purpose |
|---|---|---|
| Commit plan | `<repo>/.omni/state/commit.json` | Generated commit(s) title/body + file list |
| PR plan | `<repo>/.omni/state/pr.json` | Generated PR title/body |
| Release plan | `<repo>/.omni/state/release.json` | Generated release tag/title/body |

### 1.2 Settings

Settings are per-user:

| OS | Path |
|---|---|
| macOS | `~/Library/Application Support/omni/settings.json` |
| Linux | `~/.local/share/omni/settings.json` |
| Windows | `%APPDATA%\\omni\\settings.json` |

### 1.3 Model references

| Model ref | Meaning | Notes |
|---|---|---|
| `afm` | Local AFM sidecar | macOS + Apple Silicon only |
| `ollama:<model>` | Local Ollama model | e.g. `ollama:gemma3:1b` |
| `remote:<id>` | Remote model configured in Omni settings | requires remote config + key |

### 1.4 Output rules (agents/CI)

- `--json` prints machine-readable JSON to **stdout**.
- Progress (spinner) and internal logs print to **stderr**.

---

## 2) Help

| Command | What it does |
|---|---|
| `omni --help` | show CLI usage |
| `omni -h` | alias |
| `omni help` | alias |

---

## 3) Pick a model (setup)

### 3.1 Model commands (simple → advanced)

| Command | What it does | Output |
|---|---|---|
| `omni model --current` | print current default model ref | text |
| `omni model --current --json` | print `{ "default_model": "..." }` | JSON |
| `omni model list` | list usable model refs | text |
| `omni model list --json` | list usable model refs | JSON |
| `omni model use afm` | set default model to AFM | `ok` |
| `omni model use ollama:<model>` | set default model to Ollama | `ok` |
| `omni model use remote:<id>` | set default model to remote model | `ok` |

### 3.2 Remote models (BYOK) commands

| Command | What it does | Output |
|---|---|---|
| `omni remote list` | list configured remote models | text |
| `omni remote list --json` | list configured remote models | JSON |
| `omni remote add <id> <provider> <modelId> <name...>` | add/update remote model metadata | `ok` |
| `omni remote set-key <id> [apiKey]` | store key in OS keychain | `ok` |
| `OMNI_REMOTE_API_KEY=... omni remote set-key <id>` | same via env var | `ok` |
| `omni remote test <id>` | test key + modelId works | text |
| `omni remote test <id> --json` | same as JSON | JSON |
| `omni remote rm <id>` | remove remote model metadata | `ok` |
| `omni remote rm <id> --delete-key` | also delete key from keychain | `ok` |

Providers supported today: `openai`, `anthropic`, `google`.

Keychain entry name:

- `remote-model:<id>` (service: `com.omni.app`)

---

## 4) Commit workflows

### 4.1 What files does `omni commit` include?

By default, Omni uses the same file set as the GUI:

- tracked **unstaged** changes
- tracked **staged** changes
- **untracked** files (but not gitignored)

### 4.2 Commit flags

| Flag | Meaning |
|---|---|
| `--preview` | generate plan only (default behavior) |
| `--apply` | apply an existing saved plan |
| `--edit` | print the plan file path |
| `--yes` | one-shot: generate plan then apply immediately |
| `--split` (alias `--auto-split`) | auto-split into multiple commits (batch size = 3 files) |
| `--push` | after apply, push the current branch to origin |
| `--json` | print JSON to stdout |

### 4.3 Commit commands (simple → advanced)

| Command | What it does | Best for |
|---|---|---|
| `omni commit` | preview (generate plan) | humans |
| `omni commit --preview` | preview (explicit) | humans |
| `omni commit --preview --split` | preview multi-commit plan | humans |
| `omni commit --preview --json` | preview + plan JSON | agents |
| `omni commit --preview --split --json` | split + plan JSON | agents |
| `omni commit --edit` | print plan file path | humans/agents |
| `omni commit --apply` | apply saved plan | humans |
| `omni commit --apply --push` | apply + push | humans/CI |
| `omni commit --yes` | generate + apply | CI/agents |
| `omni commit --yes --split` | generate split plan + apply | CI/agents |
| `omni commit --yes --split --push` | generate + apply + push | CI/agents |

### 4.4 Recommended flows

**Human (review first):**

```bash
omni commit --preview --split
omni commit --apply
omni commit --apply --push
```

**Agent (capture plan JSON):**

```bash
omni commit --preview --split --json > /tmp/commit-plan.json
```

**CI (one-shot):**

```bash
omni commit --yes --split --push
```

---

## 5) PR workflows

PR has two phases:

1) preview = generate a PR plan (title/body)
2) apply = create the PR on GitHub using that plan

### 5.1 PR commands (simple → advanced)

| Command | What it does | Best for |
|---|---|---|
| `omni pr` | preview (generate PR plan) | humans |
| `omni pr --preview` | preview (explicit) | humans |
| `omni pr --preview --json` | preview + plan JSON | agents |
| `omni pr --edit` | print PR plan path | humans/agents |
| `omni pr --apply` | create PR from saved plan | humans |
| `omni pr --apply --json` | create PR and print JSON response | CI/agents |
| `omni pr --yes` | apply-only alias for `--apply` | CI |

### 5.2 Recommended flows

**Human:**

```bash
omni pr --preview
omni pr --apply
```

**CI (two-step; `pr --yes` is apply-only today):**

```bash
omni pr --preview --json > /tmp/pr-plan.json
omni pr --apply --json
```

---

## 6) Release workflows

Release also has two phases:

1) preview = generate a release plan (tag/title/body)
2) apply = create the GitHub release (or draft)

### 6.1 Release commands (simple → advanced)

| Command | What it does | Best for |
|---|---|---|
| `omni release --preview --tag v1.2.3` | preview using explicit tag | humans/CI |
| `omni release --preview --json` | preview + plan JSON | agents |
| `omni release --edit` | print plan path | humans/agents |
| `omni release --apply` | create GitHub release from saved plan | humans |
| `omni release --apply --draft` | create GitHub release draft | humans |
| `omni release --apply --json` | create release and print JSON response | CI/agents |

### 6.2 Recommended flows

**Human:**

```bash
omni release --preview --tag v1.2.3
omni release --apply
```

**CI (two-step):**

```bash
omni release --preview --tag v1.2.3 --json > /tmp/release-plan.json
omni release --apply --json
```

> Note: GitHub release creation requires `GITHUB_TOKEN`/`GH_TOKEN` (GitHub Actions provides `GITHUB_TOKEN` automatically).

---

## 7) Troubleshooting

### Nothing to commit / gitignore filtering

Omni maps common errors like “no files after gitignore filtering” / “nothing to commit” into actionable messages.

Typical fix:

```bash
git status
git add <files>
omni commit --preview
```

### PR already exists (GitHub 422)

If GitHub returns 422 during PR creation, Omni shows a friendly “PR already exists” message.

---

## 8) Debug environment variables

| Env var | Purpose |
|---|---|
| `OMNI_DEBUG_PROMPTS=1` | enable prompt tap (writes temp prompt files) |
| `OMNI_DEBUG_AFM=1` | keep AFM payload temp file |
| `OMNI_AFM_SIDECAR_PATH=/abs/path/to/omni` | override AFM sidecar path |
| `OMNI_REMOTE_API_KEY=...` | provide key for `omni remote set-key <id>` |