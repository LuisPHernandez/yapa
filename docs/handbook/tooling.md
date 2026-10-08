# Tooling: what is in this project and why

Every tool, config file, and convention added to the project is listed here, with what it does and why we have it. When a PR adds or removes a tool, it updates this file in the same PR.

For how to actually make a change day to day, see [workflow.md](workflow.md).

## Status

| Tool | Status | Section |
| --- | --- | --- |
| Git conventions (`.gitignore`, `.gitattributes`) | Done | [Git](#git) |
| GitHub pull requests + protected `main` | PR template done; ruleset is manual setup in GitHub | [GitHub](#github) |
| Decision records | Done | [Decision records](#decision-records) |
| GitHub Issues + Projects board | Manual setup | [GitHub](#github) |
| uv | Planned | |
| Docker Compose (PostGIS + Redis) | Planned | |
| Environment variables (`django-environ`) | Planned | |
| Ruff + pre-commit | Planned | |
| pytest + pytest-django + factory_boy | Planned | |
| mypy + django-stubs | Planned | |
| GitHub Actions | Planned | |
| Dependabot | Planned | |
| Flutter lints + `dart format` | Planned, with the Flutter project | |
| OpenAPI breaking-change check | Planned, once endpoints exist | |
| Sentry | Planned, at the first staging deploy | |

## Repository layout

```
.github/          GitHub configuration (PR template; CI workflows later)
backend/          Django project
docs/decisions/   One file per important decision
docs/handbook/    This file and workflow.md
docs/ideas/       Notes and plans that are not decisions yet
```

## Git

### `.gitignore`

Lists files Git must never track: secrets (`.env`), things each machine generates for itself (virtual environments, caches, build output), and editor or OS clutter.

The `.env` rule matters most. It ignores `.env` and every `.env.something` variant, except `.env.example`, which is the committed template with fake values. A secret that reaches GitHub has to be treated as leaked and rotated, even if the commit is deleted afterwards.

### `.gitattributes`

Windows ends lines with CRLF, Linux and macOS with LF. Without a rule, the same file can differ between two machines, which produces diffs where every line "changed", and breaks shell scripts when they run inside Linux Docker containers.

`* text=auto eol=lf` makes Git store and check out every text file with LF on every machine, regardless of each person's local `core.autocrlf` setting. Windows-only scripts (`.bat`, `.cmd`, `.ps1`) keep CRLF, and binary files such as images are marked so Git never rewrites them.

## GitHub

### Pull requests and protected `main`

Nobody pushes to `main` directly. Every change is made on a short-lived branch and merged through a pull request that the other person approves. With two people this still pays off:

- Both of us know all of the code, not just the half we wrote.
- A second reader catches bugs and unclear code cheaply.
- CI gets a place to run before code reaches `main`, so `main` is always deployable.

This is enforced by a ruleset on GitHub (Settings → Rules → Rulesets), not by trust. The ruleset on `main`:

| Rule | Effect |
| --- | --- |
| Restrict deletions | `main` cannot be deleted |
| Block force pushes | History on `main` cannot be rewritten |
| Require a pull request before merging, 1 approval | No direct pushes; the other person must approve |
| Dismiss stale approvals when new commits are pushed | An approval covers only the code that was reviewed |
| Require conversation resolution | Every review comment is answered before merge |
| Require status checks to pass | Added when GitHub Actions exists |

Repository settings that go with it (Settings → General → Pull Requests):

- **Allow squash merging only, with "Pull request title" as the default commit message.** Each PR becomes one commit on `main`, named after the PR, so history reads as a list of changes rather than a list of work-in-progress commits, and reverting a change is reverting one commit.
- **Automatically delete head branches.** Merged branches are removed from GitHub.
- **Collaborators** (Settings → Collaborators) must be added before the ruleset is activated, or nobody can give the required approval.

### PR template

`.github/pull_request_template.md` pre-fills the description of every new PR with three prompts: what and why, how it was tested, and a short checklist. It grows as tools arrive (for example, an item for the regenerated OpenAPI file once the API exists).

### Issues and Projects

Work is tracked as GitHub Issues on one Projects board with three columns: To do, Doing, Done. A PR that says `Closes #12` in its description closes the issue and moves the card when it merges. This is all the project management we use.

## Decision records

`docs/decisions/` holds one short file per important decision: the context, what we chose, what we rejected, and the consequences. See [the README there](../decisions/README.md) for when and how to write one.
