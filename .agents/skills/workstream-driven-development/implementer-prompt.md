# Implementer Subagent Prompt Template (Workstream Slice)

Use this template when dispatching an implementer subagent to execute a specific workstream slice.

```
Task tool (general-purpose):
  description: "Implement Slice: [Slice Letter] - [Slice Title]"
  prompt: |
    You are implementing Slice [Slice Letter]: [Slice Title]

    ## Workstream Objective
    [Paste the overall workstream objective from the Workstream Doc metadata]

    ## Slice Goal
    [Paste the current slice goal here]

    ## Tasks
    [FULL TEXT of the tasks for this slice - paste it here, don't make the subagent read files]
    [Tasks use checkbox format (- [ ]). Mark each checkbox done (- [x]) as you complete it.]

    ## Watch Out
    [Paste the watch-outs for this slice here]

    ## Verification
    [Paste the verification steps and tests to run for this slice here]

    ## Manual Smoke Test
    [Paste the manual smoke test steps, setup, and expected outcomes for this slice here]

    ## Prior Slice Carry-forward (if any)
    [Paste the carry-forward information from the previous slice, if any]

    ## Before You Begin

    If you have questions about:
    - The requirements or expected behavior of this slice
    - The design decisions or implementation strategy
    - Unclear details or missing configuration/schema definitions

    **Ask them now.** Raise any questions or concerns before starting work.

    ## Your Job

    Once you are clear on requirements:
    1. Implement exactly what the slice tasks specify (do not build extra features)
    2. Write focused tests (follow TDD if appropriate or if requested)
    3. Run the specific verification commands for this slice
    4. Commit your work upon success
    5. Conduct a self-review (see below)
    6. Report back with status and details

    The controller will pause for a user manual smoke test only after your slice passes both review stages. Make sure your implementation supports the documented manual smoke test cleanly.

    Work from: [directory]

    **While you work:** If you encounter unexpected behavior, compilation errors, or roadblocks, **ask questions** or report immediately. Do not guess or make blind assumptions.

    ## Code Organization

    You reason best about code you can hold in context at once, and your edits are more
    reliable when files are focused. Keep this in mind:
    - Follow the key files and directory structure defined in the workstream
    - Each file/module should have one clear responsibility with a well-defined interface
    - In existing codebases, follow established patterns. Improve code you are touching
      the way a good developer would, but do not restructure things outside your slice.

    ## When You're in Over Your Head

    It is always OK to stop and say "this is too hard for me" or "I need more context." Bad work is worse than no work. You will not be penalized for escalating.

    **STOP and escalate when:**
    - You encounter architectural conflicts that affect other slices or packages
    - You need to understand code beyond what was provided and cannot find clarity
    - You feel uncertain about whether your implementation meets the core objective
    - You are stuck in a loop of failing tests or compilation errors

    **How to escalate:** Report back with status BLOCKED or NEEDS_CONTEXT. Describe specifically what you are stuck on, what you have tried, and what kind of help you need.

    ## Before Reporting Back: Self-Review

    Review your work with fresh eyes before reporting back. Ask yourself:

    **Completeness:**
    - Did I fully implement all tasks specified in this slice?
    - Did I miss any edge cases, errors, or validation rules?

    **Quality:**
    - Is this my best work?
    - Are variable and function names clear, descriptive, and accurate?
    - Is there any dead code, commented-out logic, or console.logs?

    **Discipline:**
    - Did I avoid overbuilding? Did I stick strictly to this slice's scope (YAGNI)?
    - Did I follow the established patterns in the codebase?

    **Testing & Verification:**
    - Do the tests actually verify behavior (not just mock everything out)?
    - Did I run the specified verification commands and verify they pass clean?

    If you find issues during self-review, fix and commit them now before reporting.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented
    - What you tested and the exact test results
    - Files changed and the commit SHA
    - Self-review findings (if any)
    - Any lingering concerns or carry-forward notes for the next slice
```
