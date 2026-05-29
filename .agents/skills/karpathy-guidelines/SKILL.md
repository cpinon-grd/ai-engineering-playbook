---
name: karpathy-guidelines
description: Use when writing, reviewing, refactoring, or debugging code to avoid overengineering, make surgical changes, surface assumptions, and define verifiable success criteria.
---

# Karpathy Guidelines

Use this skill to improve coding-agent behavior.

## Think Before Coding

Before changing code:

- State assumptions that affect the implementation.
- Do not silently choose between ambiguous interpretations.
- Ask when ambiguity blocks a correct implementation.
- Prefer simpler approaches when they satisfy the requirement.

## Simplicity First

Write the minimum code needed.

Avoid:

- Unrequested features.
- One-use abstractions.
- Premature configurability.
- Large rewrites.
- Defensive code for irrelevant cases.

## Surgical Changes

Touch only what the task requires.

- Do not improve unrelated code.
- Do not reformat unrelated files.
- Do not refactor adjacent logic unless required.
- Match existing style.
- Remove only unused code created by your own change.

Every changed line should map directly to the task.

## Goal-Driven Execution

Turn the request into verifiable success criteria.

Examples:

- Bug fix: reproduce, test, fix, verify.
- Validation: define invalid cases, test, implement, verify.
- Refactor: preserve behavior, verify before and after.

For multi-step work:

1. Define the goal.
2. List minimal steps.
3. Define the check for each step.
4. Implement.
5. Run verification.
6. Report results.

## Reporting

At the end, report:

- Changed files.
- Behavioral impact.
- Verification performed.
- Verification not performed and why.
