---
name: claude-code-headless
description: >-
  This skill should be used when the user asks to "run Claude Code headless",
  "use claude -p", "execute a headless prompt", "Claude Code 腳本",
  "pipe output from Claude Code", "auto-approve tools",
  or discusses running Claude Code programmatically, integrating into scripts,
  cron jobs, CI/CD workflows, structured JSON output, or Claude Code CLI
  features, flags, hooks, SDK, and MCP server configuration on macOS.
version: 0.1.0
tools: Bash
argument-hint: "[prompt or flags]"
---

# Claude Code Headless (macOS)

Use the locally installed **Claude Code** CLI (`claude -p`) in headless (non-interactive) mode on **macOS**.

> This skill is for driving the `claude` CLI programmatically, not the Claude API directly.

## Environment checks

```bash
# Verify claude is installed
which claude        # Expected: /Users/$USER/.local/bin/claude
claude --version

# Quick smoke test
claude -p "Return only the single word OK."
```

If `claude` is not found, install via:
```bash
# Recommended (native installer)
curl -fsSL https://claude.ai/install.sh | bash

# Or via Homebrew
brew install --cask claude-code
```

---

## Headless mode basics

Add `-p` (or `--print`) to run non-interactively. All CLI options work with `-p`.

### Simple prompt

```bash
claude -p "What does the auth module do?"
```

### Run in a specific directory

```bash
claude -p "Summarize this project" --cwd /path/to/repo
```

---

## PTY wrapper (macOS-specific)

Claude Code may hang without a TTY. On **macOS (BSD)**, use `script` to allocate a pseudo-terminal:

```bash
# macOS BSD syntax (different from Linux!)
script -q /dev/null claude -p "Your prompt here"
```

**Do NOT use Linux syntax** (`script -q -c "cmd" /dev/null`) -- it will fail on macOS.

A wrapper script is provided that handles this automatically:

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  -p "Your prompt here"
```

---

## Structured output

### JSON output

```bash
claude -p "Summarize this project" --output-format json
```

Response includes `result` (text), `session_id`, and metadata. Extract with `jq`:

```bash
claude -p "Summarize this project" --output-format json | jq -r '.result'
```

### JSON Schema (typed output)

```bash
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'
```

### Streaming

```bash
claude -p "Explain recursion" \
  --output-format stream-json --verbose --include-partial-messages
```

Filter for text deltas only:

```bash
claude -p "Write a poem" \
  --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

---

## Auto-approve tools

Use `--allowedTools` with permission rule syntax. Trailing ` *` enables prefix matching.

```bash
# Allow read, edit, and bash
claude -p "Run tests and fix failures" \
  --allowedTools "Bash,Read,Edit"

# Allow specific git commands only
claude -p "Look at staged changes and create a commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

> Keep `--allowedTools` narrow (principle of least privilege), especially in automation.

---

## Custom system prompt

```bash
# Append to default system prompt
gh pr diff "$1" | claude -p \
  --append-system-prompt "You are a security engineer. Review for vulnerabilities." \
  --output-format json

# Replace system prompt entirely
claude -p "Analyze this code" \
  --system-prompt "You are a code reviewer focused on performance."
```

---

## Continue / resume conversations

```bash
# Continue the most recent conversation
claude -p "Now focus on the database queries" --continue

# Capture session ID and resume later
session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')
claude -p "Continue that review" --resume "$session_id"
```

---

## macOS-specific integrations

```bash
# Copy output to clipboard
claude -p "Summarize this project" | pbcopy

# Paste clipboard as input
pbpaste | claude -p "Review this code"
```

See `references/recipes.md` for more macOS patterns (notifications, open files, combined workflows).

---

## Permission modes

Use `--permission-mode` to control what Claude can do:

| Mode | Description |
|------|-------------|
| `plan` | Read-only planning, no edits |
| `acceptEdits` | Allow file edits, ask for bash |
| `bypassPermissions` | Allow everything (use with caution) |

```bash
# Safe read-only analysis
claude -p "Analyze and propose a plan" --permission-mode plan

# Allow edits
claude -p "Refactor the auth module" --permission-mode acceptEdits --allowedTools "Read,Edit"
```

---

## Interactive mode (tmux)

For slash commands or long-running tasks that need interactive input, use the Python wrapper with tmux:

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --mode interactive \
  --tmux-session my-session \
  --permission-mode acceptEdits \
  --allowedTools "Bash,Read,Edit,Write" \
  -p "Your interactive prompt here"
```

Monitor:
```bash
tmux attach -t my-session
```

---

## Common recipes

```bash
# Git commit from staged changes
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"

# Pipe build errors for diagnosis
npm run build 2>&1 | claude -p "Explain the root cause and suggest a fix"
```

See `references/recipes.md` for more recipes (PR review, batch processing, plan-then-implement, fix test failures).

---

## Background mode (non-blocking)

Run tasks in the background with `--background` (or `--bg`). The wrapper returns immediately with PID and log path.

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --background --notify \
  -p "Run all tests and fix failures" \
  --allowedTools "Bash,Read,Edit"
```

See `references/recipes.md` for detailed background mode examples, output format, and flag reference.

---

## Best practices

1. **Always give Claude a way to verify** -- include tests or build commands in your prompt
2. **Explore -> Plan -> Implement** -- use `--permission-mode plan` first, then switch to execution
3. **Keep `--allowedTools` narrow** -- principle of least privilege
4. **Use `--output-format json`** for machine-readable results in scripts
5. **Capture `session_id`** for multi-turn conversations
6. **Use CLAUDE.md** for persistent per-project rules (build commands, style guides)
7. **Pipe liberally** -- treat `claude -p` as a Unix utility

---

## Looking up Claude Code documentation

For unfamiliar flags, hooks, SDK, or MCP configuration, use the **smart-search** skill to query documentation. See `references/recipes.md` for example queries and guidance on when to look up docs.

---

## Additional Resources

- **[references/recipes.md](references/recipes.md)** -- Common recipes, macOS integration patterns, background mode details, and documentation lookup examples
