# Products Domain — Domain Design Draft

## Purpose

The Products domain defines how all goods are represented in the system.

Its goals are:
- avoid product duplication
- support multiple product categories (alcohol, wine, tobacco, soft drinks, etc.)
- enable structured, human‑in‑the‑loop moderation
- provide a foundation for analytics, integrations, and future marketplaces

The domain core is **category‑agnostic**.  
Category‑specific behavior is implemented via attributes.

---

## Core Concepts Overview

The domain is split into several logical levels, reflecting real‑world market structure:

1. Brand / Producer
2. Product Label (commercial product)
3. Product Variant (specific sellable unit)
4. Category and Category Attributes

---

## 1. Brand

### Definition

A Brand represents a producer, manufacturer, or legal entity behind a product.

**Examples:** Diageo, Antinori, Coca‑Cola Company.

### Notes
- Brand visibility in the UI is optional.
- Brand is primarily used for moderation, analytics, and B2B/producer integrations.
- Brand **does not** define packaging, volume, or barcode.

---

## 2. Product Label (Base Product)

### Definition

A Product Label represents a commercial product as it exists on the market,  
**independent of packaging, volume, or vintage**.

**Examples:** Talisker 10, Tignanello, Schweppes Tonic.

### Characteristics
- Belongs to exactly one Category.
- May be linked to a single Brand.
- Has one canonical name.
- May have one default image.

### What a Product Label is **not**
- Not a specific bottle or package.
- Not inventory‑countable.
- Not associated with a barcode.

A Product Label is what users **search for and recognize**.

---

## 3. Product Variant

### Definition

A Product Variant represents a concrete sellable unit that is actually counted during inventory sessions.

**Examples:**
- Talisker 10, 700ml, glass bottle
- Schweppes Tonic, 0.33L, glass
- Schweppes Tonic, 0.5L, PET
- Tignanello, vintage 2016
- Tignanello, vintage 2017 (no barcode)

### Characteristics
- Always belongs to exactly one Product Label.
- May have:
  - volume (in ml)
  - packaging / container type
  - vintage (optional)
  - barcode(s) – optional, **non‑unique**
- Is the **only entity** referenced by the Inventory domain.

### Key Principles
- Barcode is **not** a primary identifier.
- One Variant may have multiple barcodes (stored in a separate table `product_barcodes`).
- Multiple Variants may share the same barcode.
- Some Variants may have no barcode at all (e.g., wine vintages, local products, custom items).

### Source (origin)
Every Variant has a `source` field:
- `global` – from the moderated global catalog, available to all users.
- `team` – local variant created by a team; not moderated (or rejected). Visible only within that team’s scope.

---

## 4. Category

### Definition

A Category defines the type of product and the rules governing its attributes.

**Examples:** Whisky, Wine, Soft Drink, Tobacco, **Other** (special category).

### Category Responsibilities
- Defines allowed attributes.
- Specifies validation rules.
- Determines moderation requirements.

### Special Category: `Other`

The `Other` category is used for:
- user‑local products
- products not intended for the global catalog
- products rejected by moderation (`ignore`)

Products in the `Other` category:
- are **not sent to moderation**
- are visible only within their own team/organization
- do **not** contribute to the growth of the global catalog
- have `source = 'team'`

---

## 5. Category Attributes

### Definition

Category Attributes are structured properties applicable **only to specific categories**.

Attributes are typed and validated. They are **not free‑form user input**.

### Attribute Examples

**Whisky**  
- region (Islay, Highlands, etc.)
- style (single malt, blended, rye)
- age (optional)
- cask type (optional)

**Wine**  
- vintage
- grape varieties
- appellation
- color

**Soft Drink**  
- carbonation (boolean)
- sugar_free (boolean)

### Storage Principle
- Attributes are defined at the Category level.
- They are stored as key‑value pairs on Product Variants (e.g., in a `product_attributes` JSONB or separate table).
- Validation rules are enforced by the application, not the database.

---

## 6. Moderation (AS‑IS)

### General Principles

Moderation is a cross‑cutting process that ensures the quality of the global product catalog.  
It is performed **only by internal Getti staff** (via Diabolics). External moderators never exist.

### Current Implementation (AS‑IS)

- **All products** except those in the `Other` category are sent to the moderation queue (`moderation_requests`).
- Automatic approval does **not** exist.
- A moderator may:
  - modify **any fields** (label, variant, barcode, category, attributes)
  - **accept** – the product becomes global (`source = 'global'`). User notified.
  - **assign to** an existing product (by ID) – user’s product linked to existing one. User notified.
  - **ignore** – reject the product.

### Behavior on `ignore`
- The product remains local (`source = 'team'`).
- Its category is automatically set to `Other` (unless the user already set a different category manually).
- The product is excluded from the global catalog.
- User notified.

### Edit of a global product (`edit_product`)
- When a user edits a global product, the system creates a **local override** (copy with `source = 'team'`) and a moderation request.
- The original global product is untouched until the moderator decides.
- If accepted, the global product is updated.
- If ignored, the local override stays local (category becomes `Other`).

### Important
Moderation affects **visibility and reuse**, but **does not rewrite inventory history** (snapshots protect historical data).

---

## 7. Pricing

In the current implementation, **prices are not stored** in the Products domain.  
Inventory reports show `price = 0.00`.

In a future phase, prices may come from supplier price lists (Procurement domain) or be entered manually by users.

---

## 8. Relations to Other Domains

- **Inventory Domain** – references only `product_variant_id`. Synchronous existence check (`is_active = TRUE`) is performed when creating inventory items.
- **Recipes Domain** – uses Product Variants as ingredients. Same synchronous check.
- **Procurement Domain** – references Product Variants for order items.
- **Users, Teams & Organizations** – ownership and visibility of local products (`source = 'team'`).
- **Analytics Domain** (future) – aggregates by Product Label and Brand.
- **Marketplace / Creator Economy** (future) – builds on top of Labels and Variants.
- **Platform Domain (Diabolics)** – moderation tools and global catalog management.

---

## Key Architectural Principles

> Inventory counts Variants.  
> Users search by Labels but may choose between Variants.  
> Brands serve analytics and integrations.  
> Moderation affects visibility, not history.  
> Local products (`Other` category) never enter the global catalog.  
> Synchronous existence check on reference; no eventual consistency at this layer.

This separation is fundamental and **must not be violated** in UI, business logic, or integrations.