# GlobalTrack v11.1 — Backend Audit & Stability Fix

GlobalTrack is a Flask + SQLite shipping/tracking application designed as an interview-friendly full-stack project.

## What was fixed in v11.1

This release audits the v11 backend and fixes several issues that could cause confusing runtime behavior:

- Removed invalid escaped quotes from Jinja `url_for()` expressions.
- Removed the old demo JavaScript that blocked real Login, Register, and Create Shipment POST requests.
- Homepage tracking form now submits to the real Flask tracking route.
- Login now redirects users according to their role.
- Delivery Agent and Admin routes are protected by server-side role checks.
- The generic frontend fallback no longer exposes protected templates by guessing filenames.
- Customer profile and support pages now have explicit authenticated routes.
- Registration accepts only public account types (`Customer` and `Business`). Privileged roles cannot be self-selected.
- Terms checkbox is validated on the server.
- Login `next` redirects are restricted to local paths to avoid unsafe external redirects.
- SQLite foreign-key enforcement is enabled for each request.
- Customer dashboard count queries safely return zero instead of `NULL`.
- Delivery-agent status changes follow a controlled state transition map.
- Pickup can only be confirmed from `Pending Pickup`.
- Delivery can only be confirmed from a delivery-ready state.
- Added `create_admin.py` for local development/testing of the protected admin area.
- Verified Python syntax, Jinja template syntax, JavaScript syntax, endpoint references, and static CSS references.

## Roles

### Customer
- Register and log in.
- Create shipments.
- Track shipments publicly.
- View shipment history.
- View notifications.

### Business
- Uses the current customer shipment workflow.
- Business-specific permissions can be added later.

### Delivery Agent
- Created explicitly with `create_delivery_agent.py`.
- Can see only shipments assigned through `shipments.agent_id`.
- Can confirm pickup.
- Can update operational shipment status.
- Can confirm delivery.

### Admin
- Created explicitly with `create_admin.py`.
- Admin pages are protected by server-side role checks.
- Full admin shipment/user operations are the next backend feature area.

## Run locally

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

Open:

```text
http://127.0.0.1:5000
```

## Create test privileged accounts

Start the application once first so the database tables exist.

Delivery agent:

```bash
python create_delivery_agent.py
```

Admin:

```bash
python create_admin.py
```

Do not expose these development scripts as public web routes.

## Project structure

```text
app.py                         Flask routes, authentication, database logic
create_delivery_agent.py      Local delivery-agent account utility
create_admin.py                Local administrator account utility
requirements.txt               Python dependencies
globaltrack.db                 SQLite database created at runtime
templates/                     Jinja HTML pages
static/css/                    Dedicated CSS files
static/js/                     Frontend JavaScript
```

## Interview explanation

> GlobalTrack separates presentation, application logic, and data storage. Jinja templates handle the HTML structure, dedicated CSS files handle page presentation, JavaScript handles lightweight browser interactions, Flask handles HTTP requests and authorization, and SQLite stores users, shipments, tracking events, and notifications.

> Authentication uses Werkzeug password hashing and Flask sessions. Authorization is enforced on the server with role-based decorators, so a user cannot gain access simply by typing a protected URL. Shipment status changes are recorded as separate tracking-event records, which gives the customer a historical timeline rather than only the current status.

## Important development note

Profile settings and support currently contain frontend-only interactions. Their full database-backed workflows should be implemented before production deployment.

Security hardening such as CSRF protection, production secret management, rate limiting, email verification, audit logging, and stronger input validation should also be added before production use.
