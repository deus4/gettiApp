# Recipes Domain — Domain Design Draft

## Purpose

The Recipes Domain defines how composite products (recipes)
are created, stored, versioned, and used across the system.

Its goals:
- support bar pre-batches and cocktails
- allow automatic inventory breakdown into components
- preserve historical correctness
- enable reuse across locations and teams

This domain intentionally avoids business models and monetization logic.
It focuses on structure, correctness, and reusability.

---

## Core Concepts Overview

The Recipes Domain consists of the following core entities:

1. Recipe
2. Recipe Version
3. Recipe Ingredient
4. Recipe Ownership & Visibility
5. Recipe Usage in Inventory

---

## 1. Recipe

### Definition

A Recipe represents a conceptual composition created by a user
(e.g. cocktail, pre-batch, kitchen prep).

It is an abstract entity that may evolve over time.

Examples:
- Negroni
- Espresso Martini (House Style)
- Gin Basil Smash Pre-Batch

### Characteristics
- Has an author (User)
- Belongs to an ownership scope (who may use it)
- Has multiple versions
- Is NOT directly counted in inventory

### Inventory Usage Clarification

- Inventory never counts Recipes directly
- Inventory references Recipe Versions
- Actual inventory impact is calculated via ingredients
- Breakdown results are stored as Inventory Items

Recipes are composition blueprints,
not inventory units.

### Author vs Ownership (Important Distinction)

- Author represents the human creator of a recipe
- Ownership defines who may use the recipe operationally

Current phase:
- Author is always a User within an Organization
- Ownership is typically Team or Organization

Future phases may allow:
- cross-organization sharing
- licensing
- creator-controlled distribution

This distinction is intentionally preserved
even if author and ownership currently overlap.

### Key Fields (conceptual)
- id
- author_user_id
- owner_type (User | Team | Organization)
- owner_id
- created_at
- is_archived (boolean)

---

## 2. Recipe Version

### Definition

A Recipe Version represents a concrete, immutable specification
of a recipe at a given point in time.

Inventory, analytics, and reporting always reference
a specific Recipe Version — never the Recipe itself.

### Characteristics
- Belongs to exactly one Recipe
- Is immutable once created
- Has a version identifier
- May be marked as active or deprecated

### Key Fields (conceptual)
- id
- recipe_id
- version
- created_at
- created_by_user_id
- notes (optional)

---

## 3. Recipe Ingredient

### Definition

A Recipe Ingredient represents a single component used
within a Recipe Version.

### Characteristics
- References a Product Variant
- Has a fixed quantity and unit
- Order is optional and presentation-level only

### Fields (conceptual)
- recipe_version_id
- product_variant_id
- quantity
- unit (ML, CL, G, PCS, etc.)
- is_optional (boolean)

### Rules
- Ingredients must reference Product Variants
- Free-text ingredients are not allowed
- All units must be normalized

---

## 4. Recipe Ownership & Visibility

### Ownership Scope

Recipes follow the same ownership principles
as Inventory Sessions.

Supported scopes:
- User (personal recipes)
- Team
- Organization

### Visibility Rules
- Recipes are private by default
- Visibility is limited to ownership scope
- No public discovery is enabled in the current phase

Ownership affects:
- reuse
- access control
- aggregation and analytics

---

## 5. Recipe Usage in Inventory

### Inventory Breakdown

Recipes may be used as inventory-countable entities.

Example:
- User counts "Negroni Pre-Batch" as 3.5L
- System automatically breaks it down into:
  - Gin
  - Vermouth
  - Campari

### Rules
- Inventory references Recipe Versions
- Breakdown uses frozen ingredient quantities
- Breakdown results are stored as Inventory Snapshots
- Product changes never rewrite historical usage

---

## 6. Historical Integrity & Snapshots

### Snapshot Principle

When a Recipe Version is used:
- Ingredient list is snapshotted
- Quantities and units are frozen
- Product Variant attributes are resolved at usage time

This guarantees:
- auditability
- correct historical analytics
- stable reporting

---

## 7. Relation to Other Domains

- Products Domain:
  - Recipes reference Product Variant IDs only
- Inventory Domain:
  - Inventory may reference Recipe Versions
  - Breakdown results become Inventory Items
- Users, Teams & Organizations Domain:
  - Controls ownership and permissions
- Analytics Domain:
  - Aggregates recipe usage and component consumption

---

## 8. Future Phase: Marketplace & Creator Economy (Explicitly Out of Scope)

The following features are **explicitly not part of the current implementation**
and are listed here to clarify long-term architectural intent only.

### Possible Future Extensions
- Public recipe discovery
- Recipe licensing and monetization
- Marketplace for professional recipes
- NFT-backed recipe ownership
- Royalty and commission models

### Important Notes
- These features will be implemented as a separate domain or layer
- No blockchain, token, or NFT logic exists in the current Recipes Domain
- Current architecture merely allows safe extension later

---

## Key Architectural Principles

> Recipes are structured data, not text  
> Inventory and analytics reference versions, not drafts  
> History is immutable  
> Ownership is explicit  
> Future monetization must not affect operational correctness  

This domain is designed to remain stable, auditable,
and production-safe regardless of future business models.
