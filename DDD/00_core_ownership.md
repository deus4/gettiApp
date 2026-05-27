# 00_core_ownership — Core Ownership & Identity Layer (TO-BE)

## Purpose

This document describes the **target identity and ownership layer** for the entire system. In the current implementation (first release), some concepts may be simplified or deferred. The key difference: **Inventory Domain** uses a direct `team_id`, while Recipes and other domains with creator economy potential use polymorphic `owner_type + owner_id`.

## Implementation by Domain (Current vs Future)

| Domain       | Current (v1)                                 | Future (TO-BE)                       |
| ------------ | -------------------------------------------- | ------------------------------------ |
| Inventory    | `team_id` (FK)                               | remains `team_id`                    |
| Recipes      | `owner_type+owner_id` (already author-ready) | polymorphic, including `user`        |
| Procurement  | `owner_type+owner_id`                        | unchanged                            |
| Products     | global + `source='team'`                     | unchanged                            |

## Goals of this layer

- separate **people (actors)** from **operational entities**
- define unified ownership rules
- enable scalability (multi‑team, multi‑location, enterprise)
- be a common foundation for all domains

This layer **contains no business logic**.  
It defines *who exists* and *who owns what*.

---

## Core Principles

1. **Users are actors, not owners**
2. **Ownership is explicit and immutable**
3. **Teams are the primary operational unit**
4. **Organizations are aggregation & billing units**
5. **All domains rely on this layer**

---

## Core Entities Overview

1. User
2. Organization
3. Team
4. Membership
5. Location
6. Ownership Reference (conceptual)

---

## 1. User

### Concept

A User is a **real person** who can:
- authenticate
- perform actions
- hold roles

A User **is never the direct owner of operational data**.

### Responsibilities

- acting (who performed the action)
- authentication / authorization (outside domains)
- auditing (`created_by`, `updated_by`)

### Invariants

- User ≠ Owner
- A User may be a member of multiple Teams
- A User may be a member of multiple Organizations

### Conceptual Fields

- `id`
- `email`
- `name`
- `status` (active / suspended)
- `created_at`
- `last_login_at`

---

## 2. Organization

### Concept

An Organization is the **top‑level operational and legal entity**.

Used for:
- data aggregation
- billing
- enterprise analytics
- shared ownership

### Characteristics

- contains Teams
- can be an owner of domain data
- does not depend on specific Users

### Bootstrap Rule

The first User who creates an Organization:
- receives an admin role
- **does not become the data owner**

This is a temporary initialisation rule.

### Invariants

- Organization‑level ownership is immutable
- An Organization may exist without active Users

### Conceptual Fields

- `id`
- `name`
- `status` (active / suspended)
- `created_at`

---

## 3. Team

### Concept

A Team is the **primary working unit** of the system.

All operational actions
(inventory, recipes, orders)
happen **in the context of a Team**.

### Characteristics

- belongs to exactly one Organization
- contains Users via Membership
- may own Locations
- can be the owner of domain entities

### Invariants

- A Team cannot exist without an Organization
- Team‑level ownership is immutable

### Conceptual Fields

- `id`
- `organization_id`
- `name`
- `created_at`
- `is_active`

---

## 4. Membership

### Concept

Membership links:
- User
- Team
- Role

Membership **defines permissions**, not a user’s status.

### Characteristics

- A User may have different roles in different Teams
- Roles are not global
- No implicit permissions

### Invariants

- Membership is always team‑scoped
- Deleting a Membership ≠ deleting the User

### Conceptual Fields

- `user_id`
- `team_id`
- `role`
- `created_at`
- `is_active`

---

## 5. Location

### Concept

A Location is a **physical or logical storage place**.

Used for:
- separating inventory
- analytics
- operational clarity

### Characteristics

- belongs to exactly one Team
- optional for Inventory Sessions
- not an owner by itself

### Invariants

- A Location cannot change its Team
- A Location may be deactivated but not deleted

### Conceptual Fields

- `id`
- `team_id`
- `name`
- `type` (bar / storage / kitchen / other)
- `is_active`

---

## 6. Ownership Reference (Conceptual Layer)

### Concept

Ownership is an **explicit reference to the data owner**.

Every domain entity
(Inventory Session, Recipe, Procurement Order, etc.)
must have **exactly one owner**.

### Supported Ownership Types

- Team
- Organization

### Ownership Rules

- `owner_type` + `owner_id`
- immutable after creation
- independent of author / user

### Why Explicit Ownership

This enables:
- safe aggregation
- enterprise analytics
- multi‑team scenarios
- future sharing models

---

## Ownership vs Author (Important)

| Aspect        | Author (User)        | Owner (Team / Org) |
| ------------- | -------------------- | ------------------ |
| Who created   | ✅                   | ❌                 |
| Who owns      | ❌                   | ✅                 |
| Can be changed| yes                  | ❌                 |
| Used for      | audit, history       | access, aggregation|

---

## Cross‑Domain Contract

All domains **must**:
- reference ownership via `owner_type` + `owner_id`
- avoid implicit ownership
- never derive ownership from `user_id`

Violating this contract is considered an **architectural error**.

---

## Out of Scope

This layer does **NOT** handle:
- permissions logic
- authentication flows
- billing rules
- enterprise hierarchy (parent/child orgs)

These may be added **on top** without modifying this layer.

---

## Key Takeaways

> Users act  
> Teams operate  
> Organizations aggregate  
> Ownership is explicit  
> History is preserved  

This layer is the foundation.  
All other domains are built **on top of it**, not alongside it.