# Platform & Internal Roles Domain — Domain Design Draft

## Purpose

The Platform Domain (historically and internally called **Diabolics**) describes the internal infrastructure available **exclusively to the Getti team**.

Its goals:
- manage the global product catalogue
- moderate user‑submitted content (products, attributes)
- administer customers (Teams, Organisations, Users)
- monitor system health and logs

The Platform Domain is **completely isolated** from client‑facing parts of the system.  
External users, clients, and suppliers **never have access** to this domain.

---

## Architectural Isolation

| Aspect          | Client‑facing part          | Platform (Diabolics)               |
|-----------------|-----------------------------|-------------------------------------|
| Authentication  | JWT (client)                | Separate JWT                       |
| Users table     | `users`                     | `platform_users`                   |
| Roles           | Team‑scoped (`owner`, `manager`, etc.) | Platform‑scoped (`super_admin`, `moderator`, `support`) |
| Access          | Web / iPad                  | Internal web only                   |
| Registration    | Self‑service + invitation   | Manual creation only                |

- `platform_users` has **no foreign keys** to `organisations` or `teams` – intentional full isolation.
- Platform JWT is not accepted by client APIs; client JWT is not accepted by Platform APIs.

---

## Platform User

### Definition

A Getti team member with access to Diabolics.

### Characteristics
- Created manually (no self‑registration).
- Separate authentication.
- Exactly one platform role.
- Not a member of any client Organization.

### Key Fields
- `id` (UUID v4)
- `email` (CITEXT, unique)
- `name`
- `password_hash`
- `role` (super_admin | moderator | support)
- `is_active`
- `created_at`
- `last_login_at`

---

## Platform Roles

### `super_admin`
Full access to everything:
- manage `platform_users`
- all `moderator` and `support` permissions
- system settings
- notification templates (TO‑BE)

### `moderator`
Work with the product catalogue and moderation queue:
- moderation queue (view, approve, assign, ignore)
- edit global catalogue (any product)
- add products directly (bypassing queue)
- upload and process images

**No access** to client inventory, orders, teams, or user data.

### `support`
Work with customers:
- view Teams and Organisations (read‑only)
- view Users
- basic support operations

**No access** to product moderation or system settings.

---

## Functional Sections of Diabolics

### SEARCH – Global Catalogue

Primary tool for working with the moderated product database.

**Capabilities:**
- search by name, barcode, ID, type, subtype
- edit any product attributes (label, variant, category, barcode, tare weights, volume)
- add products directly (bypassing moderation queue) via dedicated forms (e.g., "ADD NEW ALC ROW", "ADD NEW SD ROW", "ADD NEW CIGA ROW")

**Access:** `moderator`, `super_admin`

---

### MODERATE – Moderation Queue

Central tool for processing user‑submitted content.

#### Request Types

| Type            | Description                                                       |
|-----------------|-------------------------------------------------------------------|
| `new_product`   | User created a new product from scratch.                         |
| `edit_product`  | User proposed changes to an existing global product.             |

#### Request Card Contains
- source Team, request type, date
- product name as submitted by the user
- all attributes (origin, type, subtype, volume, alcohol %, full tare, empty tare, etc.)
- for `edit_product`: original vs proposed changes (diff view)

#### Moderator Decisions

| Action         | Description                                                                          | User notification |
|----------------|--------------------------------------------------------------------------------------|-------------------|
| **Accept**     | Product becomes global (`source = 'global'`).                                        | Push              |
| **Assign to**  | Link the user’s product to an existing global product (by ID). Useful for duplicates.    | Push              |
| **Ignore**     | Reject the request. Product remains local (`source = 'team'`, category `Other`).     | Push              |

**Access:** `moderator`, `super_admin`

---

### TEAMS / BARS – Customer View

Read‑only view of client structures.

- list of Organisations and Teams
- view team members (Users)
- basic support operations (e.g., resetting permissions)

**Access:** `support`, `super_admin`

---

### SUPPLIERS

View suppliers and their pairing status.  
Extended as the Procurement domain evolves.

**Access:** `support`, `super_admin`

---

### LOGS – System Logs

- iPad logs (submitted during `Save & Send`)
- failed notifications (from Notifications Domain)
- system errors

**Access:** `super_admin`

---

### Statistics Dashboard

| Metric          | Description                                      |
|-----------------|--------------------------------------------------|
| UPLOAD IMAGES   | Number of uploaded product images               |
| TYPES           | Categories present in the global catalogue        |
| MODERATE        | Products pending moderation                     |
| TOTAL           | Total products in global catalogue                |

**Access:** `moderator`, `super_admin`

---

## Audit Log

Every moderation action is logged in `platform_audit_log`:

- `platform_user_id` – who performed the action
- `action` – e.g., `product.accepted`, `product.rejected`, `product.assigned`, `product.edited`
- `entity_type`, `entity_id` – what was affected
- `created_at`
- `payload` (JSONB) – before/after snapshot for sensitive changes

**Retention:** Logs are kept for 3 months, then automatically deleted.

---

## Image Management

### AS‑IS
- 30,000+ images in the database
- Manual processing (background removal → transparent PNG)
- Upload via Diabolics

### TO‑BE (Future)
- AI‑assisted background removal
- Batch upload
- Quality validation

This is a separate task and does not block the first release.

---

## Security

- Diabolics on a separate domain (e.g., `diabolics.gettiapp.com`).
- IP whitelist – **TO‑BE** (recommended before scaling the team).
- No self‑registration of `platform_users`.
- All actions logged.

---

## Relations to Other Domains

- **Products Domain** – moderation queue, global catalogue editing.
- **Notifications Domain** – alerts for failed notifications.
- **Users / Teams Domain** – read‑only view of client structures.
- **Inventory Domain** – iPad logs.
- **Procurement Domain** – supplier view (read‑only).

---

## Out of Scope

- Client‑facing features.
- Any form of external moderation (suppliers, brand managers, etc.) – **never**.
- Billing or payment processing.

---

## Key Architectural Principles

> Diabolics is for the Getti team only.  
> Absolute isolation – separate auth, separate users.  
> Moderators see products, not client inventory or orders.  
> Every action is logged.  
> External moderators will never exist.

This domain is the internal backbone for maintaining data quality and system health. It must remain secure, auditable, and isolated from client systems.
