# Products Domain — Domain Design Draft

## Purpose

The Products domain defines how all goods are represented in the system.

Its goals are to:
- avoid product duplication
- support multiple product categories  
  (alcohol, wine, tobacco, soft drinks, etc.)
- enable structured, human-in-the-loop moderation
- provide a foundation for analytics, integrations,
  and future marketplaces

The domain core is **category-agnostic**.
Category-specific behavior is implemented via attributes.

---

## Core Concepts Overview

The domain is split into several logical levels,
reflecting real-world market structure:

1. Brand / Producer
2. Product Label (commercial product)
3. Product Variant (specific sellable unit)
4. Category and Category Attributes

---

## 1. Brand

### Definition

A Brand represents a producer, manufacturer,
or legal entity behind a product.

Examples:
- Diageo
- Antinori
- Coca-Cola Company

### Notes
- Brand visibility in UI is optional
- Brand is primarily used for:
  - moderation
  - analytics
  - B2B and producer integrations
- Brand does **not** define:
  - packaging
  - volume
  - barcode

---

## 2. Product Label (Base Product)

### Definition

A Product Label represents a commercial product
as it exists on the market,
**independent of packaging, volume, or vintage**.

Examples:
- Talisker 10
- Tignanello
- Schweppes Tonic

### Characteristics
- Belongs to exactly one Category
- May be linked to a single Brand
- Has one canonical name
- May have one default image

### What a Product Label is NOT
- Not a specific bottle or package
- Not inventory-countable
- Not associated with a barcode

A Product Label is what users **search for and recognize**.

---

## 3. Product Variant

### Definition

A Product Variant represents a concrete sellable unit
that is actually counted during inventory sessions.

Examples:
- Talisker 10, 700ml, glass bottle
- Schweppes Tonic, 0.33L, glass
- Schweppes Tonic, 0.5L, PET
- Tignanello, vintage 2016
- Tignanello, vintage 2017 (no barcode)

### Characteristics
- Always belongs to exactly one Product Label
- May have:
  - volume
  - packaging / container type
  - vintage (optional)
  - barcode(s) (optional, **non-unique**)
- Is the **only entity** referenced by the Inventory domain

### Key Principles
- Barcode is **not** a primary identifier
- One Variant may have multiple barcodes
- Multiple Variants may share the same barcode
- Some Variants may have no barcode at all  
  (wine vintages, local products, custom items)

---

## 4. Category

### Definition

A Category defines the type of product
and the rules governing its attributes.

Examples:
- Whisky
- Wine
- Soft Drink
- Tobacco
- Other (special category)

### Category Responsibilities
- Defines allowed attributes
- Specifies validation rules
- Determines moderation requirements

### Special Category: `Other`

The `Other` category is used for:
- user-local products
- products not intended for the global catalog
- products rejected by moderation (`ignore`)

Products in the `Other` category:
- are not sent to moderation
- are visible only within the user’s team / organization
- do not contribute to the growth of the global catalog

---

## 5. Category Attributes

### Definition

Category Attributes are structured properties
applicable **only to specific categories**.

Attributes are typed and validated.
They are **not free-form user input**.

### Attribute Examples

#### Whisky
- region (Islay, Highlands, etc.)
- style (single malt, blended, rye)
- age (optional)
- cask type (optional)

#### Wine
- vintage
- grape varieties
- appellation
- color

#### Soft Drink
- carbonation (boolean)
- sugar_free (boolean)

### Storage Principle

Attributes:
- are defined at the Category level
- are stored as key–value pairs
  on Product Variants

---

## 6. Moderation (AS-IS)

### General Principles

Moderation is a cross-cutting process
that ensures the quality of the global product catalog.

### Current Implementation (AS-IS)

- **All products**, except those in the `Other` category,
  are sent to the moderation queue
- Automatic approval does **not** exist
- A moderator may:
  - modify **any fields**
    (label, variant, barcode, category, attributes)
  - approve the product  
    (add it to the global catalog)
  - assign the product to an existing one  
    (`assign to` by existing product ID)
  - reject the product (`ignore`)

### Behavior on `ignore`
- The product becomes user-local
- Its category is automatically set to `Other`
  (unless the user explicitly selected another category)
- The product is excluded from the global catalog

Moderation affects **visibility and reuse**,
but **does not rewrite inventory history**.

---

## 7. Relations to Other Domains

- Inventory Domain:
  - references **only** Product Variant IDs
- Recipes Domain:
  - uses Product Variants as ingredients
- Analytics Domain (future development):
  - aggregates data by Product Label and Brand
- Procurement Domain:
  - uses products for ordering and supplier workflows
- Marketplace / Creator Economy (future development):
  - builds on top of Product Labels and Variants

---

## Key Architectural Principle

> Inventory counts Variants  
> Users search by Labels but may choose between Variants  
> Brands serve analytics and integrations  

This separation is fundamental
and **must not be violated**
in UI, business logic, or integrations.