# LDAP Interactive Dashboard

An interactive LDAP administration dashboard built for **Siddaganga Institute of Technology (SIT), Tumkur**. Browse your directory tree, search users in real time, manage accounts from a React admin UI, and handle student account requests through a separate portal — all backed by a Flask API over OpenLDAP or Active Directory.

> [!WARNING]
> **This is a public showcase repository. The production codebase, deployment configuration, and institution-specific settings live in a private repository and are not published here.**

## Problem

College IT teams need more than command-line LDAP tools. Managing hundreds of directory accounts means navigating OUs, finding users quickly, reviewing access requests, and applying changes — often across a mix of manual steps and slow, one-at-a-time updates.

## Solution

| Component | Role |
|-----------|------|
| **Admin Dashboard** (React + Vite) | Directory overview, user search, account management, request review |
| **Student Portal** (React + Vite) | Self-service account request submission |
| **Flask API** | Directory tree, search, user operations, authentication, request queue |

## Highlights

- **Interactive directory tree** — explore domains, OUs, containers, and users in a hierarchical overview
- **Global user search** — fast server-side lookup across identity attributes (`uid`, `sAMAccountName`, `cn`, `mail`, and more)
- **Unified admin dashboard** — manage users, review pending requests, and run batch operations from one UI
- **Student request workflow** — submit → admin review → approve creates LDAP user or deny clears the queue
- **Parallel worker engine** — apply many user changes at once when batch updates are needed
- **OpenLDAP + Active Directory** — auto-detects server capabilities; AD user creation uses secure channels (`LDAPS` / StartTLS + `unicodePwd`)
- **CSV/JSON import** — upload user lists for batch changes from the dashboard

## Architecture

```text
┌─────────────────────┐     ┌─────────────────┐
│  Admin Dashboard    │     │ Student Portal  │
│  (React/Vite)       │     │ (React/Vite)    │
└──────────┬──────────┘     └────────┬────────┘
           │                         │
           └────────────┬────────────┘
                        │ REST API
                 ┌──────▼──────┐
                 │ Flask API   │
                 │             │
                 │ ┌─────────┐ │
                 │ │ SQLite  │ │  ← account request queue
                 │ └─────────┘ │
                 └──────┬──────┘
                        │ LDAP/LDAPS
                 ┌──────▼──────┐
                 │ OpenLDAP /  │
                 │ Active Dir. │
                 └─────────────┘
```

## Tech Stack

- **Backend:** Python, Flask, LDAP3
- **Frontend:** React, Vite, TypeScript
- **Data:** SQLite (request queue)
- **Auth:** LDAP bind against directory

## API Overview

### Directory & search

- `GET /ldap-directory-tree` — hierarchical domain tree for the dashboard overview
- `GET /ldap-users` — flat user list (optional `baseDn` scope)
- `GET /ldap-users/search?q=<query>` — fast identity search

### User operations

`POST /bulk-update?workers=N`

Supports `create_user`, `delete_user`, `set_email`, `set_password`, and related operations. Accepts a JSON list or `{ "users": [...] }` body.

### Account requests

- `POST /account-requests` — student submission (no login required)
- `GET /account-requests?status=pending` — admin queue
- `POST /account-requests/<id>/review` — approve or deny

## Project Structure (private repo)

```text
src/ldap_bulk_api/
  api/routes.py
  core/config.py
  services/ldap_service.py
  services/auth_service.py
  services/request_service.py
frontend/          # Admin dashboard
student-portal/  # Student request form
app.py
requirements.txt
```

## Local Development (private repo)

The runnable application lives in the private repository. Typical setup:

```bash
pip install -r requirements.txt
cp .env.example .env   # configure LDAP connection (never commit secrets)
python app.py
```

Admin dashboard and student portal each use `npm install && npm run dev` with `VITE_API_BASE_URL` pointing at the API.

## Security & Privacy

- No credentials, `.env` files, or production hostnames are included in this public repo
- AD password writes require a secure LDAP channel
- Request queue data stays on the application host (`data/requests.db` by default)

## Status

Developed as a real-world institutional IT project at **SIT Tumkur**. This repository documents the design and capabilities for portfolio purposes.

## License

[MIT](LICENSE)
