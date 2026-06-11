# Subagent Usage Examples: Design & Plan Review

This document provides ready-to-use examples for integrating artifact review into your workflow using subagents.

---

## Example 1: Single Design Spec Review

**Scenario:** After brainstorming creates a design spec, immediately review it.

```typescript
// In your brainstorm → design spec workflow
const specPath = "docs/design-specs/feature-auth.md";

const reviewResult = subagent({
  agent: "design-spec-reviewer",
  task: `Review this design spec for completeness and clarity:
  
Spec artifact: ${specPath}
Context: This spec defines the new OAuth2 authentication flow for the API.

Please review for:
- Missing acceptance criteria
- Ambiguous requirements
- Architectural concerns (async, caching, security)
- Dependencies not clearly stated
- Risk factors not addressed

Return format:
✅ Summary (one line assessment)

Strengths:
- [specific strength with file/section references]

Issues:
- Critical: [issue that blocks implementation]
- Important: [should fix before implementation]
- Minor: [nice to clarify]

Recommendations:
- [suggested improvement]`,

  async: true, // Non-blocking
  output: "reviews/design-spec-review.md",
  context: "fresh", // Clean context, no session history pollution
});

// Continue with planning while review runs
// Later: check reviewResult or read the output file
```

---

## Example 2: Parallel Design Reviews (Multi-Angle)

**Scenario:** Get diverse perspectives on your design spec before planning.

```typescript
const specPath = "docs/design-specs/feature-auth.md";

subagent({
  tasks: [
    {
      agent: "design-spec-reviewer",
      task: `Architecture & Design Review:
      
Spec: ${specPath}

Review for:
- Design patterns (SOLID, DRY, separation of concerns)
- System boundaries (what's in scope, what's external)
- Data flow (how data moves through system)
- Scaling considerations
- Maintainability concerns`,

      output: "reviews/spec-architecture-review.md",
    },
    {
      agent: "reviewer", // Builtin generic reviewer
      task: `Requirements & Completeness Review:
      
Spec: ${specPath}

Review for:
- All user stories/acceptance criteria present
- Edge cases considered
- Error handling specified
- Security requirements explicit
- Non-functional requirements (performance, compliance)`,

      output: "reviews/spec-requirements-review.md",
    },
    {
      agent: "researcher", // External research
      task: `OAuth2 Implementation Research:

Research current best practices for:
- OAuth2 flow implementation patterns in 2026
- Security vulnerabilities in OAuth2
- Common pitfalls when implementing OAuth2
- Recommended libraries/frameworks

Then assess: Are the patterns in ${specPath} aligned with current best practices?

Return: findings.md with recommendations`,

      output: "reviews/spec-oauth2-research.md",
    },
  ],

  concurrency: 3, // Run all 3 in parallel
  context: "fresh", // Each gets independent context
  async: true, // Non-blocking
});

// Continue work while all 3 review in parallel
// Later: read the 3 review files and synthesize findings
```

---

## Example 3: Sequential Spec Review → Plan Review Chain

**Scenario:** Validate design spec, then validate implementation plan against it.

```typescript
subagent({
  chain: [
    {
      agent: "design-spec-reviewer",
      task: `Review the design spec:
      
Spec: docs/design-specs/feature-auth.md

Check for completeness, clarity, and architectural soundness.
Return standard review format.`,

      output: "chain-artifacts/spec-review.md",
    },
    {
      agent: "plan-reviewer",
      task: `Validate the implementation plan against the approved design spec.

Design spec: docs/design-specs/feature-auth.md
Spec review: {previous}
Implementation plan: docs/plans/auth-implementation-plan.md

Verify:
- All design spec requirements are addressed
- Tasks appropriately break down requirements
- Task sequencing is logical
- Dependencies are explicit
- Testing strategy is complete

Return: Plan review in standard format`,

      output: "chain-artifacts/plan-review.md",
    },
  ],

  context: "fresh",
  async: true,
});

// After chain completes:
// 1. Read spec-review.md
// 2. Synthesize any issues
// 3. Update design spec if needed
// 4. Read plan-review.md
// 5. Update plan if needed
// 6. Proceed to implementation phase
```

---

## Example 4: Design Spec Review with Revision Loop

**Scenario:** Review → revise → re-review until approved.

```typescript
async function reviewSpecWithLoop(specPath, maxIterations = 3) {
  let iteration = 0;
  let approved = false;

  while (iteration < maxIterations && !approved) {
    iteration++;
    console.log(`Spec review iteration ${iteration}...`);

    // Review the spec
    const review = subagent({
      agent: "design-spec-reviewer",
      task: `Review design spec (iteration ${iteration}):
      
      File: ${specPath}
      
      Report: ✅ Approved, or ❌ Issues found with specific feedback`,

      async: false, // Blocking - we need result to decide next step
    });

    // Check if approved (assuming review output contains ✅ or ❌)
    const content = fs.readFileSync("reviews/design-spec-review.md", "utf-8");
    if (content.includes("✅")) {
      approved = true;
      console.log("✅ Spec approved!");
      return true;
    }

    // Not approved - revise and loop
    console.log("❌ Issues found. Needs revision.");
    if (iteration < maxIterations) {
      console.log("Ready to revise spec based on feedback...");
      // Here you'd prompt user or auto-revise based on feedback
    }
  }

  return false; // Max iterations reached
}

// Usage:
// const isApproved = await reviewSpecWithLoop("docs/design-specs/feature-auth.md");
```

---

## Example 5: Plan Review in Context of Spec

**Scenario:** Implement plan reviewer as a specialized agent, review in isolation.

**File: `~/.pi/agent/agents/plan-reviewer.md`**

```markdown
---
name: plan-reviewer
description: Implementation plan reviewer — validates plan for scope, tasks, sequencing
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Implementation Plan Reviewer

You are the plan reviewer. Your job is to validate implementation plans for quality, accuracy, and feasibility.

## Your Role

Do not implement or edit code. Read the plan and design spec carefully. Return a structured review.

## What to Check

1. **Scope Alignment**
   - Does the plan cover all design spec requirements?
   - Any missing features or requirements?
   - Any scope creep (features not in spec)?

2. **Task Breakdown**
   - Are tasks appropriately sized (not too large, not too small)?
   - Are tasks focused and cohesive?
   - Can tasks be implemented independently?

3. **Sequencing & Dependencies**
   - Are tasks in logical order?
   - Are dependencies explicit and correct?
   - Any circular dependencies?

4. **Validation & Acceptance**
   - Are acceptance criteria testable?
   - Are validation approaches adequate?
   - Any missing test scenarios?

5. **Feasibility**
   - Do tasks assume available tools/infrastructure?
   - Any risky or unclear technical approaches?
   - Reasonable time/effort estimates?

## Report Format

✅ **Plan Assessment:** [One-line judgment]

**Strengths:**

- [strength with section reference]

**Issues:**

- Critical: [blocks implementation]
- Important: [should fix before implementation]
- Minor: [nice to clarify]

**Recommendations:**

- [specific improvement]
```

**Dispatch the plan reviewer:**

```typescript
subagent({
  agent: "plan-reviewer",
  task: `Review this implementation plan:

Design Spec: docs/design-specs/feature-auth.md
Implementation Plan: docs/plans/auth-implementation-plan.md

This plan defines the tasks to implement OAuth2 authentication.
Validate that the plan is well-structured, complete, and feasible.`,

  output: "reviews/plan-review.md",
  context: "fresh",
});
```

---

## Example 6: Design Spec Reviewer Agent

**File: `~/.pi/agent/agents/design-spec-reviewer.md`**

```markdown
---
name: design-spec-reviewer
description: Design spec reviewer — validates design artifacts for completeness and feasibility
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash
---

# Design Spec Reviewer

You are the design spec reviewer. Your job is to validate design specs before they go into planning.

## Your Role

Do not implement. Read the spec carefully. Return a structured review.

## What to Check

1. **Completeness**
   - All user stories / use cases covered?
   - Acceptance criteria present and clear?
   - Edge cases considered?

2. **Clarity**
   - Requirements unambiguous?
   - Technical terms defined?
   - Diagrams or examples where helpful?

3. **Feasibility**
   - Realistic with current technology?
   - Dependencies on external systems clear?
   - Scaling/performance requirements explicit?

4. **Architecture**
   - System boundaries defined?
   - Data flow described?
   - Integration points clear?

5. **Quality**
   - Security requirements addressed?
   - Error handling strategy?
   - Non-functional requirements (performance, availability)?

## Report Format

✅ **Spec Assessment:** [One-line judgment]

**Strengths:**

- [strength with section reference]

**Issues:**

- Critical: [blocks planning/implementation]
- Important: [should clarify before planning]
- Minor: [nice to clarify]

**Recommendations:**

- [specific improvement]
```

**Dispatch the design spec reviewer:**

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: `Review this design specification:

Spec: docs/design-specs/feature-auth.md
Context: New OAuth2 authentication flow for the API

Validate for completeness, clarity, and feasibility.
Look for missing requirements, ambiguous language, and architectural gaps.`,

  output: "reviews/design-spec-review.md",
  context: "fresh",
});
```

---

## Example 7: Full Workflow Integration

**Scenario:** Complete cycle from brainstorm → spec review → plan review → implementation

```typescript
async function executeFullWorkflow() {
  console.log("Starting feature workflow...\n");

  // Phase 1: Brainstorm & Design
  console.log("Phase 1: Brainstorm → Design Spec");
  const specPath = "docs/design-specs/new-feature.md";
  // [brainstorm creates spec]

  // Phase 2: Spec Review (async)
  console.log("Launching async spec review...");
  const specReview = subagent({
    agent: "design-spec-reviewer",
    task: `Review: ${specPath}`,
    async: true,
    output: "reviews/spec.md",
  });

  // Phase 3: Planning (while spec review runs)
  console.log("Phase 2: Creating implementation plan...");
  const planPath = "docs/plans/new-feature-plan.md";
  // [planner creates plan]

  // Phase 4: Plan Review (async)
  console.log("Launching async plan review...");
  const planReview = subagent({
    agent: "plan-reviewer",
    task: `Review plan against spec:\n\nPlan: ${planPath}\nSpec: ${specPath}`,
    async: true,
    output: "reviews/plan.md",
  });

  // Phase 5: Synthesis (while reviews run)
  console.log("Phase 3: Preparing for implementation...");
  // Check reviews when ready

  // Phase 6: Implementation (when plan is approved)
  console.log("Phase 4: Dispatching implementation subagents...");
  // [implementer, spec-reviewer, code-quality-reviewer cycle]

  console.log("\n✅ Workflow complete!");
}
```

---

## Example 8: Conditional Dispatch Based on Artifact Type

**Scenario:** Smart dispatch that routes to different reviewers based on artifact.

```typescript
function dispatchArtifactReview(artifactPath, artifactType) {
  if (artifactType === "design-spec") {
    return subagent({
      agent: "design-spec-reviewer",
      task: `Review design spec: ${artifactPath}`,
      async: true,
      output: `reviews/${path.basename(artifactPath, ".md")}-review.md`,
    });
  }

  if (artifactType === "implementation-plan") {
    return subagent({
      agent: "plan-reviewer",
      task: `Review implementation plan: ${artifactPath}`,
      async: true,
      output: `reviews/${path.basename(artifactPath, ".md")}-review.md`,
    });
  }

  if (artifactType === "feature-proposal") {
    return subagent({
      tasks: [
        {
          agent: "reviewer",
          task: `Review proposal for clarity: ${artifactPath}`,
          output: "reviews/proposal-clarity.md",
        },
        {
          agent: "oracle",
          task: `Review proposal for feasibility and risks: ${artifactPath}`,
          output: "reviews/proposal-feasibility.md",
        },
      ],
      async: true,
    });
  }

  throw new Error(`Unknown artifact type: ${artifactType}`);
}

// Usage:
// dispatchArtifactReview("docs/design-specs/auth.md", "design-spec")
// dispatchArtifactReview("docs/plans/auth-plan.md", "implementation-plan")
// dispatchArtifactReview("docs/proposals/api-redesign.md", "feature-proposal")
```

---

## Example 9: Model Selection Per Review Type

**Scenario:** Use different models for different review focuses.

```typescript
// Create agent overrides in ~/.pi/agent/settings.json

{
  "subagents": {
    "agentOverrides": {
      "design-spec-reviewer": {
        "description": "High-thinking design reviewer",
        "model": "opencode-go/qwen3.6-plus",  // Best for design thinking
        "thinking": "high",
        "tools": "read,grep,find,ls"
      },
      "plan-reviewer": {
        "description": "Plan quality reviewer",
        "model": "opencode-go/deepseek-v4-flash",  // Fast, capable
        "thinking": "medium",
        "tools": "read,grep,find,ls"
      },
      "architecture-reviewer": {
        "description": "Deep architecture analysis",
        "model": "opencode-go/qwen3.6-plus",  // Strongest reasoning
        "thinking": "high",
        "tools": "read,grep,find,ls,bash"
      },
      "feasibility-reviewer": {
        "description": "Quick feasibility check",
        "model": "opencode-go/deepseek-v4-flash",  // Fast
        "thinking": "low",
        "tools": "read,grep,find,ls"
      }
    }
  }
}
```

**Then dispatch with model-specific agents:**

```typescript
subagent({
  tasks: [
    {
      agent: "design-spec-reviewer", // Uses qwen3.6-plus, high thinking
      task: "Review spec for design quality...",
    },
    {
      agent: "feasibility-reviewer", // Uses deepseek-v4-flash, low thinking
      task: "Quick feasibility check...",
    },
    {
      agent: "architecture-reviewer", // Uses qwen3.6-plus, high thinking
      task: "Deep architecture analysis...",
    },
  ],
  concurrency: 3,
  async: true,
});
```

---

## Example 10: Error Handling & Fallbacks

**Scenario:** Handle review failures gracefully.

```typescript
async function safeDispatchReview(agent, task, fallback = null) {
  try {
    return subagent({
      agent,
      task,
      async: false, // Blocking to catch errors
    });
  } catch (error) {
    console.error(`Review failed with agent '${agent}':`, error.message);

    if (fallback) {
      console.log(`Falling back to '${fallback}' agent...`);
      return subagent({
        agent: fallback,
        task,
      });
    }

    // No fallback - return error result
    return {
      status: "error",
      agent,
      error: error.message,
      suggestion: `Try dispatching manually with a different model override`,
    };
  }
}

// Usage with fallback:
// subagent({ agent: "design-spec-reviewer", task: "..." },
//           "reviewer")  // Fallback to generic reviewer if fails
```

---

## Example 11: Synthesis After Parallel Reviews

**Scenario:** Gather multiple review outputs and synthesize findings.

```typescript
async function synthesizeReviews(reviewFiles) {
  const reviews = {};

  for (const file of reviewFiles) {
    const content = fs.readFileSync(file, "utf-8");
    reviews[file] = content;
  }

  const synthesis = subagent({
    agent: "reviewer", // Or "context-builder"
    task: `Synthesize these reviews and produce a consolidated assessment:
    
    ${reviewFiles.map((f) => `Review ${f}: ${reviews[f]}`).join("\n\n")}
    
    Produce:
    - Consensus on approval status
    - List of critical issues (must fix)
    - List of important issues (should fix)
    - List of nice-to-haves
    - Final recommendation`,

    output: "reviews/synthesis.md",
  });

  return synthesis;
}

// Usage:
// const reviews = [
//   "reviews/spec-architecture-review.md",
//   "reviews/spec-requirements-review.md",
//   "reviews/spec-oauth2-research.md"
// ];
// await synthesizeReviews(reviews);
```

---

## Quick Reference: Common Dispatch Patterns

### Pattern 1: Single Review, Async

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review spec",
  async: true,
  output: "reviews/spec.md",
});
```

### Pattern 2: Parallel Reviews

```typescript
subagent({
  tasks: [
    { agent: "design-spec-reviewer", task: "...", output: "reviews/a.md" },
    { agent: "reviewer", task: "...", output: "reviews/b.md" },
  ],
  concurrency: 2,
  async: true,
});
```

### Pattern 3: Sequential Chain

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
      task: "Review plan using {previous}",
      output: "reviews/plan.md",
    },
  ],
  async: true,
});
```

### Pattern 4: With Fresh Context

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review spec",
  context: "fresh", // Clean context
  async: true,
});
```

### Pattern 5: With Model Override

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review spec",
  model: "opencode-go/qwen3.6-plus", // Override agent's default
  async: true,
});
```

---

## Summary

You now have:

1. ✅ Pattern for creating custom review agents (`design-spec-reviewer.md`, `plan-reviewer.md`)
2. ✅ Examples for all dispatch patterns (async, parallel, chain, with fallbacks)
3. ✅ Ready-to-use agent definitions for spec and plan review
4. ✅ Integration examples showing full workflow
5. ✅ Error handling and synthesis patterns

**Next:** Create your agents and test with your next brainstorm/spec/plan cycle.
