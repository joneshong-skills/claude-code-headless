---
name: claude-code-headless
description: "Run Claude Code in headless mode (`claude -p`) on macOS. Use when the user asks to run Claude Code programmatically, execute headless prompts, use `claude -p`, get structured JSON output, auto-approve tools with --allowedTools, pipe output, create commits via CLI, integrate Claude Code into scripts, cron jobs, and CI/CD workflows on macOS, or asks about Claude Code CLI features, flags, hooks, SDK, or MCP server configuration."
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
npm install -g @anthropic-ai/claude-code
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

### Copy output to clipboard (pbcopy)

```bash
claude -p "Summarize this project" | pbcopy
```

### Paste clipboard as input (pbpaste)

```bash
pbpaste | claude -p "Review this code"
```

### Open generated files

```bash
claude -p "Generate an HTML report" --allowedTools "Write"
open report.html
```

### Desktop notification on completion (osascript)

```bash
claude -p "Run all tests and fix failures" --allowedTools "Bash,Read,Edit"; \
  osascript -e 'display notification "Claude Code task finished" with title "Claude Code"'
```

### Combine: run, notify, and copy result

```bash
result=$(claude -p "Summarize this project" --output-format json | jq -r '.result'); \
  echo "$result" | pbcopy; \
  osascript -e 'display notification "Result copied to clipboard" with title "Claude Code"'
```

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

### Git commit from staged changes

```bash
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

### Code review a PR

```bash
gh pr diff 123 | claude -p "Review this PR for bugs and security issues" \
  --append-system-prompt "You are a senior engineer. Be thorough." \
  --output-format json | jq -r '.result'
```

### Fix test failures

```bash
claude -p "Run the test suite and fix any failures" \
  --allowedTools "Bash,Read,Edit"
```

### Explain + Plan first, then implement

```bash
# Step 1: plan (read-only)
claude -p "Analyze the auth system and propose improvements" --permission-mode plan

# Step 2: implement (continue the conversation)
claude -p "Implement the plan" --continue --allowedTools "Read,Edit,Bash"
```

### Batch processing with a loop

```bash
for file in src/**/*.py; do
  claude -p "Add type hints to $file" \
    --allowedTools "Read,Edit" \
    --output-format json | jq -r '.result'
done
```

### Pipe build errors for diagnosis

```bash
npm run build 2>&1 | claude -p "Explain the root cause and suggest a fix"
```

---

## Background mode (non-blocking)

Run tasks in the background — the wrapper returns immediately with PID and log path.

```bash
# Fire-and-forget with notification when done
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --background --notify \
  -p "Run all tests and fix failures" \
  --allowedTools "Bash,Read,Edit"

# Custom log directory
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --background --log-dir /tmp/claude-logs \
  -p "Refactor the auth module" \
  --allowedTools "Read,Edit"
```

Output:
```
Background process started:
  PID:  12345
  Log:  ~/.claude/logs/headless/claude-20260210-143022.log
  Tail: tail -f ~/.claude/logs/headless/claude-20260210-143022.log
  Stop: kill 12345
```

| Flag | Description |
|------|-------------|
| `--background` / `--bg` | Run in background, return immediately |
| `--log-dir <path>` | Log directory (default: `~/.claude/logs/headless`) |
| `--notify` | macOS notification when background task finishes |

---

## Best practices

1. **Always give Claude a way to verify** -- include tests or build commands in your prompt
2. **Explore -> Plan -> Implement** -- use `--permission-mode plan` first, then switch to execution
3. **Keep `--allowedTools` narrow** -- principle of least privilege
4. **Use `--output-format json`** for machine-readable results in scripts
5. **Capture `session_id`** when you need multi-turn conversations
6. **Use CLAUDE.md** for persistent per-project rules (build commands, style guides)
7. **Pipe liberally** -- treat `claude -p` as a Unix utility

---

## Looking up Claude Code documentation via DeepWiki

When encountering unfamiliar flags, features, hooks, SDK usage, MCP server configuration, or any Claude Code topic not covered above, query the official repository documentation using the DeepWiki MCP server.

### Query the repo

```
mcp__deepwiki__ask_question
  repoName: "anthropics/claude-code"
  question: "<specific question about Claude Code>"
```

### Example queries

```
"What CLI flags are available for claude -p headless mode?"
"How do hooks work in Claude Code? Show PreToolUse and PostToolUse examples."
"How to configure MCP servers in Claude Code settings?"
"What is the Claude Code Agent SDK and how to build custom agents?"
"How does the permission system work with --allowedTools syntax?"
"How to use CLAUDE.md files for project-level configuration?"
"What keyboard shortcuts and slash commands are available?"
"How does context compaction work in Claude Code?"
```

### Browse available topics

```
mcp__deepwiki__read_wiki_structure
  repoName: "anthropics/claude-code"
```

Use this to discover which documentation sections exist before asking a targeted question.

### When to query DeepWiki

- A flag or option is not documented in this skill
- The user asks about hooks, SDK, or MCP server integration
- Troubleshooting unexpected behavior or error messages
- Verifying the latest syntax or available options
- Any Claude Code feature beyond basic headless mode usage
