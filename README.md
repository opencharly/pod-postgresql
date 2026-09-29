# pod-postgresql

The `postgresql` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It ships a Docker-compatible
PostgreSQL server that self-initializes its cluster and accepts client
connections.

## What it provides

Installs the PostgreSQL server (`postgres`, `initdb`, `pg_ctl`), the `psql`
client and the `pg_isready` readiness probe, and the `pgvector`
similarity-search extension. A generated entrypoint wrapper
(`/usr/local/bin/postgresql-entrypoint.sh`) performs first-run `initdb`, database
creation, password handling, and `/docker-entrypoint-initdb.d` processing, then
execs the server on a unix socket under `~/.postgresql` listening on
`127.0.0.1:5432`.

| Property | Value |
|---|---|
| Port | `5432` |
| Service | `postgresql` (`/usr/local/bin/postgresql-entrypoint.sh`, `restart: always`, priority 10) |
| Volume | `pgdata` → `~/.postgresql/data` |
| Env | `PGDATA=~/.postgresql/data`, `POSTGRES_HOST_AUTH_METHOD=scram-sha-256` |
| Packages | `postgresql` + `pgvector` (arch), `postgresql-server` + `postgresql-contrib` + `pgvector` (fedora) |
| Consumed by | `pod-immich`, `pod-immich-ml` |

The entrypoint is root-capable: root-posture boxes re-exec the body under the
distro's `postgres` OS user after preparing the declared data/socket dirs (initdb
refuses root); uid-1000 boxes are unchanged. It also honors
`POSTGRES_SHARED_PRELOAD_LIBRARIES` so extension candies can load their libraries.

## How to use it

```yaml
my-image:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/pod-postgresql:<tag>'
```

```bash
charly box build my-image
charly config my-image
charly start my-image
```

Consumers such as `pod-immich` override `POSTGRES_USER` / `POSTGRES_DB`; the
`charly config` step injects `PGHOST` / `PGPORT` for service discovery.

## Layout

- `charly.yml` — the `postgresql:` candy entity (description, `require`,
  `distro`, `env`, `port`, `volume`, `service`, `plan`) plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:postgresql` — the candy properties, the
  env contract, the entrypoint features, and verification.
- `/charly-infrastructure:vectorchord` — sets `POSTGRES_SHARED_PRELOAD_LIBRARIES`.
- `/charly-infrastructure:redis` — often paired in service stacks.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
