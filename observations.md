# Observations — claude-code-headless

## Pending

### 2026-02-11 — npm install deprecated, native installer recommended
- **Category**: tech
- **Evidence**: External research confirms Anthropic recommends native installer (curl/Homebrew) over npm. Current SKILL.md line 26 still shows `npm install -g @anthropic-ai/claude-code`.
- **Research**: npm package @anthropic-ai/claude-code 2.1.38 still published, but official docs recommend native installer. Agent SDK 0.2.38 also available.
- **Confidence**: Medium
- **Trigger**: Verify via official Claude Code docs. If npm install is officially deprecated with a warning, update the skill.

## Resolved
