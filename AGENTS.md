# AGENTS.md — pod-postgresql

Standalone candy repo for the `postgresql` candy — a Docker-compatible PostgreSQL
server with pgvector and persistent data, on port `5432`. The entire candy lives
in `charly.yml` at the repo root. There is no source tree — the entrypoint is
authored inline in the plan.

Canonical files:

- `charly.yml` — the `postgresql:` candy entity (description, `require`, `env`,
  `distro`, `port`, `volume`, `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:postgresql` — the owning skill: the candy properties,
  the env contract, the entrypoint features, and verification. Load before
  editing, building, deploying, or troubleshooting this candy.
- `/charly-infrastructure:vectorchord` — the extension candy that sets
  `POSTGRES_SHARED_PRELOAD_LIBRARIES`.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; volumes, ports, `env_provide`).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `write:` / `check:`, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the server / `psql` / `initdb` / `pg_isready` binaries, the
  executable entrypoint, the `pgvector` package, the initialized data directory,
  the ready unix socket, the running `postgresql` service, and that the server
  never runs as root.

## Modify this repo

- Edit the `postgresql:` candy entity in `charly.yml`; the `skill:` entity in the
  same file is the owning skill's source — a candy change and its skill change
  land together.
- The entrypoint is authored inline in the `plan:` (`write:` step). Keep the
  root-capable branch, the `PGDATA` / `SOCKET_DIR` pinning, and the
  `POSTGRES_SHARED_PRELOAD_LIBRARIES` handling in step with the checks.
- The `pgdata` volume at `~/.postgresql/data` is the persistent store; keep it in
  step with `PGDATA` and the socket dir.
- The `skill:` entity is the source for `/charly-infrastructure:postgresql`;
  never edit the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
