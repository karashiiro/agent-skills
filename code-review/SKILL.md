---
name: code-review
description: Use when reviewing code for quality, maintainability, and test coverage. Analyzes implementation and tests to identify code smells, architectural issues, and coverage gaps.
---

# Code Review

## Overview

Systematic code review covering implementation quality, architectural alignment, and test coverage. Review both the code AND its tests together.

## When to Use

- After writing a new function or module
- Before merging significant changes
- When refactoring existing code
- When asked to "review", "check", or "look over" code

## Review Process

### 1. Gather Context

| Step | Action |
|------|--------|
| Identify files | Use Glob to find implementation + test files |
| Read implementation | Focus on the changed/new code |
| Read tests | Find co-located `*.test.ts`, `*.spec.ts`, or `__tests__/` files |
| Check project patterns | Look at similar files for conventions |

### 2. Analyze Implementation

Check each category, note specific issues with file:line references:

| Category | What to Check |
|----------|---------------|
| **Correctness** | Logic errors, edge cases, error handling |
| **Clarity** | Naming, comments, complexity |
| **Maintainability** | DRY, single responsibility, coupling |
| **Architecture** | Follows project patterns, proper abstractions |
| **Security** | Input validation, injection risks, sensitive data |

### 3. Analyze Tests

| Category | What to Check |
|----------|---------------|
| **Coverage** | Happy paths, edge cases, error conditions |
| **Assertions** | Tests assert meaningful behavior, not just that code runs |
| **Isolation** | Tests are independent, no shared mutable state |
| **Integration** | Cross-component behavior is tested |
| **Naming** | Test names clearly describe what they're testing |

### 4. Output Format

```markdown
## Summary
[One paragraph overview and overall assessment]

## Issues Found

### Critical (Must Fix)
- [file:line] Issue description and fix

### Concerns (Should Address)  
- [file:line] Significant issues that aren't blocking

### Suggestions (Consider)
- [file:line] Minor improvements or alternatives

## Test Coverage Analysis
[Assessment of test quality and coverage gaps]

## Verdict: [APPROVED / NEEDS REVISION / REJECTED]
[Justification for verdict]
```

## Code Smells

| Smell | Signs |
|-------|-------|
| God class/function | Too many responsibilities |
| Long parameter list | >3-4 params, use options object |
| Deep nesting | >3 levels, use early returns |
| Magic values | Hardcoded strings/numbers without explanation |
| Copy-paste code | Duplicate blocks, extract utility |
| Silent error swallowing | Empty catch blocks, unhandled promise rejections, missing error propagation |
| Async context issues | Sync blocking in async code, unhandled async callbacks, missing try-catch in async handlers |
| Mutable state misuse | Mutable where immutable would be cleaner |
| Tight coupling | Unrelated components depending on each other |
| TODO/FIXME/HACK | Left without justification or tracking |

## Architectural Concerns

| Concern | What to Look For |
|---------|------------------|
| Single responsibility | One class/function doing too many things |
| Error boundaries | Missing error handling at component boundaries |
| Leaky abstractions | Implementation details exposed |
| State management | Patterns that won't scale |
| Async patterns | Blocking in async contexts, callback error handling |
| Testability | Patterns that make testing difficult |
| Hard-coded config | Values that should be configurable |

## Verdict Criteria

**APPROVED** when:
- No critical issues present
- Test coverage sufficient for complexity
- Hacky code has clear justification
- Architecture supports maintainability

**NEEDS REVISION** when:
- Significant concerns but approach is sound
- Test coverage has gaps but tests exist
- Issues fixable without major restructuring

**REJECTED** when:
- Fundamental architectural problems
- Critical bugs or security issues
- No meaningful tests for complex logic
- Code is unmaintainable

## Review Guidelines

- **Be specific**: Reference exact line numbers and code snippets
- **Be constructive**: Explain WHY something is problematic, suggest alternatives
- **Acknowledge good patterns**: Note well-designed aspects
- **Consider context**: Prototype vs production have different standards
- **Respect conventions**: Check CLAUDE.md or project guidelines
- **Don't nitpick**: Focus on issues that matter for maintainability
- **Ask for clarification**: If intent is unclear, ask rather than assume wrong
