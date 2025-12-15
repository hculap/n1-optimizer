---
description: Analyze codebase for N+1 queries and performance issues using parallel agents
allowed-tools: Task, TaskOutput, Read, Glob, Grep, TodoWrite
argument-hint: [directory]
---

# N+1 Optimizer Analysis

Perform comprehensive performance analysis of the user's codebase by launching 4 specialized agents IN PARALLEL. Each agent focuses on a different layer of the application stack.

## Instructions

1. First, briefly explore the codebase structure to understand:
   - What tech stack is used (detect from package.json, requirements.txt, go.mod, etc.)
   - Where source code is located
   - General project architecture

2. Launch ALL 4 agents IN PARALLEL using a single message with multiple Task tool calls:

   **Agent 1: database-analyzer**
   - Prompt: "Analyze this codebase for database performance issues. Focus on: N+1 queries, missing indexes, inefficient JOINs, unbounded queries, query patterns in loops. Tech stack detected: [stack]. Source directories: [dirs]"

   **Agent 2: backend-analyzer**
   - Prompt: "Analyze this codebase for backend performance issues. Focus on: inefficient algorithms, unnecessary loops, blocking operations, memory leaks, redundant computations. Tech stack detected: [stack]. Source directories: [dirs]"

   **Agent 3: frontend-analyzer**
   - Prompt: "Analyze this codebase for frontend performance issues. Focus on: unnecessary re-renders, large bundle sizes, missing memoization, inefficient state updates, render blocking resources. Tech stack detected: [stack]. Source directories: [dirs]"

   **Agent 4: api-analyzer**
   - Prompt: "Analyze this codebase for API performance issues. Focus on: over-fetching, under-fetching, missing pagination, inefficient endpoints, N+1 API calls from frontend. Tech stack detected: [stack]. Source directories: [dirs]"

3. Wait for all agents to complete using TaskOutput.

4. Consolidate results into a single report grouped by category:

```markdown
# Performance Analysis Report

## Summary
- Total issues found: X
- High impact: X | Medium impact: X | Low impact: X

## Database Issues (X High, X Medium, X Low)

### [HIGH] Issue title
- **Location**: file:line
- **Problem**: Description of the issue
- **Suggestion**: How to fix it

### [MEDIUM] Issue title
...

## Backend Issues (X High, X Medium, X Low)
...

## Frontend Issues (X High, X Medium, X Low)
...

## API Issues (X High, X Medium, X Low)
...

## Quick Wins
Top 3 issues that are easy to fix with high impact:
1. ...
2. ...
3. ...
```

5. Present the consolidated report to the user.

## Important Notes

- Always launch agents IN PARALLEL (single message, multiple Task calls)
- Each agent should use subagent_type matching its name (e.g., "database-analyzer")
- If a layer doesn't exist (e.g., no frontend), that agent will return "No relevant code found"
- Severity is based on performance IMPACT:
  - HIGH: Causes significant slowdowns, scales poorly (O(n) queries, N+1)
  - MEDIUM: Noticeable impact under load
  - LOW: Minor inefficiency, good to fix but not urgent
