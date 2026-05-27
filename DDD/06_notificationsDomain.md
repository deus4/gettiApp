# Notifications Domain — Domain Design Draft

## Purpose

The Notifications Domain defines the cross‑cutting notification infrastructure used by all other domains.

Its goals:
- single point of delivery for all system notifications
- support for multiple delivery channels
- reliable delivery with retry logic
- auditable history of notifications
- extensibility without changing business domains

The Notifications Domain **contains no business logic**.  
It is an **infrastructure service**.  
Decisions about *when* to send a notification are made in the respective domains.  
The Notifications Domain is responsible only for *how*.

---

## Delivery Channels

### Supported Channels (AS‑IS / TO‑BE)

| Channel   | Recipient               | Usage                                                |
|-----------|-------------------------|------------------------------------------------------|
| Push      | User (iOS)              | Operational events, reminders                        |
| Email     | User                    | Confirmations, reports, invitations                  |
| Email     | Supplier                | Pairing requests, orders, status changes             |
| In‑app    | User (web dashboard)    | Dashboard notifications (TO‑BE, not in first release) |

### Principles
- Push – via APNs (Apple Push Notification service).
- Email – via external SMTP / transactional email provider (Postmark, Resend, or similar).
- In‑app notifications – TO‑BE, not in first release.
- SMS – out of scope.

---

## Core Concepts

### NotificationEvent

An event is a fact that occurred in another domain and requires a notification.

An event contains:
- type (`notification_type`)
- source (`source_domain`, `source_entity_id`)
- recipients (`recipients`)
- context (`payload`)

### NotificationRecord

Every notification attempt is recorded.

**Fields:**
- `id` (UUID)
- `notification_type` (string)
- `channel` (push | email | in_app)
- `recipient_type` (user | supplier | platform_user)
- `recipient_id` (optional)
- `recipient_email` / `device_token`
- `payload` (JSONB)
- `status` (pending | sent | failed | bounced)
- `attempts` (integer)
- `last_attempted_at`
- `sent_at`
- `source_domain`, `source_entity_id`
- `created_at`

### Retry Logic

- On failure, retry after: **5 minutes**, **30 minutes**, **2 hours**.
- After 3 failed attempts → status `failed`, alert in Diabolics (Platform Domain).
- Email bounce → status `bounced`.

---

## Event Registry by Domain

### Inventory Domain

| Event                             | Channel        | Recipient                | Trigger                                                      |
|-----------------------------------|----------------|--------------------------|--------------------------------------------------------------|
| `inventory.pending_finalization`  | Push + Email   | User (Owner / Manager)   | 24 hours after `Save & Send` without `Finish`                |
| `inventory.pending_reminder_2`    | Push + Email   | User (Owner / Manager)   | 48 hours after `Save & Send` without `Finish` (final reminder) |

**Rule:** First reminder sent once at 24h. Second and final reminder at 48h. No further reminders.

---

### Procurement Domain

| Event                              | Channel        | Recipient         | Trigger                                                       |
|------------------------------------|----------------|-------------------|---------------------------------------------------------------|
| `supplier.pairing_request_sent`    | Email          | Supplier          | User sent a pairing request                                   |
| `supplier.pairing_accepted`        | Push + Email   | User              | Supplier accepted pairing                                     |
| `supplier.pairing_rejected`        | Push + Email   | User              | Supplier rejected pairing                                     |
| `supplier.pairing_cooldown_expired`| Push           | User              | 24 hours after `rejected` (user may retry)                    |
| `supplier.no_reply_warning`        | In‑app / Email | User              | Supplier email looks like no‑reply (warning, not block)       |
| `order.submitted`                  | Email          | Supplier          | User submitted an order                                       |
| `order.accepted` (or revision accepted) | Push + Email | User          | Supplier accepted order / revision → order `agreed`           |
| `order.rejected` (or revision rejected) | Push + Email | User          | Supplier rejected a revision (order remains `in_negotiation`) |
| `order.message_received`           | Push + Email   | User              | Supplier left a comment on an order                           |

---

### Users / Teams Domain

| Event                    | Channel | Recipient           | Trigger                                      |
|--------------------------|---------|---------------------|----------------------------------------------|
| `user.invited`           | Email   | New user            | Manager added user to a team (invitation)    |
| `user.registration_code` | Email   | User                | Step of registration – confirmation code     |

---

### Products / Moderation Domain

| Event                           | Channel | Recipient    | Trigger                                                       |
|---------------------------------|---------|--------------|---------------------------------------------------------------|
| `product.moderation_accepted`   | Push    | User         | Moderator accepted product → global catalog                   |
| `product.moderation_rejected`   | Push    | User         | Moderator rejected product (`ignore`)                         |
| `product.moderation_assigned`   | Push    | User         | Moderator used `assign to` (linked to existing product)       |

---

## Email‑First for Suppliers

### Principle

Suppliers never register in the system.  
All interactions are via email, containing read‑only HTML pages or simple action buttons.

### Email Structure for Suppliers

Each email includes:
- context of the order or pairing request
- one or more action buttons (Accept / Reject / Comment)
- unique token in the URL to identify the action **without login**

### Token Links

Each action button uses a **one‑time signed token**:
- stored in `supplier_action_tokens` table
- TTL: **7 days**
- After use: `used_at` is set, token invalidated
- Repeated use → HTTP 410 Gone

### No‑Reply Validation

Before sending any email to a supplier, the system checks the contact email for no‑reply indicators:
- prefixes: `noreply`, `no-reply`, `donotreply`, `notifications`
- known auto‑responder domains

If no‑reply detected:
- email **is not sent**
- user receives a warning (in‑app or email)
- The user is prompted to enter a different contact email

**This is a warning, not a hard error** – the user may override by providing a different address.

---

## Notification Templates

All templates:
- stored as separate files (HTML + plain text fallback)
- support localisation (EN by default)
- contain **no business logic**
- managed via Diabolics (Platform Domain) – TO‑BE

---

## Relations to Other Domains

- **Inventory Domain** – triggers `inventory.pending_finalization` events
- **Procurement Domain** – triggers pairing and order events
- **Products Domain** – triggers moderation events
- **Users / Teams Domain** – triggers invitations and registration codes
- **Platform Domain (Diabolics)** – receives alerts for `failed` notifications

---

## Out of Scope

- Business logic of when to send notifications
- Decisions about who should receive a notification
- Marketing campaigns
- SMS
- In‑app notifications (TO‑BE, future release)

---

## Key Architectural Principles

> Notifications are infrastructure, not business logic.  
> Every attempt is recorded (audit trail).  
> Suppliers never log in – token links only.  
> No‑reply emails are a warning, not a block.  
> Retry logic is predictable and limited (max 3 attempts).  
> Templates contain no logic – only formatting.

This domain is designed to be a reliable, observable, and extensible notification backbone for the entire system.
