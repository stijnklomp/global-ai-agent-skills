---
name: docker-runtime
description: MUST USE when executing commands, running code, installing dependencies, running tests, or managing environments in any project. Enforces a Docker-first execution strategy to avoid polluting the host system.
---

# Docker-First Execution

This skill ensures that the agentic AI always prefers containerized execution via Docker or Docker Compose over installing packages or tools directly on the host system.

## Philosophy

The host system is shared and fragile. Installing package managers, runtimes, or dependencies directly on the host can lead to misconfigured environments, version conflicts, and broken tooling for other projects. **Always use Docker first.** Only fall back to host-level installation when no Docker or Docker Compose configuration exists in the project.

## Execution Strategy

Before running any command that requires a runtime (Node, Bun, Python, etc.) or before installing anything, determine the project's Docker capabilities:

1. **Read the project's README.md FIRST**
   - Look for sections like "Getting Started", "Development", "Docker", "Running with Docker", or "Testing"
   - If the README contains Docker-specific commands, follow them exactly
   - Only proceed to step 2 if the README does not contain Docker instructions

2. **Check for Docker Compose**
   - Look for `docker-compose.yml`, `docker-compose.yaml`, `compose.yml`, or `compose.yaml`
   - If present, use `docker compose` (or `docker-compose`) for all operations
   - Prefer Docker Compose profiles (e.g., `--profile test`, `--profile dev`) if they exist

3. **Check for Dockerfiles**
   - Look for `Dockerfile`, `Dockerfile.*`, or files in a `docker/` directory
   - If present but no Compose file exists, use `docker build` and `docker run`

4. **Only install on the host as a last resort**
   - If no Docker configuration exists whatsoever, then and only then install the required package manager/runtime manually

## Finding the Right Commands

Every project is different. **Never assume generic commands.** You should have already read the project's `README.md` in step 1 of the Execution Strategy above. If you skipped it, go back and read it now before proceeding.

- Look for sections like "Getting Started", "Development", "Docker", "Running with Docker", or "Testing"
- Follow the exact commands and arguments documented in the README
- If multiple options are shown (e.g., local vs. Docker), choose the Docker option

**If the README does not contain Docker instructions**, use your best judgment with sensible generic commands, but verify they work before proceeding.

## Reusing Containers (Default Behavior)

When working with Docker or Docker Compose, start the environment once and reuse it for the duration of your response. Do not create a new container for every command.

### Why this matters

- Creating a container for every command is slow. It re-runs entrypoints, re-mounts volumes, and re-initializes the environment.
- Reusing a running container is fast. It preserves the warm environment, cached dependencies, and initialized state.

### How to reuse containers

1. **Start services in detached mode** when you first need them. Use background/detached flags (e.g., `-d`) so the container stays alive after the command returns.
2. **Run subsequent commands inside the already-running container**. If a container is already running for this project, execute new commands inside it rather than starting a fresh one.
3. **Only stop containers when you are completely finished** responding to the user.

### When not to reuse

Only create a fresh container for each command when:
- The service is designed as a **one-off job** (e.g., a migration runner, a seed script, a CI runner)
- The command must run in a **completely fresh, isolated container** with no shared state
- The service has no long-running process (e.g., it runs a script and exits immediately)

For **development services, test runners, and application servers**, prefer reusing the same container.

### Checking if a container is already running

Before starting anything, check for existing containers:
```bash
docker ps
```

If a container for this project is already running, run subsequent commands inside it instead of starting a new one.

## Container Conflict Prevention

Even though Docker Compose provides isolation, you must ensure existing containers do not conflict:

- **Check running containers before starting new ones:**
  ```bash
  docker ps
  ```
- **Check for existing Compose projects:**
  ```bash
  docker compose ls
  ```
- **If a conflicting container or Compose project is running:**
  - If it's the same project and you need a fresh state: restart it or recreate it cleanly
  - If it's the same project and you just need to run commands: reuse it
  - If it's a different project using the same ports/resources: stop it first
  - **Never assume ports are free.** Always verify before starting. If a port is already taken, stop the conflicting container or use a different host port via `docker run -p` flags — **do not modify `docker-compose.yml`, `Dockerfile`, or any project Docker configuration files just to avoid a port conflict**

## Generic Fallback Commands

Only use these if the project has Docker configuration but the README provides no specific instructions:

**Installing Dependencies (Docker Compose):**
```bash
docker compose run --rm <service-name> <install-command>
```

**Running Tests (Docker Compose):**
```bash
docker compose run --rm <service-name> <test-command>
# or if a test profile exists:
docker compose --profile test up --build --exit-code-from <test-service>
```

**Running the Application (Docker Compose):**
```bash
docker compose up --build
```

**One-Off Commands (Docker Compose):**
```bash
docker compose run --rm <service-name> <command>
```

**With Dockerfile only:**
```bash
docker build -t <project-name> .
docker run --rm -p <host-port>:<container-port> <project-name>
```

**Without Docker (last resort):**
```bash
# Only after confirming no Docker configuration exists
<package-manager> install
```

## Important Rules

- **Read README.md before inspecting docker-compose.yml or Dockerfile** — the README often contains the exact commands to use
- **Never install a package manager (Bun, Node, npm, yarn, pnpm, pip, poetry, etc.) on the host if the project has any Docker configuration**
- **Prefer `docker compose` over `docker run`** when both are available
- **Always check `docker ps` and `docker compose ls` before bringing up new infrastructure** to avoid port conflicts and resource contention
- **Reuse running containers** for subsequent commands instead of creating new ones with `--rm` for every command
- **Start services in detached/background mode** and keep them running for the duration of your response. Only tear down when you are completely finished.
- **Only fall back to host execution** when you have explicitly verified that no `docker-compose.yml`, `compose.yml`, `Dockerfile`, or `docker/` directory exists in the project
- **Respect existing Docker networks** — if the project defines custom networks in Compose, do not create conflicting external networks

## Verification Checklist

Before executing any project command, ask yourself:

1. Have I read the `README.md` for Docker-specific instructions?
2. Does this project have a `docker-compose.yml` or `compose.yml`?
3. Does this project have a `Dockerfile`?
4. Are there already running containers that might conflict?
5. Can I run this command inside an existing container instead of on the host?

If the answer to (1) is yes, follow the README exactly. If the answer to (2) or (3) is yes, use Docker. If the answer to (4) is yes, resolve the conflict first.
