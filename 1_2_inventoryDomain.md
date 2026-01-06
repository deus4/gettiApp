# Inventory Domain — Domain Design Draft

## Purpose

The Inventory Domain defines how stock-taking (inventory counting) is performed,
stored, finalized, aggregated, and exported.

Its goals:
- support fast and accurate inventory sessions
- work reliably in offline / poor network conditions
- preserve historical data integrity
- allow aggregation across teams and locations
- integrate with POS systems (CompuCash, Poster, Micros)
- provide structured data for analytics and future automation

Inventory is a **time-based process**, not a static state.

---

## Core Concepts Overview

The Inventory Domain consists of the following core entities:

1. Inventory Session
2. Inventory Ownership (Operational Scope)
3. Inventory Item
4. Inventory Snapshot (historical reference)
5. Inventory Result / Report
6. Inventory Lifecycle & States

---

## 1. Inventory Session

### Definition

An Inventory Session represents a single stock-taking process
performed by a user (or team) within a specific ownership scope
and optional physical location.

Examples:
- Monthly bar inventory
- Weekly stock check
- Partial recount after delivery

### Characteristics
- Belongs to exactly one Ownership Scope
- May be associated with a specific Location
- Has a strict lifecycle
- References Product Variants (never Product Labels)

### Key Fields (conceptual)
- id
- owner_type (Team | Organization | Group)
- owner_id
- location_id (optional)
- started_at
- finished_at (nullable)
- status
- started_by_user_id

### Structure (Conceptual)
InventorySession
├── owner_type
├── owner_id
├── location_id (optional)
├── started_at
├── finished_at
├── status
└── items[]

## 2. Inventory Ownership (Operational Scope)

### Definition

Inventory Sessions belong to a single operational scope at creation time.
This scope defines who owns the inventory and how it may be aggregated.

### Current Supported Scopes
- Team
- Location (Bar / Tab)

### Future Supported Scopes (Domain-level)
- Organization
- Multi-Team Group
- Enterprise Account

Inventory Domain does **not** enforce access control rules.
Ownership is used strictly for aggregation, analytics, and hierarchy.

### Ownership Rules
- Inventory Session has exactly ONE owner
- Ownership is immutable after session creation
- Finished sessions may be aggregated across scopes

### Ownership Hierarchy (Conceptual)
Organization
├── Team
│    ├── Location / Bar
│    │    └── InventorySession
│    └── InventorySession
└── Team
└── Location
└── InventorySession

---

## 3. Inventory Item

### Definition

An Inventory Item represents the counted result of a single
Product Variant within a specific Inventory Session.

### Characteristics
- References exactly one Product Variant
- Exists only within one Inventory Session
- May be counted via weight or units
- Is immutable after session finalization

### Fields (conceptual)
- inventory_session_id
- product_variant_id
- counted_quantity
- unit (CL, L, PCS, KG, etc.)
- counted_weight (optional)
- calculation_method (weight / unit)
- created_at

### Aggregation Note

Inventory Items are always stored per Product Variant.
User-side grouping or summation across variants (e.g. different volumes)
does not alter stored data and is treated as a presentation-layer concern.

---

## 4. Previous Inventory & Difference Calculation

### Concept

Inventory is always relative to previous data.

For each Inventory Item, the system may calculate:

- **previous_quantity**
- **difference**
- **usage / consumption**

### Rules
- Previous values are taken from the last finished inventory
  for the same product variant within the same ownership scope
  and (if applicable) the same location
- Previous inventory data is snapshotted, never live-linked

---

## 5. Inventory Snapshot (Historical Integrity)

### Definition

A Snapshot represents immutable historical data used for comparisons
and reporting.

### Purpose
- prevent recalculation if product attributes change later
- preserve correct historical reporting
- guarantee auditability

### Snapshot includes:
- product_variant_id
- physical attributes used at the time
  - volume
  - tare weights
  - density (if applicable)
- calculated quantities

Snapshots are stored alongside finished inventory sessions.

---

## 6. Offline / Online Behavior

### Offline-first Principle

Inventory collection must work **without active internet connection**.

### Flow
1. On first login:
   - Download product catalog (without images)
2. During inventory:
   - All counts stored locally
3. On `Save & Send`:
   - Requires online connection
   - Uploads inventory data to backend
4. If offline:
   - Session remains local
   - User retries later

### Rules
- Images may be lazy-loaded
- Product data is cached
- Conflicts are resolved server-side (session-level, not item-level)

---

## 7. Finalization & Reports

### Finalization (`Finish`)

When user presses `Finish`:
- Timer stops
- Inventory status changes to `Finished`
- All calculations are frozen
- Session becomes read-only

### Reports

Reports are generated from frozen data:
- PDF
- XLSX
- CSV

Reports may include:
- counted quantity
- previous quantity
- difference
- usage
- price (if available from POS integration)

---

## 8. POS Integration (Conceptual)

Inventory Domain supports external POS systems.

### Integration Principles
- Inventory Session must exist in POS before pushing data
- Product mapping is done via external IDs
- Units must be normalized

### Example Systems
- CompuCash
- Poster
- Micros (future)

Inventory Domain does NOT own pricing logic,
only quantity, structure, and aggregation.

Inventory aggregation across multiple scopes
does not require POS support and is handled
entirely within the Inventory Domain.

---

## 9. Relation to Other Domains

- Products Domain:
  - Inventory references Product Variant IDs only
- Users, Teams & Organizations Domain:
  - Controls permissions, hierarchy, and ownership scopes
- Recipes Domain:
  - Inventory uses recipes for component breakdown
- Analytics Domain:
  - Aggregates inventory results over time and scopes

---

## Key Architectural Principles

> Inventory is a process, not a table  
> Historical data is immutable  
> Ownership is hierarchical by design  
> Aggregation is explicit and scope-aware  
> Product changes must never rewrite history  

This domain must remain stable and predictable even as
products, recipes, and integrations evolve.
