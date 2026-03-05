---
name: analyzing-claude-sessions
description: Use when analyzing Claude Code session logs to extract insights, debug agent behavior, measure performance, or audit tool usage — provides jq recipes for the JSONL transcript format
---

# Analyzing Claude Code Session Logs

Extract insights from Claude Code session transcripts using jq. Session logs are JSONL files where each line is a message in the conversation.

## Log Locations

| Log Type | Path |
|----------|------|
| Main session | `~/.claude/projects/{project-hash}/{session-id}.jsonl` |
| Subagent | `~/.claude/projects/{project-hash}/{session-id}/subagents/agent-{agent-id}.jsonl` |
| Tool results | `~/.claude/projects/{project-hash}/{session-id}/tool-results/` |

Find the most recent session:
```bash
ls -t ~/.claude/projects/*/*.jsonl | head -1
```

List subagents for a session:
```bash
ls ~/.claude/projects/{project-hash}/{session-id}/subagents/
```

## Message Structure

Each JSONL line has this shape:
```json
{
  "type": "user|assistant|progress|system|file-history-snapshot",
  "message": {
    "role": "user|assistant",
    "content": "string" | [{"type": "text|tool_use|tool_result", ...}],
    "model": "claude-sonnet-4-5-...",
    "usage": {"input_tokens": N, "output_tokens": N, ...},
    "stop_reason": "end_turn|tool_use|..."
  },
  "timestamp": "ISO8601",
  "uuid": "...",
  "agentId": "...",
  "sessionId": "..."
}
```

Assistant messages have `content` as an array of blocks:
- `{"type": "text", "text": "..."}` — text output
- `{"type": "tool_use", "name": "Read", "id": "toolu_...", "input": {...}}` — tool call
- `{"type": "tool_result", "tool_use_id": "toolu_...", "content": "...", "is_error": bool}` — tool response

## Shell Quoting Note

jq filters work fine inline as long as the file path is either a literal or expanded in the same `&&` chain as the variable assignment. The `select(.type == "tool_use")` syntax with inner double quotes works correctly inside single-quoted jq filters.

For array content blocks, use the `if type == "array"` guard since user messages have string content:
```bash
jq -r '.message.content | if type == "array" then [.[] | select(.type == "tool_use") | .name] | .[] else empty end' session.jsonl
```

For loops over subagent files, assign the directory path first:
```bash
SUBS="path/to/subagents" && for f in "$SUBS"/agent-*.jsonl; do
  jq -r '...' "$f"
done
```

## Quick Reference: jq Recipes

### Session Overview

```bash
# Message type distribution
jq -r '.type' session.jsonl | sort | uniq -c | sort -rn

# Time span
jq -r '.timestamp' session.jsonl | sed -n '1p;$p'

# Model used
jq -r '.message.model // empty' session.jsonl | sort -u
```

### Tool Usage

```bash
# Tool call frequency
jq -r '.message.content[]? | select(.type == "tool_use") | .name' session.jsonl | sort | uniq -c | sort -rn

# What files were read
jq -r '.message.content[]? | select(.type == "tool_use" and .name == "Read") | .input.file_path' session.jsonl

# Bash commands run
jq -r '.message.content[]? | select(.type == "tool_use" and .name == "Bash") | .input.command' session.jsonl

# Files written
jq -r '.message.content[]? | select(.type == "tool_use" and .name == "Write") | .input.file_path' session.jsonl

# Tool errors
jq -r '.message.content[]? | select(.type == "tool_result" and .is_error == true) | .content[:200]' session.jsonl
```

### Agent Dispatches (main session only)

```bash
# Subagent launch sequence
jq -r 'select(.message.content[]? | .type == "tool_use" and .name == "Agent") | "\(.timestamp) \(.message.content[] | select(.type == "tool_use" and .name == "Agent") | .input.description)"' session.jsonl

# Agent types used
jq -r '.message.content[]? | select(.type == "tool_use" and .name == "Agent") | .input.subagent_type' session.jsonl | sort | uniq -c | sort -rn
```

### Token Usage

```bash
# Total output tokens
jq -r 'select(.type == "assistant") | .message.usage.output_tokens // 0' session.jsonl | paste -sd+ | bc

# Cache hit rate
jq -r 'select(.type == "assistant") | .message.usage | "\(.cache_read_input_tokens // 0) cached / \(.input_tokens // 0) total"' session.jsonl
```

### Subagent Analysis

```bash
# Role and cost per subagent
SUBS="path/to/subagents" && for f in "$SUBS"/agent-*.jsonl; do
  turns=$(wc -l < "$f" | tr -d ' ')
  role=$(head -1 "$f" | jq -r '.message.content[:80]')
  echo "$(basename "$f" .jsonl) ($turns turns): $role"
done

# Tool usage per subagent
SUBS="path/to/subagents" && for f in "$SUBS"/agent-*.jsonl; do
  echo "--- $(basename "$f") ---"
  jq -r '.message.content | if type == "array" then [.[] | select(.type == "tool_use") | .name] | .[] else empty end' "$f" | sort | uniq -c | sort -rn
done

# Files written per subagent
SUBS="path/to/subagents" && for f in "$SUBS"/agent-*.jsonl; do
  files=$(jq -r '.message.content | if type == "array" then [.[] | select(.type == "tool_use" and (.name == "Write" or .name == "Edit")) | "\(.name): \(.input.file_path | split("/") | last)"] | .[] else empty end' "$f" 2>/dev/null)
  [ -n "$files" ] && echo "--- $(basename "$f") ---" && echo "$files"
done

# Token totals per subagent
SUBS="path/to/subagents" && for f in "$SUBS"/agent-*.jsonl; do
  jq -r 'select(.type == "assistant") | [(.message.usage.input_tokens // 0), (.message.usage.cache_read_input_tokens // 0), (.message.usage.output_tokens // 0)] | @csv' "$f" | awk -F, -v name="$(basename "$f")" '{i+=$1; c+=$2; o+=$3} END{printf "%s: %d in (%d cached), %d out\n", name, i, c, o}'
done
```

### Content Search

```bash
# Find messages containing a keyword
jq -r 'select(.message.content | tostring | test("keyword"; "i")) | "\(.timestamp) [\(.type)]"' session.jsonl

# Extract all assistant text blocks
jq -r 'select(.type == "assistant") | .message.content[]? | select(.type == "text") | .text' session.jsonl

# Find tool errors
jq -r '.message.content[]? | select(.type == "tool_result") | .content' session.jsonl | grep -i "error"
```

## Analysis Workflows

### Performance Audit

1. Count total tool calls vs turns — high tool:turn ratio means parallel calls (good), low means sequential
2. Check for redundant reads: same file path appearing multiple times
3. Check for Bash doing what dedicated tools handle (cat/grep/head instead of Read/Grep)
4. Look for retry loops: same tool called with same args multiple times
