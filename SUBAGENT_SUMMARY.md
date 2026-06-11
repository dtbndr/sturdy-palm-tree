# Subagent Exploration: Executive Summary

## Your Goal

Explore how to use Pi subagents for artifact review in your workflow:

```
Brainstorm → Design Spec (review) → Implementation Plan (review) → Implementation
```

Currently: CLI agents for design/plan review  
Goal: Unified subagent dispatch from Pi session

---

## Key Findings

### ✅ Everything You Need Already Exists

| Component                | Status       | Details                                                                                   |
| ------------------------ | ------------ | ----------------------------------------------------------------------------------------- | ------ | ------------------- |
| **Builtin agents**       | ✅ Available | 8 agents: scout, planner, worker, reviewer, context-builder, researcher, delegate, oracle |
| **Custom agent pattern** | ✅ Proven    | Your `implementer`, `spec-reviewer`, `code-quality-reviewer` all use frontmatter config   |
| **Dispatch mechanism**   | ✅ Ready     | `subagent()` tool with role-based dispatch                                                |
| **Async execution**      | ✅ Works     | `async: true` for non-blocking reviews                                                    |
| **Parallel dispatch**    | ✅ Supported | `tasks: [...]` for multi-angle reviews                                                    |
| **Sequential chains**    | ✅ Supported | `chain: [...]` for spec → plan → implementation workflows                                 |
| **Model selection**      | ✅ Per-agent | Set `model:` in frontmatter, override in settings                                         |
| **Thinking levels**      | ✅ Per-agent | `thinking: high                                                                           | medium | low` in frontmatter |
| **Tool isolation**       | ✅ Works     | Reviewers get no-edit tools (read, grep, find, ls)                                        |

### ✅ What You Have in ~/.pi/agent/agents/

```
implementer.md
  • Role: Writes code, tests, self-reviews
  • Model: deepseek-v4-pro (high thinking)
  • Tools: read, grep, find, ls, bash, edit, write
  • Dispatch: subagent({ agent: "implementer", task: "..." })

spec-reviewer.md
  • Role: Verifies implementation matches spec requirements
  • Model: qwen3.6-plus (high thinking)
  • Tools: read, grep, find, ls, bash
  • Dispatch: subagent({ agent: "spec-reviewer", task: "..." })

code-quality-reviewer.md
  • Role: Verifies code is clean, tested, maintainable
  • Model: deepseek-v4-flash (low thinking)
  • Tools: read, grep, find, ls, bash
  • Dispatch: subagent({ agent: "code-quality-reviewer", task: "..." })
```

This is the **exact pattern** you should use for design-spec-reviewer and plan-reviewer.

---

## What's Missing (What You Need to Create)

### 1. Design Spec Reviewer Agent

**File:** Create `~/.pi/agent/agents/design-spec-reviewer.md`

```markdown
---
name: design-spec-reviewer
description: Design spec reviewer — validates design artifacts for clarity, completeness, feasibility
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Your design review methodology here

# (Check spec for missing requirements, ambiguous language, architectural gaps)
```

### 2. Implementation Plan Reviewer Agent

**File:** Create `~/.pi/agent/agents/plan-reviewer.md`

```markdown
---
name: plan-reviewer
description: Implementation plan reviewer — validates plan for scope, task breakdown, sequencing
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Your plan review methodology here

# (Check plan matches spec, tasks are well-decomposed, sequencing is logical)
```

**That's it.** Same pattern as your existing agents. Just copy the frontmatter, customize the body.

---

## How to Dispatch

### Simple: Single Design Spec Review

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: `Review this design spec:
  
  File: docs/design-specs/feature-auth.md
  
  Check for missing requirements, ambiguous language, architectural gaps.`,

  async: true, // Non-blocking
  output: "reviews/design-spec-review.md",
});
```

### Advanced: Parallel Reviews (Multiple Angles)

```typescript
subagent({
  tasks: [
    {
      agent: "design-spec-reviewer",
      task: "Architecture review",
      output: "reviews/arch.md",
    },
    {
      agent: "reviewer",
      task: "Requirements review",
      output: "reviews/reqs.md",
    },
    {
      agent: "researcher",
      task: "Best practices research",
      output: "reviews/research.md",
    },
  ],
  concurrency: 3,
  async: true,
});
```

### Advanced: Sequential Chain (Spec → Plan → Implementation)

```typescript
subagent({
  chain: [
    {
      agent: "design-spec-reviewer",
      task: "Review spec",
      output: "reviews/spec.md",
    },
    {
      agent: "plan-reviewer",
      task: "Review plan against spec: {previous}",
      output: "reviews/plan.md",
    },
    {
      agent: "implementer",
      task: "Implement based on approved plan: {previous}",
    },
  ],
  async: true,
});
```

---

## Your Workflow: Before & After

### Before: Separate CLI Agents

```
Pi session (brainstorm)
    ↓ creates docs/design-specs/feature.md

Switch to Claude Code / Gemini CLI / etc.
    → Review design spec manually
    ↓ returns review-notes.txt

Switch back to Pi (planning)
    → Copy review notes into context
    ↓ creates docs/plans/feature-plan.md

Switch to another CLI agent
    → Review plan manually
    ↓ returns more-notes.txt

Switch back to Pi (implementation)
    → Dispatch implementer (existing)
```

**Friction:** Context switching, manual copy-paste, no unified flow

### After: Unified Subagent Dispatch

```
Pi orchestrator session
    ↓
Brainstorm → Design Spec (artifact)
    ├─→ dispatch design-spec-reviewer async ─→ reviews/spec.md
    │
    ├─ Continue planning (while review runs)
    │
    ├─→ Plan (artifact)
    │   ├─→ dispatch plan-reviewer async ──→ reviews/plan.md
    │   │
    │   ├─ Continue work (while review runs)
    │   │
    │   ├─→ Implementation Phase
    │       ├─→ dispatch implementer
    │       ├─→ dispatch spec-reviewer
    │       └─→ dispatch code-quality-reviewer
    │
    └─ Done (all in same session)
```

**Benefits:** No context switching, async non-blocking, reviews in one session, unified artifacts

---

## Frontmatter Extensibility

All agents use **YAML frontmatter**. You can add custom fields:

```yaml
---
name: design-spec-reviewer
description: ...
model: opencode-go/qwen3.6-plus
thinking: high

# Standard Pi fields (processed by Pi)
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash

# Custom fields (for your documentation/workflows)
artifact_type: design_spec
review_focus: [clarity, completeness, feasibility]
checklist:
  - Missing requirements
  - Ambiguous language
  - Architectural gaps
---
```

**Custom fields are:**

- ✅ Visible in agent metadata
- ✅ Can be referenced in your prompts
- ✅ Good for documentation
- ❌ Not auto-processed by Pi (but you can add logic to read them)

---

## Model Selection Per Agent

You can assign different models to different reviewers:

In `~/.pi/agent/settings.json`:

```json
{
  "subagents": {
    "agentOverrides": {
      "design-spec-reviewer": {
        "model": "opencode-go/qwen3.6-plus", // Strong reasoning
        "thinking": "high"
      },
      "plan-reviewer": {
        "model": "opencode-go/deepseek-v4-flash", // Fast, capable
        "thinking": "medium"
      },
      "quick-check": {
        "model": "deepseek/deepseek-v4-flash", // Quick
        "thinking": "low"
      }
    }
  }
}
```

Then dispatch any agent with its custom config automatically applied.

---

## Integration with Existing Workflow

Your **existing agents stay the same:**

- `implementer` — writes code
- `spec-reviewer` — checks code against requirements
- `code-quality-reviewer` — checks code quality

**New agents to create:**

- `design-spec-reviewer` — checks design spec
- `plan-reviewer` — checks implementation plan

**Full workflow:**

```
DESIGN PHASE (NEW):
  brainstorm → design-spec-reviewer → plan-reviewer

IMPLEMENTATION PHASE (EXISTING):
  implementer → spec-reviewer → code-quality-reviewer
```

**Both phases** use the same subagent dispatch mechanism, same frontmatter pattern, same async/parallel capabilities.

---

## Extensibility Possibilities

### 1. Frontmatter Metadata

You can add fields like:

```yaml
review_checklist: [...]
artifact_type: design_spec
methodology_source: ~/skills/design-review.md
```

These are visible in agent metadata and can be used in orchestration logic.

### 2. Settings-Based Overrides

Control agent behavior via `~/.pi/agent/settings.json` without editing agent files.

### 3. Parallel Reviews for Diverse Angles

```typescript
subagent({
  tasks: [
    { agent: "design-spec-reviewer", task: "Architecture" },
    { agent: "reviewer", task: "Requirements" },
    { agent: "oracle", task: "Design decisions" },
  ],
});
```

### 4. Conditional Dispatch

Create logic that routes artifacts to appropriate reviewers based on type/size/complexity.

### 5. Chain Workflows

Spec → Plan → Implementation with reviews at each stage, using `{previous}` to pass context.

### 6. Review Synthesis

After parallel reviews, dispatch a synthesizer to consolidate findings.

---

## Key Constraints & Guarantees

### ✅ What Works Guaranteed

1. **Fresh context** - Each subagent gets independent context (no pollution)
2. **Model selection** - You can assign models per agent
3. **Tool isolation** - Reviewers get read-only tools (no accidental edits)
4. **Async execution** - `async: true` = non-blocking, parent continues work
5. **Parallel dispatch** - Multiple agents run concurrently
6. **Sequential chains** - `{previous}` passes output to next step
7. **Output artifacts** - Reviews saved to files for synthesis

### ⚠️ Current Limitations

1. **Custom frontmatter** - Extensible but not auto-processed by Pi
2. **Nesting depth** - Default max 2 levels deep (configurable)
3. **Intercom coordination** - Optional, requires both agents to support it
4. **Review artifacts** - Not auto-committed to git (you decide)

### 🚀 What's Possible with Extra Work

1. Create custom frontmatter processor (external tooling)
2. Build orchestration logic around artifact types
3. Create skill for artifact-review workflows
4. Integrate with git for review history tracking

---

## Quick Start: 3 Steps

### Step 1: Create design-spec-reviewer.md

```bash
mkdir -p ~/.pi/agent/agents
cat > ~/.pi/agent/agents/design-spec-reviewer.md << 'EOF'
---
name: design-spec-reviewer
description: Design spec reviewer — validates design artifacts
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Design Spec Reviewer

You are the design spec reviewer. Your job is to validate design specs before they go to planning.

## What to Check

1. **Completeness** - Are all requirements and use cases covered?
2. **Clarity** - Are requirements unambiguous and well-defined?
3. **Feasibility** - Is this achievable with available technology?
4. **Architecture** - Are system boundaries and data flow clear?
5. **Quality** - Are security, performance, and error handling addressed?

## Report Format

✅ **Summary:** [One-line assessment]

**Strengths:**
- [strength with section reference]

**Issues:**
- Critical: [blocks planning]
- Important: [should clarify]
- Minor: [nice to improve]

**Recommendations:**
- [specific improvement]
EOF
```

### Step 2: Create plan-reviewer.md

```bash
cat > ~/.pi/agent/agents/plan-reviewer.md << 'EOF'
---
name: plan-reviewer
description: Implementation plan reviewer — validates plan artifacts
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Implementation Plan Reviewer

You are the plan reviewer. Your job is to validate implementation plans.

## What to Check

1. **Scope Alignment** - Does plan cover all spec requirements?
2. **Task Breakdown** - Are tasks appropriately sized and focused?
3. **Sequencing** - Are tasks in logical order with explicit dependencies?
4. **Feasibility** - Are tasks realistic? Any risky approaches?
5. **Validation** - Are acceptance criteria testable?

## Report Format

✅ **Summary:** [One-line assessment]

**Strengths:**
- [strength with task reference]

**Issues:**
- Critical: [blocks implementation]
- Important: [should fix before starting]
- Minor: [nice to clarify]

**Recommendations:**
- [specific improvement]
EOF
```

### Step 3: Test Dispatch

```typescript
// In a Pi session, test dispatch:
subagent({
  agent: "design-spec-reviewer",
  task: "Review docs/design-specs/feature.md for completeness",
  async: true,
  output: "reviews/spec.md",
});

// Check output file:
// cat reviews/spec.md
```

**Done!** Now integrate into your brainstorm/planning workflow.

---

## Documentation Files Provided

1. **SUBAGENT_EXPLORATION.md** (17KB)
   - Comprehensive discovery of builtin agents
   - Existing custom agents in your setup
   - Frontmatter configuration options
   - Architecture and patterns

2. **SUBAGENT_USAGE_EXAMPLES.md** (18KB)
   - 11 ready-to-use dispatch patterns
   - Design spec reviewer examples
   - Plan reviewer examples
   - Integration with full workflow
   - Error handling and fallbacks

3. **EXTENSION_ROADMAP.md** (15KB)
   - Migration path from CLI agents to subagents
   - 3-phase implementation plan
   - Weekly breakdown
   - Testing checklist
   - Open questions for future exploration

4. **SUBAGENT_SUMMARY.md** (this file)
   - Executive summary
   - Key findings
   - Quick start guide
   - What's possible and what's limited

---

## What You've Discovered

### ✅ Confirmed

- Builtin agents are available and ready to use
- Custom agent pattern (frontmatter) is well-documented
- Your existing agents follow the right pattern
- Dispatch mechanism is simple and powerful
- Async/parallel/chain workflows all supported
- Model selection is flexible per agent

### ✅ Opportunities

- Create design-spec-reviewer using existing pattern
- Create plan-reviewer using existing pattern
- Integrate reviews into artifact generation workflows
- Run reviews async while continuing work
- Parallel review for diverse angles
- Review synthesis and feedback loops

### ⚠️ Limitations

- Frontmatter extensibility is manual (no auto-processing)
- Custom fields are for documentation only (for now)
- Default nesting depth is 2 (configurable if needed)

### 🚀 Next Steps

1. Create the two new agent files (design-spec-reviewer, plan-reviewer)
2. Test dispatch pattern with sample spec/plan
3. Integrate into brainstorm/planning workflow
4. Document learnings and optimizations

---

## Bottom Line

**You have everything you need.** The infrastructure exists:

- ✅ 8 builtin agents (scout, planner, worker, reviewer, etc.)
- ✅ Working pattern for custom agents (your implementer, spec-reviewer, code-quality-reviewer)
- ✅ Unified dispatch mechanism (subagent tool)
- ✅ Async, parallel, and sequential execution
- ✅ Model and thinking level control

**All you need to do:**

1. Create 2 new agent files (design-spec-reviewer.md, plan-reviewer.md) using the existing pattern
2. Dispatch them in your workflow using `subagent()` tool
3. Get reviews as artifacts in your session
4. Iterate on specs/plans based on review feedback

**The barrier to entry is very low.** The pattern is proven. The tooling is ready.
