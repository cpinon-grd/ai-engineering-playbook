---
name: python-development-guidelines
description: Use when writing, reviewing, refactoring, debugging, testing, documenting, or configuring Python code, especially projects that use pytest, Ruff, uv, virtual environments, environment-variable configuration, logging, CI, typed functions, and PEP 257 docstrings.
---

# Python Development Guidelines

Apply these rules when generating or reviewing Python code. Optimize for clear structure, maintainability, explicit typing, testability, and AI-assisted readability.

## Project Structure

- Separate source code, tests, docs, and configuration.
- Prefer `src/<package_name>/` for application code when creating a package layout.
- Place tests under `tests/`.
- Keep modules focused by concern, such as models, services, controllers, and utilities, when those boundaries fit the project.
- Add `__init__.py` when creating packages under `tests/` or `src/<package_name>/`.

## Configuration

- Use environment variables for runtime configuration and secrets.
- Keep configuration loading separate from business logic.
- Provide typed config objects when configuration is shared across modules.

## Typing and Docstrings

- Add type annotations to every Python function and method signature.
- Include explicit return types, including `None` where appropriate.
- Add descriptive docstrings to all Python functions and classes.
- Follow PEP 257 docstring conventions.
- Update existing docstrings when changing behavior.
- Keep existing comments unless they are made false by the current change; update stale comments instead of silently deleting them.

```python
def normalize_username(value: str) -> str:
    """Return a normalized username suitable for lookup."""
    return value.strip().lower()
```

For classes, type methods and class attributes where relevant.

```python
class UserService:
    """Coordinate user lookup and profile updates."""

    def __init__(self, repository: UserRepository) -> None:
        self.repository = repository
```

## Error Handling and Logging

- Use specific exception types.
- Capture useful context in logs without leaking secrets.
- Let unexpected exceptions propagate when the caller or framework should handle them.
- Avoid broad `except Exception` blocks unless they add context and re-raise or handle a known boundary.

## Testing

- Use `pytest` and pytest plugins only; do not use `unittest`.
- Put all tests under `tests/`.
- Add typing annotations to all test functions, fixtures, and helper functions.
- Add docstrings to test functions, fixtures, and test helper classes.
- Write tests before fixing bugs when practical.
- Cover relevant edge cases and error conditions.

```python
def test_normalize_username_strips_and_lowercases() -> None:
    """Normalize usernames by trimming whitespace and lowercasing."""
    assert normalize_username(" Alice ") == "alice"
```

When type-checking pytest fixtures, import these names inside a `TYPE_CHECKING` block only as needed.

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from _pytest.capture import CaptureFixture
    from _pytest.fixtures import FixtureRequest
    from _pytest.logging import LogCaptureFixture
    from _pytest.monkeypatch import MonkeyPatch
    from pytest_mock.plugin import MockerFixture
```

## Tooling

- Prefer `uv` for dependency and virtual environment management when the project does not already use another tool.
- Use Ruff for linting and formatting consistency when available.
- Match existing project tooling if it already has a clear convention.

## Documentation

- Use docstrings for public APIs, complex behavior, and non-obvious side effects.
- Keep README guidance accurate when creating or changing project-level setup, usage, or testing workflows.
- Keep examples minimal and runnable.

## CI/CD

- Use GitHub Actions or GitLab CI when adding CI to a new Python project.
- Run tests and Ruff checks in CI when those tools are part of the project.
- Keep CI jobs focused and reproducible.

## AI-Friendly Coding

- Prefer direct, readable code over clever shortcuts.
- Use small modules and explicit interfaces.
- Keep explanations and code snippets aligned with the project's existing structure and tooling.
- Preserve existing style unless the requested change requires introducing a new convention.
