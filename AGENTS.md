# Agent Working Guidelines

Use these rules when writing, reviewing, refactoring, or debugging code.

## Think Before Coding

Before changing code:

- State relevant assumptions.
- Do not silently choose between ambiguous interpretations.
- If the request is unclear and the ambiguity affects the implementation, ask.
- If a simpler approach exists, prefer it and explain the tradeoff.
- Do not start broad rewrites without a concrete reason.

## Simplicity First

Implement the minimum code that solves the task.

- Do not add features that were not requested.
- Do not introduce abstractions for one-off logic.
- Do not add configurability unless there is an explicit requirement.
- Do not add defensive handling for impossible or irrelevant scenarios.
- If the solution becomes significantly larger than necessary, simplify it.

## Surgical Changes

Touch only the files and lines needed for the task.

- Do not reformat unrelated code.
- Do not refactor adjacent code unless required.
- Match the existing project style.
- Remove only imports, variables, functions, or files made unused by your own change.
- If you notice unrelated dead code or design problems, mention them instead of changing them.

Every changed line should be traceable to the user request.

## Goal-Driven Execution

Convert tasks into verifiable outcomes.

Examples:

- Bug fix: reproduce the bug, add or identify a failing test, implement the fix, then verify.
- Validation: define invalid cases, test them, implement validation, then verify.
- Refactor: preserve behavior, verify before and after, and avoid unrelated changes.

For multi-step work:

1. State the goal.
2. List the minimal implementation steps.
3. Define how each step will be verified.
4. Run the relevant checks when possible.
5. Report what changed and what was verified.

## Verification

Prefer project-native checks:

- Unit tests.
- Type checks.
- Linters.
- Format checks.
- Build commands.
- Targeted reproduction commands.

If verification cannot be run, say exactly why and provide the command the user should run.
