# Dev Environment

This project uses a containerized dev environment. Claude Code runs on the **host**; the Docker container provides tools and services via MCP.

```
Host (Claude Code)                   Docker container (devenv)
├── reads/writes files directly      ├── runs shell commands (tests, builds, etc.)
├── manages git                      ├── has project toolchains installed
└── talks to container via MCP       └── connects to companion services (postgres, etc.)
```

## Rules

- specs of project, features and implementation plans are available on trame mcp tool
- **shell commands** → use the `shell_exec` MCP tool (may appear as `mcp__devenv__shell_exec`) (runs inside the container at `/workspace`)
- **file operations** → read and write directly on the host, do NOT use `shell_exec` for file I/O
- **companion services** (postgres, redis, etc.) → accessible by service name from inside the container — see `denv/docker-compose.base.yml` for connection details
- always work with a team. each teammate should claim an implementation plan from trame, and work in an isolated worktree
- worktree should be created with --relative-paths in ./wortrees/<worktree name>
- each teammate creates its own docker compose stack with `docker compose -p <worker name> up -d` so service ports don't collide between workers
- use latest sonnet model for teammate
- the team lead merges teammate branches back into main once their work is done
- teammate stop their docker compose stack and delete their worktree when done

## Bootstrapping

The `shell_exec` MCP tool is provided by the `devenv` MCP server (`denv/mcp.sh`), which auto-starts the Docker stack. If `shell_exec` is not available, bootstrap the environment yourself using Bash:

1. **Build the image** (if missing) — `docker build denv/ -t <project>-denv`
2. **Start the stack** — run `denv/mcp.sh` or:
   ```
   docker compose -p <project>-coord -f denv/docker-compose.coord.yml up -d --wait
   ```
3. **Fallback** — if the MCP tool remains unavailable, run commands inside the container via Bash:
   ```
   docker compose -p <project>-coord -f denv/docker-compose.coord.yml exec devenv <command>
   ```

`<project>` is the repo directory name (e.g. `trame`).

## Customization

Files marked `[ADAPT]` in `denv/` are meant to be edited per-project.

<!-- Add project-specific instructions below (e.g. how to run tests, which package manager to use, migration commands) -->
