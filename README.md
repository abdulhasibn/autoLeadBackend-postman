# AutoLead Backend — Postman

Standalone Postman collection for the [autoLeadBackend](https://github.com/abdulhasibn/autoLeadBackend) Express API.
Share this repo with the team — no paid Postman plan required.

## Files

| File | Purpose |
| ---- | ------- |
| `AutoLead-API.postman_collection.json` | Health + Auth + Users + Owners |
| `AutoLead-Local.postman_environment.json` | Local `baseUrl` (`http://localhost:3000`) and sample variables |

## Collection layout

Every feature ships as its own **top-level folder**. Do not leave new feature requests at collection root.

| Folder | Routes |
| ------ | ------ |
| Health | `GET /health` (public) |
| Auth | `POST /auth/login`, `POST /auth/refresh`, `GET /auth/me` |
| Users | Staff CRUD under `/users` (Admin) |
| Owners | Owner CRUD under `/owners` (Admin or Salesperson; deactivate is Admin-only) |

## Import

1. Clone this repo.
2. Open Postman → **Import** → select the collection and `AutoLead-Local.postman_environment.json`.
3. Select **AutoLead Local**.

## Smoke flow

`baseUrl` defaults to `http://localhost:3000`. The local environment ships with the bootstrapped admin `email` / `password` (`admin@example.com`). Login and Refresh store `accessToken` and `refreshToken`. Create Staff stores `staffUserId`. Create Owner stores `ownerId`.

1. **Health → Health**
2. **Auth → Login** → **Me** (optional **Refresh** rotates the session)
3. Staff (Admin token):
   1. **Users → Create Staff** (stores `staffUserId`; default role `salesperson`)
   2. **List Staff** / **Get Staff** / **Update Staff**
   3. **Replace Staff Roles** (`admin` or `salesperson`; cannot drop the last admin)
   4. **Deactivate Staff** (cannot deactivate yourself or the last admin)
4. Owners (Admin or Salesperson; deactivate needs Admin):
   1. **Owners → Create Owner** (stores `ownerId`; live phone must be unique)
   2. **List Owners** / **Get Owner** / **Update Owner** → **Deactivate Owner**

Shared error envelope:

```json
{ "error": { "code": "STRING_CODE", "message": "Human-readable message" } }
```

Pagination on list routes: `limit` (default 20, max 100), `offset` (default 0). Page shape: `{ items, total, limit, offset }`.

## Safety

Do not commit real access tokens or refresh tokens. Keep secret values in your local Postman environment only. The committed password is the local bootstrap admin password, not a production secret.
