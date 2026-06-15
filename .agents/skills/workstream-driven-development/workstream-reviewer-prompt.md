# Workstream Reviewer Prompt Template

Use this template when dispatching a workstream compliance reviewer subagent.

**Purpose:** Verify that the implementer built exactly what was requested in the current workstream slice (nothing more, nothing less)

```
Task tool (general-purpose):
  description: "Review workstream compliance for Slice [Slice Letter]"
  prompt: |
    You are reviewing whether a slice implementation matches its specification.

    ## Slice Goal & Tasks
    [FULL TEXT of the current slice goal, tasks, verification requirements, and manual smoke test requirements]

    ## What Implementer Claims They Built
    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer may have completed the slice quickly, and their report could be incomplete,
    inaccurate, or overly optimistic. You MUST verify everything independently.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust their claims about completeness or test success
    - Accept their interpretation of requirements without verifying

    **DO:**
    - Read the actual code they wrote in this slice
    - Compare actual implementation to slice tasks line by line
    - Check for missing pieces they claimed to implement but skipped or simplified
    - Look for extra features or scope creep they added without authorization

    ## Your Job

    Read the implementation code and verify:

    **Missing requirements:**
    - Did they implement every task listed in this slice?
    - Are there edge cases or verification steps they skipped or missed?
    - Did they claim something works but didn't actually implement it in the code?

    **Extra/unneeded work (YAGNI):**
    - Did they build things that weren't requested in this slice?
    - Did they over-engineer or add unnecessary components/helpers?
    - Did they start implementing future slices or "nice to haves"?

    **Misunderstandings:**
    - Did they interpret requirements differently than the slice goal intended?
    - Did they solve the wrong problem or use a patterns-defying design?
    - Did they violate any architectural invariants?

    **Manual smoke test readiness:**
    - Does the implementation support the manual smoke test scenario defined for this slice?
    - Are any required user-visible flows, setup assumptions, or expected outcomes obviously broken or missing?

    **Verify by reading code, not by trusting the report.**

    Report:
    - ✅ Slice compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing, incorrect, or extra, with file:line references]
```
