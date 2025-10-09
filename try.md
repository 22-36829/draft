### Inventory Management – Detailed Functional/Flow Summary

Below is a comprehensive map of the features, data flows, and interaction patterns across the Inventory Management experience. Use this to study end‑to‑end behavior quickly.

### 1) Key UI Areas

- Inventory tab
  - Table columns: Product, Category, Location, Unit Price, Current Stock.
  - Actions per product: Deliveries (modal), Edit, Deactivate/Reactivate, Hard Delete (admin).
  - Header actions: Reports modal, Add Product modal.
  - Search/filter: Keyword (product/category/location), Category dropdown, Items per page.
  - Pagination: Unified compact control (chevrons + 1–5 page buttons + “Go to page” input).

- Low Stock / Reorder tab
  - Filters: Keyword, Category, “Only low stock” toggle.
  - Derived table: Product, Category, Current Stock, editable “Order Qty” (suggested qty is prefilled).
  - Actions: Remove item from plan, Clear, Print (clean tabular report), Export CSV (Product, Category, Current Stock, Order Qty).

- Requests tab
  - Filters: Keyword, Staff, Date From, Date To, Status (always visible).
  - Table: Product, Qty Change, Type, Status, Requested By, Created, Actions (Approve/Reject).
  - Actions: Clear filters, Print filtered requests.

- Returns tab
  - Filters: Keyword, Staff, Date From, Date To (always visible).
  - Cards: Return #, Sale #, item count, total refund, reason, created at, itemized products.
  - “Updated by … • timestamp” shown if backend provides `updated_by`/`updated_at`.
  - Actions: Delete Return (manager/admin), Clear filters, Print filtered returns.

- Deliveries modal (from Inventory product row)
  - Search within deliveries (supplier/date/qty/cost), inline editing with keyboard shortcuts.
  - Actions: Add/Edit/Delete deliveries; auto‑create supplier by name if not found.
  - Pagination: Unified compact control + per‑page selector.

- Reports modal
  - Report types: Current Stock, Expired Products, Sales by Staff, Sales by Period.
  - Parameters: Staff, Period (day/week/month), Date ranges, Status/Category (for expired/stock).
  - Unified pagination and print layouts (specialized for Sales by Period and Expired).

### 2) Data Model & Important Fields (frontend expectations)

- Products: `id`, `name`, `category_name`, `location`, `unit_price`, `current_stock`.
- Deliveries (batches): `id`, `quantity`, `expiration_date`, `delivery_date`, `supplier_name`, `cost_price`.
- Requests: `id`, `product_name`, `quantity_change`, `trans_type`, `status`, `requested_by`/`requested_by_email`, `created_at`.
- Returns: `id`, `return_number`, `sale_number`, `item_count`, `total_refund_amount`, `reason`, `created_at`, `items[]`, optional `updated_by`, `updated_at`.
- Staff: `id`, `first_name`, `last_name` (list filtered server‑side by manager’s pharmacy).
- Reports: typed payloads (e.g., for expired, sales period) with shape aligned to table/print renderers.

### 3) State, Derivations, and Memoization

- Top-level state: products, categories, requests, returns, deliveries list for selected product, staff list, filters, pagination cursors.
- Derived lists (via `useMemo`):
  - `filteredProducts`: keyword + category.
  - `lowStockList`: from `filteredProducts` + low‑stock logic; attaches suggested qty.
  - `filteredRequests` / `filteredReturns`: keyword + staff + date range (+ status for requests).
- Pagination derivations:
  - Compute totals, pages, current page clamp, slice window.
  - Reused compact component pattern everywhere for consistency.

### 4) Core Flows (step‑by‑step)

- Add/Edit/Delete Deliveries
  1) Open modal on product row → fetch deliveries for `product.id`.
  2) Add or inline‑edit fields; supplier lookup/auto‑create by name.
  3) Submit → backend updates; reload deliveries + products; success toast.
  4) Inline edit saves on Ctrl+Enter; Esc cancels.

- Manage Products
  1) Edit → name/price/location/category (create category if needed).
  2) Deactivate/Reactivate (soft) or Hard Delete (admin).
  3) Table refreshes on success; unified toasts.

- Requests Approval
  1) Filter → Approve/Reject visible for manager/admin if status `pending`.
  2) Action call → reload requests (and products for stock adjustments if relevant).
  3) Optional: Print filtered requests for audit.

- Returns Handling
  1) Filter → view cards. “Delete Return” shown for manager/admin.
  2) Delete → confirm → call delete endpoint → reload returns.
  3) Optional: Print filtered returns; “Updated by …” if backend provides.

- Reporting
  1) Open reports modal → choose report type → set parameters.
  2) Generate → backend returns typed rows; unified pagination; print with professional header.
  3) Sales by Period: period header with quantities/totals; Expired: suppliers, expiration range, estimated loss.

- Low Stock / Reorder Planning
  1) Filter: keyword/category/toggle.
  2) `lowStockList` builds with suggested order qty = `max(0, reorderPoint*2 - currentStock)`; you can override the suggestion.
  3) Print: Product, Category, Current Stock, Order Qty (clean table).
  4) CSV: same columns; downloaded via a safe in‑browser generator.

### 5) Business Rules and Data Integrity

- Current Stock excludes expired batches (backend logic).
- Currency formatting: ₱ with two decimals across UI & print.
- Expired report “Estimated Loss”:
  - Aggregates only priced deliveries: sum(cost_price × quantity).
  - Shows “—” if no priced entries found.
- Sales computations:
  - Period revenue fixed to sum of line items (not duplicated by joins).
  - Staff metrics: totals/averages (daily/weekly/monthly) computed server‑side, rounded to 2 decimals.
- Print headers:
  - Always show date/time, report title, “Requested by: First Last”.

### 6) Error Handling, Safety, and UX

- Errors
  - Modal‑scoped banners for Deliveries/Reports; global error hidden while modals open.
  - Guarded number formatting: `Number(value || 0).toFixed(2)` prevents `toFixed()` runtime errors.
- Keyboard
  - Ctrl+K search, Ctrl+P reports, Ctrl+F deliveries search, Ctrl+Enter save inline edit, Esc close/cancel.
- Responsiveness
  - Filter bars are always visible; compact (text‑sm), responsive grid 1/2/6 to avoid wrap glitches.
  - Unified pagination improves navigation and reduces clutter.

### 7) Printing and CSV Exports

- Print (Reorder):
  - Minimal professional layout; Product, Category, Current Stock, Order Qty with aligned numeric columns.
- CSV (Reorder):
  - Safe generator with escaping; filename `reorder_YYYY-MM-DD.csv`.
- Reports/Expired/Sales Period:
  - Tailored print with headings, subtle color blocks, totals; includes date range and requester.

### 8) Security / Roles

- Manager/Admin:
  - Approve/Reject Requests, Delete Returns, product lifecycle actions (deactivate/reactivate/hard delete).
- Staff:
  - Read‑only portions as suitable; all server routes still enforce authorization.

### 9) Extensibility / Next Steps

- Persist reorder “drafts” with user-entered quantities for later export/submit.
- CSV/Excel export for Requests/Returns/Inventory lists with filters applied.
- Email/PDF export of prints.
- Frontend filter of staff by pharmacy (backend already scopes list; we can further limit UI list if needed).

### 10) File and Important Spots

- Main UI: `frontend/src/pages/manager/InventoryManagement.js`
  - Tabs and filter bars, tables, unified pagination, modals, print/export functions, delete return, auto‑create supplier, inline delivery editing.
- APIs consumed: `POSAPI`, `InventoryAPI`, `ManagerAPI` (context‑aware calls tied to manager’s pharmacy).

This document mirrors the concrete UI behavior and the underlying data logic so you can study the flow from inputs → state → derived lists → actions → persistence → printing/exports with confidence.
