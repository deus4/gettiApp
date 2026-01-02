# Products Domain — Domain Design Draft

## Purpose

The Products Domain defines how all goods are represented in the system.
Its goal is to:
- avoid duplication
- support different categories (alcohol, wine, tobacco, soft drinks, etc.)
- allow structured moderation
- enable future analytics, integrations, and marketplaces

This domain is category-agnostic at the core and category-specific via attributes.

---

## Core Concepts Overview

The domain is split into several logical levels reflecting real-world structure:

1. Brand / Producer
2. Product Label (commercial product)
3. Product Variant (specific sellable unit)
4. Category and Category Attributes

---

## 1. Brand

### Definition
A Brand represents a producer, manufacturer, or brand owner.

Examples:
- Diageo
- Antinori
- Coca-Cola Company

### Notes
- Brand visibility in UI is optional
- Brand is mainly used for:
  - moderation
  - analytics
  - B2B / producer integrations
- Brand does NOT define packaging, volume, or barcode

---

## 2. Product Label (Product Base)

### Definition
A Product Label represents a commercial product as it is known on the market,
independent of packaging, volume, or vintage.

Examples:
- Talisker 10
- Tignanello
- Schweppes Tonic

### Characteristics
- Belongs to exactly one Category
- Can be linked to one Brand
- Has a single canonical name
- Can have one default image

### What it is NOT
- Not a specific bottle
- Not inventory-countable
- Not tied to a barcode

---

## 3. Product Variant

### Definition
A Product Variant represents a specific sellable unit of a Product Label.

Examples:
- Talisker 10, 700ml, glass bottle
- Schweppes Tonic, 0.33L, glass
- Schweppes Tonic, 0.5L, PET
- Tignanello, vintage 2016
- Tignanello, vintage 2017 (no barcode)

### Characteristics
- Always belongs to a Product Label
- Can have:
  - volume
  - packaging type
  - vintage (optional)
  - barcode (optional, non-unique)
- Is the ONLY entity used in inventory sessions

### Physical Properties (Variant-level)

Product Variants may include physical properties required for inventory counting:
- full_tare_weight (grams)
- empty_tare_weight (grams)
- density (optional, liquids only)

These properties are mandatory for weight-based inventory and BLE integration.

### Important Notes
- Barcode is NOT a primary identifier
- Multiple variants may share the same barcode
- Some variants may have no barcode at all

---

## 4. Category

### Definition
Category defines the type of product and the rules for its attributes.

Examples:
- Whisky
- Wine
- Soft Drink
- Tobacco

### Responsibilities
- Determines which attributes are applicable
- Defines validation rules
- Defines moderation requirements

---

## 5. Category Attributes

### Definition
Category Attributes define structured properties that are applicable only
to certain categories. Attributes are typed and validated per category (not free-form user input).

Examples:

#### Whisky Attributes
- region (Islay, Highlands, etc.)
- style (single malt, blended, rye)
- age (optional)
- cask type (optional)

#### Wine Attributes
- vintage
- grape varieties
- appellation
- color

#### Soft Drink Attributes
- carbonation (boolean)
- sugar_free (boolean)

### Storage Principle
Attributes are defined per category and stored as key-value pairs on Product Variants.

---

## 6. Moderation Rules (High-level)

Moderation affects visibility and reuse scope (global vs local),
but does not affect historical inventory data.

- Product Labels are moderated entities
- Product Variants may be:
  - auto-approved
  - queued for moderation
- Attribute values may require moderation depending on category

---

## 7. Relation to Other Domains

- Inventory Domain references ONLY Product Variant IDs
- Recipes reference Product Variants
- Analytics aggregate via Product Label and Brand
- Marketplace/NFT layers build on top of Product Labels and Variants

---

## Key Architectural Principle

> Inventory counts Variants  
> Users see Labels  
> Brands serve analytics and integrations  

This separation is fundamental and must not be violated.
