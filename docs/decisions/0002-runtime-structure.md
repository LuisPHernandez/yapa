# 0002: Runtime structure

- **Status:** Accepted
- **Date:** 2026-10-24

## Context

The backend has to run the same on our laptops, on a single server now, and possibly on a managed platform later. There is no income yet, so production starts small. GeoDjango needs native libraries (GDAL, GEOS, PROJ) that are easy to install on Linux and awkward elsewhere. The database decision is in [0001](0001-self-hosted-database-first.md).

## Decision

- **One backend image**, built from one Dockerfile, used by every backend process with a different start command: `web` (Gunicorn + Django, answers the apps' HTTP requests) and `worker` (background jobs).
- **Official images for everything else** (PostGIS, Redis, the reverse proxy). We configure them; we don't build them.
- **Production starts on one Linux server running Docker Compose.** A reverse proxy is the only container exposed to the internet and handles HTTPS. The database runs on the same server per 0001.
- **Outside the server:** S3-compatible storage for images and database backups, Firebase Cloud Messaging, Sentry, the payment gateway.
- **Portability rules every feature follows:**
  - All configuration comes from environment variables.
  - Nothing is stored on a container's disk; uploads go to storage.
  - No state is kept in memory between requests; any `web` container can serve any request.
  - Logs go to standard output.

## Alternatives considered

- **A managed platform from day one.** Less operational work, but a monthly cost before there is revenue.
- **Python and GDAL installed directly on the server.** No Docker to learn, but the server drifts from development and is hard to rebuild.
- **A separate image per process.** Unnecessary when one image with different start commands does the job, and it would let processes drift apart.

## Consequences

- Moving the database to a managed service, moving the backend to a managed platform, or adding more `web` servers needs no code changes.
- Until then the single server is a single point of failure.
- Every new feature must respect the portability rules: for example, user uploads must go to storage, never to the local disk.
