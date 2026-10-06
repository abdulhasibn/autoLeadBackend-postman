# AutoLead Backend — Postman

Standalone Postman collection for the [autoLeadBackend](https://github.com/abdulhasibn/autoLeadBackend) Express API.
Share this repo with the team — no paid Postman plan required.

The source of truth is `postman/` in the backend repo, and this repo mirrors it. The contract for every route is in [`docs/api.md`](https://github.com/abdulhasibn/autoLeadBackend/blob/main/docs/api.md).

## Files

| File | Purpose |
| ---- | ------- |
| `AutoLead-API.postman_collection.json` | Health, Auth, Users, Owners, Catalog, Vehicles, Leads, Notifications, Dashboard |
| `AutoLead-Local.postman_environment.json` | Local `baseUrl` (`http://localhost:3000`) and sample variables |

## Collection layout

Every feature ships as its own **top-level folder**. Do not leave new feature requests at collection root.

| Folder | Routes |
| ------ | ------ |
| Health | `GET /health` (public) |
| Auth | `POST /auth/login`, `POST /auth/refresh`, `GET /auth/me` |
| Users | Staff CRUD under `/users` (Admin) |
| Owners | Owner CRUD under `/owners` (Admin or Salesperson; deactivate is Admin-only) |
| Catalog | `GET /catalog/makes`, `/makes/:makeId/models`, `/models/:modelId/variants` (Admin or Salesperson) |
| Vehicles | Create / list / get / update (list and get carry a signed `frontImageUrl`), status (`open` / `linked` / `dropped` / `sold`) + status history (with `changedByName`), signed photo and document uploads (documents take an optional `fileName`) (Admin or Salesperson; drop/re-list and deletes are Admin-only) |
| Leads | Walk-in create (optional catalog preference), set preference, associate vehicle (reads include a `linkedVehicle` summary), assign (Admin), follow-up, status (`new` / `not_now` / `booking_confirmed` / `converted` / `lost` / `vehicle_unavailable`), convert (sells the vehicle) (Admin or Salesperson; a salesperson sees only leads assigned to them) |
| Notifications | Own inbox (`follow_up_due`, `lead_assigned`) + mark read (Admin or Salesperson) |
| Dashboard | `GET /dashboard` — KPIs, attention lists, today's follow-ups (Admin) |

## Import

1. Clone this repo.
2. Open Postman → **Import** → select the collection and `AutoLead-Local.postman_environment.json`.
3. Select **AutoLead Local**.

## Smoke flow

`baseUrl` defaults to `http://localhost:3000`. The local environment ships with the bootstrapped admin `email` / `password` (`admin@example.com`). Login and Refresh store `accessToken` and `refreshToken`. Create Staff stores `staffUserId`. Create Owner stores `ownerId`. Catalog list requests store `makeId` / `modelId` / `variantId`. Create Vehicle stores `vehicleId`. Media and document upload/confirm requests store `mediaStoragePath` / `mediaId` and `documentStoragePath` / `documentId`. Create Lead stores `leadId`. List Notifications stores `notificationId`.

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
5. Vehicle + lead (Admin token; a salesperson can do every step except those marked Admin):
   1. **Catalog → List Makes** → **List Models** → **List Variants**
   2. **Vehicles → Create Vehicle** (`showroomId` is optional; defaults to your home showroom) → **List Vehicles** / **Get Vehicle** / **Update Vehicle**
   3. **Create Media Upload** → `PUT` the bytes to the returned `uploadUrl` → **Confirm Media** → **List Media** (same for documents)
   4. New vehicles start `open`. Linking a lead makes them `linked`, and converting a lead makes them `sold`. **Change Vehicle Status** (Admin) only drops or re-lists: dropping a vehicle with active leads first returns `422 VEHICLE_HAS_LINKED_LEADS` with `error.details.linkedLeadCount`; resend with `confirmUnlinkLeads: true`. **List Vehicle Status History** shows each step.
   5. **Leads → Create Lead** (uses `modelId` as the preferred model) → **Set Lead Preference** (optional; uses `variantId`) → **Associate Vehicle** → **Assign Lead** (Admin; uses `staffUserId`) → **Schedule Follow-up** → **Change Lead Status**
   6. Move the lead to `booking_confirmed` (needs a vehicle), then **Convert Lead**. It sells the vehicle to this lead and moves the other active leads on that vehicle to `vehicle_unavailable`.
   7. **Notifications → List Notifications** → **Mark Notification Read**. The assignee sees `lead_assigned` right away and `follow_up_due` at the scheduled time.
   8. **Dashboard → Get Dashboard** (Admin).

Shared error envelope:

```json
{ "error": { "code": "STRING_CODE", "message": "Human-readable message" } }
```

Pagination on list routes: `limit` (default 20, max 100), `offset` (default 0). Page shape: `{ items, total, limit, offset }`.

## Safety

Do not commit real access tokens or refresh tokens. Keep secret values in your local Postman environment only. The committed password is the local bootstrap admin password, not a production secret.
