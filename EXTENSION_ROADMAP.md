# Extension Roadmap: From CLI Agents to Pi Subagents

This document maps your current workflow to subagent equivalents and shows how to incrementally migrate.

---

## Current State: CLI Agents for Artifact Review

```
Your workflow:
┌─────────────────┬──────────────────┬────────────────┐
│ Brainstorm      │ Design Spec      │ CLI Agent 1    │
│ (Pi session)    │ (markdown)       │ (review spec)  │
│                 │                  │                │
│                 ├──────────────────┼────────────────┤
│                 │ Implementation   │ CLI Agent 2    │
│                 │ Plan (markdown)  │ (review plan)  │
│                 │                  │                │
│                 ├──────────────────┼────────────────┤
│                 │ Code (git)       │ CLI Agents 3+4 │
│                 │                  │ (implement,    │
│                 │                  │  review)       │
└─────────────────┴──────────────────┴────────────────┘
```

**Current tools:** Separate CLI agents (Claude Code, Gemini CLI, etc.)

**Friction points:**

- Context switching between tools
- No unified dispatch mechanism
- Artifacts copied manually between tools
- Reviews not integrated into workflow
- No persistent review chain

---

## Proposed State: Unified Subagent Dispatch

```
New workflow:
┌──────────────────────────────────────────────────────┐
│ Your Pi Orchestrator Session (Parent)                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Brainstorm                                          │
│    ↓                                                 │
│  Design Spec (markdown artifact)                     │
│    ├─→ [dispatch design-spec-reviewer async] ─┐    │
│    ├─→ Plan (markdown artifact)                │    │
│    │    ├─→ [dispatch plan-reviewer async] ─┐ │    │
│    │    └─→ Implementation Phase             │ │    │
│    │         ├─→ [dispatch implementer]       │ │    │
│    │         ├─→ [dispatch spec-reviewer]     │ │    │
│    │         └─→ [dispatch code-quality-rev] │ │    │
│    │                                         │ │    │
│    └─ spec-review.md ←─────────────────────┘ │    │
│                                                │    │
│ plan-review.md ←────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**New approach:** Subagents + unified dispatch from Pi

**Benefits:**

- Same session, no context switching
- Async non-blocking reviews
- Reviews integrated into workflow
- Persistent review artifacts
- Same model selection and thinking level per agent

---

## Migration Path (3 Phases)

### Phase 1: Establish Artifact Review Agents (Week 1)

**Goal:** Create the design-spec-reviewer and plan-reviewer agents

**Steps:**

1. **Create design-spec-reviewer agent**

   ```bash
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

   [Your review methodology here]
   EOF
   ```

2. **Create plan-reviewer agent**

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

   # Plan Reviewer

   [Your review methodology here]
   EOF
   ```

3. **Test dispatch pattern**

   ```typescript
   // In a test session, verify dispatch works:
   subagent({
     agent: "design-spec-reviewer",
     task: "Review a test spec",
     async: true,
     output: "test-review.md",
   });
   ```

4. **Document in AGENTS.md**
   - Add section: "Artifact Review Agents"
   - List: design-spec-reviewer, plan-reviewer
   - Note: Ready for next phase

---

### Phase 2: Integrate into Brainstorm Workflow (Week 2)

**Goal:** Dispatch reviews automatically after each artifact generation

**Steps:**

1. **Update brainstorming skill** (or create wrapper)
   - After spec generation, dispatch design-spec-reviewer async
   - Continue planning while review runs
   - Include review results in spec generation summary

2. **Update planning skill** (or create wrapper)
   - After plan generation, dispatch plan-reviewer async
   - Compare plan against spec requirements
   - Include review results in plan generation summary

3. **Create review synthesis process**
   - After reviews complete, synthesize findings
   - Mark design spec as "Approved" or "Needs revision"
   - Same for implementation plan

4. **Testing**
   - Run full cycle: brainstorm → spec (review) → plan (review)
   - Verify reviews run async
   - Verify output artifacts created
   - Test revision loops

---

### Phase 3: Advanced Workflows (Week 3+)

**Goal:** Extend to parallel reviews, different angles, conditional dispatch

**Steps:**

1. **Parallel design review** (multiple angles)
   - Architecture review (design patterns)
   - Requirements review (completeness)
   - Security review (threat modeling)
   - Feasibility review (technical risks)

2. **Conditional dispatch**
   - Route artifacts to appropriate reviewers
   - Use different models for different focuses
   - Parallel reviews for large specs/plans

3. **Model optimization**
   - Measure which models work best for each review type
   - Create agent overrides in settings.json
   - Document optimal model per review

4. **Integration with implementation**
   - Link approved specs to implementer context
   - Link approved plans to task dispatch
   - Feed synthesis into worker prompts

---

## Integration Points with Existing Agents

### Current Custom Agents (Your Setup)

```
~/.pi/agent/agents/
├── implementer.md (deepseek-v4-pro, high thinking)
├── spec-reviewer.md (qwen3.6-plus, high thinking) ← code spec reviewer
├── code-quality-reviewer.md (deepseek-v4-flash, low thinking)
│
├── design-spec-reviewer.md (NEW - to create)
└── plan-reviewer.md (NEW - to create)
```

### Proposed Workflow Integration

```
Design Phase (NEW):
  brainstorm → design-spec-reviewer → plan-reviewer

Implementation Phase (EXISTING):
  implementer → spec-reviewer → code-quality-reviewer

Full cycle:
  Design phase → Implementation phase
  (reviews → artifacts → implementation → code reviews)
```

---

## Comparison Table: CLI Agents vs. Subagents

| Aspect            | CLI Agents              | Subagents                      | Winner    |
| ----------------- | ----------------------- | ------------------------------ | --------- |
| Context switching | Manual (switch windows) | None (same session)            | Subagents |
| Review dispatch   | Manual prompt/copy      | `subagent()` tool              | Subagents |
| Artifact handoff  | Copy-paste files        | Direct file paths              | Subagents |
| Async execution   | Manual wait             | Built-in `async: true`         | Subagents |
| Parallel reviews  | Start multiple CLIs     | `tasks: [...]`                 | Subagents |
| Chain reviews     | Manual sequencing       | `chain: [...]`                 | Subagents |
| Output location   | Varies per tool         | Unified `output:` path         | Subagents |
| Model selection   | Per-tool default        | Per-agent, overridable         | Subagents |
| Integration       | External                | Part of Pi                     | Subagents |
| Discovery         | Browser                 | `subagent({ action: "list" })` | Subagents |

---

## Frontmatter Extensions You Can Use

### Design Spec Reviewer Frontmatter

```yaml
---
name: design-spec-reviewer
description: Design spec reviewer — validates design artifacts
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash

# Custom metadata (for your reference/tooling)
artifact_type: design_spec
review_focus: [clarity, completeness, feasibility, architecture]
output_format: markdown
checklist:
  - Missing requirements
  - Ambiguous language
  - Architectural gaps
  - Dependency unmapped
  - Feasibility risks
  - Security concerns

# Optional: reference to methodology
methodology_source: ~/.pi/agent/skills/design-review/SKILL.md
---
```

### Plan Reviewer Frontmatter

```yaml
---
name: plan-reviewer
description: Implementation plan reviewer — validates plan artifacts
model: opencode-go/qwen3.6-plus
thinking: high
systemPromptMode: replace
inheritProjectContext: true
inheritSkills: false
tools: read, grep, find, ls, bash

# Custom metadata
artifact_type: implementation_plan
review_focus: [scope_alignment, task_breakdown, sequencing, feasibility]
output_format: markdown
checklist:
  - Requirements covered
  - Task decomposition
  - Dependencies explicit
  - Feasibility assessed
  - Testing strategy
  - Risk identification

# Optional: reference to methodology
methodology_source: ~/.pi/agent/skills/plan-review/SKILL.md
---
```

**Note:** Custom fields are metadata for documentation. The tool reads them as YAML but Pi doesn't process them specially. Your prompts and workflows can reference them.

---

## Settings Configuration

### Add to ~/.pi/agent/settings.json

```json
{
  "subagents": {
    "agentOverrides": {
      "design-spec-reviewer": {
        "description": "High-thinking design spec validator",
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high",
        "tools": "read,grep,find,ls,bash"
      },
      "plan-reviewer": {
        "description": "Implementation plan validator",
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high",
        "tools": "read,grep,find,ls,bash"
      },
      "parallel-design-review": {
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high"
      },
      "quick-feasibility-check": {
        "model": "opencode-go/deepseek-v4-flash",
        "thinking": "low"
      }
    }
  }
}
```

Then dispatch:

```typescript
subagent({ agent: "design-spec-reviewer", ... })
subagent({ agent: "plan-reviewer", ... })
subagent({ agent: "quick-feasibility-check", ... })
```

---

## Testing & Validation Checklist

### Phase 1: Agent Creation ✓

- [ ] design-spec-reviewer.md created
- [ ] plan-reviewer.md created
- [ ] Both agents discoverable via `subagent({ action: "list" })`
- [ ] Manual dispatch test successful

### Phase 2: Workflow Integration ✓

- [ ] Brainstorm creates spec artifact
- [ ] design-spec-reviewer dispatched async
- [ ] Review output saved to file
- [ ] Planning continues while review runs
- [ ] Plan reviewer dispatched async
- [ ] Parallel execution confirmed (both async)

### Phase 3: Review Cycles ✓

- [ ] Design spec review → found issues → revise → re-review loop works
- [ ] Plan review → found issues → revise → re-review loop works
- [ ] Reviews show issues in consistent format
- [ ] Synthesis of parallel reviews works

### Phase 4: Model Optimization ✓

- [ ] Identified optimal models per review type
- [ ] Settings overrides tested
- [ ] Cost vs. quality trade-offs documented
- [ ] Performance measurements taken

---

## Rollout Plan

### Week 1: Foundation

- ✅ Document current state (this doc)
- ✅ Create exploration docs (SUBAGENT_EXPLORATION.md)
- ✅ Create example docs (SUBAGENT_USAGE_EXAMPLES.md)
- [ ] Create design-spec-reviewer agent
- [ ] Create plan-reviewer agent
- [ ] Test individual dispatch

### Week 2: Integration

- [ ] Integrate into brainstorm workflow
- [ ] Integrate into planning workflow
- [ ] Test async execution
- [ ] Test review synthesis

### Week 3: Optimization

- [ ] Parallel reviews for complex specs
- [ ] Model selection per review type
- [ ] Settings-based configuration
- [ ] Performance tuning

### Week 4+: Advanced

- [ ] Custom frontmatter for artifact metadata
- [ ] Conditional dispatch logic
- [ ] Integration with implementation phase
- [ ] Feedback loop: reviews → workflow optimization

---

## Open Questions for Future Exploration

1. **Frontmatter extensions**
   - Can we create custom fields that Pi processes? (E.g., `review_checklist:` that agent auto-reads)
   - How to best document artifact types in frontmatter?

2. **Intercom coordination**
   - Can reviewers ask clarifying questions via `contact_supervisor`?
   - Example: "Spec is ambiguous on error handling. Should this be synchronous or async?"

3. **Review artifacts**
   - Should reviews be committed to git?
   - How to track review history for features?

4. **Conditional dispatch**
   - Create a dispatcher that routes based on artifact characteristics?
   - E.g., large spec → parallel reviews, small spec → single review

5. **Model selection**
   - Create a skill for model optimization per review type?
   - Measure quality vs. cost for each model?

6. **Integration with CLI agents**
   - Can you still use CLI agents alongside subagents?
   - How to migrate existing workflows incrementally?

7. **Skill creation**
   - Create "artifact-review" skill that encapsulates dispatch patterns?
   - Create "design-review" and "plan-review" skills?

---

## Files Created

1. **SUBAGENT_EXPLORATION.md** — Comprehensive discovery of existing agents and capabilities
2. **SUBAGENT_USAGE_EXAMPLES.md** — Ready-to-use dispatch examples
3. **EXTENSION_ROADMAP.md** (this file) — Migration path from CLI agents to subagents

## Next Steps

1. ✅ Read SUBAGENT_EXPLORATION.md to understand what's available
2. ✅ Read SUBAGENT_USAGE_EXAMPLES.md for concrete examples
3. ⬜ Create design-spec-reviewer.md using provided template
4. ⬜ Create plan-reviewer.md using provided template
5. ⬜ Test dispatch with sample spec/plan
6. ⬜ Integrate into your brainstorm/planning workflow
7. ⬜ Document learnings and optimizations

---

## Summary

| Component           | Current                        | Proposed                      | Status    |
| ------------------- | ------------------------------ | ----------------------------- | --------- |
| Design spec review  | CLI agent                      | design-spec-reviewer subagent | To create |
| Plan review         | CLI agent                      | plan-reviewer subagent        | To create |
| Code implementation | implementer subagent           | ✅ Exists                     | Using     |
| Code spec review    | spec-reviewer subagent         | ✅ Exists                     | Using     |
| Code quality review | code-quality-reviewer subagent | ✅ Exists                     | Using     |
| Dispatch mechanism  | Manual/copy-paste              | subagent() tool               | Available |
| Async execution     | Manual wait                    | Built-in async: true          | Available |
| Parallel reviews    | Multiple CLI windows           | tasks: [...]                  | Available |
| Chain reviews       | Manual sequencing              | chain: [...]                  | Available |

**Key insight:** You have everything you need. The custom agents already exist (`implementer`, `spec-reviewer`, `code-quality-reviewer`). You just need to create the design/plan review agents using the same pattern and integrate them into your workflow.
