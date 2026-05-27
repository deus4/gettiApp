# Procurement Domain — Domain Design Draft

## Purpose

The Procurement Domain defines:
- how goods are ordered from suppliers
- how supplier relationships are managed
- How procurement workflows integrate with inventory

Its goals:
- fast ordering based on inventory data
- email‑first, supplier‑friendly workflows (suppliers never need to log in)
- support for negotiation via append‑only revisions
- full history of order changes and messages
- foundation for future automation and analytics

Procurement is an **operational workflow**, not a financial system.  
Pricing, billing, payments, and accounting are **explicitly out of scope**.

**Orders are a free entry‑level feature of the system.**  
Inventory remains the primary paid feature.

---

## Core Concepts Overview

1. Supplier  
2. Supplier Relationship (Pairing)  
3. Procurement Order  
4. Order Revision (append‑only negotiation model)  
5. Order Message (text chat)  
6. Supplier Action Tokens (one‑time, signed)  
7. Procurement Lifecycle  

---

## 1. Supplier

### Definition

A Supplier is an external entity that provides goods to a Team or Organisation.  
Suppliers are **never system users** – no registration, no login.

**Examples:** alcohol distributor, beverage wholesaler, tobacco supplier, local food vendor.

### Key Fields
- `id` (UUID v4)
- `owner_type` (team | organization)
- `owner_id`
- `name`
- `legal_name` (optional)
- `contact_email`
- `contact_name` (optional)
- `phone` (optional)
- `notes` (optional)
- `pairing_status` (inactive | pending | accepted | rejected)
- `pairing_requested_at`
- `pairing_resolved_at`
- `last_rejected_at` (for cooldown)
- `is_noreply` (boolean – flagged if email looks like no‑reply)
- `created_by` (user_id)
- `created_at`

---

## 2. Supplier Relationship (Pairing)

### Definition

An operational link between a Team/Organisation and a Supplier is required before orders can be sent.

### Pairing Flow
1. User creates a Supplier (status `inactive`).
2. User sends a **pairing request**.
3. System checks email for no‑reply patterns (prefixes: `noreply`, `no-reply`, `donotreply`, `notifications`, known auto‑responders).
4. If no‑reply detected → `is_noreply = TRUE`, user warned, email **not sent**.
5. Otherwise, email sent with HTML page and Accept/Reject buttons (token links).
6. Supplier clicks a button → token validated, status updated.
7. User receives push/email notification.

### States

inactive → pending → accepted
↘ rejected → (cooldown 24h) → inactive (user may retry)

| Status     | Orders allowed |
|------------|----------------|
| `inactive` | ❌ No           |
| `pending`  | ❌ No           |
| `accepted` | ✅ Yes          |
| `rejected` | ❌ No           |

### Cooldown after `rejected`
- User may resend the pairing request only after **24 hours**.
- Logic: `last_rejected_at < NOW() - INTERVAL '24h'`.
- Rationale: The supplier may have clicked Reject by accident; 24h is sufficient cooldown.

---

## 3. Procurement Order

### Definition

A request to a Supplier for one or more products. Supports negotiation via revisions.

### Order Statuses


draft → submitted → in_negotiation → agreed → completed
↘ cancelled (before agreed)

| Status            | Description                                                       |
|-------------------|-------------------------------------------------------------------|
| `draft`           | User created, not yet sent. Editable.                             |
| `submitted`       | Sent to supplier. First revision (v1) created. Immutable.         |
| `in_negotiation`  | Supplier proposed changes (new revision) or the user rejected a proposal. |
| `agreed`          | Both parties accepted a revision. Order frozen.                   |
| `completed`       | User manually marks as completed after physical delivery.         |
| `cancelled`       | Cancelled before `agreed`.                                        |

### Key Fields
- `id` (UUID v4)
- `supplier_id`
- `owner_type`, `owner_id`
- `status`
- `source_session_id` (nullable – references Inventory Session if created from inventory data)
- `notes`
- `created_by` (user_id)
- `submitted_at`
- `agreed_at`
- `completed_at`
- `created_at`

---

## 4. Order Revision (Negotiation Model)

### Problem
In real negotiations, order contents change:
- supplier may offer different quantities (“only 50 instead of 100”)
- supplier may propose a substitute product (“no brand X, but brand Y at discount”)
- The user must accept or counter‑propose.

### Solution: Append‑only revisions
Every change to the order items creates a **new revision**. Old revisions are never deleted. Full negotiation history is preserved.

### Order Revision

One version of the order item set, proposed by one party.

**Fields:**
- `id` (UUID v4)
- `order_id`
- `version` (integer, 1‑based, `UNIQUE(order_id, version)`)
- `proposed_by` (user | supplier)
- `proposed_by_user_id` (nullable, if `proposed_by = user`)
- `status` (pending_review | accepted | superseded)
- `notes`
- `created_at`

**Revision statuses:**
| Status           | Description                                                       |
|------------------|-------------------------------------------------------------------|
| `pending_review` | Awaiting response from the other party.                           |
| `accepted`       | Other party accepted → order status becomes `agreed`.             |
| `superseded`     | Replaced by a newer revision (when a counter‑proposal is made).   |

### Order Revision Item

One item within a specific revision.

**Fields:**
- `id` (UUID v4)
- `revision_id`
- `product_variant_id`
- `quantity`
- `unit`
- `price` (optional – supplier may provide)
- `change_type` (original | quantity_changed | added | removed)
- `previous_item_id` (nullable – references the item from previous revision)
- `notes`

**`change_type`** allows diff between revisions:
- `original` – unchanged item.
- `quantity_changed` – same product, different quantity.
- `added` – new item in this revision.
- `removed` – item deleted in this revision.

### Negotiation Flow Example
1. User creates order → **Revision v1** (`proposed_by = user`). Order status → `submitted`.
2. Supplier sees order and proposes changes → **Revision v2** (`proposed_by = supplier`). v1 becomes `superseded`. Order status → `in_negotiation`.
3. User accepts v2 → v2 status `accepted`, order status → `agreed`.
4. If user rejects v2, supplier may propose another revision (v3).

---

## 5. Order Messages (Text Chat)

### Definition
Text exchange between the user and supplier within an order. Does not change order items – only comments.

### Characteristics
- Belongs to an order.
- Optionally linked to a specific revision item.
- Supplier writes via token link (no login).
- User writes via normal authentication.

### Fields
- `id` (UUID v4)
- `order_id`
- `author_type` (user | supplier)
- `author_user_id` (nullable, if `author_type = user`)
- `revision_item_id` (nullable – comment on a specific item)
- `body`
- `created_at`

---

## 6. Supplier Action Tokens

### Purpose
One‑time signed tokens embedded in email links, allowing suppliers to perform actions without login or registration.

### Supported Actions

| `action_type`       | Description                                      |
|---------------------|--------------------------------------------------|
| `accept_pairing`    | Accept pairing request.                         |
| `reject_pairing`    | Reject pairing request.                         |
| `accept_revision`   | Accept a proposed revision (order → `agreed`).  |
| `reject_revision`   | Reject a revision (supplier may propose another).|
| `propose_revision`  | Open form to propose changes to an order.       |
| `add_message`       | Leave a text comment.                           |

### Characteristics
- TTL: **7 days**.
- One‑time: after use, `used_at` is set. Repeated use → HTTP 410 Gone.
- Unique token (`TEXT UNIQUE`).

### Fields
- `id` (UUID v4)
- `token` (unique)
- `supplier_id`
- `order_id` (nullable)
- `revision_id` (nullable)
- `action_type`
- `expires_at`
- `used_at` (nullable)
- `created_at`

---

## 7. Procurement Lifecycle (Full Flow)

1. **Inventory session finished** → system identifies low stock (or user starts manually).
2. User creates **Procurement Order** (`draft`). `source_session_id` may be set.
3. User adds items (via Product Variants). First revision (v1) created implicitly.
4. User submits order → status `submitted`. Email sent to the supplier with token links to view the order.
5. Supplier reviews. May:
   - Accept order as‑is → order `agreed` (no negotiation needed).
   - Propose changes → creates new revision, order → `in_negotiation`. User notified.
6. The user may accept or reject the proposal. If rejected, the supplier may propose another revision.
7. Once a revision is accepted → order status is `agreed`. No further changes.
8. Physical delivery happens outside the system.
9. User marks order `completed`.

### Important
> Procurement **does NOT automatically update inventory**.  
> Stock updates happen only through the next physical inventory session.

---

## 8. No‑Reply Email Handling

Before sending any email to a supplier:
- Check contact email against known no‑reply prefixes (`noreply`, `no-reply`, `donotreply`, `notifications`) and known auto‑responder domains.
- If match → set `is_noreply = TRUE`, **do not send email**, show warning to user, suggest entering a different email.

This is a **warning, not a hard block** – user can override by entering a different address.

---

## 9. Ownership & Aggregation

Supported ownership scopes:
- Team
- Organization

Ownership defines visibility, aggregation, and analytics.

---

## 10. Relations to Other Domains

- **Products Domain** – references Product Variants in order items. Synchronous existence check when adding items.
- **Inventory Domain** – may be the source of demand (`source_session_id`).
- **Users / Teams / Organizations Domain** – ownership and permissions.
- **Notifications Domain** – email delivery to suppliers, push/email to users.
- **Analytics Domain** (future) – aggregation of orders, supplier performance.

---

## Explicitly Out of Scope

- Payments and invoicing
- Accounting
- Enforced pricing (prices are informational only)
- Supplier authentication / registration
- Automatic inventory update after delivery

---

## Key Architectural Principles

> Procurement is operational, not financial.  
> Suppliers are external – they never log in.  
> Every change to order content creates a new revision (append‑only).  
> Full negotiation history is preserved.  
> Email is the primary channel for suppliers.  
> Inventory drives orders, not the opposite.  
> Ownership is explicit and immutable.

This domain is designed to remain lightweight, supplier‑friendly, and extensible for future AI‑agent integration (agent‑ready API with clean revision history).
