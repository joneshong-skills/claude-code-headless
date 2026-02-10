# claude-code-headless

Run Claude Code in headless mode (`claude -p`) on macOS. Use it to run Claude Code programmatically, execute headless prompts, get structured JSON output, auto-approve tools, pipe output, create commits via CLI, and integrate Claude Code into scripts, cron jobs, and CI/CD workflows.

## What It Does

This skill provides a reliable way to use the Claude Code CLI non-interactively on macOS. It includes:

- **PTY wrapper** -- A Python script that allocates a pseudo-terminal via macOS BSD `script(1)` to prevent Claude Code from hanging without a TTY.
- **Headless mode** -- Run `claude -p` with structured JSON output, JSON schema typed output, and streaming.
- **Interactive mode** -- Launch Claude Code inside a tmux session for prompts that need interactive input or slash commands.
- **Background mode** -- Fire-and-forget tasks that log to a file and optionally send a macOS desktop notification on completion.
- **macOS integrations** -- Clipboard (pbcopy/pbpaste), desktop notifications (osascript), and `open` for generated files.

## Installation

1. Copy this skill directory to `~/.claude/skills/claude-code-headless/`.
2. Ensure Claude Code CLI is installed:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
3. (Optional) Install tmux for interactive mode:
   ```bash
   brew install tmux
   ```

## Usage

### Headless prompt

```bash
claude -p "Summarize this project"
```

### Using the wrapper script

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  -p "Your prompt here"
```

### Structured JSON output

```bash
claude -p "Summarize this project" --output-format json | jq -r '.result'
```

### Auto-approve specific tools

```bash
claude -p "Run tests and fix failures" --allowedTools "Bash,Read,Edit"
```

### Background mode with notification

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --background --notify \
  -p "Run all tests and fix failures" \
  --allowedTools "Bash,Read,Edit"
```

### Interactive mode via tmux

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --mode interactive \
  --tmux-session my-session \
  -p "Your interactive prompt here"
```

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Skill definition with full reference documentation |
| `scripts/claude_headless.py` | Python wrapper for reliable headless/interactive execution |

## License

This skill is provided as-is for use with Claude Code.
