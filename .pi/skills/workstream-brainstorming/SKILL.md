---
name: workstream-brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements, and design before implementation, generating a Workstream Document instead of separate specs and plans."
---

# Workstream Brainstorming

Help turn ideas into fully formed designs and sequential execution slices through natural collaborative dialogue, culminating in a single **Workstream Document**.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design and get user approval.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design/workstream and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## The Workstream Document Concept

We explicitly do NOT separate design specs (`docs/superpowers/specs/`) and implementation plans (`docs/superpowers/plans/`). Instead, we capture both in a single **Workstream Document** written under `docs/workstreams/YYYY-MM-DD-<topic>.md`. 

A Workstream Document contains:
1. **Metadata**: Title, Date, Status (Planned/In Progress/Complete), Objective, Target Branch.
2. **Context**: Problem description, Approved simplifications, Target behavior, Deliberate simplifications, Architecture invariants.
3. **Scope**: In scope, Out of scope.
4. **Key Files**: Table mapping file paths, packages, and their roles.
5. **Scoped Slices**: Sequential slices (Slice A, B, C...) containing Goals, specific Tasks, Watch outs, Verification steps, and Carry-forwards.
6. **Final Verification**: Comprehensive validation checklist (compilation, typecheck, tests, lint, format).
7. **Success Criteria**: Clear, testable outcomes that must be met.

This document serves as the single source of truth for both architectural design and the task checklist used by `workstream-driven-development`.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Explore project context** — check files, docs, recent commits
2. **Offer visual companion** (if topic involves visual questions) — this is its own message, not combined with a clarifying question.
3. **Ask clarifying questions** — one at a time, understand purpose/constraints/success criteria
4. **Propose 2-3 approaches** — with trade-offs and your recommendation
5. **Present design and slice breakdown** — in sections scaled to their complexity, get user approval after each section
6. **Write Workstream Document** — save to `docs/workstreams/YYYY-MM-DD-<topic>.md` and commit
7. **Workstream self-review** — check for placeholders, contradictions, completeness, and make sure slices are independently logical
8. **User reviews Workstream Document** — ask user to review the workstream file before proceeding
9. **Transition to implementation** — invoke `workstream-driven-development` skill

## Process Flow

```dot
digraph workstream_brainstorming {
    "Explore project context" [shape=box];
    "Visual questions ahead?" [shape=diamond];
    "Offer Visual Companion\n(own message, no other content)" [shape=box];
    "Ask clarifying questions" [shape=box];
    "Propose 2-3 approaches" [shape=box];
    "Present design & slices" [shape=box];
    "User approves design?" [shape=diamond];
    "Write Workstream Document" [shape=box];
    "Workstream self-review\n(fix inline)" [shape=box];
    "User reviews workstream?" [shape=diamond];
    "Invoke workstream-driven-development" [shape=doublecircle];

    "Explore project context" -> "Visual questions ahead?";
    "Visual questions ahead?" -> "Offer Visual Companion\n(own message, no other content)" [label="yes"];
    "Visual questions ahead?" -> "Ask clarifying questions" [label="no"];
    "Offer Visual Companion\n(own message, no other content)" -> "Ask clarifying questions";
    "Ask clarifying questions" -> "Propose 2-3 approaches";
    "Propose 2-3 approaches" -> "Present design & slices";
    "Present design & slices" -> "User approves design?";
    "User approves design?" -> "Present design & slices" [label="no, revise"];
    "User approves design?" -> "Write Workstream Document" [label="yes"];
    "Write Workstream Document" -> "Workstream self-review\n(fix inline)";
    "Workstream self-review\n(fix inline)" -> "User reviews workstream?";
    "User reviews workstream?" -> "Write Workstream Document" [label="changes requested"];
    "User reviews workstream?" -> "Invoke workstream-driven-development" [label="approved"];
}
```

**The terminal state is invoking workstream-driven-development.** Do NOT invoke `writing-plans` or standard implementation skills.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits).
- Ask clarifying questions one at a time to refine the idea, focusing on purpose, constraints, and success criteria.

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs, leading with your recommended option and explaining why.

**Presenting the design & slice breakdown:**
- Once you understand what you're building, present the design.
- Scale each section to its complexity: cover architecture, components, data flow, testing.
- **Decompose into slices**: Break down the execution plan into sequential, logical groupings of tasks (Slice A, Slice B, Slice C...). Slices must represent clean milestones that can be independently developed, compiled, and tested.

## Document Structure & Schema

Use this exact structure for the Workstream Document written to `docs/workstreams/YYYY-MM-DD-<topic>.md`:

````markdown
# <Topic> Workstream

**Date**: YYYY-MM-DD
**Status**: Planned / In Progress / Complete
**Merged**: Target branch (e.g. `dev` or `main`)
**Objective**: Clear, concise statement of the ultimate goal.

## Context

### Problem
What is currently wrong, inefficient, or missing?

### Approved simplification
What constraints have been agreed upon to simplify the task and avoid over-engineering?

### Target behaviour
How should the system behave once the workstream is finished?

### Deliberate simplification for this workstream
What is explicitly deferred, out of scope, or accepted as a limitation for this workstream?

### Architecture invariant
What are the strict design, folder, or boundary rules that must never be violated?

---

## Workstream scope

**In scope**
- Bullet 1
- Bullet 2

**Out of scope**
- Bullet 1
- Bullet 2

---

## Key files

| File | Package | Current role |
| ---- | ------- | ------------ |
| `path/to/file` | Name or `@pkg` | Brief description of its role in this workstream |

---

## Scoped workstream slices

### Slice A: <Slice Title>

**Goal**: What does this slice achieve?

#### Tasks

**`file/to/modify`**
- [ ] Specific step 1
- [ ] Specific step 2

#### Watch out
- Caveats, risks, or API constraints

#### Verification
- Commands to run and assertions to check (use the project's actual test/build tooling)

#### Carry-forward
- State, APIs, or schema elements established in this slice that the next slice assumes to be present.

*(Repeat for Slice B, Slice C, etc.)*

---

## Final verification

Run all of the following before handoff. Each must pass clean:
- [ ] Tests pass
- [ ] Type check passes
- [ ] Linter passes
- [ ] Formatter passes

---

## Success criteria

All criteria met:
- [ ] Specific behavior 1
- [ ] Specific behavior 2
````

## Self-Review and Approval

After writing the workstream document, look at it with fresh eyes:
1. **Placeholder scan:** Any "TBD", "TODO", or vague requirements? Fix them.
2. **Slice logical flow:** Is each slice clean? Does Slice B properly build on Slice A's carry-forward? Are they too large or too small?
3. **Ambiguity check:** Pick one concrete path and eliminate double-interpretations.

Ask the user to review the written Workstream Document:

> "Workstream Document written and committed to `docs/workstreams/YYYY-MM-DD-<topic>.md`. Please review it and let me know if you want to make any changes before we transition to `workstream-driven-development`."

Wait for approval. Once approved, invoke `workstream-driven-development` to execute.
