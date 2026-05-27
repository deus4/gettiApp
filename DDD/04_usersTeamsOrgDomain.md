# Users, Teams & Organisations Domain — Domain Design Draft

## Purpose

The Users, Teams & Organisations Domain defines:
- who uses the system
- how users are grouped
- how responsibilities are distributed
- how operational ownership is structured

Its goals:
- support single bars and enterprise structures
- enable multi‑team and multi‑location setups
- separate human identity from data ownership
- provide a stable foundation for Inventory, Recipes, Products, and Analytics
- remain flexible for future business models

This domain is **structural and organisational**.  
It does **not** implement inventory, recipe, or order business logic.

---

## Core Concepts Overview

The domain consists of:

1. User  
2. Organisation  
3. Team  
4. Membership & Roles  
5. Location  
6. Invitation  
7. Ownership & Scope Resolution  

---

## 1. User

### Definition

A User represents a real human being who can authenticate, interact with the system, and perform actions based on assigned roles.

### Characteristics
- Represents a single person
- May belong to multiple Teams
- May belong to multiple Organizations
- **Does not own operational data**
- Acts only within granted scopes

### Authentication
- JWT‑based authentication.
- Refresh tokens stored in `user_sessions` together with device info (`device_id`, `push_token`, `platform`) for push notifications.
- Authorization is handled in the **application layer (middleware)**, not in the domain.

### Key Fields
- `id` (UUID v4)
- `email` (CITEXT, unique)
- `name`
- `password_hash`
- `status` (active | suspended)
- `created_at`
- `last_login_at`

### Core Principle

> Users are actors, not owners of operational data.  
> Deleting a user from a team does **not** affect existing data – `created_by_user_id` remains as an audit field.

---

## 2. Organization

### Definition

An Organization represents a legal or business entity that groups teams and locations under a single operational and analytical umbrella.

**Examples:** restaurant group, hotel chain, enterprise customer, franchise owner.

### Bootstrap Rule
When an Organization is created, the creating user is assigned the highest‑level role (`owner`) within that Organisation.  
This rule exists solely to bootstrap the system and guarantee at least one administrator. It does **not** imply permanent ownership or fixed roles.

### Characteristics
- Top‑level ownership scope
- Contains one or more Teams
- Used for aggregation, analytics, and billing
- Contractual and billing boundary

### Fields
- `id` (UUID v4)
- `name`
- `status` (active | suspended)
- `created_at`

### Current Implementation Note
Organisations are **flat** – no parent/child hierarchies. Hierarchical organisations are a possible future extension.

---

## 3. Team

### Definition

A Team is the **primary operational unit** within an Organisation.  
It is the main working context for users.

**Examples:** single bar, kitchen, brand‑specific unit, temporary event team.

### Characteristics
- Belongs to exactly one Organisation
- Contains Users via Memberships
- May own Locations
- Is a valid ownership scope for Inventory and Recipes

### Fields
- `id` (UUID v4)
- `organization_id`
- `name`
- `is_active`
- `created_at`

### Responsibilities
- Day‑to‑day operations
- Inventory execution
- Recipe usage
- Procurement (future)

---

## 4. Membership & Roles

### Definition

A Membership connects a User to a Team and defines what actions the user may perform within that Team.

Users may have different roles in different Teams. Roles are **team‑scoped** – no global roles exist by default.

### Fields
- `id` (UUID v4)
- `user_id`
- `team_id`
- `role`
- `is_active`
- `created_at`
- `UNIQUE(user_id, team_id)`

### Example Roles

| Role               | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `owner`            | Full access, team management                                                |
| `manager`          | Operational management, can finalise inventory sessions                    |
| `bartender`        | Can perform inventory counting                                              |
| `inventory_manager`| Manages inventory sessions, may also edit products                          |
| `accountant`       | View‑only access to reports and financial data                             |
| `viewer`           | Read‑only access to all team data                                           |

### Role Principles
- Roles are **team‑scoped** – no global roles.
- Permissions are resolved as `role + ownership scope`.
- Authorization is handled in the **application layer**, not in the domain model.
- Repositories receive `team_id` explicitly – they do not filter by the current user themselves.

---

## 5. Location

### Definition

A Location represents a physical or logical place where inventory is stored and counted.

**Examples:** bar, storage room, kitchen, warehouse, mobile unit.

### Characteristics
- Belongs to exactly one Team
- Optional for Inventory Sessions
- **Not an ownership scope** – only refines a Team scope

### Fields
- `id` (UUID v4)
- `team_id`
- `name`
- `type` (bar | storage | kitchen | mobile | other)
- `is_active`
- `created_at`

### Hierarchy (Conceptual)

Team
├── Location (Bar)
├── Location (Storage)
└── Location (Kitchen)


An Inventory Session may be:
- tied to a specific Location, or
- team‑wide (`location_id = NULL`).

---

## 6. Invitation

### Invitation Flow
1. A manager enters a new staff member’s email and selects a role.
2. The system generates a signed, one‑time token.
3. An email is sent (via Notifications Domain) with a link containing the token.
4. The user clicks the link, registers or logs in.
5. The user is automatically added to the Team with the specified role.

### Fields
- `id` (UUID v4)
- `team_id`
- `email` (CITEXT)
- `role`
- `token` (unique, signed)
- `expires_at` (standard TTL for such flows)
- `accepted_at` (nullable)
- `created_by` (user_id)
- `created_at`

---

## 7. Ownership & Scope Resolution

### Ownership Concept

Ownership defines:
- who owns operational data
- how data is aggregated
- how visibility is resolved

**Supported ownership scopes:** Team, Organization.

**Future scopes (not in current phase):** Multi‑Team Group, Enterprise Account.

### Ownership Rules
- Each operational entity (Inventory Session, Recipe, Procurement Order) has exactly one owner.
- Ownership is **immutable after creation**.
- Ownership is **independent of the user or author**.
- Some domains use a polymorphic `(owner_type, owner_id)` (Recipes, Procurement).  
  **Inventory uses a direct `team_id` foreign key** – this is a conscious decision because inventory is always tied to an operational unit.

### Ownership Hierarchy (Conceptual)

Organization
├── Team
│ ├── Location
│ │ └── InventorySession
│ └── InventorySession (without location)
└── Team
└── InventorySession


---

## 8. Relation to Other Domains

- **Inventory Domain** – owned by Team (direct `team_id`). Access via membership.
- **Recipes Domain** – author = User, owner = Team/Organization (polymorphic).
- **Products Domain** – global catalog shared; local products scoped to Team (`source = 'team'`).
- **Procurement Domain** – orders and suppliers owned by Team/Organisation.
- **Notifications Domain** – uses `user_sessions` to store device tokens for push.
- **Platform Domain (Diabolics)** – completely isolated; separate `platform_users` table.

---

## Key Architectural Principles

> Users are actors, not owners.  
> Ownership is explicit and immutable.  
> Teams are the primary operational unit.  
> Organisations enable scale and aggregation.  
> Permissions are resolved in the application layer, not in the domain.  
> Data outlives the user – removing a user does not delete their audit records.

This domain provides the organisational backbone for the entire system and must remain stable as the system evolves.
