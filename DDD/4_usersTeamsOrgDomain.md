# Users, Teams & Organizations Domain — Domain Design Draft

## Purpose

The Users, Teams & Organizations Domain defines:
- who uses the system
- how users are grouped
- how responsibilities are distributed
- how operational ownership is structured

Its goals:
- support single bars and enterprise structures
- enable multi-team and multi-location setups
- separate human identity from data ownership
- provide a stable foundation for Inventory, Recipes, Products, and Analytics
- remain flexible for future business models

This domain is **structural and organizational**.  
It does NOT implement inventory, recipe, or order business logic.

---

## Core Concepts Overview

The domain consists of:

1. User
2. Organization
3. Team
4. Membership & Roles
5. Location
6. Ownership & Scope Resolution

---

## 1. User

### Definition

A User represents a real human being
who can authenticate, interact with the system,
and perform actions based on assigned roles.

### Characteristics
- Represents a single person
- May belong to multiple Teams
- May belong to multiple Organizations
- Does not own operational data
- Acts only within granted scopes

### Key Fields
- id
- email
- name
- status
- created_at
- last_login_at

### Core Principle

> Users are actors, not owners of operational data.

---

## 2. Organization

### Definition

An Organization represents a legal or business entity
that groups teams and locations
under a single operational and analytical umbrella.

Examples:
- restaurant group
- hotel chain
- enterprise customer
- franchise owner

---

### Bootstrap Rule

When an Organization is created,
the creating user is assigned the highest-level role.

This exists solely to bootstrap the system
and guarantee at least one administrator.

It does not imply permanent ownership or fixed roles.

---

### Characteristics
- Top-level ownership scope
- Contains one or more Teams
- Used for aggregation and analytics
- Contractual and billing boundary

### Fields
- id
- name
- created_at
- status

---

### Structure (Conceptual)

Organization  
├── Team  
│    ├── Location  
│    │    └── InventorySession  
│    └── InventorySession  
└── Team  

---

### Current Implementation Note

Organizations are currently flat.
Hierarchical org structures are a future extension.

---

## 3. Team

### Definition

A Team is the primary operational unit
within an Organization.

It is the main working context for users.

### Characteristics
- Belongs to one Organization
- Contains users via memberships
- May own locations
- Valid ownership scope for Inventory and Recipes

### Fields
- id
- organization_id
- name
- created_at
- is_active

---

## 4. Membership & Roles

### Definition

A Membership connects a User to a Team
and defines permitted actions.

Users may have different roles across teams.

### Fields
- user_id
- team_id
- role
- created_at
- is_active

### Role Principles
- Roles are team-scoped
- No global roles
- Permissions resolved via role + ownership scope

---

## 5. Location

### Definition

A Location represents a physical or logical place
where inventory is stored and counted.

### Characteristics
- Belongs to one Team
- Optional for inventory
- Not an ownership scope

### Fields
- id
- team_id
- name
- type
- is_active

---

## 6. Ownership & Scope Resolution

### Ownership Concept

Ownership defines:
- who owns operational data
- how data is aggregated
- how visibility is resolved

Supported scopes:
- Team
- Organization

Future scopes:
- Multi-Team Group
- Enterprise Account

### Ownership Rules
- Each Inventory Session has exactly one owner
- Ownership is immutable
- Ownership is independent of user or author

---

## 7. Relation to Other Domains

- Inventory — owned by Team or Organization
- Recipes — authored by User, owned by Team or Organization
- Products — shared within ownership scopes
- Procurement (future) — ownership-based
- Analytics — aggregation via ownership hierarchy

---

## Key Architectural Principles

> Users are actors, not owners  
> Ownership is explicit and immutable  
> Teams are the primary operational unit  
> Organizations enable scale and aggregation  
> Permissions are resolved outside domain data  

This domain provides the organizational backbone
for the entire system and must remain stable as it evolves.