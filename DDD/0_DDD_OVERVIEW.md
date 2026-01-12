# Domain-Driven Design Overview

## Document Purpose

This document describes the overall
Domain-Driven Design (DDD) approach used in the system.

Its goals are to:
- define clear domain boundaries
- assign ownership and responsibilities
- prevent business logic leakage
- establish a shared Ubiquitous Language
  across product, engineering, and business

The document reflects the **current operational reality (AS-IS)**,
while explicitly marking future directions (TO-BE),
without mixing them into active core logic.

---

## Core DDD Principles

### 1. Operations First

The system is designed around
real workflows of bars, restaurants,
and corporate clients.

> Features are irrelevant
> if they break operational flow.

---

### 2. Inventory Is the Source of Truth

- All quantitative data originates
  from the Inventory domain
- Inventory history is immutable
- Corrections create new states,
  never rewrite the past

---

### 3. Users Act, Domains Own

- Users initiate actions
- Domains own data and rules
- UI contains no business logic

---

### 4. AS-IS and TO-BE Must Not Mix

- AS-IS reflects what actually works today
- TO-BE is explicitly marked as future
- Future layers must not distort
  current architectural decisions

---

## Domain Landscape

### CORE DOMAINS

These domains deliver
the primary business value.
├── Products
├── Inventory
├── Recipes
├── Users / Teams / Organizations
└── Procurement

---

### SUPPORTING DOMAINS

These domains are not directly monetized,
but are critical for scale and quality.
├── Moderation
├── Analytics
├── POS Integrations
└── Notifications

---

### APPLICATION / FUTURE LAYERS

These layers do **not participate in AS-IS operations**,
but are built on top of existing domains.
├── Creator Economy
├── Marketplace
├── NFT / Licensing
└── Payments & Billing

---

## Domain Summaries

### Products Domain

Defines how products are represented.

- Category-agnostic core
- Category-specific attributes
- Global and local catalogs
- Human-in-the-loop moderation
- Foundation for analytics and marketplaces

Products do **not** track quantities —
they describe entities.

---

### Inventory Domain (CORE FLOW)

Responsible for actual stock accounting.

- Snapshot-based model
- Offline-first
- Single source of quantitative truth
- Operates exclusively on Product Variants
- Independent from post-factum moderation

Inventory is the **primary paid feature**.

---

### Recipes Domain

Handles recipes, pre-batches, and compound items.

- Recipes are templates
- Mandatory versioning
- Inventory references recipe versions
- Breakdown is stored in snapshots

In AS-IS, recipes are not standalone
inventory entities.

---

### Users / Teams / Organizations Domain

Defines:
- users
- teams
- organizations
- roles and permissions

Key principles:
- users may belong to multiple teams
- organization is a responsibility container
- the first user has maximum access
- hierarchical organizations are future development

---

### Procurement Domain

Handles ordering and suppliers.

- Integrated with inventory
- Email-first supplier interaction
- Supplier and order lifecycle
- Closes the loop:
  Inventory → Order → Inventory

---

## Cross-Cutting Processes

### Moderation

- Applies to products and attributes
- All products (except `Other`) are moderated
- Affects visibility and reuse,
  but never rewrites history

---

### Analytics

- Built on top of inventory snapshots
- Does not affect operational data
- Aggregates across:
  Variant → Label → Brand

---

## Key Architectural Invariants

> Inventory counts Variants  
> Users search by Labels  
> Moderation never breaks history  
> AS-IS over TO-BE  
> Monetization must not break operations  

---

## Document Role

This overview:
- serves as an architectural map
- is used for team onboarding
- aligns product, business, and engineering

Detailed behavior is described
in individual Domain Design documents.