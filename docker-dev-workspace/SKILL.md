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

## Debugging a process in a container

Add debugger support when the project's runtime supports it and it will improve the local workflow. Treat it as a development-only feature: enable the runtime's inspector or debug server on a container interface, publish its port only in the development Compose configuration, and never expose it from production.

Before creating an editor-specific configuration, inspect the repository for existing editor settings and developer documentation. If they identify the supported IDE, follow that convention. Otherwise, when debugger setup is in scope, ask which editor or IDE developers use and whether its project configuration should be committed. Treat Windows, macOS, and Linux compatibility as the default requirement; do not ask the user to choose an operating system. Do not ask when the requested work does not include an editor integration; provide the portable debug endpoint and connection details instead.

Docker does not receive breakpoints. It only routes the debug connection from the host to the runtime. The developer's editor or IDE attaches to the published localhost port and sends breakpoint requests using the runtime's debug protocol. Configure the editor explicitly rather than assuming it discovers the container automatically.

For an editor attach configuration, supply the published host address and port plus a mapping between local source paths and container paths. Enable source maps when the runtime executes transpiled code, such as TypeScript. With a process watcher that replaces the runtime after source edits, use the IDE's supported reconnect option or document that the developer must reattach. Verify by placing a breakpoint in mounted source, invoking the relevant code path, and inspecting the pause.

Do not generate VS Code, JetBrains, or another editor's files unless that editor is already used by the project or the user asks for it. Explain the portable ingredients—runtime debug server, Compose port mapping, and local-to-container source mapping—so equivalent attach settings can be made in any IDE.

## Developer experience and verification

Provide a concise start command, local URLs/ports, stop command, and any first-run requirements. Do not put real secrets in images or Compose files; use ignored local environment files plus committed examples.

Before finishing, verify proportionally to the project:

1. Build and start the development stack.
2. Confirm health checks and service status.
3. Confirm the application reaches its expected local endpoint.
4. Edit a watched source file or otherwise demonstrate that the intended reload path works.
5. When debugger support is included, attach and stop at a real breakpoint.
6. State any unverified platform-specific assumption rather than presenting it as established fact.
