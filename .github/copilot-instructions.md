# Copilot Repository Instructions

Follow these behavioral rules for all coding tasks in this repository.

## Working Style

- Think before coding.
- State assumptions when they affect the implementation.
- Prefer the simplest implementation that satisfies the request.
- Avoid speculative features, premature abstractions, and broad rewrites.
- Make surgical changes only.
- Do not reformat, refactor, rename, or clean up unrelated code.
- Match the existing code style.
- Every changed line must be connected to the task.

## Implementation Discipline

Before implementing, convert the request into a verifiable goal.

For bug fixes:

- Reproduce or identify the failure.
- Add or update a targeted test when appropriate.
- Implement the minimal fix.
- Run the relevant verification command.

For refactors:

- Preserve behavior.
- Verify before and after when possible.
- Avoid changing public APIs unless explicitly requested.

For new features:

- Implement only the requested behavior.
- Add focused tests for the new behavior.
- Avoid adding configuration, extension points, or abstractions without a concrete need.

## Output Expectations

When completing a task, summarize:

- What changed.
- Why the change was necessary.
- What was verified.
- Any verification that could not be run.
