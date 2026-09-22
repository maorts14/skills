---
name: docker-dev-workspace
description: Create or improve a Docker Compose local development workspace with reproducible services and reliable automatic reload across Windows, macOS, and Linux. Use when a project needs a containerized local setup, hot reload, or cross-platform file watching.
---

# Docker Dev Workspace

Build a local development environment that makes the project's normal start path one reproducible Docker Compose command, while preserving fast feedback after source changes.

## Start with the project, not a template

Inspect the repository's runtime, package scripts, existing Docker files, environment examples, exposed services, and developer documentation before proposing changes. Reuse existing conventions and make the smallest coherent change. Do not overwrite an existing Compose, Dockerfile, lockfile, database volume, or developer environment without explicit authorization.

Containerize only what the application actually needs: for example an API, web client, worker, database, cache, or message broker. Keep service-to-service traffic on the Compose network and expose only ports developers genuinely need. Use named volumes for stateful local data and bind mounts for editable source code. Add health checks and `depends_on` conditions only when a service truly cannot start successfully before its dependency is ready.

Keep development configuration separate from production configuration when development needs bind mounts, debuggers, seeds, reload tools, or published internal ports. Production must not inherit these conveniences by accident.

## Automatic reload

Use the stack's native development server or watcher where available (for example Vite HMR or a Node watcher). Mount the relevant source paths, restrict watch scopes to source and configuration files, and avoid watching generated output, dependencies, caches, secrets, or the entire repository without need.

File content synchronization and filesystem change notifications are separate concerns. On Linux, regular event-based watching normally works with bind mounts. With Docker Desktop on Windows, a repository stored on the Windows filesystem can expose the changed files to a Linux container without reliably delivering Linux `inotify` events; macOS setups can also vary by Docker Desktop sharing mode. Do not claim polling is mandatory everywhere.

When event-based reload is unreliable, use the watcher's polling option and explain its CPU/latency trade-off. Prefer an opt-in environment variable or a clearly documented Compose override when the team has mixed operating systems; use polling as the default only if reliable cross-platform behavior is more valuable than its small continuous cost. If practical, test a source change in the running container before choosing the default.

For Nodemon, `--legacy-watch` (or `-L`) enables polling. For Chokidar-based tools, use their documented polling setting rather than inventing a busy loop. Note the recommended Windows alternative: keep the repository in the WSL/Linux filesystem when using Docker Desktop, then verify normal file-event watching before removing polling.

## Developer experience and verification

Provide a concise start command, local URLs/ports, stop command, and any first-run requirements. Do not put real secrets in images or Compose files; use ignored local environment files plus committed examples.

Before finishing, verify proportionally to the project:

1. Build and start the development stack.
2. Confirm health checks and service status.
3. Confirm the application reaches its expected local endpoint.
4. Edit a watched source file or otherwise demonstrate that the intended reload path works.
5. State any unverified platform-specific assumption rather than presenting it as established fact.
