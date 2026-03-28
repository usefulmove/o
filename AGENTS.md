# Agent Harness Protocol

[enso – an agent harness for reliable, persistent AI collaboration](https://github.com/usefulmove/enso)

enso v0.5.0

A single-file seed for managing context across LLM agents, sessions, and tools.

## 1. Purpose

This protocol defines how to manage context when working with LLM agents on software projects. The workflow:

1. Drop this file into a project directory
2. Point an agent to it
3. Agent bootstraps the directory structure
4. Agent creates a PRD from conversation with human
5. Documents evolve as work unfolds, staying compact and focused
6. **Agent builds its own tools—extending its capabilities over time**

**The goal:** maintain the smallest set of high-signal tokens needed for the next step. The harness treats context as data—each operation transforms the current context into a new context, enabling recursive, verifiable workflows.

**The philosophy:** Software building software. Agents improve by becoming authors of their own tooling.

## 2. The Six Operations

**Context is finite.** The context window is the agent's working memory. Every token competes for attention. Treat context as a scarce resource.

| Operation | What It Does | Why It Matters |
|-----------|--------------|----------------|
| **Write** | Persist information outside the context window | Working memory is temporary; persistence survives sessions |
| **Select** | Load only what's needed right now | Don't waste tokens on irrelevant context |
| **Probe** | Actively search (grep, LSP, glob) for answers | Don't assume you know what's in the codebase |
| **Compress** | Summarize to fit the token budget | When context gets full, condense instead of dropping |
| **Isolate** | Split work across multiple scopes | Divide complex tasks to stay within limits |
| **Assign** | Choose the ideal agent for each task | Match task requirements to agent capabilities |

**Keep current, not historical.** Documents reflect the present state. Git preserves history. Don't accumulate cruft in docs.

**Progressive disclosure.** Load context only when needed. Frontmatter before full docs. Summaries before details.

## 2.1 Self-Improvement: The "Seventh Operation"

**Agents improve by building their own tools.**

Just as a skilled engineer develops custom scripts and workflows, an agent should extend its own capabilities during normal work. When you encounter friction—repetitive tasks, complex procedures, or missing functionality—don't just push through. Build a tool.

This is the essence of *software building software*: the agent writes code that the agent itself will use. Every tool you build becomes part of your persistent capabilities, compounding over time.

**The self-extension loop:**
1. **Encounter friction** — a task you do repeatedly, a complex procedure, a missing capability
2. **Build the minimal solution** — a script, a skill, a helper
3. **Capture it** — persist to `docs/skills/` so it's discoverable
4. **Use it** — your future self benefits from your past work
5. **Iterate** — improve the tool as you use it

**Key insight:** The agent IS the tool builder. Not a user of downloaded skills—an author of its own capabilities.

## 3. Terminology

| Term | Definition |
|------|------------|
| **Working Context** | Active tokens in the current LLM call. Ephemeral, limited by context budget. |
| **Persistent Context** | Markdown documents that survive across sessions. Includes Core, Stories, Reference, Skills, and Logs. |
| **Reference Context** | Queryable external sources: codebase (LSP, grep), RAG indexes, web. Not stored in the doc system. |
| **Compaction** | Summarizing working context into persistent context before it's lost. |
| **Context Budget** | The token limit of the model's context window. |
| **Context Scope** | Per-story declaration of file boundaries: what the agent can write, read, or must exclude. |

**Hierarchy:**

```
WORKING CONTEXT (ephemeral, token-limited)
    |
    | <- Select (load)     -> Write (persist)
    v                         v
PERSISTENT CONTEXT (markdown, durable)
  |-- Core Docs (PRD, Architecture, Standards)
  |-- Stories (active tasks)
  |-- Reference (conventions, completed work)
  |-- Skills (on-demand capabilities)
  |-- Logs (session summaries)

REFERENCE CONTEXT (external, queryable)
  |-- Codebase (LSP, grep, git)
  |-- RAG indexes
  |-- External sources (web, APIs)
```

## 4. Directory Structure

```text
docs/
  core/           # Source of Truth (PRD, Architecture)
  stories/        # Active Units of Work (The "Ticket" system)
  reference/      # Long-term Memory (Lessons, Conventions)
  skills/         # Local Capabilities (Scripts, Tests)
  logs/           # Session History
```

## 5. Bootstrapping

When an agent encounters this file in a new project:

1. **Create structure**
   ```bash
   mkdir -p docs/{core,stories,reference/completed,skills,logs}
   touch docs/reference/LESSONS.md
   ```

2. **Add retrieval-led reasoning instruction** to root `AGENTS.md`:
   ```
   IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning 
   for framework-specific and domain-specific tasks.
   ```

3. **Gather context** — Prompt the human for the problem, success criteria, scope, and constraints.

4. **Generate PRD** — Create `docs/core/PRD.md` from the conversation.

5. **System Mapping** — Probe the codebase to create `ARCHITECTURE.md`, identify capabilities, and document conventions.

6. **First story** — Create the initial story in `docs/stories/`.

7. **Begin work**

## 6. Planning Phase

**Plan before you execute.** No file modifications until the story's Approach section is complete. Planning is not optional—it is the first act of execution.

**Required steps before touching any file:**

1. **Create or locate the story** — If no story exists for this task, create one in `docs/stories/` now.
2. **Complete the Approach section** — Fill in Steps, Risks & Unknowns, and Verification before writing any code.
3. **Verify scope** — Confirm the Context Scope (Write/Read/Exclude) is declared and accurate.
4. **Then execute** — Only after the above are done, begin modifying files.

**Why this matters:** Agents that skip planning produce work that drifts from intent, miss edge cases, and require costly rework. A complete plan externalizes reasoning so it can be reviewed, corrected, and resumed across sessions. The plan is the first deliverable.

**If the task is small** (single-file, low-risk change): a minimal story with a one-line Steps entry still satisfies this requirement. The goal is intentionality, not ceremony.

## 7. Document Lifecycle

Context is living code. Refactor documentation as aggressively as you refactor code. Stale context is technical debt.

**Core Docs** (PRD, Architecture, Standards) — Update in place when scope changes. Don't preserve history—git does.

**Stories** — Create when planning, update during execution, move to `reference/completed/` when done.

**Reference** — Conventions and lessons. Read-only during execution; prune when irrelevant. **Update `LESSONS.md` with new learnings.**

**Skills** — Add as needed, update when procedures change, remove when obsolete.

**Logs** — Append session summaries after compaction; prune when no longer informative.

## 8. Context Scope

Every story declares its context boundaries:

```markdown
## Context Scope

**Write** (files this task will modify):
- src/auth/login.ts
- src/auth/session.ts

**Read** (files for reference only):
- docs/core/ARCHITECTURE.md
- src/auth/types.ts

**Exclude** (ignore these):
- src/legacy/
- *.test.ts
```

**Enforcement:**
- Agent must not modify files outside Write scope
- Read files are a seed; use Probe to dynamically discover related context
- Agent should consult Read files before making changes
- Agent should avoid loading Excluded paths into context
- Scope changes require explicit human approval

## 9. Skills

On-demand capabilities for **vertical, action-specific workflows** (migrations, upgrades, transformations).

**When to use:**
- **Skills**: One-time actions (migrate, upgrade, refactor)
- **AGENTS.md + docs/core/**: Always-available knowledge

**Location:** `docs/skills/<skill-name>/`

**Structure:**
```
SKILL.md      # Required: frontmatter + when to use
scripts/      # Optional: executable code
references/   # Optional: additional docs
assets/       # Optional: templates, data files
```

**Discovery:** Agent scans directories at session start, reads frontmatter (~100 tokens) for discovery, loads full skill only when needed.

## 9.2. Self-Extension: Building Your Own Tools

**The agent is the tool builder.**

When solving a task requires a script, transformation, or repeatable procedure, **build it yourself**. Don't download it. Don't discard it after single use. Extend your own capabilities.

This is how agents truly improve over time—not by accumulating external dependencies, but by becoming authors of their own tooling.

**The self-extension loop:**
1. **Encounter friction** — a task you'll do again, a complex procedure, a missing capability
2. **Build the minimal tool** — a script, skill, or helper that solves YOUR specific need
3. **Capture it** — persist to `docs/skills/<tool-name>/` with documentation
4. **Use it** — your future sessions benefit from your past work
5. **Iterate** — improve it as you use it

**Before building:** Scan `docs/skills/` for your own existing tools. Reuse and extend your own work before rebuilding.

**Build when:**
- Any task you'll do more than once
- Any procedure that took real effort to figure out
- Any workaround that will likely recur
- Any gap between what you need and what's available

**The compounding effect:** A tool built today saves derivation cost in every future session. After months of work, an agent should have dozens of custom tools tailored to its specific workflows and codebase.

**Pi Principle:** Software building software. The agent extends itself.

## 9.1. Framework Documentation Index

Store version-matched framework docs in `docs/core/framework/` and add an index to `AGENTS.md`:

```markdown
## Framework Documentation
Location: docs/core/framework/

| Section | Files |
|---------|-------|
| Routing | routing.md, navigation.md |
| Caching | cache-directives.md, cache-lifecycle.md |
```

**Why this works:** Always present, standard Markdown, retrieval-led (100% accuracy vs. 79% with on-demand skills).

## 10. Compaction

Moves insights from working context to persistent context.

**Triggers:** ~80% token utilization, completing a story, circular conversation, ending session.

**Process:** Summarize decisions, list artifacts, extract lessons to `LESSONS.md`, write summary to `logs/`, continue with fresh context.

## 11. Templates

Templates are guidelines, not rigid forms. Start minimal, expand as needed.

### PRD

```markdown
# [Project] PRD

## Problem
What problem are we solving? Why does it matter?

## Goals
What does success look like?

## Scope
**In scope:** ...
**Out of scope:** ...
```

### Architecture

```markdown
# [Project] Architecture

## Overview
High-level description.

## Components
| Component | Responsibility |
|-----------|----------------|
| ... | ... |

## Key Decisions
| Decision | Rationale |
|----------|-----------|
| ... | ... |
```

### Story

```markdown
# [STORY-ID] [Title]

## Goal
What are we trying to accomplish?

## Acceptance Criteria
- [ ] ...

## Context Scope
**Write:** ...
**Read:** ...
**Exclude:** ...

## Approach & Verification Plan

### Steps
1. ...

### Risks & Unknowns
- ...

### Verification
How to confirm success (tests, manual checks, etc.)

**IMPORTANT: Do not begin execution until this section is complete.**
```

### Session Summary

```markdown
# Session: [Topic]
**Date:** YYYY-MM-DD

## Overview
What was accomplished.

## Key Decisions
- ...

## Artifacts Modified
- ...

## Next Steps
- ...
```

## 12. Agent Guidelines

- **Plan before executing.** Write the story and complete the Approach section before modifying any file. No exceptions.
- **Search first.** Exhaust search tools before asking for paths.
- **Tool Selection.** Skills for vertical workflows, framework docs for APIs, external tools for navigation.
- **Prefer retrieval over training.** Consult `docs/` for framework specifics.
- **Read before writing.** Check Context Scope and Read files.
- **Update, don't accumulate.** Modify docs in place—git tracks history.
- **Compact proactively.** Don't wait for context overflow.
- **Stay in scope.** Don't modify files outside Write scope without approval.
- **Be concise.** Terse, technical communication.
- **Test early.** Prefer writing or consulting tests alongside implementation. Tests are a verification signal—let them guide design, not just confirm it.
- **Lint before done.** Run static analysis before marking any task complete. Treat lint errors as bugs.
- **Use accuracy tools.** Prefer LSP (go-to-definition, find-references) for code navigation and Context7 for framework/library docs when available—both reduce hallucination risk.
- **Extend yourself.** When you encounter repetitive work, build a tool. The agent is the tool builder—don't just use capabilities, create them. Software building software.
- **Capture your own wisdom.** Skills are not downloaded, they're authored. Every tool you build should be discoverable for your future sessions.
- **Compound your capabilities.** A month of work should leave you with dozens of custom tools. Your future self should be significantly more capable than your past self.

## 13. The Pi Principle

This harness draws inspiration from [Pi](https://github.com/badlogic/pi-mono/), a minimal coding agent with a powerful philosophy: **agents should extend themselves**.

Rather than downloading pre-built tools or skills, Pi agents write their own extensions. When they need new functionality, they build it. The result is software that writes more software—an agent that becomes increasingly capable over time by authoring its own tooling.

**Key insight:** The most powerful agents are not those with the most downloaded dependencies, but those that have built the most custom tools for their specific workflows.

**In practice:**
- Extensions are authored, not installed
- Tools are tailored to YOUR codebase and workflows
- Capabilities compound session over session
- The agent becomes uniquely capable for its specific domain

This is the essence of self-improvement: the agent as tool author.
