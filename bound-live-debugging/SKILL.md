---
name: bound-live-debugging
description: Use when debugging a running bound agent instance — covers database inspection, MCP-based probing, TDD fix cycles, and techniques for discovering issues through live agent conversation.
---

# Live Debugging a Bound Agent

## Overview

Debug a running bound instance by combining direct database inspection, MCP-based agent conversations, and targeted TDD fix cycles. Core loop: probe via conversation, verify via DB, fix with TDD, restart and re-probe.

## When to Use

- Bound agent exhibiting unexpected behavior
- Investigating command failures, context assembly issues, or memory gaps
- Verifying fixes end-to-end against a live instance
- Probing for undiscovered issues through open-ended agent interaction

## Setup

Start bound from the data directory so config paths resolve:

```bash
cd ~/bound && bun /path/to/bound/packages/cli/src/bound.ts start 2>&1 &
```

Kill and restart after each code change. Bun runs TypeScript source directly.

## Core Patterns

### 1. Conversation as Probe

Use `bound_chat` (Polaris MCP) to hold diagnostic conversations. The agent is itself a capable debugger — ask it to inspect its own state with `query`, `memorize`, `forget`, `hostinfo`, `commands`, `advisory`, `skill-list`.

### 2. DB as Ground Truth

`bound_chat` may return stale content from a previous turn (polling race). Always verify via direct DB query:

```bash
THREAD=$(sqlite3 ~/bound/data/bound.db "SELECT id FROM threads WHERE interface='mcp' ORDER BY created_at DESC LIMIT 1;")
sqlite3 ~/bound/data/bound.db "SELECT substr(content,1,1000) FROM messages WHERE thread_id='$THREAD' AND role='assistant' ORDER BY created_at DESC LIMIT 1;"
```

### 3. Schema Before Query

Always check `.schema <table>` before querying unfamiliar tables. Common traps: `advisories.proposed_at` (not `created_at`), `tasks` has no `name` column, `turns.tokens_cache_write`/`tokens_cache_read` (may be NULL).

### 4. Agent Self-Report

Ask open-ended questions about functioning. Agents notice real issues: "Event emitted: undefined" exposed a parsing bug. "purge requires --thread-id" exposed a singleton context gap. Watch for repeated analysis across conversations — signals missing cross-session memory.

### 5. TDD Fix Cycle

RED (failing test) → GREEN (minimal fix) → typecheck (`bun run typecheck`) → full suite (`bun test packages/agent packages/sandbox --recursive`) → commit → restart bound → re-probe via MCP.

### 6. Web API for Feedback Loops

Some tests require the web API to simulate operator actions:

```bash
curl -s -X POST "http://localhost:3000/api/advisories/$ID/approve"
curl -s -X POST "http://localhost:3000/api/advisories/$ID/apply"
```

Advisory lifecycle: proposed → approved → applied (two steps required).

## Effective Probe Sequences

**System health sweep:** Ask the agent to run `query` for task counts by status, memory count, `hostinfo`, and `commands`.

**Memory lifecycle:** `memorize` a key → `query` to verify → `forget` the key → `query` to confirm deletion.

**Advisory feedback loop:** Agent posts `advisory` → apply via web API (approve then apply) → next turn agent sees `[Advisory notification]`.

**Cross-thread continuity:** `memorize` in thread A → start thread B → check if value appears in Recent Activity Digest (not memory delta — delta baseline is thread creation time).

**Schedule+cancel:** `schedule --in 1h --payload "test"` → `cancel <id>` → `query` to verify status=cancelled.

**Concurrent behavior:** Send two messages to same thread while loop is active — second should be re-queued (findPendingUserMessage in finally block).

## Common Pitfalls

| Pitfall | Mitigation |
|---------|------------|
| Trusting MCP response text | Verify via DB — polling may return stale turn |
| Wrong column names in queries | `.schema <table>` before every unfamiliar query |
| Assuming ctx.threadId is set | Verify loopContextStorage wiring (AsyncLocalStorage) |
| Missing typecheck | `bun run typecheck` — test files excluded from tsc |
| Forgetting to restart bound | Changes require process restart |
| Single-turn MCP threads | Cache breakpoints only activate with ≥2 history messages |

## Quick Reference: Tool Combinations

| Goal | Tools |
|------|-------|
| Verify command output | `bound_chat` + `sqlite3` DB check |
| Test advisory feedback | `bound_chat` → `curl` (approve+apply) → `bound_chat` |
| Test cross-thread memory | `bound_chat` (thread A) → `bound_chat` (new thread B) |
| Verify concurrent safety | `Promise.all` with `loopContextStorage.run` in tests |
| Check token budget | Insert large ContentBlock[], set small contextWindow, verify truncation |

## Diagnostic SQL Queries

```sql
-- Task failures by error type
SELECT substr(error,1,100), count(*) FROM tasks
WHERE status='failed' AND deleted=0 GROUP BY substr(error,1,80) ORDER BY count(*) DESC;

-- Cache token population
SELECT model_id, tokens_cache_write, tokens_cache_read FROM turns ORDER BY rowid DESC LIMIT 5;

-- Thread summaries
SELECT id, substr(title,1,40), substr(summary,1,100) FROM threads
WHERE deleted=0 AND summary IS NOT NULL ORDER BY last_message_at DESC LIMIT 5;

-- Task-thread linkage
SELECT id, thread_id, substr(payload,1,60) FROM tasks WHERE deleted=0 ORDER BY modified_at DESC LIMIT 5;

-- Advisory status
SELECT id, status, title FROM advisories WHERE deleted=0 ORDER BY proposed_at DESC LIMIT 5;
```
