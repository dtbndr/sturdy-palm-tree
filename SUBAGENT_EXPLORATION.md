# Subagent Exploration: Your Artifact Review Workflow

## Current Workflow

```
Brainstorm → Design Spec (artifact) → Implementation Plan (artifact) → CLI agents review artifacts
```

## Goal

Explore how subagents can be leveraged for design spec review and implementation plan review using existing dispatch infrastructure and frontmatter configuration patterns similar to `implementer`, `spec-reviewer`, and `code-quality-reviewer`.

---

## Discovery Summary

### ✅ What Exists: Builtin Agents

From `pi-subagents` skill, these builtin agents are available globally:

| Agent             | Purpose                       | Best For               | Output                         |
| ----------------- | ----------------------------- | ---------------------- | ------------------------------ |
| `scout`           | Fast codebase recon           | Initial exploration    | `context.md`                   |
| `planner`         | Creates implementation plans  | Multi-step work        | `plan.md`                      |
| `worker`          | Implementation + decisions    | Writing code/fixes     | Implementation with escalation |
| `reviewer`        | Review-and-fix specialist     | Code inspection        | Can edit/fix                   |
| `context-builder` | Requirements/codebase handoff | Structured context     | Context files for handoff      |
| `researcher`      | Web research                  | External evidence      | `research.md`                  |
| `delegate`        | Lightweight generic           | Generic tasks          | Depends on task                |
| `oracle`          | Advisory review               | Architecture/decisions | Advisory (no edits)            |

### ✅ What Exists: Custom Agents (Your Setup)

Located in `~/.pi/agent/agents/`:

```markdown
1. implementer
   - Model: deepseek/deepseek-v4-pro (high thinking)
   - Tools: read, grep, find, ls, bash, edit, write
   - Role: Writes code, tests, self-reviews
   - Dispatch: subagent({ agent: "implementer", task: "..." })

2. spec-reviewer
   - Model: opencode-go/qwen3.6-plus (high thinking)
   - Tools: read, grep, find, ls, bash (no edits)
   - Role: Verifies implementation matches requirements
   - Dispatch: subagent({ agent: "spec-reviewer", task: "..." })

3. code-quality-reviewer
   - Model: opencode-go/deepseek-v4-flash (low thinking)
   - Tools: read, grep, find, ls, bash (no edits)
   - Role: Verifies clean, tested, maintainable code
   - Dispatch: subagent({ agent: "code-quality-reviewer", task: "..." })
```

### ✅ Frontmatter Configuration

All agents use **frontmatter-based configuration**. Fields you can customize:

```yaml
---
name: agent-name # Local name for discovery
package: optional-namespace # Optional: creates runtime name package.name
description: Brief description # What the agent does
model: provider/model-name # Explicit model (inherits parent default if omitted)
thinking: high|medium|low # Reasoning budget
systemPromptMode: replace|prepend # Prepend to builtin prompts or replace
inheritProjectContext: true|false # Include .pi/settings.json + project files
inheritSkills: true|false # Include available skills
tools: read, grep, edit, write # CSV list of available tools
defaultProgress: true|false # Show progress updates
defaultReads: file1, file2 # Auto-read files before task
output: path/to/file # Fixed output path
maxSubagentDepth: N # Nesting depth (default: 2)
---
Your system prompt here...
```

**Reference:** See `~/.pi/agent/agents/implementer.md` for the working pattern.

---

## Opportunity: Artifact Review Agents

Your current workflow uses **different CLI agents** to review design specs and implementation plans. These can be **unified as Pi subagents** with consistent dispatch patterns.

### Proposed Structure

#### 1. **Design Spec Reviewer Agent**

**Purpose:** Review design spec artifacts (markdown) for clarity, completeness, and feasibility.

**Where to create:** `~/.pi/agent/agents/design-spec-reviewer.md` (or `.pi/agents/` for project-specific)

**Proposed frontmatter:**

```yaml
---
name: design-spec-reviewer
description: Design spec reviewer — validates design artifacts for clarity, completeness, and feasibility
model: opencode-go/qwen3.6-plus # High-thinking model for design analysis
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls # Review-only, no writes
---
# Your review methodology here
```

**Dispatch pattern:**

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: `Review this design spec for completeness and feasibility:
  
  Spec: /path/to/design-spec.md
  Context: Feature for [XYZ]
  
  Check for: 
  - Missing acceptance criteria
  - Ambiguous requirements
  - Architectural gaps
  - Feasibility concerns
  - Dependencies not addressed
  
  Report: Strengths, Issues (Critical/Important/Minor), Recommendations`,
});
```

#### 2. **Implementation Plan Reviewer Agent**

**Purpose:** Review implementation plan artifacts for scope accuracy, task breakdown quality, and feasibility.

**Where to create:** `~/.pi/agent/agents/plan-reviewer.md`

**Proposed frontmatter:**

```yaml
---
name: plan-reviewer
description: Implementation plan reviewer — validates plan artifacts for scope, task breakdown, and sequencing
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls
---
# Your plan review methodology here
```

**Dispatch pattern:**

```typescript
subagent({
  agent: "plan-reviewer",
  task: `Review this implementation plan for quality:
  
  Plan: /path/to/implementation-plan.md
  Design Spec: /path/to/design-spec.md
  
  Verify:
  - Tasks align with design spec requirements
  - Task breakdown is appropriately granular
  - Dependencies are explicit
  - Validation criteria are testable
  - Sequencing is optimal
  
  Report: Assessment, Coverage gaps, Task concerns, Recommendations`,
});
```

---

## Integration Points

### Option 1: Inline Dispatch (Current Session)

After generating an artifact, dispatch reviewers immediately:

```typescript
// After brainstorm creates spec
const specReviewResult = subagent({
  agent: "design-spec-reviewer",
  task: "Review the design spec at ...",
  async: true, // Non-blocking, continues your work
  output: "reviews/design-spec-review.md",
});

// After planner creates plan
const planReviewResult = subagent({
  agent: "plan-reviewer",
  task: "Review the implementation plan at ...",
  async: true,
  output: "reviews/implementation-plan-review.md",
});

// Review results, synthesize feedback, iterate as needed
```

### Option 2: Parallel Reviews (Multi-Angle)

Launch multiple reviewers on same artifact for diverse angles:

```typescript
subagent({
  tasks: [
    {
      agent: "design-spec-reviewer",
      task: "Review for architecture and design patterns",
      output: "reviews/spec-architecture-review.md",
    },
    {
      agent: "reviewer", // Generic builtin for requirements check
      task: "Review spec for requirement completeness and clarity",
      output: "reviews/spec-requirements-review.md",
    },
    {
      agent: "researcher", // External validation
      task: "Research if proposed architecture patterns are current best practices",
      output: "reviews/spec-pattern-research.md",
    },
  ],
  concurrency: 3,
  context: "fresh",
  async: true,
});
```

### Option 3: Sequential Review Chain

Chain spec review → plan review → implementation (when ready):

```typescript
subagent({
  chain: [
    {
      agent: "design-spec-reviewer",
      task: "Review the design spec: {task}",
      output: "reviews/spec-review.md",
    },
    {
      agent: "plan-reviewer",
      task: "Review the implementation plan against the approved spec. Spec review: {previous}",
      output: "reviews/plan-review.md",
    },
    {
      agent: "planner",
      task: "If needed, revise the plan based on review feedback: {previous}",
      output: "implementation-plan-revised.md",
    },
  ],
  context: "fresh",
  async: true,
});
```

---

## Extensibility: Custom Frontmatter

The agent frontmatter is **fully extensible**. You can add custom fields:

### Example: Artifact-Specific Metadata

```yaml
---
name: design-spec-reviewer
# Standard fields
model: opencode-go/qwen3.6-plus
thinking: high
# Custom metadata (for your workflows)
artifact_type: design_spec
review_focus: clarity,feasibility,completeness
output_format: markdown
review_checklist:
  - Missing requirements
  - Ambiguous terms
  - Architectural gaps
  - Dependency unmapped
  - Feasibility risks
---
```

While custom fields in frontmatter won't be directly processed by Pi, they can be:

1. **Documented** for agent authors' reference
2. **Referenced in prompts** (agent reads its own frontmatter)
3. **Used in workflow orchestration** (your tooling can parse them)
4. **Stored in agent settings** via `subagent()` tool (model overrides, thinking, tools)

### Example: Settings-Based Overrides

Instead of creating new agent files, override existing agents in `~/.pi/agent/settings.json`:

```json
{
  "subagents": {
    "agentOverrides": {
      "reviewer": {
        "description": "Design spec reviewer variant",
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high",
        "systemPrompt": "Your design review methodology..."
      },
      "custom-plan-reviewer": {
        "model": "opencode-go/deepseek-v4-flash",
        "thinking": "medium",
        "tools": "read,grep,find,ls"
      }
    }
  }
}
```

Then dispatch:

```typescript
subagent({ agent: "custom-plan-reviewer", task: "..." });
```

---

## Existing Patterns You Can Adopt

### From `subagent-driven-development` skill:

1. **Two-stage review pattern**
   - Spec compliance review (does it match requirements?)
   - Quality review (is it well-built?)
   - **For your workflow:** Design completeness → Implementation feasibility

2. **Report format pattern**

   ```markdown
   ✅ Assessment Summary

   Strengths:

   - [...]

   Issues:

   - Critical: [...]
   - Important: [...]
   - Minor: [...]

   Recommendations:

   - [...]
   ```

3. **Re-review loops**
   - If reviewer finds issues → revise artifact
   - Dispatch reviewer again
   - Continue until approved

4. **Async execution with continuous progress**
   - Launch reviewer asynchronously (`async: true`)
   - Continue with next work item
   - Check results when needed

### From `pi-subagents` skill:

1. **Fresh context for adversarial review**
   - Each reviewer gets clean context
   - No history pollution
   - Consistent standards across reviews

2. **Parallel review for diverse angles**
   - Launch multiple agents with different focuses
   - Synthesize findings
   - Catch issues from multiple perspectives

3. **Intercom coordination (advanced)**
   - Reviewers can ask clarifying questions
   - Contact parent with `contact_supervisor`
   - Parent decides via `intercom` reply

---

## Comparison: Builtin vs. Custom Agents

### When to Use Builtin (`reviewer`, `oracle`, etc.)

✅ **Good for:**

- Generic review tasks (no specialized methodology needed)
- Parallel angles on same artifact
- One-off reviews
- Leveraging existing agent investment

Example:

```typescript
subagent({
  tasks: [
    { agent: "reviewer", task: "Review spec for architectural concerns" },
    {
      agent: "oracle",
      task: "Review spec for design decisions that need approval",
    },
  ],
});
```

### When to Create Custom Agents

✅ **Good for:**

- Repeated, standardized review (design specs, plans)
- Domain-specific methodology (artifact-type validation)
- Specific model preferences (qwen3.6 for spec review, deepseek for quality)
- Behavioral guardrails (no edits, specific tool set)

Example: Your `spec-reviewer`, `code-quality-reviewer`, `implementer` setup

---

## Recommended Next Steps for Exploration

### 1. **Prototype a Design Spec Reviewer**

- Create `~/.pi/agent/agents/design-spec-reviewer.md`
- Use high-thinking model (qwen3.6-plus or similar)
- Base methodology on your current CLI agent approach
- Test dispatch: `subagent({ agent: "design-spec-reviewer", task: "..." })`

### 2. **Prototype an Implementation Plan Reviewer**

- Create `~/.pi/agent/agents/plan-reviewer.md`
- Similar structure to spec-reviewer
- Check plan against spec and feasibility criteria
- Test with a sample plan

### 3. **Create a Review Workflow Chain**

- Spec generation → spec review → plan generation → plan review
- Use async execution to keep work flowing
- Test with your next brainstorm/spec/plan cycle

### 4. **Extend with Parallel Reviews**

- Launch multiple reviewers (design, feasibility, completeness)
- Synthesize findings
- Create revision cycle if needed

### 5. **Integrate with Existing Tools**

- Add to your brainstorming skill workflow
- Trigger from plan-writing workflow
- Pipe results into implementation prep

---

## Key Insights

### ✅ What Works Today

1. **Agent files** - Fully supported, use frontmatter like your `implementer.md`
2. **Dispatch patterns** - `subagent()` tool + custom agents ready to use
3. **Model selection** - You can assign specific models per agent (per frontmatter)
4. **Tool isolation** - Each agent gets explicit tool list (no edits for reviewers)
5. **Thinking levels** - High/medium/low for complexity matching
6. **Async execution** - Launch reviewers non-blocking, continue work
7. **Output files** - Save reviews to artifacts for synthesis

### ⚠️ Current Limitations

1. **No pre-built artifact-review agents** - You'd create them (but pattern exists)
2. **Frontmatter is extensible but not auto-processed** - Custom fields are visible but not acted upon by Pi
3. **No specialized "review-sync" mode yet** - You control when reviews happen via `async: true/false`
4. **Intercom coordination optional** - Works great without it, but requires both agents to have the bridge

### 🚀 What's Possible

1. Create `design-spec-reviewer` and `plan-reviewer` agents once, reuse forever
2. Dispatch in your workflow with one-liner: `subagent({ agent: "design-spec-reviewer", task: "..." })`
3. Run async reviews while continuing brainstorm/planning work
4. Parallel-review artifacts from multiple angles
5. Chain reviews into handoff plans for implementation
6. Use existing `implementer`, `spec-reviewer`, `code-quality-reviewer` for code phase

---

## Architecture: Your Proposed Review Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ Your Orchestrator Session (Parent)                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Brainstorm → Design Spec (artifact)                        │
│       ↓                                                     │
│  [dispatch design-spec-reviewer async] ──┐                 │
│       ↓                                   ↓                 │
│  Planning (while review runs) ←── spec-review.md           │
│       ↓                                                     │
│  Implementation Plan (artifact)                            │
│       ↓                                                     │
│  [dispatch plan-reviewer async] ──┐                        │
│       ↓                            ↓                        │
│  Ready for implementation ←── plan-review.md               │
│       ↓                                                     │
│  [dispatch implementer] (existing workflow)                │
│       ↓                                                     │
│  [dispatch spec-reviewer, code-quality-reviewer]           │
│       ↓                                                     │
│  Complete                                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## File Locations Reference

### Builtin Agents

- **Source:** `/Users/dtbndr/.volta/tools/image/packages/@earendil-works/pi-coding-agent/lib/node_modules/@earendil-works/pi-coding-agent/dist/builtin-agents.md` (built-in)
- **Invoke:** `subagent({ agent: "scout" })` — uses builtin

### Your Custom Agents

- **Location:** `~/.pi/agent/agents/`
- **Files:**
  - `~/.pi/agent/agents/implementer.md`
  - `~/.pi/agent/agents/spec-reviewer.md`
  - `~/.pi/agent/agents/code-quality-reviewer.md`

### Project-Specific Agents (Optional)

- **Location:** `.pi/agents/` (in your project root)
- **Precedence:** Project agents override user agents override builtins
- **Example:** `/Users/dtbndr/Development/test-ws/pi-demo/.pi/agents/`

### Skills / Methodologies

- **Subagent-driven-development:** `~/.pi/agent/skills/subagent-driven-development/SKILL.md`
- **Pi-subagents:** `~/.pi/agent/npm/node_modules/pi-subagents/skills/pi-subagents/SKILL.md`

---

## Summary: You Have Everything You Need

| Component            | Status        | How to Use                              |
| -------------------- | ------------- | --------------------------------------- |
| Builtin agents       | ✅ Available  | `subagent({ agent: "reviewer" })`       |
| Custom agent pattern | ✅ Proven     | See `~/.pi/agent/agents/implementer.md` |
| Frontmatter config   | ✅ Extensible | Add custom fields, document in prompt   |
| Dispatch mechanism   | ✅ Ready      | `subagent()` tool with role names       |
| Async execution      | ✅ Works      | `async: true` for non-blocking reviews  |
| Parallel reviews     | ✅ Supported  | `tasks: [...]` in subagent call         |
| Chain workflows      | ✅ Supported  | `chain: [...]` for sequential reviews   |
| Model override       | ✅ Per-agent  | Set `model:` in frontmatter             |
| Thinking levels      | ✅ Per-agent  | Set `thinking:` in frontmatter          |

**Next action:** Create `design-spec-reviewer.md` and `plan-reviewer.md` using the existing agent file pattern.
