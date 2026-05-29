# Target Architecture (TO‑BE)

*This document is a **brief summary** of the planned architecture after the rewrite. For detailed domain boundaries and specifications, see the [`DDD/`](./DDD) folder and the [`ROADMAP.md`](./ROADMAP.md).*

The target architecture **preserves all operational value** of the current system (offline‑first inventory, product catalogue, moderation) while moving to a scalable, maintainable, and extensible foundation.

---

## High‑Level Vision

Mobile (iPad) / Web Dashboard
│
▼
Explicit REST API (Node.js)
│
▼
PostgreSQL (Source of Truth)
│
▼
Async Workers (exports, sync, notifications)


- **Firebase is no longer the source of truth** – it may stay temporarily for authentication and push notifications (FCM), but all business logic moves to the new backend.
- **No implicit side effects** – every state change is triggered by an explicit API call.

---

## Core Components

### 1. Backend API (Node.js)
- **Language:** JavaScript (strict mode, no TypeScript – disciplined “use strict”).
- **Responsibilities:** All domain logic (Products, Inventory, Recipes, Procurement, Users, Notifications).
- **Offline sync:** Explicit endpoints for mobile to upload full inventory sessions atomically – no conflict resolution magic.
- **Authentication:** JWT (separate from Firebase, though Firebase Auth may be used initially).

### 2. PostgreSQL Database (Source of Truth)
- **Schema:** Explicit, managed via migrations.
- **Immutable history:** Snapshots stored directly in rows (e.g., `inventory_items` with frozen tare/volume).
- **Referential integrity:** Enforced at the application level (no polymorphic foreign keys, but careful use of `owner_type`+`owner_id` where needed).
- **JSONB** only for optional payloads (e.g., `payload` in `notification_records`).

### 3. Bounded Contexts (Domains)
The system is decomposed into independent domains as described in the DDD folder:
- Products, Inventory, Recipes, Users/Teams/Orgs, Procurement, Notifications, Platform.
- Each domain has its own clear API and data model, with explicit dependencies (e.g., Inventory references Product Variants by ID, but not the reverse).

### 4. Offline‑First Mobile (iPad)
- **Remains offline‑first** – the app stores counts locally when offline.
- **Sync:** On `Save & Send`, the client sends the **entire session as an atomic JSON document** to the backend. No incremental sync or two‑way real-time listeners.
- **Images** are lazy‑loaded; placeholders when offline.

### 5. Async Workers
- **Exports** (PDF, XLSX) – generated in the background.
- **Notifications** – email and push sent via workers (retry logic, audit).
- **Moderation** – queue processing (future).

### 6. Anti‑Corruption Layers (ACLs)
- For POS integrations (CompuCash, Poster, etc.) – each system gets its own adapter that maps external data to domain models.
- For delivery merchant apps (future) – separate bounded context.

---

## Key Principles

- **Explicit > implicit** – No database listeners, no hidden side effects.
- **Snapshots guarantee history** – Once data is written, it never changes.
- **Domains are decoupled** – They communicate via IDs and explicit API calls, not shared tables.
- **Offline is a feature, not a bug** – But sync is explicit and atomic.
- **Firebase is temporary** – Used only for auth/push during migration.

---

## Comparison with Current System

| Aspect            | Current (AS‑IS)                       | Target (TO‑BE)                            |
|-------------------|---------------------------------------|--------------------------------------------|
| Source of truth   | Firebase Realtime Database             | PostgreSQL                                 |
| Sync mechanism    | Firebase listeners (implicit)          | Explicit API, atomic session upload        |
| Business logic    | Scattered (PHP, Swift, Firebase rules) | Centralised in Node.js domain layer        |
| Inventory history | Mutable (can be overwritten)           | Immutable snapshots                        |
| Procurement       | External email form                    | Integrated domain with revisions           |
| Scalability       | Limited (Firebase constraints)         | Horizontally scalable (PostgreSQL + API)   |

---

## Migration Path

- **Phase 1** (current): PostgreSQL schema + Node.js API (products, inventory, users). Dual‑write or read from Firebase initially.
- **Phase 2**: Recipes & procurement (orders) fully on new stack.
- **Phase 3**: Retire Firebase writes; use only for auth/push fallback.
- **Phase 4**: Migrate auth to native JWT (optional).

*See [`ROADMAP.md`](./ROADMAP.md) for detailed timeline and milestones.*

---

## Out of Scope (Not in This Architecture Document)

- Specific cloud provider (we use Zone.ee).
- Exact API specification (REST, JSON).
- UI frameworks (web frontend will be rewritten separately, but not detailed here).

*For domain‑level details, refer to the DDD files in the [`DDD/`](./DDD) folder.*
