# Subagent Quick Reference Card

## 🎯 Your Goal
```
Brainstorm → Design Spec (review) → Implementation Plan (review) → Implement
                    ↓                         ↓
            [design-spec-reviewer]   [plan-reviewer]
            (to create)              (to create)
```

---

## ✅ What Exists

### Builtin Agents (8)
```
scout, planner, worker, reviewer, context-builder, researcher, oracle, delegate
```

### Your Custom Agents (3)
```
implementer (deepseek-v4-pro)
spec-reviewer (qwen3.6-plus)
code-quality-reviewer (deepseek-v4-flash)
```

### Dispatch Mechanism
```typescript
subagent({ agent: "name", task: "..." })
```

---

## 📝 Create These Two Agents

### 1. Design Spec Reviewer
```bash
cat > ~/.pi/agent/agents/design-spec-reviewer.md << 'AGENT'
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

Review design specs for:
- Completeness (all requirements present)
- Clarity (unambiguous language)
- Feasibility (achievable with available tech)
- Architecture (system boundaries clear)
- Quality (security, performance, error handling addressed)

Report format:
✅ **Summary:** [one-line assessment]

**Strengths:**
- [strength with section reference]

**Issues:**
- Critical: [blocks planning]
- Important: [should clarify]
- Minor: [nice to improve]

**Recommendations:**
- [specific improvement]
AGENT
```

### 2. Implementation Plan Reviewer
```bash
cat > ~/.pi/agent/agents/plan-reviewer.md << 'AGENT'
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

Review implementation plans for:
- Scope alignment (covers all spec requirements)
- Task breakdown (appropriately sized, focused)
- Sequencing (logical order, explicit dependencies)
- Feasibility (realistic, no risky assumptions)
- Validation (acceptance criteria testable)

Report format:
✅ **Summary:** [one-line assessment]

**Strengths:**
- [strength with task reference]

**Issues:**
- Critical: [blocks implementation]
- Important: [should fix before starting]
- Minor: [nice to clarify]

**Recommendations:**
- [specific improvement]
AGENT
```

---

## 🚀 Usage Patterns

### Single Async Review
```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review docs/design-specs/feature.md",
  async: true,
  output: "reviews/spec.md"
})
```

### Parallel Multi-Angle Reviews
```typescript
subagent({
  tasks: [
    { agent: "design-spec-reviewer", task: "Architecture review", output: "reviews/arch.md" },
    { agent: "reviewer", task: "Requirements review", output: "reviews/reqs.md" },
    { agent: "researcher", task: "Best practices research", output: "reviews/research.md" }
  ],
  concurrency: 3,
  async: true
})
```

### Sequential Chain (Spec → Plan → Impl)
```typescript
subagent({
  chain: [
    { agent: "design-spec-reviewer", task: "Review spec", output: "reviews/spec.md" },
    { agent: "plan-reviewer", task: "Review plan: {previous}", output: "reviews/plan.md" },
    { agent: "implementer", task: "Implement: {previous}" }
  ],
  async: true
})
```

### Review with Revision Loop
```typescript
// Review → Found issues? → Revise → Re-review
// Repeat until approved
```

### Fresh Context (No History Pollution)
```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review spec",
  context: "fresh",  // Clean context
  async: true
})
```

---

## 📊 Model Selection

### For Design/Architecture Reviews (High Thinking)
```yaml
model: opencode-go/qwen3.6-plus
thinking: high
```

### For Quality/Feasibility Checks (Medium)
```yaml
model: opencode-go/deepseek-v4-flash
thinking: medium
```

### For Quick Checks (Low)
```yaml
model: opencode-go/deepseek-v4-flash
thinking: low
```

---

## 🔧 Configuration

### In ~/.pi/agent/settings.json
```json
{
  "subagents": {
    "agentOverrides": {
      "design-spec-reviewer": {
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high"
      },
      "plan-reviewer": {
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high"
      }
    }
  }
}
```

---

## 📍 File Locations

### Custom Agents Location
```
~/.pi/agent/agents/
├── implementer.md ✅
├── spec-reviewer.md ✅
├── code-quality-reviewer.md ✅
├── design-spec-reviewer.md (CREATE)
└── plan-reviewer.md (CREATE)
```

### Builtin Agents Access
```typescript
subagent({ agent: "scout" })      // Works automatically
subagent({ agent: "planner" })    // Works automatically
```

---

## ✨ Advanced Patterns

### Conditional Dispatch
```typescript
function reviewArtifact(path, type) {
  if (type === "design-spec")
    return subagent({ agent: "design-spec-reviewer", task: `Review ${path}` });
  if (type === "plan")
    return subagent({ agent: "plan-reviewer", task: `Review ${path}` });
}
```

### Model Override Per Dispatch
```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "...",
  model: "opencode-go/qwen3.6-plus"  // Override agent's default
})
```

### Synthesis After Parallel Reviews
```typescript
// Run 3 parallel reviews, then:
subagent({
  agent: "reviewer",
  task: "Synthesize these 3 reviews and produce consensus...",
  async: true
})
```

### Error Handling
```typescript
try {
  subagent({ agent: "design-spec-reviewer", task: "..." })
} catch (error) {
  console.error("Review failed:", error)
  // Fallback to builtin reviewer
  subagent({ agent: "reviewer", task: "..." })
}
```

---

## 🎯 Workflow Before & After

### BEFORE: CLI Agents
```
Pi session (brainstorm)
  ↓ create spec
Switch to Claude Code / Gemini
  ↓ review manually
Switch back to Pi (planning)
  ↓ copy-paste notes
Switch to another CLI
  ↓ review manually
Switch back to Pi (implement)
  ↓ copy-paste notes
```
**Problem:** Context switching, manual copy-paste, no unified flow

### AFTER: Subagent Dispatch
```
Pi orchestrator
  ├─ Brainstorm → Spec
  │  ├─ dispatch design-spec-reviewer async
  │  └─ continue planning (non-blocking)
  ├─ Plan → Implementation Plan
  │  ├─ dispatch plan-reviewer async
  │  └─ continue work (non-blocking)
  └─ Implementation
     ├─ dispatch implementer
     ├─ dispatch spec-reviewer
     └─ dispatch code-quality-reviewer
```
**Benefit:** Same session, async non-blocking, unified flow

---

## 📋 Checklist

### Week 1: Create Agents
- [ ] Create design-spec-reviewer.md
- [ ] Create plan-reviewer.md
- [ ] Test dispatch with sample spec
- [ ] Test dispatch with sample plan
- [ ] Verify output files created

### Week 2: Integrate Into Workflow
- [ ] Add to brainstorm workflow (auto-review spec)
- [ ] Add to planning workflow (auto-review plan)
- [ ] Test async execution (reviews run parallel)
- [ ] Test output artifact creation
- [ ] Document review format

### Week 3: Optimize
- [ ] Test parallel reviews (multiple angles)
- [ ] Implement review synthesis
- [ ] Measure model performance vs. cost
- [ ] Optimize thinking levels
- [ ] Consider conditional dispatch

---

## 💡 Tips

### Tip 1: Start Simple
```typescript
// Don't start with complex chains
// Start with: single agent, single dispatch
subagent({ agent: "design-spec-reviewer", task: "Review spec", async: true })
```

### Tip 2: Use Fresh Context
```typescript
// Reviews should have clean context
context: "fresh"  // No session history pollution
```

### Tip 3: Run Async
```typescript
// Non-blocking = keep moving
async: true  // Continue work while review runs
```

### Tip 4: Save to Artifacts
```typescript
// Persistent outputs for synthesis
output: "reviews/spec-review.md"
output: "reviews/plan-review.md"
```

### Tip 5: Check Output Files
```bash
# After dispatch completes:
cat reviews/spec-review.md
cat reviews/plan-review.md
```

---

## ❓ Quick FAQ

**Q: Do I have to create new agents?**
A: No. But custom agents give you model/thinking control per review type.

**Q: Can I use builtin `reviewer` instead?**
A: Yes! But it won't be optimized for design/plan review.

**Q: How long does a review take?**
A: Depends on spec/plan size. Usually 30sec-2min. Async means you don't wait.

**Q: Where do reviews go?**
A: Wherever you specify in `output:` parameter. Default: `reviews/file.md`

**Q: Can I run reviews in parallel?**
A: Yes! Use `tasks: [...]` with `concurrency: 3`

**Q: Can I chain reviews?**
A: Yes! Use `chain: [...]` and `{previous}` for output passing.

---

## 🔗 References

### Documentation Files
- **Start here:** SUBAGENT_SUMMARY.md (5 min)
- **Deep dive:** SUBAGENT_EXPLORATION.md (20 min)
- **Code examples:** SUBAGENT_USAGE_EXAMPLES.md (30 min)
- **Implementation plan:** EXTENSION_ROADMAP.md (20 min)
- **Navigation guide:** README_SUBAGENT_EXPLORATION.md

### Pi Documentation
- **Subagents skill:** `~/.pi/agent/npm/node_modules/pi-subagents/skills/pi-subagents/SKILL.md`
- **Subagent dispatch:** `~/.pi/agent/skills/pi-subagent-dispatch/SKILL.md`

### Your Agents
- **Location:** `~/.pi/agent/agents/`
- **Pattern reference:** `implementer.md`

---

## 🚀 Next Action

```bash
# 1. Create agents (copy-paste from above)
cat > ~/.pi/agent/agents/design-spec-reviewer.md << ...
cat > ~/.pi/agent/agents/plan-reviewer.md << ...

# 2. Test dispatch
# (in Pi session)
subagent({
  agent: "design-spec-reviewer",
  task: "Review a test spec",
  async: true,
  output: "test-review.md"
})

# 3. Check output
cat test-review.md

# 4. Integrate into workflow
# (add dispatch calls to brainstorm/planning)

# 5. Done!
```

**Time to first working review: ~20 minutes**

---

**Key insight:** You have the pattern. You have the tools. You're ready to go. Just create the 2 agent files and dispatch them.
