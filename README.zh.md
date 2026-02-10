[English](README.md) | [繁體中文](README.zh.md)

# claude-code-headless

在 macOS 上以 headless 模式（`claude -p`）執行 Claude Code。用於程式化執行 Claude Code、執行 headless 提示詞、取得結構化 JSON 輸出、自動核准工具、管線輸出、透過 CLI 建立提交，以及將 Claude Code 整合到腳本、排程任務和 CI/CD 工作流程中。

## 功能特色

此技能提供在 macOS 上可靠地非互動式使用 Claude Code CLI 的方式，包括：

- **PTY 封裝器** — 透過 macOS BSD `script(1)` 分配偽終端的 Python 腳本，防止 Claude Code 在無 TTY 時掛起。
- **Headless 模式** — 使用結構化 JSON 輸出、JSON Schema 型別輸出和串流模式執行 `claude -p`。
- **互動模式** — 在 tmux 工作階段中啟動 Claude Code，用於需要互動輸入或斜線命令的提示。
- **背景模式** — 即發即忘的任務，記錄到檔案並可選擇在完成時發送 macOS 桌面通知。
- **macOS 整合** — 剪貼簿（pbcopy/pbpaste）、桌面通知（osascript），以及用 `open` 開啟產生的檔案。

## 安裝

1. 將此技能目錄複製到 `~/.claude/skills/claude-code-headless/`。
2. 確認已安裝 Claude Code CLI：
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
3. （可選）安裝 tmux 以使用互動模式：
   ```bash
   brew install tmux
   ```

## 使用方式

### Headless 提示

```bash
claude -p "Summarize this project"
```

### 使用封裝腳本

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  -p "Your prompt here"
```

### 結構化 JSON 輸出

```bash
claude -p "Summarize this project" --output-format json | jq -r '.result'
```

### 自動核准特定工具

```bash
claude -p "Run tests and fix failures" --allowedTools "Bash,Read,Edit"
```

### 背景模式加桌面通知

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --background --notify \
  -p "Run all tests and fix failures" \
  --allowedTools "Bash,Read,Edit"
```

### 透過 tmux 的互動模式

```bash
python3 ~/.claude/skills/claude-code-headless/scripts/claude_headless.py \
  --mode interactive \
  --tmux-session my-session \
  -p "Your interactive prompt here"
```

## 檔案

| 檔案 | 說明 |
|------|------|
| `SKILL.md` | 技能定義及完整參考文件 |
| `scripts/claude_headless.py` | 用於可靠 headless/互動執行的 Python 封裝器 |

## 授權

本技能以現狀提供，供 Claude Code 使用。
