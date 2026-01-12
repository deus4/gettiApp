# Recipes Domain — Domain Design Draft

## Purpose

The Recipes Domain defines how composite products (recipes)
are created, stored, versioned, and used across the system.

Its goals:
- support cocktails, pre-batches, and preparations
- allow automatic inventory breakdown into components
- preserve historical correctness
- enable reuse across teams and locations

This domain intentionally avoids business models and monetization.
It focuses on structure, correctness, and extensibility.

---

## Core Concepts Overview

The Recipes Domain consists of:

1. Recipe
2. Recipe Version
3. Recipe Ingredient
4. Ownership & Visibility
5. Recipe Usage in Inventory

---

## 1. Recipe

### Definition

A Recipe represents a conceptual composition created by a user.
It is an abstract entity that may evolve over time.

Examples:
- Negroni
- Espresso Martini (House Style)
- Gin Basil Smash Pre-Batch

### Characteristics
- Has an author (User)
- Belongs to an ownership scope
- Has multiple versions
- Is not directly inventory-countable

---

### Inventory Usage

- Inventory never counts Recipes directly
- Inventory references Recipe Versions only
- Actual impact is calculated via ingredients
- Breakdown results are stored as Inventory Items

Recipes are composition blueprints, not inventory units.

---

### Author vs Ownership

- Author — human creator
- Ownership — who may use it operationally

Current phase:
- Author: User within Organization
- Ownership: Team or Organization

Future:
- cross-org sharing
- licensing
- creator-controlled distribution

---

### Key Fields
- id
- author_user_id
- owner_type (User | Team | Organization)
- owner_id
- created_at
- is_archived

---

## 2. Recipe Version

### Definition

A Recipe Version is an immutable specification of a recipe
at a specific point in time.

Inventory and analytics always reference versions.

### Characteristics
- Belongs to one Recipe
- Immutable
- Versioned
- Active or deprecated

### Fields
- id
- recipe_id
- version
- created_at
- created_by_user_id
- notes

---

## 3. Recipe Ingredient

### Definition

A single component within a Recipe Version.

### Characteristics
- References Product Variant
- Fixed quantity and unit
- Order is UI-only

### Fields
- recipe_version_id
- product_variant_id
- quantity
- unit
- is_optional

### Rules
- Product Variant only
- No free-text
- Units normalized

---

## 4. Ownership & Visibility

Supported scopes:
- User
- Team
- Organization

### Rules
- Private by default
- Visibility limited to ownership scope
- No public discovery

---

## 5. Usage in Inventory

Example:
- User counts “Negroni Pre-Batch” as 3.5L
- System breaks it down into ingredients

### Rules
- Recipe Version referenced
- Quantities frozen
- Results stored as Inventory Items
- History never rewritten

---

## 6. Historical Integrity

When a Recipe Version is used:
- ingredients snapshotted
- quantities frozen
- product attributes resolved at usage time

---

## 7. Relation to Other Domains

- Products — Product Variant
- Inventory — Recipe Version
- Users / Teams / Organizations — ownership
- Analytics — aggregation

---

## 8. Future: Marketplace & Creator Economy (Out of Scope)

- Public discovery
- Licensing
- Marketplaces
- NFT-backed ownership
- Royalties

Implemented as a separate domain later.

---

## Key Architectural Principles

> Recipes are structured data, not text  
> Inventory works with versions  
> History is immutable  
> Ownership is explicit  
> Monetization must not break operations