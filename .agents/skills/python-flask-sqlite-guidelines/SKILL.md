---
name: python-flask-sqlite-guidelines
description: Use when writing, reviewing, refactoring, debugging, testing, documenting, or configuring Python applications that use Flask, SQLite, Flask-SQLAlchemy, SQLAlchemy, Alembic, Jinja2 templates, static assets, authentication, REST APIs, pytest, Black, isort, or Flask deployment patterns.
---

# Python Flask SQLite Guidelines

Apply these rules when generating or reviewing modern Python applications built with Flask and SQLite. Favor clear structure, predictable database access, secure defaults, and maintainable tests.

## Project Structure

- Use a `src/your_package_name/` layout for package code.
- Place tests in `tests/` parallel to `src/`.
- Keep configuration in `config/` or load it from environment variables.
- Store dependencies in `requirements.txt` or `pyproject.toml`, matching the existing project.
- Place static files in `static/`.
- Place Jinja2 templates in `templates/`.

## Code Style

- Follow Black formatting with an 88-character line length.
- Use isort for import ordering when the project uses it.
- Follow PEP 8 naming conventions:
  - Use `snake_case` for functions and variables.
  - Use `PascalCase` for classes.
  - Use `UPPER_CASE` for constants.
- Prefer absolute imports over relative imports.

## Type Hints

- Add type hints for all function parameters and return values.
- Import typing helpers from `typing`.
- Use `Optional[Type]` instead of `Type | None` when following this guideline set.
- Use `TypeVar` for generic types.
- Define shared custom types in `types.py` when the project needs them.
- Use `Protocol` for duck typing and interface-style dependencies.

## Flask Structure

- Use the Flask application factory pattern.
- Organize routes with Blueprints.
- Use Flask-SQLAlchemy or SQLAlchemy for database access.
- Implement error handlers for application and HTTP errors.
- Use Flask-Login for session-based authentication when authentication is needed.
- Keep views, services, models, and forms or schemas separated by responsibility.

```python
from flask import Flask


def create_app() -> Flask:
    """Create and configure the Flask application."""
    app = Flask(__name__)
    app.config.from_prefixed_env()

    from .routes import main_bp

    app.register_blueprint(main_bp)
    return app
```

## Database

- Use SQLAlchemy ORM for database models.
- Use Alembic for database migrations when schema changes need to be tracked.
- Define models in separate modules.
- Model relationships explicitly.
- Add indexes for frequent lookup, join, and filtering paths.
- Use proper connection handling and pooling settings for the deployment target.

## Authentication

- Use Flask-Login for session management.
- Use Flask-OAuth or the project's selected OAuth library for Google OAuth.
- Hash passwords with bcrypt when password authentication exists.
- Configure secure sessions.
- Implement CSRF protection for forms and browser-facing state-changing requests.
- Use role-based access control where authorization differs by user role.

## API Design

- Use Flask-RESTful for REST APIs when the project uses that extension.
- Validate requests before business logic runs.
- Return appropriate HTTP status codes.
- Use consistent response formats.
- Handle errors consistently.
- Implement rate limiting for public or abuse-prone endpoints.

## Testing

- Use pytest for tests.
- Place tests under `tests/`.
- Write route tests for Flask endpoints.
- Use pytest-cov when tracking coverage.
- Use fixtures for app, client, database, and authenticated-user setup.
- Use pytest-mock for mocking.
- Test error paths and edge cases.

## Security

- Use HTTPS in production.
- Configure CORS intentionally and narrowly.
- Sanitize and validate all user input.
- Use secure session configuration.
- Log security-relevant events without exposing secrets.
- Follow OWASP guidance for authentication, authorization, input validation, and session handling.

## Performance

- Use Flask-Caching when caching is appropriate.
- Optimize database queries and avoid unnecessary N+1 query patterns.
- Configure connection pooling for the deployment context.
- Implement pagination for large result sets.
- Use background tasks for heavy or slow operations.
- Monitor application performance in production.

## Error Handling

- Create custom exception classes for domain errors.
- Use targeted `try`/`except` blocks.
- Log useful context at error boundaries.
- Return consistent error responses.
- Handle expected edge cases explicitly.
- Keep user-facing error messages clear without leaking internals.

## Documentation

- Use Google-style docstrings.
- Document public APIs and non-obvious behavior.
- Keep `README.md` updated when setup, usage, or testing flows change.
- Use inline comments for why, not what.
- Generate API documentation when the project already has an API docs workflow.
- Document environment setup and required variables.

## Development Workflow

- Use virtual environments.
- Use pre-commit hooks when available.
- Follow the project's Git workflow.
- Use semantic versioning when the project publishes versions.
- Add CI/CD checks for tests, formatting, linting, and security where appropriate.
- Implement structured and useful application logging.

## Dependencies

- Pin dependency versions for reproducible deployments.
- Use `requirements.txt` for production dependencies when that is the project's convention.
- Separate development dependencies.
- Use compatible package versions.
- Update dependencies regularly.
- Check dependencies for known security vulnerabilities.
