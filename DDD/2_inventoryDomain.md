# Inventory Domain — Domain Design Draft

## Purpose

The Inventory Domain defines how stock-taking
is performed, stored, finalized,
aggregated, and exported.

Its goals:
- support fast and accurate inventory sessions
- work reliably in offline / poor network conditions
- preserve historical data integrity
- allow aggregation across teams and locations
- integrate with POS systems
  (CompuCash, Poster, Micros)
- provide structured data for analytics
  and future automation

Inventory is a **time-based process**,
not a static stock state.

---

## Core Concepts Overview

The Inventory Domain consists of:

1. Inventory Session
2. Inventory Ownership (Operational Scope)
3. Inventory Item
4. Inventory Snapshot
5. Inventory Result / Report
6. Inventory Lifecycle & States

---

## 1. Inventory Session

### Definition

An Inventory Session represents
a single stock-taking process
performed by a user or team
within a specific ownership scope
and an optional physical location.

Examples:
- monthly bar inventory
- weekly stock check
- partial recount after delivery

### Characteristics
- Belongs to exactly one ownership scope
- May be associated with a Location
- Has a strict lifecycle
- References **Product Variants only**
  (never Product Labels)

### Key Fields (conceptual)
- id
- owner_type (Team | Organization)
- owner_id
- location_id (optional)
- started_at
- finished_at (nullable)
- status
- started_by_user_id

### Structure (conceptual)

InventorySession  
├── owner_type  
├── owner_id  
├── location_id (optional)  
├── started_at  
├── finished_at  
├── status  
└── items[]  

---

## 2. Inventory Ownership (Operational Scope)

### Definition

At creation time,
an Inventory Session belongs
to exactly one operational scope.
This scope defines:
- who owns the inventory
- how data may be aggregated

### Supported Scopes (AS-IS)
- Team
- Team + Location

### Future Scopes (TO-BE)
- Organization
- Multi-Team Group
- Enterprise Account

The Inventory Domain does **not**
enforce access control.
Ownership is used strictly for:
- aggregation
- analytics
- data hierarchy

### Ownership Rules
- An Inventory Session has exactly one owner
- Ownership is immutable after creation
- Finished sessions may be aggregated
  across higher scopes

### Ownership Hierarchy (conceptual)

Organization  
├── Team  
│    ├── Location  
│    │    └── InventorySession  
│    └── InventorySession  
└── Team  
     └── InventorySession  

---

## 3. Inventory Item

### Definition

An Inventory Item represents
the counted result of a single
Product Variant within one
Inventory Session.

### Characteristics
- References exactly one Product Variant
- Exists only within one Inventory Session
- May be counted by:
  - units
  - weight
- Immutable after session finalization

### Fields (conceptual)
- inventory_session_id
- product_variant_id
- counted_quantity
- unit (CL, L, PCS, KG, etc.)
- counted_weight (optional)
- calculation_method (weight / unit)
- created_at

### Important Note

User-side summation across variants
(e.g. multiple volumes of the same product)
is a **presentation-layer concern**.

Stored data always remains
at the Product Variant level.

---

## 4. Previous Inventory & Difference Calculation

### Concept

Inventory is always relative
to previous data.

For each Inventory Item,
the system may calculate:
- previous_quantity
- difference
- usage / consumption

### Rules
- Previous values are taken from
  the last finished inventory
  of the same Product Variant
  within the same ownership scope
  and (if applicable) the same Location
- Previous inventory data is snapshotted,
  never live-linked

---

## 5. Inventory Snapshot (Historical Integrity)

### Definition

A Snapshot represents immutable
historical reference data
used for comparison and reporting.

### Purpose
- prevent recalculation when products change later
- preserve correct historical reporting
- ensure auditability

### Snapshot includes:
- product_variant_id
- physical attributes valid at the time:
  - volume
  - tare weight
  - density (if applicable)
- calculated quantities

Snapshots are stored alongside
finished Inventory Sessions.

---

## 6. Offline / Online Behavior

### Offline-first Principle

Inventory collection must work
**without an active internet connection**.

### Flow
1. On first login:
   - download product catalog (without images)
2. During inventory:
   - all data stored locally
3. On `Save & Send`:
   - an active internet connection is required
   - inventory data is uploaded to the backend
   - after initial data review
     (by a manager or responsible person),
     additional actions may be required:
       - data correction
       - adding missing items
   - in this case, the Inventory Session
     is marked as requiring revision
   - after changes are made,
     the user must press `Save & Send` again
4. If offline:
   - session remains local
   - user retries later

Important:
`Save & Send` represents data submission,
not business acceptance.

An Inventory Session is considered
accepted only after passing review
without requiring further revisions.

### Rules
- Images are lazy-loaded
- Product data is cached
- Conflicts are resolved
  at the session level,
  not item level

---

## 7. Finalization & Reports

### Finalization (`Finish`)

When the user presses `Finish`:
- timer stops
- status becomes `Finished`
- all calculations are frozen
- session becomes read-only

### Reports

Reports are generated
from frozen data only:
- PDF
- XLSX
- CSV

Reports may include:
- counted quantity
- previous quantity
- difference
- usage
- price (if available via POS)

---

## 8. POS Integration (Conceptual)

The Inventory Domain supports
external POS systems.

### Integration Principles
- Inventory Session must exist in POS
  before pushing data
- Product mapping uses external IDs
- Units are normalized

### TO-BE: Extended Integrations

In future phases, integrations with external systems
may include not only inventory data exchange,
but also product catalog synchronization.

This includes:
- creating new products in external systems
  (POS, ERP, etc.)
- using the built-in barcode scanner
  as a product data input source
- creating Product Labels and Product Variants
  in third-party systems based on data
  validated within the Products Domain

Product data remains governed by the Products Domain
and its moderation processes.
The Inventory Domain is not the source of truth
for product definitions.

### Example Systems
- CompuCash
- Poster
- Micros (future)

The Inventory Domain does **not**
own pricing logic —
only quantity, structure, and aggregation.

Aggregation across scopes
is handled entirely
inside the Inventory Domain.

---

## 9. Relation to Other Domains

- Products Domain:
  - Inventory references **Product Variant IDs only**
- Users / Teams / Organizations Domain:
  - controls permissions and hierarchy
- Recipes Domain:
  - used for component breakdown
- Analytics Domain:
  - aggregates inventory results over time and scopes

---

## Key Architectural Principles

> Inventory is a process, not a table  
> Historical data is immutable  
> Ownership defines aggregation, not access  
> Aggregation is explicit and scope-aware  
> Product changes must never rewrite history  

The Inventory Domain must remain
stable and predictable
even as products, recipes,
and integrations evolve.