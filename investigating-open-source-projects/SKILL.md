---
name: investigating-open-source-projects
description: Use when investigating how external or open-source projects implement features, make design decisions, or evolve over time — guides research strategies across code, issues, and documentation
---

# Investigating Open-Source Projects

## Overview

Choose a research strategy based on the type of question. Different question types benefit from different tool combinations. GitHub's structured API tools excel at code-level and issue-level investigation; web search excels at finding blogs, release announcements, and ecosystem context.

## Research Patterns

### Implementation Discovery

**Trigger:** "How does X implement Y?" / "What patterns do libraries use for Z?"

Search for source code across repositories, then read implementations directly.

1. Search code with targeted queries scoped by repo, language, and path
2. Read 2-3 of the most relevant source files
3. Compare patterns across implementations

```lua
-- Find implementations
local results = github.search_code({
  query = 'symbol:retryWithBackoff language:typescript path:src NOT path:test'
}):await()

-- Read a file from results
local file = github.get_file_contents({
  owner = "some-org", repo = "some-repo", path = "src/retry.ts"
}):await()

result({ search = results, file = file })
```

Best for: finding how multiple projects solve the same problem. Especially strong for obscure libraries with sparse documentation — code search finds implementations that web search cannot.

### Decision Archaeology

**Trigger:** "Why did project X choose Y over Z?" / "What alternatives were considered?"

Reasoning lives in issue discussions, PR comments, and blog posts. Search for the discussion, not the code.

1. Search issues and PRs in the project for the feature/decision
2. Read key issues to find debate and rationale
3. Check blog posts or release notes for additional context

```lua
-- Find relevant issues
local issues = github.search_issues({
  query = 'repo:prisma/prisma "json filtering" is:issue'
}):await()

-- Read an issue with comments
local detail = github.issue_read({
  owner = "prisma", repo = "prisma", issueNumber = 2444
}):await()

result({ issues = issues, detail = detail })
```

Best when combined with web search for blog posts and release announcements that explain the "why" in the team's own words.

### Evolution Tracking

**Trigger:** "How has X changed over time?" / "What's the history of feature Y?"

Evolution narratives are told in blog posts, release announcements, and changelogs. Start with web search, supplement with GitHub for precision.

1. Web search for blog posts, release notes, and changelogs
2. Supplement with releases and specific PRs for dates and details
3. Reconstruct timeline from multiple sources

```lua
-- List releases to identify version milestones
local releases = github.list_releases({
  owner = "vercel", repo = "next.js", perPage = 20
}):await()

-- Read a specific PR for detail
local pr = github.pull_request_read({
  owner = "vercel", repo = "next.js", pullNumber = 59447
}):await()

result({ releases = releases, pr = pr })
```

Web search is the primary tool here — teams write blog posts about major changes. Use GitHub tools to fill in precise dates, PR numbers, and technical details.

### Deep Dive

**Trigger:** "How does X work internally?" / "What's the mechanism behind Y?"

Understanding internals requires reading source code AND the concepts behind it.

1. Find and read the key source files
2. Search for related issues/bugs that reveal edge cases
3. Look up external documentation for underlying APIs or protocols

```lua
-- Find core implementation files
local code = github.search_code({
  query = 'repo:ceifa/wasmoon "lua_yieldk" language:typescript path:src'
}):await()

-- Find related bug reports
local bugs = github.search_issues({
  query = 'repo:ceifa/wasmoon await coroutine is:issue'
}):await()

result({ code = code, bugs = bugs })
```

Combine GitHub tools for code and issues with web search for external API/protocol documentation.

## GitHub Search Quick Reference

| Syntax | Example | What it does |
|--------|---------|-------------|
| `"exact phrase"` | `"exponential backoff"` | Exact string match |
| `/regex/` | `/Math\.pow.*retry/` | Regular expression |
| `language:` | `language:typescript` | Filter by language |
| `path:` | `path:src NOT path:test` | Filter by file path |
| `symbol:` | `symbol:exponentialBackoff` | Find definitions only |
| `repo:` | `repo:vercel/next.js` | Scope to one repo |
| `org:` | `org:microsoft` | Scope to an organization |
| `NOT` | `NOT path:node_modules` | Exclude results |
| `OR` | `(language:ts OR language:js)` | Boolean or |

**Tips:**
- Start with `repo:` or `org:` scoping to reduce noise
- Use `symbol:` to find definitions, not just references
- `path:src NOT path:test NOT path:vendor` eliminates most noise
- `"exact phrase"` beats vague keywords for precision
- Combine multiple filters: `symbol:fetchWithRetry language:typescript repo:org/repo`

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using only web search for code questions | Code search finds actual implementations; web search finds descriptions |
| Using only GitHub tools for "why" questions | Blog posts and release notes explain reasoning better than raw PR lists |
| Vague keyword searches | Use `symbol:`, `"exact phrase"`, or `/regex/` for precision |
| Not filtering by path | `NOT path:test NOT path:vendor` dramatically reduces noise |
| Searching one repo when the impl is in a dependency | Check package.json/Cargo.toml for where the real implementation lives |
| Giving up when a file isn't on `main` | `get_file_contents` handles branch resolution; or use `get_repository_tree` to discover the default branch |
| Using `path_filter` in `get_repository_tree` like a deep path lookup | `path_filter` is a prefix filter on entries at the current tree level, not a recursive path navigator. To reach `packages/effect/src`, walk the tree by SHA: get root → extract `packages` SHA → extract `effect` SHA → extract `src` SHA |
