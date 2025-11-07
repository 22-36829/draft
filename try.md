## Role and Access Overview (for DFD)

### Roles

- **Admin**: Platform administration across all pharmacies; manages users, subscriptions, announcements; full Admin portal access.
- **Manager**: Manages a single pharmacy; can manage staff, inventory, POS, forecasting, reports, announcements, support.
- **Staff**: Operates in a single pharmacy; POS, inventory requests, own sales, waste & expiry, AI assistant.

### Role Responsibilities (what each role does)

- **Admin**
  - Create, activate/deactivate, and assign roles to user accounts across pharmacies
  - Manage pharmacy profiles and subscription lifecycle (plan, status, payments)
  - Publish platform announcements and review platform-level dashboards/metrics

- **Manager**
  - Create and manage staff accounts within their pharmacy
  - Oversee inventory (stock updates, approvals), run POS operations, and reconcile sales
  - Generate and review reports (sales, period, detailed), use forecasting to plan stock
  - Handle pharmacy announcements and support tickets

- **Staff**
  - Operate POS to record sales and issue receipts
  - Submit inventory adjustment requests (e.g., corrections, shrinkage, waste/expiry)
  - Review own sales history and contribute operational data to reports/forecasts
  - Use the AI assistant for recommendations/information

### Role–System Interactions (how each role uses the system)

- **Admin → System**
  - Authenticates and obtains a session token
  - Accesses Admin portal to:
    - Manage Users: create/update users, set role, (cross-pharmacy) → updates Users data store
    - Manage Subscriptions: plans, status, payments → updates Subscriptions/Pharmacies data stores
    - Announcements: create/publish → updates Announcements data store, notifies clients
  - Views platform metrics → aggregated reads across Users/Pharmacies/Sales

- **Manager → System**
  - Authenticates and obtains a session token scoped to their pharmacy
  - Staff Management: create/list/update/deactivate staff → updates Users (within pharmacy)
  - Inventory: create/approve adjustments; view stock/low stock → reads/writes Inventory and Products
  - POS: conduct transactions → writes Sales, updates Inventory
  - Forecasting: request forecasts for products/categories → reads historical Sales, loads model files, returns forecast series
  - Reporting: run date/period/detailed reports → reads Sales/Products with pharmacy scope
  - Announcements & Support: create/read → writes/reads Announcements and Tickets

- **Staff → System**
  - Authenticates and obtains a session token scoped to their pharmacy
  - POS Operations: record sales → writes Sales; triggers inventory decrements
  - Inventory Requests: submit adjustments for approval → writes Adjustment Requests pending manager review
  - Own Sales & Waste/Expiry: view/history submission → reads Sales; writes Waste/Expiry records if applicable
  - AI Assistant: request recommendations/info → reads supported knowledge/services; returns guidance

### Where access is enforced (conceptual, no code)

- Client application shows only the sections allowed by the user’s role
- Server validates the user’s role and pharmacy scope on every protected request
- Data operations are constrained by `role` and `pharmacy_id` to prevent cross-tenant access

### Data flow touchpoints (for DFD diagrams)

- External Entities: Admin, Manager, Staff (browsers)
- Processes: Auth, Admin Services, Manager Services, Staff Services, Forecasting Service
- Data Stores: Users, Pharmacies, Products, Inventory, Sales, Subscriptions, Announcements, Tickets, Model Files
- Flows:
  - Credentials → Auth → Session Token
  - Role-scoped requests → Services → Reads/Writes to data stores
  - Forecast requests → Forecasting Service → Model Files + Sales history → Forecast output to client

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
