# GETTI
*(formerly poirot.systems)*

## System Overview & Developer Introduction
### Current State → Target Architecture

---

## 0. What This System Is

**Getti** is an **inventory management system for HoReCa**,
primarily designed for bars and restaurants.

The system solves real operational problems:
- fast and accurate inventory counting,
- reduction of human errors,
- offline-first operation on site,
- centralised product catalogue,
- POS integrations.

The product was originally built **Firebase-first**
to quickly launch an MVP and validate hypotheses.
The system is **live**, has **paying customers**,
but the current architecture **does not scale well**.

**Current goal:**  
Gradually rebuild the backend and data layer
while preserving business logic and UX,
and move away from legacy Firebase usage.

---

## 1. System Participants

### 1.1 Users (Clients)

**Typical roles:**
- **Owner / Manager** — team, product, and inventory management
- **Bartender / Staff** — inventory execution
- *(future)* accounting / financial role

**Interfaces:**
- **Mobile / iPad app** — inventory (offline-first)
- **Web app** — management, finalization, reports

---

### 1.2 Moderator (Internal)

Getti internal staff only.

**Responsibilities:**
- moderating new products,
- maintaining the global catalogue,
- fixing data issues,
- duplicate prevention.

Moderation tools are **not available to clients**.

---

### 1.3 Developer

Developer responsibilities:
- understand the domain,
- work with backend and data models,
- **do not extend Firebase legacy logic**,
- participate in gradual architecture migration.

---

## 2. Current Architecture (As-Is)

### 2.1 High-Level View
Web  → Firebase ← Mobile
↓
SQL (partial replica)

Firebase currently acts as:
- authentication provider,
- database,
- event bus,
- sync mechanism,
- offline support.

This was acceptable for MVP,
but is now an architectural bottleneck.

---

### 2.2 Firebase Responsibilities

Firebase stores:
- users and teams,
- locations,
- products (global and local),
- moderation queue,
- inventories,
- suppliers and orders,
- legacy functionality.

A lot of logic relies on:
- real-time listeners,
- implicit side effects,
- automatic synchronisation.

---

### 2.3 Web Application (As-Is)

Stack: HTML / CSS / JS / PHP

Responsibilities:
- authentication,
- team management,
- product management,
- inventory finalisation,
- PDF / XLSX exports.

Technically:
- writes directly to Firebase,
- partially reads from SQL.

---

### 2.4 Mobile / iPad Application (As-Is)

Stack: Swift

Responsibilities:
- offline inventory counting,
- BLE scale integration,
- local storage,
- data submission via **Save & Send**.

Offline-first is a **core feature** and must be preserved.

---

### 2.5 SQL Database (As-Is)

Stack: MySQL

Used for:
- partial normalisation,
- global reference data.

SQL is **not the source of truth**.

---

## 3. Domain‑Driven Design (DDD)

The architecture is defined through **Domain‑Driven Design**.
Each domain has its own specification in the [`DDD/`](./DDD) folder.

### Core domains

| Domain               | Description                                                                 |
| -------------------- | --------------------------------------------------------------------------- |
| **Products**         | Global product catalogue, local products, moderation, categories & attributes |
| **Inventory**        | Stock‑taking sessions, offline‑first, weight‑based counting, snapshots      |
| **Recipes**          | Structured recipes, versioning, breakdown into product variants             |
| **Users & Teams**    | Authentication, roles, memberships, invitations                             |
| **Procurement**      | Supplier management, ordering, negotiation (revisions), email‑first         |
| **Notifications**    | Push/email delivery, retry logic, token‑based actions for suppliers       |
| **Platform (Internal)** | Moderation tools, global catalogue management, internal roles (Diabolics)  |

**Key architectural document:** [`DDD/00_core_ownership.md`](./DDD/00_core_ownership.md)
– defines the ownership & identity layer used by all domains.

---

## 4. Target Architecture (To‑Be)

### 4.1 Key Shift

❌ Firebase as system brain  
✅ **Backend + PostgreSQL as source of truth**

Firebase temporarily remains for:
- authentication (until migrated),
- push notifications (FCM),
- backward compatibility.

---

### 4.2 Target Architecture

Mobile / Web
↓
Backend API (Node.js)
↓
PostgreSQL (source of truth)
↓
Async workers (sync, moderation, exports)


**Mobile app:**
- remains offline‑first,
- syncs via explicit APIs,
- does not rely on Firebase listeners.

---

### 4.3 Planned Stack

| Component          | Technology                                          |
| ------------------ | --------------------------------------------------- |
| Backend            | Node.js (JavaScript, strict mode, no TypeScript)    |
| Database           | PostgreSQL (explicit schema, migrations)            |
| Async workers      | Node.js (bull or similar)                           |
| Infrastructure     | Zone.ee                                             |
| Backups            | 3 months retention                                 |

**No TypeScript** – we use disciplined JavaScript with `"use strict"` and explicit contracts.

---

## 5. Business Processes & Roadmap

- **Business processes** (inventory, procurement, moderation, invitations, recipes) are described in [`BUSINESS_PROCESSES.md`](./BUSINESS_PROCESSES.md).
- **Development roadmap** is available in [`ROADMAP.md`](./ROADMAP.md).

---

## 6. How to Read Project Documentation

1. `README.md` – this file, system overview.
2. [`DDD/00_core_ownership.md`](./DDD/00_core_ownership.md) – fundamental ownership & identity rules.
3. Domain files in [`DDD/`](./DDD) (e.g., `1_productsDomain.md`, `2_inventoryDomain.md`, etc.).
4. [`BUSINESS_PROCESSES.md`](./BUSINESS_PROCESSES.md) – end‑to‑end workflows.
5. [`ROADMAP.md`](./ROADMAP.md) – milestones and timeline.

**When changing business logic, update the relevant domain document first.**

---

## 7. Developer Notes

- **Never extend Firebase legacy logic** – new features go into the new backend.
- **Preserve offline‑first** – the mobile app must work without internet.
- **Respect domain boundaries** – do not leak logic across domains.
- **Ownership rules** (see `00_core_ownership.md`) are mandatory for all domains.
- **When in doubt, ask** – the documentation is a living artifact.
