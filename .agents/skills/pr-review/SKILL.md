---
name: pr-review
description: Review pull requests and code changes with focused angles for security, performance, tests, and architecture. Use when the user asks to review a PR, diff, branch, pull request, or set of changes.
---

# PR Review Skill

Use this skill when the user asks to review a pull request, diff, branch, set of changes, or "this PR".

## Review angle selection

Choose the review angle based on the user's wording:

- If the user mentions security, auth, authorization, injection, secrets, privacy, data exposure, XSS, crypto, or hardening, use **SECURITY**.
- If the user mentions performance, latency, memory, scaling, throughput, N+1, allocations, async, caching, or complexity, use **PERFORMANCE**.
- If the user mentions tests, coverage, test quality, edge cases, assertions, mocks, flakes, or CI confidence, use **TESTS**.
- If the user mentions architecture, design, coupling, boundaries, abstractions, scalability, reversibility, or naming, use **ARCHITECTURE**.
- If no angle is specified, default to **SECURITY**.

## Required review discipline

For every review:

- Cite the file path and line number for each finding.
- Rank findings by severity:
  - `blocker`: must be fixed before merge.
  - `important`: should be fixed before merge unless there is an explicit risk acceptance.
  - `nit`: low-risk cleanup or clarity improvement.
- Be specific and evidence-based.
- Do not report vague concerns.
- If the diff does not provide enough context, say exactly what context is missing and ask for the surrounding file.
- Prefer defects over nice-to-haves.
- Do not invent line numbers.
- Do not assume behavior that is not visible in the diff or surrounding context.
- Group findings by severity, then by file path.
- Include a concise remediation suggestion for each finding.
- If there are no findings, say that no concrete findings were identified for the selected angle.

## Output format

Use this structure:

```text
Review angle: <SECURITY | PERFORMANCE | TESTS | ARCHITECTURE>

Findings

[blocker] path/to/file.ext:123 — Short finding title
Evidence: Explain the exact issue and why this line is problematic.
Impact: Explain the concrete risk.
Fix: Suggest a specific fix.

[important] path/to/file.ext:456 — Short finding title
Evidence: ...
Impact: ...
Fix: ...

[nit] path/to/file.ext:789 — Short finding title
Evidence: ...
Impact: ...
Fix: ...

Context needed
- path/to/file.ext: Need surrounding function/class/module because ...

Verdict: Safe to merge | needs changes | reject
````

For the architecture angle, use this final verdict line instead:

```text
Verdict: Architecturally sound | needs trim | re-think before merging
```

## Angle 1: SECURITY

Review the PR for security defects. Focus in this priority order:

1. Auth/authz:

   * New endpoints or branches missing authentication checks.
   * Missing authorization checks.
   * Role or tenant assumptions.
   * IDOR risks.
2. Input validation:

   * Untrusted input flowing into queries, shell commands, file paths, deserialization, template rendering, or eval-like behavior.
3. Injection:

   * SQL injection.
   * NoSQL injection.
   * Command injection.
   * Prompt injection.
   * Template injection.
4. Secrets:

   * Hardcoded keys, tokens, credentials, or certificates.
   * Secrets in logs.
   * Secrets included in client-bundled code.
   * Committed `.env` or local secret files.
5. Output encoding:

   * XSS through unescaped templates.
   * HTML rendered from user-controlled content.
   * JSONP-style leaks.
6. Crypto/randomness:

   * `Math.random` or weak randomness for security tokens.
   * MD5/SHA1 for security-sensitive purposes.
   * Missing IVs/nonces.
   * Custom cryptography.
7. Data exposure:

   * PII in logs.
   * Overshared API responses.
   * Missing redaction.
   * Debug data returned to clients.

Skip nice-to-haves. Report only concrete defects.

End with:

```text
Verdict: Safe to merge | needs changes | reject
```

## Angle 2: PERFORMANCE

Review the PR for performance regressions. Focus on:

1. N+1 patterns:

   * Loops doing database, filesystem, or network calls per item without batching.
2. Hot-path allocations:

   * New objects, arrays, maps, regexes, or closures inside hot loops.
   * Regexes recompiled per call.
3. Unbounded work:

   * Missing pagination.
   * Unconstrained result sets.
   * Recursion without depth limits.
   * Unbounded queues or buffers.
4. Bad async:

   * Sequential awaits where parallel execution is safe and intended.
   * `Promise.all` without concurrency limits when the input can be large.
   * Missing cancellation or timeout handling for potentially slow calls.
5. Cache misuse:

   * Cache keys that omit required variables.
   * Absent, excessive, or pathological TTLs.
   * Shared cache state that can leak across users or tenants.
6. Algorithmic complexity:

   * Hidden O(n²) or worse behavior.
   * Sorting inside loops.
   * Nested `.map`, `.filter`, `.some`, `.find`, or `.reduce` over large collections.

For each finding:

* Quote the specific file and line.
* Name the bad pattern.
* State the complexity or scaling concern when applicable.
* Suggest a concrete fix.

End with:

```text
Verdict: Safe to merge | needs changes | reject
```

## Angle 3: TESTS

Review the test coverage and test quality for the PR. Focus on:

1. Tests for new code paths:

   * Every new branch should have at least one meaningful test.
   * New error paths should be tested.
2. Edge cases:

   * Empty input.
   * Null or undefined values.
   * Boundary values.
   * Dependency failures.
   * Invalid user input.
3. Assertion strength:

   * Assertions that would pass with the wrong value.
   * Snapshot-only tests.
   * Tests that only check the happy path.
   * Tests with no meaningful failure signal.
4. Mocking discipline:

   * Mocks that do not fail when the real interface changes.
   * Over-mocking that removes the behavior under test.
   * Mocks that encode implementation details instead of observable behavior.
5. Determinism:

   * Date/time not stubbed.
   * Randomness not controlled.
   * Network/filesystem dependencies not isolated.
   * Order-dependent tests.
6. Test names:

   * Names that do not describe behavior.
   * Names that describe implementation rather than expected outcome.

A test existing is not enough. Read the assertions and determine whether the test would catch a regression.

End with:

```text
Verdict: Safe to merge | needs changes | reject
```

## Angle 4: ARCHITECTURE

Review the shape of the change rather than only line-level defects. Focus on:

1. Boundary drift:

   * UI reaching into persistence or infrastructure.
   * Domain types importing transport, framework, database, or API-specific types.
   * Business logic moving into controllers, views, scripts, or jobs.
2. Premature abstraction:

   * Interfaces, factories, base classes, config layers, or plugin systems with only one implementation.
   * Abstractions that hide simple behavior without reducing coupling.
3. Coupling:

   * Shared utilities importing feature modules.
   * Feature modules depending on each other in both directions.
   * Shared mutable state.
   * Global registries or hidden side effects.
4. Scalability:

   * What breaks first if this code path grows 10x.
   * Whether the design allows batching, async execution, pagination, partitioning, or isolation.
5. Reversibility:

   * Whether this is a one-way door.
   * Whether rollback would require data migration, API contract changes, or broad call-site edits.
6. Naming:

   * Names based on implementation instead of role.
   * Versioned implementation names such as `UserManagerImplV2`.
   * Names that obscure ownership or boundary.

For each finding:

* Cite file and line where the architectural issue is introduced.
* Explain the design pressure or boundary violation.
* Suggest the smaller, cleaner shape of the change.

End with:

```text
Verdict: Architecturally sound | needs trim | re-think before merging
```

## Context guidance

Full file context is preferred.

If the user only provides a diff and the issue may depend on code outside the diff:

* Do not guess.
* State that the diff is insufficient.
* Ask for the surrounding function, class, or full file.
* Mention the exact file and nearby lines needed.

If the repository contains generated files, vendored code, lockfiles, snapshots, or build artifacts, ignore them unless the PR specifically changes behavior through those files.

````

---

## 2. Copilot Code Review Instructions

Ruta:

```text
.github/instructions/pr-review.instructions.md
````

Contenido:

````markdown
---
applyTo: "**/*"
---

# PR Review Instructions

When reviewing a pull request, diff, branch, or set of changes, use a focused review angle.

Select the review angle from the user's wording:

- Use **SECURITY** for auth, authorization, injection, secrets, privacy, XSS, crypto, hardening, or data exposure.
- Use **PERFORMANCE** for latency, memory, scaling, throughput, N+1, allocation, async, caching, or complexity concerns.
- Use **TESTS** for coverage, edge cases, assertions, mocks, flakes, deterministic testing, or CI confidence.
- Use **ARCHITECTURE** for design, coupling, boundaries, abstractions, scalability, reversibility, or naming.
- If unspecified, default to **SECURITY**.

## Output rules

- Cite file path and line number for every finding.
- Rank each finding as `blocker`, `important`, or `nit`.
- Be specific and evidence-based.
- Do not report vague concerns.
- Do not invent line numbers.
- Do not assume behavior that is not visible in the diff or available context.
- If the diff is insufficient, say what surrounding file or function is needed.
- Prefer concrete defects over nice-to-haves.
- Include a specific fix suggestion for each finding.

Use this format:

```text
Review angle: <SECURITY | PERFORMANCE | TESTS | ARCHITECTURE>

Findings

[blocker] path/to/file.ext:123 — Short finding title
Evidence: ...
Impact: ...
Fix: ...

[important] path/to/file.ext:456 — Short finding title
Evidence: ...
Impact: ...
Fix: ...

[nit] path/to/file.ext:789 — Short finding title
Evidence: ...
Impact: ...
Fix: ...

Context needed
- path/to/file.ext: Need surrounding function/class/module because ...

Verdict: Safe to merge | needs changes | reject
````

For architecture reviews, use this final verdict instead:

```text
Verdict: Architecturally sound | needs trim | re-think before merging
```

## SECURITY angle

Focus on concrete security defects in this order:

1. Auth/authz:

   * Missing authentication.
   * Missing authorization.
   * Role or tenant assumptions.
   * IDOR.
2. Input validation:

   * Untrusted input flowing into queries, shell commands, file paths, deserialization, templates, or eval-like behavior.
3. Injection:

   * SQL, NoSQL, command, prompt, or template injection.
4. Secrets:

   * Hardcoded credentials.
   * Secrets in logs.
   * Secrets in client-bundled code.
   * Committed `.env` files.
5. Output encoding:

   * XSS through unescaped templates or rendered user content.
6. Crypto/randomness:

   * Weak randomness for security tokens.
   * MD5/SHA1 for security-sensitive flows.
   * Missing IVs/nonces.
   * Custom cryptography.
7. Data exposure:

   * PII in logs.
   * Overshared API responses.
   * Missing redaction.

Skip nice-to-haves.

## PERFORMANCE angle

Focus on regressions caused by:

1. N+1 database, filesystem, or network calls.
2. Hot-path object, array, map, regex, or closure allocations.
3. Missing pagination or unbounded result sets.
4. Recursion without depth limits.
5. Sequential awaits where parallelism is safe.
6. `Promise.all` without concurrency limits over unbounded input.
7. Cache keys that omit required variables.
8. Missing or pathological cache TTLs.
9. Hidden O(n²) behavior.
10. Sorts inside loops.

For each finding, name the bad pattern and suggest the fix.

## TESTS angle

Focus on whether the tests would catch regressions:

1. New branches and code paths should have meaningful tests.
2. Edge cases should cover empty input, null/undefined, boundaries, and dependency failures.
3. Assertions should verify behavior, not just execution.
4. Snapshot-only tests are weak unless paired with behavioral assertions.
5. Mocks should track real interfaces and avoid over-mocking.
6. Date, time, randomness, network, and filesystem dependencies should be controlled.
7. Test names should describe behavior.

A test existing is not enough; inspect the assertions.

## ARCHITECTURE angle

Focus on the shape of the change:

1. Boundary drift:

   * UI reaching into persistence.
   * Domain code importing transport, framework, database, or API-specific types.
   * Business logic moving into controllers, views, scripts, or jobs.
2. Premature abstraction:

   * Interfaces, factories, base classes, or config layers with one implementation.
3. Coupling:

   * Shared utilities importing feature modules.
   * Feature modules depending on each other cyclically.
   * Shared mutable state or hidden side effects.
4. Scalability:

   * What breaks first if this path grows 10x.
5. Reversibility:

   * Whether rollback would require migrations, API contract changes, or broad edits.
6. Naming:

   * Names based on implementation instead of role.
   * Versioned implementation names.

For each finding, cite the line where the architectural issue is introduced and suggest the smaller, cleaner design.