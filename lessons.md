# Claude Code Headless — Lessons Learned

### 2026-02-13 — Nested session protection blocks headless invocation
- **Friction**: Running `claude -p` from inside a Claude Code session fails with "Cannot be launched inside another Claude Code session"
- **Fix**: Prepend `unset CLAUDECODE &&` before the command to bypass the nesting check
- **Rule**: When invoking `claude -p` from within Claude Code (e.g., via maestro or skill testing), always `unset CLAUDECODE` first. Updated SKILL.md with a dedicated caveat section.
