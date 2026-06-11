# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable, conforming to architectural standards)

**Only dispatch after workstream compliance review passes.**

```
Task tool (general-purpose):
  Use template at requesting-code-review/code-reviewer.md

  DESCRIPTION: [slice summary, from implementer's report]
  PLAN_OR_REQUIREMENTS: Slice [Slice Letter] from Workstream Document YYYY-MM-DD-<topic>.md
  BASE_SHA: [commit before slice]
  HEAD_SHA: [current commit]
```

**In addition to standard code quality concerns, the reviewer should check:**
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the key files list and architecture invariants from the Workstream Document?
- Did this implementation create new files that are already large, or significantly grow existing files? (Do not flag pre-existing file sizes—focus on what this slice's change contributed.)
- Does the code follow the established patterns and conventions already present in the project (check existing files for idioms, logging style, connection patterns, etc.)?

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment
