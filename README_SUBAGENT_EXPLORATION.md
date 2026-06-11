# Subagent Exploration: Complete Documentation Index

This directory contains comprehensive exploration of Pi subagents for your artifact review workflow.

---

## 📋 Documentation Overview

### 1. **SUBAGENT_SUMMARY.md** ← Start here

**Quick overview (15KB, 5-min read)**

- What you discovered
- Key findings checklist
- Quick start: 3 steps to create agents
- What's possible vs. what's limited
- Bottom line: You have everything you need

**Best for:** Executive summary, quick reference, deciding next steps

---

### 2. **SUBAGENT_EXPLORATION.md** ← Deep dive

**Comprehensive discovery (17KB, 20-min read)**

- Complete builtin agents reference table
- Your custom agents breakdown (implementer, spec-reviewer, code-quality-reviewer)
- Frontmatter configuration options (all supported fields)
- Integration points (3 options for dispatch)
- Extensibility possibilities
- Comparison: Builtin vs. Custom agents
- Recommended next steps
- File locations reference

**Best for:** Understanding the full landscape, what's available, how it works

---

### 3. **SUBAGENT_USAGE_EXAMPLES.md** ← Practical examples

**Ready-to-use code examples (18KB, 30-min read)**

- 11 complete dispatch patterns
- Single review (Example 1)
- Parallel reviews (Example 2)
- Sequential chain (Example 3)
- Review with revision loop (Example 4)
- Custom agent definitions (Examples 5-6)
- Full workflow integration (Example 7)
- Conditional dispatch (Example 8)
- Model selection (Example 9)
- Error handling (Example 10)
- Review synthesis (Example 11)
- Quick reference card

**Best for:** Copy-paste ready code, implementation, testing patterns

---

### 4. **EXTENSION_ROADMAP.md** ← Implementation plan

**Migration path (15KB, 20-min read)**

- Before/after workflow comparison
- 3-phase implementation plan (weeks 1-3+)
- Integration with existing agents
- Comparison table: CLI agents vs. subagents
- Frontmatter extensions you can use
- Settings configuration examples
- Testing & validation checklist
- Rollout plan with weekly breakdown
- Open questions for future exploration

**Best for:** Planning implementation, team coordination, validation strategy

---

## 🎯 Quick Navigation

### By Use Case

**"I just want a quick overview"**
→ Read SUBAGENT_SUMMARY.md (5 min)

**"I want to understand what's available"**
→ Read SUBAGENT_EXPLORATION.md (20 min)

**"I want to implement design-spec-reviewer and plan-reviewer"**
→ Read SUBAGENT_USAGE_EXAMPLES.md Examples 5-6 (5 min)
→ Follow Quick Start in SUBAGENT_SUMMARY.md (10 min)

**"I want to integrate reviews into my workflow"**
→ Read SUBAGENT_USAGE_EXAMPLES.md Examples 7, 8, 11 (15 min)

**"I want to plan the full migration"**
→ Read EXTENSION_ROADMAP.md (20 min)
→ Follow 3-phase plan with testing checklist

**"I want everything"**
→ Read all in order: SUMMARY → EXPLORATION → EXAMPLES → ROADMAP (60 min)

---

## 📊 Key Findings Summary

### What Exists ✅

| Component                | Status       | Location                                          |
| ------------------------ | ------------ | ------------------------------------------------- | ------ | ---- |
| Builtin agents (8 total) | ✅ Ready     | Available via `subagent()`                        |
| Custom agent pattern     | ✅ Proven    | `~/.pi/agent/agents/*.md`                         |
| Your existing agents     | ✅ Working   | implementer, spec-reviewer, code-quality-reviewer |
| Dispatch mechanism       | ✅ Ready     | `subagent({ agent: "...", task: "..." })`         |
| Async execution          | ✅ Works     | `async: true`                                     |
| Parallel dispatch        | ✅ Works     | `tasks: [...]`                                    |
| Sequential chains        | ✅ Works     | `chain: [...]`                                    |
| Model selection          | ✅ Per-agent | Set in frontmatter or settings                    |
| Thinking levels          | ✅ Per-agent | `thinking: high                                   | medium | low` |

### What You Need to Create ⬜

1. **design-spec-reviewer.md** — Design spec validator
2. **plan-reviewer.md** — Implementation plan validator

Both use the same pattern as your existing agents (copy frontmatter, customize prompt).

---

## 🚀 Quick Start

### Step 1: Create Agents (10 minutes)

Copy from SUBAGENT_SUMMARY.md "Quick Start" section:

- Create `~/.pi/agent/agents/design-spec-reviewer.md`
- Create `~/.pi/agent/agents/plan-reviewer.md`

### Step 2: Test Dispatch (5 minutes)

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "Review docs/design-specs/test.md",
  async: true,
  output: "reviews/test.md",
});
```

### Step 3: Integrate Into Workflow (30 minutes)

Choose pattern from SUBAGENT_USAGE_EXAMPLES.md:

- Example 1: Single review after spec generation
- Example 7: Full workflow integration
- Example 8: Conditional dispatch based on artifact type

---

## 📍 File Locations

### Your Custom Agents

```
~/.pi/agent/agents/
├── implementer.md
├── spec-reviewer.md
├── code-quality-reviewer.md
├── design-spec-reviewer.md (CREATE)
└── plan-reviewer.md (CREATE)
```

### Project-Specific Agents (Optional)

```
.pi/agents/
├── any-project-specific-agents.md
```

### Builtin Agents

```
~/.volta/tools/image/packages/@earendil-works/pi-coding-agent/...
```

(Accessed via `subagent()` tool)

### Configuration

```
~/.pi/agent/settings.json
  - defaultModel
  - enabledModels
  - subagents.agentOverrides
```

---

## 🔍 Reference Tables

### Builtin Agents Available

| Agent             | Purpose         | Best For               |
| ----------------- | --------------- | ---------------------- |
| `scout`           | Codebase recon  | Quick exploration      |
| `planner`         | Creates plans   | Multi-step work        |
| `worker`          | Implementation  | Writing code           |
| `reviewer`        | Review-and-fix  | Code inspection        |
| `context-builder` | Handoff context | Structured information |
| `researcher`      | Web research    | External evidence      |
| `oracle`          | Advisory review | Architecture decisions |
| `delegate`        | Generic tasks   | Flexible work          |

### Your Custom Agents

| Agent                   | Model             | Thinking | Tools                                   | Role            |
| ----------------------- | ----------------- | -------- | --------------------------------------- | --------------- |
| `implementer`           | deepseek-v4-pro   | High     | read, grep, find, ls, bash, edit, write | Implementation  |
| `spec-reviewer`         | qwen3.6-plus      | High     | read, grep, find, ls, bash              | Spec compliance |
| `code-quality-reviewer` | deepseek-v4-flash | Low      | read, grep, find, ls, bash              | Quality check   |

### Dispatch Patterns

| Pattern       | Use Case     | Example                                      |
| ------------- | ------------ | -------------------------------------------- |
| Single async  | Quick review | `subagent({ agent: "...", async: true })`    |
| Parallel      | Multi-angle  | `subagent({ tasks: [...], concurrency: 3 })` |
| Chain         | Sequential   | `subagent({ chain: [...] })`                 |
| Fresh context | No pollution | `subagent({ ..., context: "fresh" })`        |

---

## 📝 Workflow Transformation

### Before: CLI Agents

```
Pi session
  → brainstorm → Design Spec artifact
  → Switch to Claude Code / Gemini CLI
    → Manually review spec
    → Return review notes
  → Switch back to Pi
    → Copy notes to context
    → Continue planning → Implementation Plan artifact
  → Switch to another CLI
    → Manually review plan
  → Switch back to Pi
    → Copy notes to context
    → Implement
```

**Problems:**

- Context switching
- Manual copy-paste
- No unified flow
- Reviews not integrated

### After: Subagent Dispatch

```
Pi orchestrator session
  ├─ Brainstorm → Design Spec
  │  ├─ dispatch design-spec-reviewer async
  │  └─ Continue planning (non-blocking)
  │
  ├─ Plan → Implementation Plan
  │  ├─ dispatch plan-reviewer async
  │  └─ Continue work (non-blocking)
  │
  ├─ Implementation
  │  ├─ dispatch implementer
  │  ├─ dispatch spec-reviewer
  │  └─ dispatch code-quality-reviewer
  │
  └─ Done (all in same session)
```

**Benefits:**

- No context switching
- Async non-blocking
- Unified flow
- Reviews integrated
- Single artifact store

---

## 🎓 Key Concepts

### Frontmatter Configuration

All agents use YAML frontmatter:

```yaml
---
name: agent-name # For discovery/dispatch
model: provider/model # Explicit model
thinking: high|medium|low # Reasoning budget
systemPromptMode: replace # Or prepend
inheritProjectContext: true # Include .pi settings
inheritSkills: false # Include available skills
tools: read, grep, edit # CSV list of tools
---
Your system prompt here...
```

### Custom Frontmatter Fields

You can add custom metadata:

```yaml
artifact_type: design_spec
review_focus: [clarity, completeness]
checklist:
  - Missing requirements
  - Ambiguous language
```

These are visible in metadata and can be used in your workflows (though Pi doesn't process them automatically).

### Settings Overrides

Control behavior in `~/.pi/agent/settings.json`:

```json
{
  "subagents": {
    "agentOverrides": {
      "design-spec-reviewer": {
        "model": "opencode-go/qwen3.6-plus",
        "thinking": "high"
      }
    }
  }
}
```

---

## ✅ Implementation Checklist

### Phase 1: Foundation (Week 1)

- [ ] Read SUBAGENT_SUMMARY.md
- [ ] Read SUBAGENT_EXPLORATION.md
- [ ] Read SUBAGENT_USAGE_EXAMPLES.md Examples 5-6
- [ ] Create design-spec-reviewer.md
- [ ] Create plan-reviewer.md
- [ ] Test dispatch with sample spec/plan
- [ ] Document in AGENTS.md

### Phase 2: Integration (Week 2)

- [ ] Integrate design-spec-reviewer into brainstorm workflow
- [ ] Integrate plan-reviewer into planning workflow
- [ ] Test async execution (reviews run parallel to work)
- [ ] Test review artifact creation
- [ ] Document output format

### Phase 3: Optimization (Week 3+)

- [ ] Test parallel reviews (multiple angles)
- [ ] Implement review synthesis
- [ ] Add conditional dispatch (route by artifact type)
- [ ] Measure model performance vs. cost
- [ ] Optimize thinking levels per review type

---

## 🔗 Related Documentation

### Pi Documentation

- Main: `~/.volta/tools/image/packages/@earendil-works/pi-coding-agent/README.md`
- Subagents skill: `~/.pi/agent/npm/node_modules/pi-subagents/skills/pi-subagents/SKILL.md`
- Subagent dispatch skill: `~/.pi/agent/skills/pi-subagent-dispatch/SKILL.md`

### Your Custom Agents

- `~/.pi/agent/agents/implementer.md`
- `~/.pi/agent/agents/spec-reviewer.md`
- `~/.pi/agent/agents/code-quality-reviewer.md`

### Superpowers Workflows

- Subagent-driven-development: `~/.pi/agent/skills/subagent-driven-development/SKILL.md`
- Requesting code review: `~/.pi/agent/skills/requesting-code-review/SKILL.md`

---

## 💡 Pro Tips

### Tip 1: Use Fresh Context for Reviews

```typescript
subagent({
  agent: "design-spec-reviewer",
  task: "...",
  context: "fresh", // Clean context, no history pollution
});
```

### Tip 2: Run Async to Keep Moving

```typescript
// Non-blocking - continue work while review runs
subagent({ agent: "...", async: true, output: "reviews/file.md" });
```

### Tip 3: Save Reviews to Artifacts

```typescript
// Persistent artifacts for synthesis
output: "reviews/spec-review.md";
output: "reviews/plan-review.md";
```

### Tip 4: Match Model to Task

```yaml
# High-thinking for design/planning
model: opencode-go/qwen3.6-plus
thinking: high

# Fast for quick checks
model: opencode-go/deepseek-v4-flash
thinking: low
```

### Tip 5: Document Your Methodology

Your agent prompts should reference your methodology:

```
Read the latest methodology from:
~/.pi/agent/skills/design-review/SKILL.md
```

---

## ❓ FAQ

**Q: Can I use builtin agents for artifact review?**
A: Yes! But custom agents are better because you can optimize model, thinking level, and tools per review type.

**Q: Do I have to create new agents?**
A: No, you can dispatch the builtin `reviewer` or `oracle` agents. But custom agents give you more control.

**Q: Can reviews ask clarifying questions?**
A: Yes, using the intercom bridge (requires `contact_supervisor` in agent and `intercom` in parent).

**Q: Where do review artifacts go?**
A: You specify the output path: `output: "reviews/spec-review.md"`. They're saved to disk.

**Q: Can I run reviews in parallel?**
A: Yes! Use `tasks: [...]` with `concurrency: N`.

**Q: Can I chain reviews?**
A: Yes! Use `chain: [...]` and pass output to next step via `{previous}`.

**Q: What models should I use?**
A: High-thinking (qwen3.6-plus) for design/planning, fast models for quick checks.

**Q: Can I override models per dispatch?**
A: Yes, in settings.json via `agentOverrides` or inline with `model: "..."` parameter.

---

## 📞 Support

If questions arise while exploring:

1. **Check SUBAGENT_EXPLORATION.md** for reference info
2. **Check SUBAGENT_USAGE_EXAMPLES.md** for code patterns
3. **Check EXTENSION_ROADMAP.md** for implementation strategy
4. **Check Pi docs:** `~/.pi/agent/npm/node_modules/pi-subagents/skills/pi-subagents/SKILL.md`
5. **Ask in Pi session:** `/subagents-doctor` for diagnostics

---

## 🎉 Summary

**You have:**

- ✅ 8 builtin agents ready to use
- ✅ Working pattern for custom agents (your 3 existing agents)
- ✅ Unified dispatch mechanism (`subagent()` tool)
- ✅ Async, parallel, and sequential execution
- ✅ Model and thinking level control

**You need to create:**

- ⬜ design-spec-reviewer.md (using existing pattern)
- ⬜ plan-reviewer.md (using existing pattern)

**That's it.** The infrastructure exists. The pattern is proven. You're ready to go.

---

**Start here:** Read SUBAGENT_SUMMARY.md (5 minutes), then create the two agents (10 minutes), then test (5 minutes).

**Total time to first working design-spec-reviewer dispatch: ~20 minutes.**
