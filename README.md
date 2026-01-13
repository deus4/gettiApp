# GETTI
_(formerly poirot.systems)_

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
- centralized product catalog,
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
- maintaining the global catalog,
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
- realtime listeners,
- implicit side effects,
- automatic synchronization.

---

### 2.3 Web Application (As-Is)

Stack: HTML / CSS / JS / PHP

Responsibilities:
- authentication,
- team management,
- product management,
- inventory finalization,
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
- partial normalization,
- global reference data.

SQL is **not the source of truth**.

---

## 3. Core Domain Concepts

### 3.1 Products

**Global Product Catalog (GPC):**
- alcohol,
- non-alcoholic beverages,
- tobacco,
- moderated,
- shared across teams.

**Local products:**
- team-specific,
- recipes and pre-batches,
- miscellaneous items,
- not moderated.

---

### 3.2 Recipes (Pre-batches)

- recipes are structured ingredient lists,
- used in inventory,
- automatically broken down into components.

---

### 3.3 Inventories

Inventory is tied to:
- team,
- location,
- time.

Includes:
- weighted items,
- unit-counted items,
- recipes.

---

## 4. Product Moderation Logic

### Scenario A — new product
1. User creates a product.
2. If not marked as `Other`, it goes to moderation.
3. Moderator:
   - **accepts** → adds to GPC,
   - **assigns** → links to existing product,
   - **ignores** → product stays local.

---

### Scenario B — editing a global product
1. User edits a GPC product.
2. A local override is created.
3. Changes go to moderation.
4. Moderator decides how to resolve.

---

## 5. Target Architecture (To-Be)

### 5.1 Key Shift

❌ Firebase as system brain  
✅ Backend + SQL as source of truth

Firebase temporarily remains for:
- authentication,
- push notifications,
- backward compatibility.

---

### 5.2 Target Architecture
Mobile / Web
↓
Backend API
↓
PostgreSQL
↓
Async workers (sync, moderation, exports)

Mobile app:
- remains offline-first,
- syncs via explicit APIs,
- does not rely on Firebase listeners.

---

### 5.3 Planned Stack

Backend (Target):
- Node.js (TypeScript) — primary backend API
- PostgreSQL — source of truth
- Async workers for sync, moderation, exports

Optional / Future:
- Python services for analytics or ML workloads

**Database**
- PostgreSQL
- explicit schema
- migrations

**Infrastructure**
- Zone.ee
- backups (3 months retention)

---

## 6. How to Read Project Documentation

- `README.md` — system overview
- `*/DDD_OVERVIEW.md` — domain map
- `*/...Domain.md` — domain specifications

When changing business logic,
update domain documents first.
