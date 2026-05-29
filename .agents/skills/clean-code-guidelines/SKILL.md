---
name: clean-code-guidelines
description: Use when writing, reviewing, refactoring, or debugging code to improve readability, maintainability, naming, structure, testing, comments, constants, encapsulation, and overall code quality.
---

# Clean Code Guidelines

Apply these rules when writing or reviewing code to keep changes clear, maintainable, and human-readable.

## Constants Over Magic Numbers

- Replace hard-coded values with named constants.
- Use descriptive constant names that explain each value's purpose.
- Keep constants near the top of the file or in a dedicated constants file when the project already uses one.

## Meaningful Names

- Choose variable, function, class, and file names that reveal purpose.
- Prefer names that explain why something exists and how it is used.
- Avoid abbreviations unless they are universally understood in the codebase or domain.

## Smart Comments

- Do not comment on what the code does when the code can be made self-documenting.
- Use comments to explain why something is done a certain way.
- Document APIs, complex algorithms, and non-obvious side effects.

## Single Responsibility

- Make each function do one thing.
- Keep functions small and focused.
- Split a function if it needs a comment just to explain what it does.

## DRY

- Extract repeated code into reusable functions when the repetition is real and stable.
- Share common logic through appropriate abstractions.
- Maintain single sources of truth for rules, constants, and transformations.

## Clean Structure

- Keep related code together.
- Organize code in a logical hierarchy.
- Follow existing file and folder naming conventions.

## Encapsulation

- Hide implementation details behind clear interfaces.
- Expose the smallest useful public surface.
- Move nested conditionals into well-named functions when that makes intent clearer.

## Code Quality Maintenance

- Refactor when it directly supports the requested change or prevents immediate confusion.
- Fix technical debt early when it is in scope.
- Leave touched code cleaner than you found it without expanding the change unnecessarily.

## Testing

- Write or update tests before fixing bugs when practical.
- Keep tests readable and maintainable.
- Cover edge cases and error conditions relevant to the change.

## Version Control

- Write clear commit messages when commits are requested.
- Make small, focused commits.
- Use meaningful branch names when creating branches.
