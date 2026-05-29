---
name: devsecops-appsec-guidelines
description: Use when writing, reviewing, refactoring, debugging, testing, documenting, or configuring security-sensitive code, including secure coding, secret handling, dependency hygiene, authentication, authorization, RBAC, input validation, database access, logging, SAST, SCA, secret scanning, IaC scanning, DAST, Policy as Code, monitoring, compliance, and audit documentation across Python, JavaScript, TypeScript, Go, Java, Ruby, PHP, C#, and shell.
---

# DevSecOps AppSec Guidelines

Apply these rules when generating or reviewing application code, scripts, CI/CD configuration, infrastructure code, or documentation where security matters. Favor least privilege, secure defaults, auditable decisions, and defense in depth.

## General Security Principles

- Never hardcode secrets, credentials, tokens, private keys, or API keys.
- Use environment variables, managed secret stores, or secure vaults for sensitive data.
- Do not include `.env`, secret config files, private keys, or unknown tokens in source control.
- Never log sensitive data, secrets, passwords, API keys, session IDs, or access tokens.
- Validate and sanitize all user input at trust boundaries.
- Escape output in the correct context, such as HTML, JavaScript, SQL, shell, or logs.
- Avoid unsafe dynamic execution functions such as `eval`, `exec`, shell interpolation, deserialization of untrusted data, or similar constructs.

## Database Security

- Use parameterized queries, prepared statements, or an ORM for all database access.
- Do not build SQL with string concatenation or interpolation of untrusted values.
- Ensure database users and service accounts have only the privileges required.
- Review and update database access policies when data access changes.
- Treat migration scripts and seed data as security-sensitive artifacts.

## Dependency Management

- Use packages only from verified and trusted sources.
- Do not add new dependencies without explicit approval and security review.
- Prefer maintained packages with active security posture and clear licenses.
- Keep dependencies updated through the project's normal process.
- Scan dependencies for known vulnerabilities with Software Composition Analysis (SCA).
- Pin or lock dependency versions when reproducibility is required.

## Authentication and Authorization

- Use secure, established authentication frameworks.
- Do not implement custom authentication unless explicitly required and reviewed.
- Store passwords using strong, salted password hashing such as Argon2 or bcrypt.
- Enforce Role-Based Access Control (RBAC) for sensitive operations.
- Apply least privilege to APIs, background jobs, admin actions, and UI actions.
- Verify authorization server-side; do not rely on hidden UI controls as enforcement.

## Secure SDLC Practices

- Integrate Static Application Security Testing (SAST) into CI.
- Integrate Software Composition Analysis (SCA) into CI.
- Scan all code for secrets before merge.
- Scan infrastructure code with IaC security tooling.
- Integrate Dynamic Application Security Testing (DAST) in CD for deployed applications where practical.
- Use Policy as Code (PaC) for automated and version-controlled security policies.
- Keep security checks actionable and documented so failures can be fixed, not ignored.

## Monitoring and Feedback

- Enable continuous vulnerability monitoring and alerting.
- Use Runtime Application Self-Protection (RASP) and Web Application Firewall (WAF) where appropriate for the deployment context.
- Encourage regular vulnerability assessments and penetration testing.
- Feed recurring vulnerability patterns back into coding rules, prompts, tests, and review checklists.
- Log security-relevant events with context while excluding secrets and sensitive payloads.

## Compliance and Documentation

- Align controls and decisions with relevant standards such as OWASP Top 10, NIST, and ISO 27001.
- Document security controls, threat assumptions, and accepted risks.
- Record security decisions in architecture docs, pull requests, or audit artifacts when they affect compliance.
- Keep compliance documentation close to the code or system it describes when possible.

## Review Checklist

Before finishing security-sensitive work, check:

- No secrets are hardcoded, logged, committed, or passed through unsafe build paths.
- Inputs are validated and outputs are escaped in the correct context.
- Database access uses parameterized queries or ORM protections.
- Authentication and authorization use established frameworks and server-side enforcement.
- New dependencies are justified, reviewed, and scanned.
- CI/CD includes relevant SAST, SCA, secret scanning, IaC scanning, DAST, or PaC checks.
- Security controls and non-obvious decisions are documented for auditability.
