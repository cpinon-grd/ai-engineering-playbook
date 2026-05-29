---
name: docker-production-guidelines
description: Use when writing, reviewing, refactoring, or debugging Dockerfiles, Docker Compose files, .dockerignore files, container build pipelines, production container images, pinned base images, multi-stage builds, non-root runtime users, healthchecks, volumes, networking, logging, and Docker security.
---

# Docker Production Guidelines

Apply these rules when generating or reviewing Docker configuration. Favor minimal, secure, reproducible production images with a small attack surface.

## Dockerfile

- Pin base image versions. Use tags like `node:20.11-alpine3.19`; never use `:latest` for production.
- Use multi-stage builds for compiled languages and build-time dependency separation.
- Preserve layer cache by copying package or lock files first, installing dependencies, then copying source.
- Combine related `RUN` commands with `&&` to reduce unnecessary layers.
- Create and switch to a non-root user before `CMD` or `ENTRYPOINT`.
- Add `HEALTHCHECK` for production services.
- Use `COPY --chown=appuser:appuser` when copying files that should be owned by the runtime user.

```dockerfile
FROM node:20.11-alpine3.19 AS runtime

WORKDIR /app
RUN addgroup -S appuser && adduser -S appuser -G appuser

COPY --chown=appuser:appuser package.json package-lock.json ./
RUN npm ci --omit=dev

COPY --chown=appuser:appuser . .

USER appuser
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

## Security

- Never run production containers as root.
- Do not put secrets in Dockerfiles, build arguments, image layers, or committed Compose files.
- Do not copy `.env` files into images.
- Scan images in CI with `docker scout`, Trivy, or the project's existing scanner.
- Keep the runtime image minimal and omit build tools unless they are required at runtime.

## `.dockerignore`

Always include a `.dockerignore` for build contexts. At minimum, exclude:

```gitignore
node_modules
.git
*.log
.env*
tests
test
__pycache__
.pytest_cache
```

Adjust entries to the project's language and build system while keeping secrets, VCS data, logs, and local dependencies out of the image context.

## Volumes

- Use named volumes for production persistence.
- Use bind mounts only for development workflows.
- Do not rely on bind mounts for production application code or persistent data.

## Networking

- Use custom bridge networks in Compose.
- Avoid host networking in production unless the deployment platform explicitly requires it.
- Reference Compose services by service name rather than hard-coded container IPs.

## Logging

- Log to `stdout` and `stderr`.
- Do not write application logs to files inside production containers unless a platform-specific sidecar or collector requires it.

## Forbidden in Production

- Do not use `:latest` tags.
- Do not use `ADD` when `COPY` is sufficient.
- Do not run as root.
- Do not pass secrets through build args or bake secrets into image layers.
- Do not copy `.env` files, VCS directories, local dependency folders, logs, or test artifacts into production images.
