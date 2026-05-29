---
name: python-project-guidelines
description: Use when writing, reviewing, refactoring, debugging, testing, documenting, or configuring Python projects that need clear project structure, modular design, environment-variable configuration, robust logging, pytest tests, docstrings, README updates, Rye dependency management, Ruff style checks, CI/CD, type hints, and AI-friendly code.
---

# Python Project Guidelines

Apply these rules when generating or reviewing Python project code and configuration. Optimize for clear structure, maintainability, debuggability, and AI-assisted development.

## Project Structure

- Separate source code, tests, docs, and configuration.
- Prefer a clear package layout such as `src/<package_name>/` when creating a package.
- Keep tests under `tests/`.
- Keep docs under `docs/` when project documentation grows beyond the README.
- Keep configuration in dedicated config modules, config files, or environment variables according to existing project conventions.

## Modular Design

- Split responsibilities into distinct modules when the project needs them:
  - `models` for data models and domain objects.
  - `services` for business logic and integrations.
  - `controllers`, `routes`, or handlers for request or command boundaries.
  - `utils` for small reusable helpers that do not belong to a richer domain module.
- Avoid unnecessary abstractions for simple scripts or small features.
- Keep public interfaces explicit and easy to test.

## Configuration

- Use environment variables for runtime configuration, secrets, and deployment-specific values.
- Keep configuration loading separate from business logic.
- Validate required settings early with clear error messages.
- Avoid hard-coded secrets, paths, hosts, and credentials.

## Error Handling and Logging

- Use specific exception types when possible.
- Capture useful context in logs, such as identifiers, operation names, and external system names.
- Do not log secrets or sensitive values.
- Add context at application boundaries and re-raise when the caller should decide how to handle the failure.
- Use structured or consistently formatted logs when the project already has a logging pattern.

## Testing

- Use `pytest` for tests.
- Prefer focused unit tests for pure logic and targeted integration tests for boundaries.
- Add regression tests before fixing bugs when practical.
- Keep fixtures small and descriptive.
- Test edge cases, invalid inputs, and error paths relevant to the change.

## Documentation

- Add docstrings for public functions, classes, and modules whose purpose is not obvious.
- Keep README instructions accurate when setup, usage, configuration, or test commands change.
- Document assumptions, environment variables, and operational requirements.
- Use comments for complex logic and non-obvious decisions, not for restating simple code.

## Dependency Management

- Use Rye for dependency management and virtual environments when the project does not already use another tool.
- Keep dependency declarations in `pyproject.toml` when using Rye.
- Separate runtime and development dependencies.
- Prefer pinned or locked dependencies for reproducible environments.
- Follow existing project dependency tooling if it is already established.

## Code Style

- Use Ruff for linting and formatting consistency when available.
- Follow existing Ruff configuration instead of introducing unrelated style churn.
- Use descriptive variable and function names.
- Add type hints for new or modified public functions and complex internal functions.
- Keep functions small enough to read and test comfortably.

## CI/CD

- Use GitHub Actions or GitLab CI when adding CI/CD to a Python project.
- Run tests and Ruff checks in CI when those tools are part of the project.
- Keep CI jobs reproducible and scoped to meaningful checks.
- Avoid CI changes unrelated to the requested task.

## AI-Friendly Coding

- Prefer direct, explicit code over clever shortcuts.
- Use names that reveal intent and data shape.
- Include rich error context for debugging.
- Keep examples and snippets aligned with the project's actual layout and tooling.
- Preserve existing conventions unless the requested change requires a new pattern.
