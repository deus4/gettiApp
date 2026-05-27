# Recipes Domain — Domain Design Draft

## Purpose

The Recipes Domain defines how composite products (recipes) are created, stored, versioned, and used across the system.

Its goals:
- support cocktails, pre‑batches, and kitchen preparations
- enable automatic inventory breakdown into components (planned)
- preserve historical correctness
- allow reuse across teams and locations

This domain intentionally avoids business models and monetisation.  
It focuses on **structure, correctness, and extensibility**.

**Current status:** Recipes are **not yet used in inventory** due to legacy limitations. The design below describes the **target state (TO‑BE)** that will be implemented in Phase 2 of the roadmap.

---

## Core Concepts Overview

The Recipes Domain consists of:

1. Recipe  
2. Recipe Version  
3. Recipe Ingredient  
4. Ownership, Author & Rights Holder  
5. Recipe Transfer (history of ownership changes)  
6. Usage in Inventory (planned)

---

## 1. Recipe

### Definition

A Recipe represents a conceptual composition created by a user.  
It is an abstract entity that may evolve over time (via versions).

**Examples:** Negroni, Espresso Martini (House Style), Gin Basil Smash Pre‑Batch.

### Characteristics
- Has an **author** (User) – who created it (immutable).
- Belongs to an **ownership scope** – who may use it operationally.
- Has a **rights holder** (User, nullable) – who receives royalties in the future creator economy.
- Has multiple **versions**.
- Is **not directly inventory‑countable** – inventory works with Recipe Versions.

### Key Fields
- `id` (UUID)
- `name`
- `author_user_id` (immutable)
- `owner_type` (user | team | organization)
- `owner_id`
- `rights_holder_user_id` (nullable, for future monetisation)
- `is_archived` (boolean)
- `created_at`

---

## 2. Recipe Version

### Definition

A Recipe Version is an **immutable** specification of a recipe at a given point in time.  
Inventory and analytics always reference a specific Recipe Version – never the Recipe itself.

### Characteristics
- Belongs to exactly one Recipe.
- Immutable after creation.
- Version numbers: 1, 2, 3… per Recipe.
- Uniqueness: `UNIQUE(recipe_id, version)` enforced by the application and database.

### Key Fields
- `id` (UUID)
- `recipe_id`
- `version` (integer, 1‑based)
- `notes` (optional)
- `created_by` (user_id)
- `created_at`

---

## 3. Recipe Ingredient

### Definition

A single component used within a Recipe Version.

### Characteristics
- References exactly one **Product Variant** (never a Product Label).
- Has a fixed quantity and unit.
- May be optional (e.g., garnish).
- Order (`sort_order`) is for UI presentation only – not business logic.

### Synchronous Validation
When an ingredient is created, the system checks that the Product Variant exists and is `is_active = TRUE`.  
No eventual consistency at this layer.

### Key Fields
- `id` (UUID)
- `recipe_version_id`
- `product_variant_id`
- `quantity` (numeric)
- `unit` (ml, cl, l, pcs, g, kg)
- `is_optional` (boolean)
- `sort_order` (integer, optional)

### Rules
- No free‑text ingredients – only Product Variants.
- Units must be normalised (e.g., always ml for liquids, g for weight, pcs for pieces).

---

## 4. Ownership, Author & Rights Holder

Three independent roles are clearly separated:

| Role            | Responsible for                                | Can be changed | Used for                     |
|----------------|------------------------------------------------|----------------|------------------------------|
| **Author**     | Who created the recipe (historical fact)       | ❌ No          | Audit, credit                |
| **Owner**      | Who may use the recipe operationally           | ✅ Yes (via Recipe Transfer) | Access, aggregation |
| **Rights Holder** | Who receives royalties in future creator economy | ✅ Yes | Future monetization |

**Current phase:**  
- The author is always a User within an Organisation.  
- Owner is typically a Team or Organisation.  
- Rights holder may be `NULL` (not used yet).

**Supported ownership scopes:** User, Team, Organisation.

**Visibility:** Recipes are private by default. Visibility is limited to the ownership scope. No public catalogue in the current phase.

---

## 5. Recipe Transfer (History of Ownership Changes)

When operational ownership of a Recipe changes from one scope to another, a **Recipe Transfer** record is created (append‑only).  
This allows full audit of ownership history.

### Typical scenarios
- A bartender creates a recipe (owner = User) and transfers it to a bar (owner = Team).
- A bar collaborates with a guest bartender – operational ownership goes to the bar, but `rights_holder_user_id` stays with the guest.

### Key Fields
- `id` (UUID)
- `recipe_id`
- `from_owner_type`, `from_owner_id`
- `to_owner_type`, `to_owner_id`
- `transferred_by` (user_id)
- `notes` (optional)
- `transferred_at`

---

## 6. Usage in Inventory (Planned – TO‑BE)

This section describes the **target behaviour** to be implemented in Phase 2 of the roadmap.

### Flow
1. User starts an Inventory Session.
2. User selects a Recipe (e.g., “Negroni Pre‑Batch”) and enters the **total volume** (or total units) counted.
3. The system takes the **latest active Recipe Version** (or a specific version if the user chooses).
4. System breaks down the total volume into ingredient quantities using **linear scaling** (for volumetric ingredients) and simple integer rounding for piece‑based ingredients (e.g., eggs, limes).
5. For each ingredient, an **Inventory Item** is created, with:
   - `product_variant_id`
   - `counted_quantity` (derived from breakdown)
   - snapshot of product attributes (weight, volume) frozen at that moment.
6. Additionally, a link is stored in `inventory_item_recipe_components` to trace which breakdown produced which items.
7. If the Recipe Version changes after the session is started, the version **at count time** is used – no recalculations.

### Important
- Recipe Versions are **immutable**. Changing a Recipe creates a new version; old inventory sessions remain unchanged.
- The breakdown respects `is_optional` ingredients (they may be excluded if zero or not used).

---

## 7. Historical Integrity & Snapshots

When a Recipe Version is used in inventory:
- The ingredient list is **snapshotted** (stored as part of the inventory session).
- Quantities and units are frozen.
- Product Variant attributes (tare weight, volume) are resolved **at usage time** and stored in the snapshot.

This guarantees auditability and correct historical reporting even if recipes or products change later.

---

## 8. Relations to Other Domains

- **Products Domain** – uses Product Variant IDs; synchronous existence check.
- **Inventory Domain** – references Recipe Versions; breakdown results become Inventory Items.
- **Users / Teams / Organizations Domain** – controls ownership and permissions.
- **Notifications Domain** – future notifications about recipe changes or transfers.
- **Analytics Domain** – aggregates recipe usage and component consumption.
- **Marketplace / Creator Economy** (future separate bounded context) – will reference `recipe_version_id` for licensing, NFTs, and royalties.

---

## 9. Future: Marketplace & Creator Economy (Out of Scope)

The following are **explicitly not part of this domain** and will be implemented as separate bounded contexts later:

- Public recipe discovery
- Recipe licensing and monetisation
- NFT‑backed recipe ownership
- Royalty payments

The current design reserves the `rights_holder_user_id` field and the Recipe Transfer log to enable these features without rewriting the core domain.

---

## Key Architectural Principles

> Recipes are structured data, not free‑text.  
> Inventory works with **versions**, not recipes.  
> History is immutable – old sessions never change.  
> Ownership, author, and rights holder are separate.  
> Ownership changes are logged (Recipe Transfer).  
> Monetisation must not break operational correctness.  
> Synchronous validation of Product Variants; no eventual consistency at this layer.

This domain is designed to remain stable, auditable, and production‑safe regardless of future business models.
