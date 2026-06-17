# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built: clean, tested, maintainable, and aligned with project architecture.

**Only dispatch after workstream compliance review passes.**

```
Task tool (general-purpose):
  description: "Review code quality for Slice [SLICE_LETTER]"
  model: [MODEL — REQUIRED: choose per SKILL.md model-selection guidance]
  prompt: |
    You are reviewing one slice's implementation for code quality after workstream compliance has already passed.

    ## What Was Requested

    Read the slice brief: [BRIEF_FILE]

    Workstream-level constraints that bind this slice:
    [GLOBAL_CONSTRAINTS]

    ## What the Implementer Claims They Built

    Read the implementer's report: [REPORT_FILE]

    ## Diff Under Review

    **Base:** [BASE_SHA]
    **Head:** [HEAD_SHA]
    **Diff file:** [DIFF_FILE]

    Read the diff file once. Do not re-run git commands. Inspect code outside the diff only if you can
    name a specific risk that requires one focused check.

    ## Tests

    The implementer already ran the tests and reported results for this slice. Do not re-run the
    suite just to confirm their report. Run a focused test only when reading the code raises a
    specific doubt you can name.

    ## Your Job

    Check:
    - Does each file have one clear responsibility with a well-defined interface?
    - Are units decomposed so they can be understood and tested independently?
    - Is the implementation following the key files list and architecture invariants from the Workstream Document?
    - Did this implementation create new files that are already large, or significantly grow existing files?
      (Do not flag pre-existing file sizes — focus on what this slice's change contributed.)
    - Does the code follow the established patterns and conventions already present in the project?
    - Are tests meaningful, behavior-focused, and sufficiently targeted for the changed code?
    - Is error handling, maintainability, and readability good enough to keep this slice healthy long-term?

    ## Calibration

    Categorize issues by actual severity:
    - **Critical:** severe quality or correctness issue that makes the slice unsafe to accept
    - **Important:** maintainability, testing, or structural issue that should be fixed before accepting the slice
    - **Minor:** non-blocking cleanup or polish suggestion

    Every issue must include file:line references.

    ## Output Format

    ### Strengths
    - [Specific things done well]

    ### Issues

    #### Critical (Must Fix)
    #### Important (Should Fix)
    #### Minor (Nice to Have)

    For each issue: file:line, what's wrong, why it matters, and how to fix it if not obvious.

    ### Assessment

    **Task quality:** [Approved | Needs fixes]

    **Reasoning:** [1-2 sentence technical assessment]
```
