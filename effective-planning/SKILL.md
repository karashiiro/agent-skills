---
name: effective-planning
description: Use when entering plan mode or creating any multi-step plan - applies three questions to each step (why, what depends on, how verify) to prevent underspecified work that causes problems during execution
---

# Effective Planning

## Overview
Plans fail when steps are underspecified. Discovering missing details mid-execution wastes time and forces compromises. Invest 5 minutes in planning clarity to save hours of rework.

## When to Use

**Use this skill when:**
- Writing implementation plans from designs
- Breaking down complex tasks into steps
- Reviewing a plan before execution starts
- Detecting vague or underspecified steps in existing plans

**Don't use for:**
- Trivial one-step tasks (just do it)
- Research/exploration (no fixed plan needed)
- Already-executing work (too late for planning)

## Quick Reference: Iterative Refinement Process

Planning is iterative. Start rough, refine until questions have solid answers.

**For each step in your plan:**

1. **Draft the step** - Write what you think needs to happen (can be rough)

2. **Ask three questions:**
   - **Why are we doing this?** (Purpose and goal)
   - **What does this depend on?** (Prerequisites, assumptions, ordering)
   - **How will we know it worked?** (Verification criteria)

3. **Identify gaps** - Which questions can't you answer with confidence?
   - If you're guessing, handwaving, or saying "probably" → you have a gap

4. **Close open questions IMMEDIATELY:**
   - **Investigate**: Dispatch agents to research the codebase in parallel
   - **Ask**: If agents can't clarify, ask the user directly
   - **Document**: Record what you learned ("Verified: Redis configured in prod")
   - **CRITICAL**: Do NOT leave questions open "to figure out later" - they may invalidate your entire plan

5. **Refine the step** - Update the step with answers from investigation

6. **Repeat** - Continue until all three questions have concrete, verified answers

**You're done when:** Every step has clear purpose, explicit dependencies, and defined verification criteria.

## What to Do
Include the answers to the following questions in the plan itself under each step or phase.

### 1. "Why are we taking this action?"
Each action we take is in service of a goal, whether that goal is to improve some system, to ensure we're creating something people will love, or to verify that our assumptions are correct. This is the most important question to ask, and every step of every plan should be in service of this goal!

While it's easy to get distracted by the task at hand, the purpose of an action is almost never to just move on to the next step of the plan; we're doing this for a *reason*.

Most of the time, the reasons are straightforward. Here are some common ones:
- If we're building a feature, we're solving a specific real-world problem people face. Make sure you know what that problem is.
- If we're designing an interface, we're ensuring our systems are well-abstracted so others can make sense of them. Don't create leaky abstractions.
- If we're writing tests, we're making sure we're not shipping bugs to the people we serve. Make every test count, and test contracts, not implementation details. If you need to reference implementation details in a test, odds are that the contract is underspecified.

Other times, the reasons are not straightforward. In these cases, don't guess, *ask the user*!

**Smell test for weak "Why" statements:**

If your "Why" answer describes WHAT you're doing or HOW you're doing it instead of WHY it matters, you have a weak answer. Look for these red flags:
- "To refactor X" / "To extract Y" / "To create Z" (describes what, not why)
- "To ensure consistency" / "To avoid duplication" (surface-level, no impact stated)
- "To validate the design" / "To prove it works" (meta-work, not the real goal)
- "Because it's the next step" / "To enable Step 3" (sequencing, not purpose)

**Connect every step to the overarching goal:**

Every plan has a goal stated at the top. Every step's "Why" should connect back to that goal. If you can't draw a line from your step to the stated goal, you haven't explained why it matters.

For each step, ask: **"How does this serve the plan's goal?"**

Example plan goal: *"Reduce cognitive load for library consumers and code reviewers by eliminating unused abstractions"*

- Bad (mechanical): "This utility class has zero callers"
- Ask: How does deleting it serve the goal of reducing cognitive load?
- Good (connects to goal): "This utility class adds 150 lines that reviewers must scan when tracing request handling, only to discover it's never instantiated. Library consumers browsing the public API see it alongside the real implementation, doubling the conceptual surface area. Deleting it removes a red herring that wastes reviewer time and confuses consumers."

Example plan goal: *"Prevent duplication bugs by consolidating sync/async implementations"*

- Bad (mechanical): "The sync handler duplicates the async handler's logic"
- Ask: How does consolidation prevent duplication bugs?
- Good (connects to goal): "The sync and async handlers both implement 8 identical validation methods. Last quarter, we fixed a null-pointer bug in the async version but forgot to patch sync, causing production crashes. Making sync delegate to async means bug fixes only need one location, eliminating the 'fix one, miss the duplicate' failure mode."

**Transform WHAT into WHY by asking: "So what? Why does that matter?"**

Examples showing the transformation:

**Feature work:**
- Bad: "We're creating this storage interface to help others provide their own implementations depending on their use cases."
- Good: "We're creating this storage interface because [FEATURE BEING BUILT] requires a storage solution, but we don't want to assume that our users use any specific one. We don't know their exact use cases, so the interface can't have escape hatches for our particular circumstances."

**Refactoring work:**
- Bad: "Extracting to a base class enforces structural consistency and avoids duplicating error-prone cleanup logic."
- Ask "So what?": Why does consistency matter? What happens when cleanup logic is duplicated?
- Good: "The async client and server handler both have identical 60-line task lifecycle blocks. When we fixed a shutdown bug last month, we only patched the server side, causing client-side resource leaks in production. Extracting to a shared base class means lifecycle bugs only need one fix."

**Test work:**
- Bad: "To add test coverage for the new endpoint."
- Ask "So what?": Why does this endpoint need coverage?
- Good: "This endpoint handles payment processing. Without tests, we risk shipping regressions that could charge users incorrectly or fail to process refunds, directly impacting revenue and trust."

### 2. "What assumptions does this step rely on?"
Every step operates in a context. Missing dependencies cause execution failures. Hidden assumptions lead to rework. Document both explicitly before execution begins.

**Common dependency types:**
- **Sequential dependencies**: Step 3 requires Step 1's output (file created, database migrated, API deployed)
- **System assumptions**: Database exists, API is accessible, environment variables are set
- **Code assumptions**: Interface exists, function is available, module is installed
- **Knowledge assumptions**: "We already know the schema", "The user confirmed the approach"
- **External dependencies**: Third-party service is available, credentials are configured

**Make dependencies explicit:**
- Bad: "Update the user service"
- Good: "Update the user service (requires: database migration from Step 2 complete, `User` model updated in Step 1)"
- Bad: "Deploy to production"
- Good: "Deploy to production (requires: staging validation passed with >95% success rate, deployment approval from user)"

**When uncertain about assumptions:**
1. Dispatch agents to investigate the codebase in parallel
2. Check with the user if agents can't provide clear answers
3. Document what you found: "Verified: Redis is already configured in production"
4. Update the plan with confirmed dependencies

If you're guessing about whether something exists or works a certain way, you haven't verified your assumptions. Stop and investigate.

### 3. "How will we know we've done this correctly?"
Every step needs a verification criterion. Before claiming completion, you must be able to answer this question with concrete evidence.

**Common verification methods:**
- **Tests pass**: Run the test suite and confirm success
- **Manual verification**: Run the application and observe expected behavior
- **Metrics/benchmarks**: Compare performance before/after
- **External validation**: API responds correctly, integration works
- **Code review**: Another person/agent confirms quality

**Do not accept vague answers:**
- Bad: "It should work"
- Bad: "Looks correct"
- Good: "All unit tests pass and integration test shows API returning 200"
- Good: "Benchmark shows 40% reduction in response time from 500ms to 300ms"

If you can't define verification upfront, you're not ready to execute the step. Stop and clarify first.

## Close Open Questions During Planning, Not Execution

**The Rule:** If you have a question about how something works, what exists, or what approach to take, close that question NOW during planning. Do not leave it for execution.

**Why this matters:**
- Open questions often reveal faulty assumptions that invalidate other parts of your plan
- Discovering these mid-execution forces costly rework and compromises
- Questions answered in parallel during planning cost minutes; discovered serially during execution cost hours

**How to close questions:**

1. **Identify the question explicitly**
   - "Does the caching layer support TTL configuration?"
   - "Which authentication middleware is currently in use?"
   - "Should we use approach A or approach B for this?"

2. **Investigate in parallel**
   - Dispatch multiple research agents simultaneously
   - Search for existing implementations, patterns, configuration
   - Read relevant files to verify assumptions

3. **Ask the user if investigation can't resolve it**
   - Technical questions agents can usually answer by reading code
   - Design decisions and preferences require user input
   - Don't guess when you should ask

4. **Document the answer in your plan**
   - "Investigated: Auth uses JWT middleware in `src/auth/jwt.ts`"
   - "User confirmed: Use approach B (webhook-based) for real-time updates"
   - "Verified: Cache config supports TTL in `redis.conf` line 47"

5. **Update dependent steps based on answers**
   - If investigation reveals surprises, revise your plan accordingly
   - Better to discover now than mid-execution

**Example:**

Bad (leaving questions open):
```
1. Add caching to API endpoints
   - Figure out which cache to use
   - Probably Redis or in-memory
```

Good (closing questions during planning):
```
[Dispatches research agent to check existing infrastructure]
[Reads config files to verify cache setup]
[Finds Redis already configured in production]

1. Add Redis caching to API endpoints
   - Verified: Redis 7.0 already running in staging/prod (config in `infrastructure/redis.yml`)
   - Verified: Connection pooling configured with 10 connections max
   - Cache layer will use existing Redis instance, no new infrastructure needed
```

## Refactoring and Architectural Work

Refactoring plans commonly have weak "Why" statements that describe the technical change without explaining the impact. This happens because the refactoring IS the task, making it easy to confuse WHAT you're doing with WHY it matters.

**The real "Why" for refactoring connects to:**
- **Maintainability**: "Current structure makes changes risky/slow/error-prone"
- **Review-ability**: "Diff is too large to review, blocking merges"
- **Bug risk**: "Duplication caused bugs before, will again"
- **Cognitive load**: "Mixing concerns makes code hard to reason about"
- **Scalability**: "Adding new cases requires editing N places"
- **Merge conflicts**: "Structure makes upstream merges impossible"

### Common Mechanical "Why" Patterns (and Fixes)

| Mechanical Why (describes WHAT) | Real Why (describes IMPACT) |
|---|---|
| "Extract base class for structural consistency" | "The server handler and async client have 60 duplicate lines. Last bug fix only patched one, causing production resource leaks. Base class means one fix location." |
| "Refactor X to extend Y" | "Current structure violates single responsibility - the async client is +1,003 lines mixing protocol handling and task logic, making reviews take hours and blocking merge of upstream changes." |
| "Create handler to absorb task logic" | "The async client's 670 lines of task code are impossible to review alongside protocol changes. Extracting lets reviewers focus on one concern at a time, unblocking the 4-month-stalled merge." |
| "Validate the abstraction works" | "If the base class design is wrong, we'll discover it when refactoring the tested server handler. Better to fail fast with tested code than build the client handler on a broken foundation." |
| "Move duplicated code to shared location" | "Every task feature requires identical changes in 3 places (client sync, client async, server). Shared code means feature work becomes 1 change instead of 3, reducing bug risk and velocity." |

### Finding the Real Why

When writing a refactoring step, ask:

1. **What problem does the current structure cause?**
   - Example: "Reviewers can't understand the diff because protocol + task logic are interleaved"

2. **What specific incident or pain point motivated this?**
   - Example: "Last month's shutdown bug fix only patched server side, caused client resource leaks"

3. **What becomes easier/safer/faster after this change?**
   - Example: "After extraction, adding new task features only touches the task handler, not the main client class"

4. **What risk does this eliminate?**
   - Example: "Eliminates the 'fix in one place, miss the duplicate' class of bugs"

If you can't answer these, you might not understand why the refactoring matters. **Ask the user** what problem the current structure causes.

## Common Problems

| Problem | Symptom | Solution |
|---------|---------|----------|
| **Leaving open questions** | "We'll decide on the cache solution later", "Need to figure out the auth approach" | Close ALL questions during planning - dispatch agents to investigate or ask the user |
| **Assuming vs. verifying** | "It probably works this way" | Dispatch agents to verify assumptions before writing the plan |
| **Vague acceptance criteria** | "Make it better", "Fix the issue" | Define specific, measurable outcomes before starting |
| **Missing dependencies** | Step 3 needs Step 1's output but order unclear | Explicitly document "This requires [Step X] because..." |
| **Skipping verification planning** | "We'll test it after" | Define HOW you'll verify BEFORE executing |
| **Paralysis by over-planning** | Spending hours planning a 10-minute task | Match planning depth to task complexity |
| **Deferring decisions** | "We can choose between A/B during implementation" | Make architectural decisions during planning when they're cheapest to change |

## Red Flag Language

If you find yourself thinking or writing any of these during planning, STOP:
- "We'll figure that out when we get there"
- "We'll decide between X and Y during implementation"
- "Need to investigate which approach to use" (without actually investigating)
- "This part is straightforward" (without explaining why)
- "Should be easy to..." (without verifying assumptions)
- "Something like..." (vague handwaving)
- "Probably works like..." (guessing instead of researching)
- "TBD" / "TODO: Research" / "To be determined"
- "We can explore options later"

These indicate underspecified plans with open questions. Address them NOW, not during execution. Dispatch agents to investigate or ask the user to close these questions before finalizing your plan.

## Example: Before and After

### ❌ Underspecified Plan
```
1. Add caching to API
2. Update tests
3. Deploy
```

**Problems:** What kind of cache? Where? What TTL? Which endpoints? How verify it works?

### ✅ Well-Specified Plan
```
1. Add in-memory caching to `/users` endpoint
   - **Why**: Reduce DB load - endpoint hit 10k times/minute, mostly duplicate queries
   - **Dependencies**: Requires cache service instance (already in staging/prod)
   - **Verification**: Response time drops from ~200ms to <50ms on cache hit, cache monitor shows hit rate >80%

2. Update integration tests to verify cache behavior
   - **Why**: Ensure cache invalidation works correctly when users update profiles
   - **Dependencies**: Step 1 complete
   - **Verification**: Test suite passes, new tests specifically check stale data not returned

3. Deploy to staging, validate metrics
   - **Why**: Confirm cache hit rate and response time improvements before prod
   - **Dependencies**: Steps 1-2 complete
   - **Verification**: Monitoring dashboard shows cache hit rate >80%, p95 latency <50ms over 1-hour observation
```
