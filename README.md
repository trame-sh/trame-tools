# trame-tools

Base Docker image and host-side runtime files for [trame](https://trame.sh) dev environments.

Published at `ghcr.io/trame-sh/trame-tools`.

## What's included

### Docker image (`docker/`)

A dev environment base image with:

- **Node.js 22** (bookworm-slim)
- **Git 2.48+** (built from source for `--relative-paths` worktree support)
- **Claude Code CLI** (for headless agent loop)
- **gosu** + UID/GID mapping entrypoint
- **MCP server** (`shell_exec` tool for Claude Code)

### Host-side tools (`tools/`)

Files that live in user projects and are updated via `denv/update.sh`:

- **`update.sh`** — Self-updating script that pulls the latest tools from this repo
- **`mcp.sh`** — MCP launcher that auto-starts the compose stack
- **`build-denv.sh`** — Force-rebuild the dev environment image (no cache)
- **`agent.sh`** — Headless agent launcher (see below)
- **`agent-loop.sh`** — Container-side agent loop (used by `agent.sh`)
- **`prompt.md`** — Default prompt template for the agent loop (`[ADAPT]` per project)
- **`AGENTS.md`** — Agent instructions for Claude Code
- **`.claude/agents/teammate.md`** — Claude Code agent definition for parallel teammates

### Headless agent (`agent.sh`)

`denv/agent.sh` launches an autonomous Claude Code agent in an isolated environment:

1. Auto-increments a worker ID from running compose projects
2. Creates a git worktree (`worktrees/worker-N`) for isolation
3. Starts a compose stack mounting the worktree
4. Runs Claude Code in a loop inside the container (`agent-loop.sh`)
5. Follows container logs in the foreground — Ctrl+C stops everything

```bash
# Use a specific model
denv/agent.sh --model opus
```

The agent loop reads `denv/prompt.md` as its prompt. Customize this file per project. If Claude creates a `.no-plan` signal file, the loop sleeps 5 minutes before retrying.

On exit, the compose stack is stopped and the worktree is removed if clean (no uncommitted changes).

## Usage

This image is meant to be extended in your project's Dockerfile:

```dockerfile
FROM ghcr.io/trame-sh/trame-tools:latest

# Add project-specific layers here
# RUN curl -fsSL https://get.pnpm.io/install.sh | SHELL="$(which bash)" bash -
```

See [trame-project-template](https://github.com/trame-sh/trame-project-template) for a ready-to-use template.

## Building locally

```bash
docker build docker/ -t trame-tools
```

## Releasing

Push a semver tag to trigger the GitHub Actions workflow:

```bash
git tag v1.0.0
git push origin v1.0.0
```

This will:
1. Build and push the Docker image to `ghcr.io/trame-sh/trame-tools:<version>` + `:latest`
2. Create a GitHub release with `tools/` packaged as `trame-tools.tar.gz`

## License

MIT
