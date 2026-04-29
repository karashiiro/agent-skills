---
name: organizing-pr-feedback
description: Use when collecting and organizing pull request review feedback - provides GitHub MCP gateway patterns for fetching PR comments, review threads, and issue comments, then consolidating into actionable revision lists
---

# Organizing PR Feedback

## Overview

Collect all feedback threads from a GitHub PR, cross-reference author replies to identify resolved vs. open items, and produce a consolidated revision table grouped by topic.

## When to Use

- Reviewing or organizing PR comments
- Identifying what still needs addressing after reviews
- Preparing a revision pass

## Fetching PR Feedback via Gateway

### Issue-level comments (general discussion)

```lua
local comments = github.issue_read({
  owner = "OWNER",
  repo = "REPO",
  issue_number = PR_NUMBER,
  method = "get_comments",
  perPage = 100
}):await()

local out = {}
for _, c in ipairs(comments.comments or {}) do
  table.insert(out, {
    id = c.id,
    author = c.user and c.user.login or "unknown",
    created = c.created_at,
    body = c.body
  })
end
result(out)
```

### Inline review threads (code-level comments)

```lua
local res = github.pull_request_read({
  method = "get_review_comments",
  owner = "OWNER",
  repo = "REPO",
  pullNumber = PR_NUMBER,
  perPage = 100
}):await()

local threads = {}
if res.review_comments and res.review_comments.reviewThreads then
  for _, thread in ipairs(res.review_comments.reviewThreads) do
    local t = {
      isResolved = thread.IsResolved,
      isOutdated = thread.IsOutdated,
      path = nil,
      comments = {}
    }
    if thread.Comments and thread.Comments.Nodes then
      for _, c in ipairs(thread.Comments.Nodes) do
        table.insert(t.comments, {
          author = c.Author and c.Author.Login or "unknown",
          body = c.Body,
          path = c.Path,
          line = c.Line
        })
        t.path = t.path or c.Path
      end
    end
    table.insert(threads, t)
  end
end
result(threads)
```

**Note:** Review thread fields use PascalCase (GraphQL style): `IsResolved`, `Comments.Nodes`, `Author.Login`, `Body`, `Path`, `Line`.

### Top-level reviews (approve/request-changes verdicts)

```lua
local res = github.pull_request_read({
  method = "get_reviews",
  owner = "OWNER",
  repo = "REPO",
  pullNumber = PR_NUMBER,
  perPage = 100
}):await()

local reviews = {}
for _, r in ipairs(res.reviews or {}) do
  table.insert(reviews, {
    author = r.user and r.user.login or "unknown",
    state = r.state,
    body = r.body
  })
end
result(reviews)
```

## Consolidation Workflow

1. Fetch all three sources (issue comments, review threads, top-level reviews).
2. Identify the PR author from PR metadata or user context.
3. For each feedback item, check if the PR author replied indicating it's resolved.
4. For each feedback item, check if the PR author agreed to address it.
5. Exclude items the author said are already resolved. Include items the author agreed to address.
6. Deduplicate across reviewers: merge same concern into one row citing both.
7. Output a table with columns: What to change | Detailed context | Reviewer(s).

## Output Format

```markdown
| **What to change** | **Detailed context** | **Reviewer(s)** |
|---|---|---|
| Short imperative description | Full context with quotes | @reviewer1, @reviewer2 |
```

Group related items (e.g. multiple schema regressions = one row). Keep "what to change" short and actionable. Put full reasoning in "detailed context".

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing inline review threads | Use `get_review_comments` not `get_comments` for code-level feedback |
| Empty review bodies | Reviews with inline comments often have empty top-level body; content is in threads |
| Wrong field casing on threads | Gateway returns PascalCase for GraphQL fields: `IsResolved`, `Author.Login` |
| Including resolved items | Cross-reference author replies before listing as open |
| Listing duplicates | Consolidate by topic, cite all reviewers in one row |
