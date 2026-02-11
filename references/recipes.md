# Claude Code Headless -- Recipes & Reference

Extended examples and reference material for the `claude-code-headless` skill.
For core workflow and basics, see the main [SKILL.md](../SKILL.md).

---

## Common Recipes

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

## macOS-Specific Integrations

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

## Background Mode (Non-Blocking)

Run tasks in the background -- the wrapper returns immediately with PID and log path.

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

### Background mode flags

| Flag | Description |
|------|-------------|
| `--background` / `--bg` | Run in background, return immediately |
| `--log-dir <path>` | Log directory (default: `~/.claude/logs/headless`) |
| `--notify` | macOS notification when background task finishes |

---

## Looking Up Claude Code Documentation

When encountering unfamiliar flags, features, hooks, SDK usage, MCP server configuration, or any Claude Code topic not covered in SKILL.md, use the **smart-search** skill.

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

### When to look up documentation

- A flag or option is not documented in the main skill
- The user asks about hooks, SDK, or MCP server integration
- Troubleshooting unexpected behavior or error messages
- Verifying the latest syntax or available options
- Any Claude Code feature beyond basic headless mode usage
