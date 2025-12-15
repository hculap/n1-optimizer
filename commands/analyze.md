---
description: Analyze codebase for N+1 queries and performance issues using parallel agents
allowed-tools: Task, TaskOutput, Read, Glob, Grep, TodoWrite
argument-hint: [directory]
---

# N+1 Optimizer Analysis

Perform comprehensive performance analysis by launching 4 specialized agents IN PARALLEL. Each agent focuses on a different layer of the application stack.

## Step 1: Detect Tech Stack

Check these files to identify the tech stack (use Read tool):

| File | Stack | Check For |
|------|-------|-----------|
| `package.json` | Node.js | Look at `dependencies` for: react/vue/angular (frontend), express/fastify/nest (backend), prisma/typeorm/sequelize (ORM) |
| `requirements.txt` or `pyproject.toml` | Python | django, flask, fastapi, sqlalchemy, tortoise-orm |
| `go.mod` | Go | gin, echo, fiber, gorm, ent |
| `pom.xml` or `build.gradle` | Java | spring-boot, hibernate, jpa |
| `Gemfile` | Ruby | rails, sinatra, activerecord |
| `composer.json` | PHP | laravel, symfony, doctrine |

Also identify source directories by checking for: `src/`, `app/`, `lib/`, `services/`, `api/`, `components/`, `pages/`.

**Record:** Frontend framework, Backend framework, ORM/Database library, Source directories.

## Step 2: Launch 4 Agents IN PARALLEL

**CRITICAL:** Launch ALL 4 agents in a SINGLE message with multiple Task tool calls. Do NOT launch them sequentially.

Use `subagent_type` matching the agent name exactly:

### Agent 1: database-analyzer
```
Prompt: "Analyze this codebase for database performance issues.

Working directory: [WORKING_DIR]
Tech stack: [DETECTED_STACK - e.g., Node.js + Prisma + PostgreSQL]
Source directories: [DIRS - e.g., src/, services/]

Focus on:
- N+1 queries (queries inside loops, lazy loading)
- Missing indexes on frequently queried columns
- Inefficient JOINs
- Unbounded queries (no LIMIT)
- Query patterns in loops

Return findings in format: [SEVERITY] Issue - file:line"
```

### Agent 2: backend-analyzer
```
Prompt: "Analyze this codebase for backend performance issues.

Working directory: [WORKING_DIR]
Tech stack: [DETECTED_STACK]
Source directories: [DIRS]

Focus on:
- O(n²) algorithms (nested loops on collections)
- Blocking operations in async code
- Memory leaks (unclosed resources, growing arrays)
- Redundant computations (missing memoization)
- Sequential awaits that could be parallel

Return findings in format: [SEVERITY] Issue - file:line"
```

### Agent 3: frontend-analyzer
```
Prompt: "Analyze this codebase for frontend performance issues.

Working directory: [WORKING_DIR]
Tech stack: [DETECTED_STACK]
Source directories: [DIRS]

Focus on:
- Unnecessary re-renders (inline objects/functions, missing memo)
- Large bundle imports (importing full lodash, moment, etc.)
- Missing memoization (useMemo, useCallback, computed)
- Inefficient state updates causing cascading renders
- Memory leaks (missing cleanup in useEffect)

Return findings in format: [SEVERITY] Issue - file:line"
```

### Agent 4: api-analyzer
```
Prompt: "Analyze this codebase for API performance issues.

Working directory: [WORKING_DIR]
Tech stack: [DETECTED_STACK]
Source directories: [DIRS]

Focus on:
- Over-fetching (returning all fields, SELECT *)
- Under-fetching (multiple requests for related data)
- Missing pagination on list endpoints
- N+1 API calls from frontend (fetch in loop)
- Inefficient endpoint design

Return findings in format: [SEVERITY] Issue - file:line"
```

## Step 3: Wait for Results

After launching all 4 agents, wait for each to complete using TaskOutput:

```
TaskOutput(task_id: [database-analyzer-id], block: true)
TaskOutput(task_id: [backend-analyzer-id], block: true)
TaskOutput(task_id: [frontend-analyzer-id], block: true)
TaskOutput(task_id: [api-analyzer-id], block: true)
```

Collect all results before proceeding to consolidation.

## Step 4: Consolidate Report

Parse each agent's output and create a unified report:

```markdown
# Performance Analysis Report

## Summary
- **Tech Stack**: [detected stack]
- **Total Issues**: X (High: X | Medium: X | Low: X)
- **Layers Analyzed**: Database, Backend, Frontend, API

## Database Issues (X High, X Medium, X Low)

### [HIGH] Issue title
- **Location**: file:line
- **Pattern**: What was detected
- **Problem**: Why this is a performance issue
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
Top 3 high-impact issues that are straightforward to fix:
1. [Issue] - [One-line fix description]
2. [Issue] - [One-line fix description]
3. [Issue] - [One-line fix description]
```

### Handling Missing Layers

If an agent returns "No relevant code found" or finds no issues:
- **Omit that section** from the report entirely
- In Summary, note: "Layers Analyzed: Database, Backend" (listing only analyzed layers)
- Do NOT include empty sections

## Important Notes

- **Parallel Execution**: Always launch all 4 agents in ONE message with multiple Task calls
- **Agent Names**: Use exact `subagent_type`: `database-analyzer`, `backend-analyzer`, `frontend-analyzer`, `api-analyzer`
- **Severity Classification**:
  - **HIGH**: N+1 queries, O(n²) algorithms, unbounded queries, memory leaks - causes significant slowdowns
  - **MEDIUM**: Missing eager loading, sequential awaits, minor over-fetching - noticeable under load
  - **LOW**: Style issues, optional optimizations - good to fix but not urgent
- **Focus on Impact**: Only report issues with real performance implications, not style preferences
