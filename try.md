## Role and Access Overview (for DFD)

### Roles

- **Admin**: Platform administration across all pharmacies; manages users, subscriptions, announcements; full Admin portal access.
- **Manager**: Manages a single pharmacy; can manage staff, inventory, POS, forecasting, reports, announcements, support.
- **Staff**: Operates in a single pharmacy; POS, inventory requests, own sales, waste & expiry, AI assistant.

### Where access is enforced

- **Frontend route gating**: `RequireAuth` only renders routes if `user.role` is allowed.

```text
frontend/src/App.js
- If not authenticated → Login
- If role not in allowed list → LandingPage
```

- **Backend authorization (source of truth)**:
  - Admin-only helpers: `_require_admin(...)` (abort 403 if not admin)
  - Manager/Admin checks: `require_manager_or_admin(...)` (abort 403 if not manager/admin)

```text
backend/routes/admin.py → _require_admin
backend/utils/helpers.py → require_manager_or_admin
```

- **Authentication**:
  - JWT issued at login; `/api/auth/me` returns `id, email, role, pharmacy_id` for RBAC and scoping.

### High-level DFD (Context)

- **External Entities**: Admin, Manager, Staff (web clients)
- **Processes**:
  - Auth Service (login, token, /me)
  - Admin Service (users, subscriptions, announcements)
  - Manager Service (staff, inventory, POS, forecasting, reports, announcements, support)
  - Staff Service (POS, inventory requests, own sales, waste & expiry)
  - Forecasting Service (model training/serving)
- **Data Stores**:
  - PostgreSQL (users with `role` and `pharmacy_id`, products, sales, subscriptions, tickets, etc.)
  - Model files under `backend/ai_models/` (saved forecasting models)
- **Data Flows**:
  - Client → Backend: credentials → JWT; subsequent API calls with JWT
  - Backend → DB: CRUD, reporting queries, sales reads
  - Backend → Forecasting: load best model, compute predictions → JSON to client

### Why this works (security & correctness)

- **Defense in depth**: Frontend hides unauthorized UI; backend enforces final authorization via RBAC helpers and JWT on protected endpoints.
- **Tenant scoping**: Most manager/staff actions validate `pharmacy_id` to prevent cross-tenant access.
- **Explicit roles**: `admin`, `manager`, `staff` are validated on user creation and used consistently across UI and API.

This section summarizes role behavior and flows so you can quickly build DFD diagrams without digging through code.
