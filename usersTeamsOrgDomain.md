# Users, Teams & Organizations Domain — Domain Design Draft

## Purpose

The Users, Teams & Organizations Domain defines
who uses the system, how users are grouped,
how responsibilities are distributed,
and how operational ownership is structured.

Its goals:
- support single bars and large enterprises
- enable multi-team and multi-location setups
- separate human identity from operational ownership
- provide a stable foundation for inventory, recipes, procurement, and analytics
- remain flexible for future business models

This domain is **structural and organizational**.
It does NOT implement business logic of inventory, recipes, or orders.

---

## Core Concepts Overview

The domain consists of the following core entities:

1. User
2. Organization
3. Team
4. Membership & Roles
5. Locations (Operational Units)
6. Ownership & Scope Resolution

---

## 1. User

### Definition

A User represents a real human being
who can authenticate, interact with the system,
and perform actions according to assigned roles.

### Characteristics
- Represents a single person
- May belong to multiple Teams
- May belong to multiple Organizations
- Has no inherent ownership of operational data
- Acts within scopes granted by memberships

### Key Fields (conceptual)
- id
- email
- name
- status (active / suspended)
- created_at
- last_login_at

### Important Principle

> Users do not own inventory, recipes, or orders by default.  
> Ownership belongs to operational entities (Teams, Organizations).

Users are **actors**, not containers.

---

## 2. Organization

### Definition

An Organization represents a legal or business entity
that groups multiple teams and locations
under a single operational and analytical umbrella.

Examples:
- Restaurant group
- Hotel chain
- Enterprise customer with multiple brands
- Franchise owner

### Bootstrap Ownership Rule

When an Organization is created, the creating User
is automatically assigned the highest-level role
within that Organization.

This rule exists solely to bootstrap the system
and ensure that each Organization has at least one administrator.

This does NOT imply:
- permanent ownership
- exclusive rights
- immutability of roles

Ownership and roles may evolve as the system grows.

### Characteristics
- Top-level ownership scope
- Contains one or more Teams
- May aggregate data across Teams and Locations
- Represents contractual and billing boundary

### Key Fields (conceptual)
- id
- name
- created_at
- status (active / suspended)

### Organization Structure (Conceptual)

Organization  
├── Team  
│    ├── Location  
│    │    └── InventorySession  
│    └── Team-level InventorySession  
└── Team  

### Current Implementation Note

In the current phase, Organizations are flat:
- Organizations do not contain sub-organizations
- All Teams belong directly to a single Organization

Hierarchical Organizations (parent / child structures)
are considered a future extension and are not implemented yet.

---

## 3. Team

### Definition

A Team represents an operational unit
within an Organization.

It is the **primary working context**
for most users.

Examples:
- Single bar
- Kitchen team
- Brand-specific unit
- Temporary event team

### Characteristics
- Belongs to exactly one Organization
- Contains Users via memberships
- May own Locations
- Is a valid ownership scope for Inventory and Recipes

### Key Fields (conceptual)
- id
- organization_id
- name
- created_at
- is_active

### Team Responsibilities
- Day-to-day operations
- Inventory execution
- Recipe usage
- Procurement (future)

---

## 4. Membership & Roles

### Membership Definition

A Membership connects a User to a Team
and defines what actions the user may perform
within that Team.

Users may have different roles in different teams.
Roles define permissions, not status.
They are operational, not hierarchical by nature.

### Key Fields (conceptual)
- user_id
- team_id
- role
- created_at
- is_active

### Example Roles (Non-Exhaustive)
- Owner
- Manager
- Bartender
- Inventory Manager
- Accountant
- Viewer

### Role Principles
- Roles are **team-scoped**
- No global roles exist by default
- Permissions are resolved via role + ownership scope

This allows:
- one user to manage multiple teams
- different responsibility levels per team

---

## 5. Location (Operational Unit)

### Definition

A Location represents a physical or logical place
where inventory is stored and counted.

Examples:
- Bar
- Storage room
- Kitchen
- Warehouse
- Mobile unit

### Characteristics
- Belongs to exactly one Team
- Optional for Inventory Sessions
- Used for physical separation of stock

### Key Fields (conceptual)
- id
- team_id
- name
- type (bar / storage / kitchen / other)
- is_active

### Location Hierarchy (Conceptual)

Team  
├── Location (Bar)  
├── Location (Storage)  
└── Location (Kitchen)  

Inventory Sessions may:
- be location-specific
- or team-wide (location = null)

---

## 6. Ownership & Scope Resolution

### Ownership Scope Concept

Ownership defines **who owns operational data**
and how it may be aggregated.

Supported ownership scopes:
- Team
- Organization

Future scopes (planned, not implemented):
- Multi-Team Group
- Enterprise Account

### Ownership Rules
- Every Inventory Session has exactly ONE owner
- Ownership is immutable after creation
- Ownership is independent from author or user

### Scope Resolution Examples

- Inventory owned by Team A
  - Visible to Team A
  - Aggregatable at Organization level

- Inventory owned by Organization
  - Visible across all Teams
  - Aggregatable globally

Ownership is used for:
- aggregation
- analytics
- reporting
- access resolution (via external rules)

---

## 7. Relation to Other Domains

- Inventory Domain:
  - Inventory Sessions belong to Team or Organization
  - Users act within Team context
- Recipes Domain:
  - Recipes have authors (Users)
  - Recipes are owned by Team or Organization
- Products Domain:
  - Products are shared within ownership scopes
- Procurement Domain (future):
  - Orders belong to Teams or Organizations
  - Supplier relationships are team-scoped
- Analytics Domain:
  - Aggregates data by ownership hierarchy

---

## Key Architectural Principles

> Users are actors, not owners  
> Ownership is explicit and immutable  
> Teams are the primary operational unit  
> Organizations enable aggregation and scale  
> Permissions are resolved outside domain data  

This domain provides a stable organizational backbone
for all present and future functionality,
from single bars to enterprise deployments.
