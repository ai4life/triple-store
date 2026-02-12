## 2026-02-11

- Converted `jena-fuseki/docker-entrypoint.sh` to Unix (LF) line endings to fix runtime errors seen when starting the container (repeated `set: illegal option -` and `: not found`). Corrected the readiness loop to use `until curl --output /dev/null --silent --head --fail http://localhost:3030; do`.

- Updated `jena-fuseki/Dockerfile` to be robust against Windows CRLF checkouts:
  - Strip CRLF from scripts after `COPY` (`sed -i 's/\r$//'`) so scripts run correctly inside Alpine/Linux images.
  - Invoke the entrypoint using `bash` under `tini` to ensure the `#!/bin/bash` shebang is honored.
  - Strip CRLF and set executable bits for `load.sh`, `tdbloader`, and `tdbloader2`.

- Rebuilt and tested the image locally; Fuseki starts successfully and container logs include "Fuseki is available :-)".
- Rebuilt and tested the image locally; Fuseki starts successfully and container logs include "Fuseki is available :-)".

- Security notes: `ADMIN_PASSWORD` is only injected when needed and is unset after use; avoid exposing secrets in build logs or image layers. Inputs used by the entrypoint are processed using `envsubst` and handled carefully.

-- Change recorded by automation per `AGENTS.md` instructions.

- Make Fuseki respect Cloud Run's `PORT` environment variable:
  - `docker-entrypoint.sh` now reads `PORT` (defaulting to 3030) and uses it for readiness checks and dataset creation.
  - `Dockerfile` `CMD` updated to run `/jena-fuseki/fuseki-server --port ${PORT:-3030}` so the server listens on the injected port (e.g. 8080 on Cloud Run).

-- Change recorded by automation per `AGENTS.md` instructions.

- Added `docker-compose.yml` at repository root to simplify local deployment:
  - Builds the `jena-fuseki` service, maps host port `3030:3030`, creates a named `fuseki-data` volume, and includes a basic HTTP healthcheck.
  - Verified via `docker compose up -d` that the service comes up and responds to `/$/ping`.

-- Change recorded by automation per `AGENTS.md` instructions.

- Updated `docker-compose.yml` to build both `jena` and `jena-fuseki` and clarified persistence:
  - Added a `jena` service that builds from `./jena` (image contains the Jena CLI/jars). The service is intended for CLI use and does not require a persistent volume.
  - Removed the unnecessary `jena-data` volume; `jena` is image-only and provides tools via `docker compose exec jena` or `docker compose run --rm jena`.
  - Kept `fuseki` service and its `fuseki-data` volume; confirmed `docker compose up -d --build` brings both services up and that `fuseki` responds to `/$/ping`.

-- Change recorded by automation per `AGENTS.md` instructions.
