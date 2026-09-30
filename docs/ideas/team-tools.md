# Team Tools

Stack: Django + Django REST Framework + GeoDjango, PostgreSQL + PostGIS, Flutter.
Team of 2, working together daily.

The goal is tooling that catches mistakes automatically and keeps both machines identical. Process-heavy tools (Jira, Slack) aren't needed at this size.

---

## Collaboration

- **GitHub pull requests, even for two people.** Every change goes through a short-lived branch and a PR the other person reviews. It spreads knowledge of the code, catches bugs, and gives CI a place to run. Keep `main` always deployable.
- **GitHub Issues + Projects** as a simple board (To do / Doing / Done). That's all the project management needed.
- **Figma** for screens. Agree on a screen before building it, especially before showing it to stores.
- **Decision records:** a `docs/decisions/` folder with one short markdown file per important decision (e.g. "why PostGIS geography", "one app vs. two"). When you talk daily, decisions live in your heads and get forgotten or remembered differently.

## Python code quality

- **uv:** modern, very fast package manager with a lockfile, so both developers get exactly the same dependency versions.
- **Ruff:** linter and formatter in one tool (replaces black, isort, flake8). No style debates, no formatting-only diffs.
- **mypy + django-stubs:** type checking catches a whole class of bugs before runtime.
- **pytest + pytest-django + factory_boy:** tests with easy test-data creation.
- **pre-commit:** runs Ruff and other checks automatically before each commit.

## Environment

- **Docker Compose** for PostgreSQL + PostGIS (`postgis/postgis` image) and Redis. Same database version for both; nobody installs PostGIS by hand.
- **Environment variables** via `django-environ`: commit a `.env.example`; never commit real `.env` files.

## Automation

- **GitHub Actions:** on every PR, run the linter, type checks, tests (with a PostGIS service container), and the OpenAPI breaking-change check.
- **Dependabot:** automatic PRs for dependency updates and security fixes.
- **Sentry:** error tracking from the first deployment.

## Flutter

- `flutter analyze` with a strict lint set (e.g. `very_good_analysis`).
- `dart format` enforced in CI.

---

A backend change, the regenerated OpenAPI file, and the Flutter update go in one pull request, so the API contract never drifts. Separate customer and store apps can later share a common Dart package inside `/mobile`.
