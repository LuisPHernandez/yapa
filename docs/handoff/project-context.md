# Project context

Everything needed to get up to speed on this project: what it is, how it is built, what has been decided, and what is still open. Written mainly for AI assistants starting a new session, and readable by any new teammate.

Read this first, then:

- [docs/handbook/tooling.md](../handbook/tooling.md): the tools that are in the project and why
- [docs/handbook/workflow.md](../handbook/workflow.md): how a change gets from a branch into `main`
- [docs/decisions/](../decisions/): decision records
- [docs/ideas/team-tools.md](../ideas/team-tools.md): the tooling plan, with a checkbox for each tool already applied

When this file and the code disagree, the code wins, and this file should be fixed in the next PR.

## The product

A marketplace for surplus food in Guatemala, similar to Too Good To Go. The repository is called `yapa`; early notes call the product 2good2go.

- Food businesses (bakeries, restaurants, supermarkets), each with one or more branches, publish surprise packages of surplus food at a discount.
- Customers find packages near them, pay by card in the app, and pick them up at the branch within a time window by showing a QR code.

## Team and goals

Two developers who work together daily. Their background is Python, JavaScript, Java, Go, PHP (Laravel) and some C/C++.

The codebase is meant to be production-ready from the start: the first release is a smaller feature set built on the final architecture, not a prototype to throw away.

## Stack and architecture

| Area | Choice |
| --- | --- |
| Backend | Django + Django REST Framework + GeoDjango, as a modular monolith. No microservices, no Kubernetes. |
| Django apps | `accounts`, `stores`, `catalog`, `orders`, `payments`, `payouts`, `invoicing` |
| Database | PostgreSQL + PostGIS |
| Mobile | Two Flutter apps (customer app and store app) sharing one backend and one database |
| API contract | drf-spectacular generates an OpenAPI spec; the Dart client is generated from it. The API is versioned (`/api/v1/`) because old app versions stay installed. |
| Background jobs | Needed for expiring reservations, closing drops, notifications, invoicing and payouts. Celery + Redis or a Postgres-based queue, not decided yet. |
| Infrastructure (planned) | Docker containers on a managed platform, managed PostgreSQL with PostGIS, S3-compatible storage + CDN for images, Firebase Cloud Messaging for push, Sentry for errors. Separate dev, staging and prod environments. |

### Repository layout

```
.github/          GitHub configuration
backend/          Django project
frontend/         Flutter apps (customer and store)
docs/decisions/   One file per important decision
docs/handbook/    What tools exist (tooling.md) and how to make changes (workflow.md)
docs/handoff/     This file
docs/ideas/       Temporary notes and plans; files here come and go
```

The Flutter code lives in `frontend/`. Older notes say `/mobile`; that name is out of date. Common Dart code (generated API client, models, auth, theme) may later move to a shared package inside `frontend/`.

## Key decisions

These are settled. Ask the team before deviating from any of them.

### Users, auth and permissions

- **Custom User model**, created before the first migration.
- **One account for both apps.** Email is unique. The customer app never shows work-related features.
- **Auth:** JWT with `djangorestframework-simplejwt`. Tokens carry minimal claims (the user ID). Flutter stores tokens in `flutter_secure_storage`.
- **No roles table and no role column on users.** Every user is a customer. A user is store staff only if they have an active `store_members` row, whose `role` is `owner`, `manager` or `staff`. `store_member_locations` says which branches they work at.
- **A user can belong to at most one store** (`store_members.user_id` is unique). This is intended.
- **Platform admins** use Django's `is_staff` / `is_superuser`.
- **Permissions are defined in code.** The role-to-permissions mapping lives in code, is enforced by DRF permission classes, and is exposed through a `/me` endpoint so the store app can show or hide screens and buttons.
- **Membership is checked against the database on every request**, never trusted from the token.

### Catalogue and stock

- **Package types vs. drops:** `package_types` is what a branch sells (description, price, value). `drops` is one day's release of a package type (quantity, pickup window).
- **Stock:** `quantity_remaining` is decremented atomically, so a drop can never be oversold.
- **Location:** branch coordinates are stored as `geography(Point, 4326)` with a GiST index. Proximity search uses `ST_DWithin` / `ST_Distance`.

### Money and orders

- **Card payments only**, no cash.
- **No separate payment table.** Orders have separate `order_status` and `payment_status` fields.
- **Stores are paid periodically** through `payouts`. Each order stores a snapshot of `unit_price` and `commission_rate`, so later price or commission changes don't alter past orders.
- **Money** is stored as decimals, in GTQ.
- **Time** is stored as `timestamptz` in UTC, with `USE_TZ = True`. Guatemala (`America/Guatemala`) has no daylight saving time.
- **Statuses** are Django `TextChoices` backed by database check constraints.

### Process

- Every change goes through a short-lived branch and a pull request approved by the other developer; `main` is protected. Details in [workflow.md](../handbook/workflow.md).
- A backend change, the regenerated OpenAPI spec and the matching Flutter update go in one PR, so the API contract never drifts.

## Data model

Once Django models exist, `models.py` is the source of truth and any schema diagram is documentation only. Until then, the draft schema has these tables:

| Table | Purpose |
| --- | --- |
| `users` | Every account: name, email, phone, status, `is_staff` / `is_superuser` |
| `stores` | A business: name, NIT (unique), commission rate, approval status |
| `store_members` | Links a user to their one store with a role |
| `locations` | A store's branches: address, zona / municipio / departamento, coordinates, rating |
| `store_member_locations` | Which branches a store member works at |
| `user_likes` | Branches a customer has saved |
| `categories` | Food and dietary tags |
| `package_types` | What a branch sells: name, price, original value, active flag |
| `package_type_categories` | Tags on a package type |
| `drops` | One day's release: date, pickup window, total and remaining quantity, optional price override, status |
| `orders` | A customer's purchase from a drop: quantity, price and commission snapshot, order and payment status |
| `payouts` | A periodic transfer to a store: period, gross, commission, net |
| `reviews` | One rating per order |

Fields still to add to the draft:

- `orders`: `pickup_code` (random, unique), `provider_transaction_id`, `paid_at`, `refunded_at`, `expires_at`, and FEL invoicing fields (`customer_nit` or "CF", FEL identifiers).
- `stores`: bank details for payouts. `payouts`: `transfer_reference`.
- `package_types`: `image`.
- A new `devices` table (`user_id`, `fcm_token`, `platform`) for push notifications.

## Open decisions

- Payment gateway
- FEL invoicing provider (FEL is Guatemala's electronic invoicing system)
- Hosting provider
- Background job queue (Celery + Redis or Postgres-based)
- Whether store staff may buy packages from their own store

When one of these is settled, it gets a record in `docs/decisions/` and moves to "Key decisions" above.

## Working with the team (for AI assistants)

- **Don't change anything on disk without explicit permission.** Propose file contents and commands in the conversation; the developer applies them or explicitly allows you to. Permission covers only the change it was given for.
- **Work in small steps and explain each one.** The developers want to understand every tool and change, not just receive them.
- **Ask before deviating** from the key decisions above.
- **Keep the docs current.** A change that adds a tool updates `docs/handbook/tooling.md`. A change to how the team works updates `docs/handbook/workflow.md`. A settled decision gets a record in `docs/decisions/`. A change to anything described here updates this file.
- **At least one developer works on Windows.** Text files use LF line endings on every machine, enforced by `.gitattributes`.
