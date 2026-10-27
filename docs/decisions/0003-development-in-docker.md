# 0003: Development runs fully in Docker

- **Status:** Accepted
- **Date:** 2026-10-24

## Context

In development, the database and Redis run in Docker in every setup we considered. The open question was where Django itself runs. GeoDjango needs GDAL, GEOS and PROJ, which are awkward to install on Windows. We want both machines identical, whatever operating system each of us uses, and as close to production ([0002](0002-runtime-structure.md)) as possible.

## Decision

Docker Compose runs the whole backend in development: Django, PostgreSQL + PostGIS and Redis. Django runs from the same Dockerfile as production, with the source code mounted into the container so changes reload without rebuilding.

## Alternatives considered

- **Django directly on Windows, only the database in Docker.** Shortest commands and best editor integration, but GDAL on Windows is fiddly and the setup differs from production.
- **Developing inside WSL2.** Linux like production and fast, but specific to Windows, and everything (Git, uv, editor tooling) moves into WSL.

## Consequences

- Nobody installs Python libraries, GDAL or PostgreSQL by hand; Docker Desktop is the only requirement for running the backend.
- Development, CI and production run the same image, so "works on my machine" problems are rare.
- Backend commands run through Compose (for example `docker compose exec backend python manage.py migrate`), which is longer to type.
- Adding a dependency means rebuilding the image.
- A local `backend/.venv` (from `uv sync`) is still kept, only for the editor's autocomplete and for tools that run outside the container.
- With the code on the Windows filesystem, file access from the container is slower and Django's autoreload falls back to polling. If that becomes a problem, the repository can move into the WSL2 filesystem.
