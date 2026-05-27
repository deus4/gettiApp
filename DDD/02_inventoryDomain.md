# Inventory Domain — Domain Design Draft

## Purpose

The Inventory Domain defines how stock‑taking (inventory counting) is performed, stored, finalized, aggregated, and exported.

Its goals:
- support fast and accurate inventory sessions
- work reliably in offline / poor network conditions
- preserve historical data integrity
- allow aggregation across teams and locations
- integrate with POS systems (CompuCash, Poster, Micros)
- provide structured data for analytics and future automation

Inventory is a **time‑based process**, not a static stock state.

**This is the primary paid feature of the system.**

---

## Core Concepts Overview

The Inventory Domain consists of:

1. Inventory Session
2. Inventory Ownership (operational scope)
3. Inventory Item (with snapshot)
4. Inventory Lifecycle & States
5. Offline / Online behavior
6. Finalization & Reports
7. POS integrations

---

## 1. Inventory Session

### Definition

An Inventory Session represents a single stock‑taking process performed by a user (or team) within a specific ownership scope and, optionally, a physical location.

**Examples:**
- monthly bar inventory
- weekly stock check
- partial recount after delivery

### Characteristics
- Belongs to exactly one **Team** (ownership scope)
- May be associated with a specific **Location**
- Has a strict lifecycle
- References **Product Variants only** (never Product Labels)

### Parallel Sessions
A Team may have multiple active sessions simultaneously, **but only for different Locations**. This is enforced at the application layer (not a database constraint) because `location_id` is nullable.

### Key Fields
- `id` (UUID v4)
- `team_id`
- `location_id` (optional)
- `status` (active, pending_finalization, finished, cancelled)
- `started_at`
- `finished_at` (nullable)
- `started_by` (user_id)
- `finished_by` (user_id, nullable)
- `pos_system`, `pos_session_id` (for POS integrations)
- `reminder_sent_at` (for pending session reminders)

---

## 2. Inventory Ownership

### Definition
Each Inventory Session belongs to exactly one **Team**. Ownership determines:
- who owns the data
- how data may be aggregated

### Rules
- Ownership is immutable after creation.
- Ownership is **not linked to a specific user** – data belongs to the Team.
- `created_by_user_id` is an audit field; removing a user from the team does not affect the data.

### Aggregation Hierarchy

Organization
├── Team
│ ├── Location
│ │ └── InventorySession
│ └── InventorySession (without location)
└── Team
└── InventorySession


**Note:** Unlike other domains (Recipes, Procurement), Inventory uses a direct `team_id` foreign key, not a polymorphic owner. This is a conscious decision because inventory is always tied to an operational unit.

---

## 3. Inventory Item

### Definition
An Inventory Item represents the counted result of a single Product Variant within a specific Inventory Session.

### Characteristics
- References exactly one Product Variant
- Has its own UUID (not a composite key)
- May be counted by units or by weight
- Contains a **snapshot of physical parameters** at the time of counting
- Immutable after session finalization

### Snapshot (stored directly in the item row)
At each count, the following are frozen:
- `snapshot_full_tare_g` – full bottle weight
- `snapshot_empty_tare_g` – empty bottle weight
- `snapshot_volume_ml` – volume

These three numbers are stored in the `inventory_items` row itself, not in a separate table.  
**Why:** If a moderator later changes the product in the global catalog (e.g., a new bottle weight), historical records are **not** recalculated. A report from January shows exactly the same numbers as when it was created.

### Continuation (`is_continuation`)
If a manager asks a bartender to continue a session after `Save & Send`, new items receive:
- `is_continuation = TRUE`
- `continuation_at` – timestamp of addition

In the UI, these items are marked separately. The user sees an alert when opening the session: “Inventory will continue. After finishing, press Save & Send again.”

### Key Fields
- `id` (UUID v4)
- `session_id`
- `product_variant_id`
- `counted_quantity`
- `unit` (ml, cl, l, pcs, kg, g)
- `calculation_method` (weight | unit)
- `raw_weight_g` (optional, for audit)
- `snapshot_full_tare_g`
- `snapshot_empty_tare_g`
- `snapshot_volume_ml`
- `is_continuation`
- `continuation_at`
- `counted_by` (user_id)
- `created_at`

---

## 4. Inventory Lifecycle & States

active → pending_finalization → finished
↘ cancelled


- **`active`** – inventory in progress on iPad. Data stored locally on the device.
- **`pending_finalization`** – `Save & Send` has been sent, data received by the server, waiting for `Finish` in the web dashboard. This state may last indefinitely (conscious decision). A manager can return to the session, add items via web, or ask a staff member to continue on iPad.
- **`finished`** – finalized via the `Finish` button in the web dashboard. All calculations frozen. Read‑only forever.
- **`cancelled`** – manually cancelled.

### Notifications for pending sessions
- 24 hours after `Save & Send` without `Finish` → first push + email to the manager.
- 48 hours → second and final reminder.
- `reminder_sent_at` records the time of the first reminder.

**Important:** `Save & Send` is data submission, not business acceptance. Only a session with status `finished` is considered final.

---

## 5. Offline / Online Behavior

### Offline‑first Principle
Inventory collection must work **without an active internet connection**.

### Flow (AS‑IS on iPad)
1. On first login – download product catalog (without images; images load online, placeholders when offline).
2. During inventory – all data stored locally on iPad.
3. On `Save & Send` – requires internet. The client sends the **entire session as an atomic document**, not incrementally.
4. After initial review (by manager), the session may require corrections or additional items. In that case, the session is marked as needing revision. After changes, the user presses `Save & Send` again.
5. If no internet – session stays local, UI shows offline status, user retries later.

### Conflict resolution
There are no conflicts by design – the client sends the full session as one atomic document. The server accepts or rejects it entirely.

### Images
Lazy‑loaded online. Offline → placeholder. Do not block inventory.

---

## 6. Finalization & Reports

### Finalization (`Finish` in web dashboard)
When `Finish` is clicked:
- timer stops
- `finished_at` is recorded
- status changes to `finished`
- all calculations frozen
- session becomes read‑only forever

### Difference calculation
For each Inventory Item, the system calculates:
- `previous_quantity` – from the last finished session of the same Variant in the same Team + Location scope
- `difference` – current minus previous
- `usage / consumption`

Previous data is taken from snapshots, not from live entities.

### Report formats
- PDF
- XLSX
- CSV
- Email with attachment (not inline text)

No automatic scheduled exports – user initiates export manually.

---

## 7. POS Integrations

### Principles
- Each system (CompuCash, Poster, Micros) has its own **Anti‑Corruption Layer (adapter)**.
- Mapping `product_variant_id` ↔ external ID is stored in the adapter, not in the domain model.
- The Inventory Session must already exist in the POS before pushing data (for CompuCash).
- Units are normalized in the adapter.
- The Inventory Domain **does not own pricing**.

### Current status
- CompuCash – partial legacy integration exists, will be rewritten on the new stack.
- Poster, Micros – future development.

### Future Extended Integrations (TO‑BE)
- Creating new products in external systems (POS, ERP) based on validated catalog data.
- Using the built‑in barcode scanner as a product data input source.
- Syncing Product Labels and Variants to third‑party systems.

However, product data remains governed by the Products Domain and its moderation process. The Inventory Domain is **not** the source of truth for product definitions.

---

## 8. Authorization

Access checks are in the **application layer (middleware)**, not in the domain.
- JWT contains `user_id`.
- Middleware reads `team_memberships` and verifies the user’s role for that Team.
- Repositories receive `team_id` explicitly – they do not filter by user themselves.

A user may view an Inventory Session if:
- they are a member of the Team that owns the session
- their role has view permission (Owner, Manager, Inventory Manager)

---

## 9. Relations to Other Domains

- **Products Domain** – references only `product_variant_id`; synchronous existence check when creating items.
- **Recipes Domain** – breakdown of recipe components into items (planned for near future).
- **Users / Teams / Organizations Domain** – ownership and access rights.
- **Procurement Domain** – a finished session may be the source for creating an order.
- **Notifications Domain** – triggers reminders for pending sessions.
- **Platform Domain** – logs from iPad.

---

## Key Architectural Principles

> Inventory is a process, not a table.  
> Historical data is immutable.  
> The snapshot is stored in the item row, not a separate table.  
> Data belongs to the Team, not the user.  
> `Save & Send` ≠ finalisation.  
> Offline is a baseline requirement, not an option.  
> No conflicts – client sends the full session atomically.

This domain must remain stable and predictable even as products, recipes, and integrations evolve.