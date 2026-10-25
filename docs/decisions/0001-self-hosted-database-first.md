# 0001: Self-hosted database first, managed later

- **Status:** Accepted
- **Date:** 2026-10-24

## Context

Production needs PostgreSQL with PostGIS. A managed database service handles backups, failover and upgrades for us, but costs money every month before the product has any income. Running PostgreSQL ourselves in a Docker container on the same server as the backend is cheaper and adds no network latency, but makes its operation our job.

## Decision

Start with PostgreSQL + PostGIS running in a Docker container on our own server, and move to a managed PostgreSQL + PostGIS service once the product has income.

The backend must not depend on where the database runs: it reads the connection from a `DATABASE_URL` environment variable, so moving to managed means changing that variable and moving the data, with no code changes.

## Alternatives considered

- **Managed database from day one.** Least operational work and safest for the data, but a fixed monthly cost before there is any revenue.
- **PostgreSQL installed directly on the server, without Docker.** Works, but the container keeps the PostgreSQL and PostGIS versions identical to development and makes the server easy to rebuild.

## Consequences

- Backups are our responsibility. Automated backups stored off the server, and a tested restore, must exist before real customer data or payments reach the database.
- If the server fails, the database goes down with it until it is restored.
- The PostGIS version we run should be one that managed providers also offer, so the later move is a plain dump and restore.
- Moving to managed needs a short maintenance window to copy the data.
