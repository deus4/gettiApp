# Current System (AS‑IS)

*This document is a **brief summary** of the system as it works today. For a full description, refer to the [`README.md`](./README.md) and the domain‑specific documents in [`DDD/`](./DDD).*

The current production system is **live and generating revenue**, but its architecture limits further growth. Below are the key facts and components.

---

## Stack & Data Flow

- **Backend / real‑time DB** – Firebase Realtime Database (source of truth, sync engine, offline cache, event bus).
- **Web dashboard** – PHP + HTML/CSS/JS, reads/writes directly to Firebase (and partially to MySQL).
- **Mobile (iPad) app** – Swift, offline‑first, stores counts locally, syncs via Firebase listeners.
- **Relational storage** – MySQL (partial replica, used for global reference data and some reporting).
- **Authentication** – Firebase Auth.
- **Push notifications** – FCM (Firebase Cloud Messaging).

---

## Live Functionality (Used by Paying Customers)

| Module              | Status                                                                 |
|---------------------|------------------------------------------------------------------------|
| **Inventory counting** | Fully functional. Offline‑first, BLE scale support, `Save & Send` → manager review → `Finish`. |
| **Product catalog**     | Global + local products. The moderation queue exists but is manual.        |
| **Reports**             | PDF / XLSX exports from finished inventory sessions.                   |
| **Teams & users**       | Basic multi‑team support, roles (owner, manager, bartender).           |
| **Recipes**             | Structured recipe data exists, **but not yet used in inventory**.      |
| **Procurement (orders)**| External email‑only form – **not integrated** with inventory.          |
| **Analytics**           | No customer‑facing analytics. Internal logs only.                      |

---

## Known Limitations (Operational Impact)

- **Inventory session finalisation** – can be inconsistent under network flakiness.
- **Historical integrity** – product changes can alter past reports (no snapshots).
- **No real‑time stock aggregation** across locations or teams.
- **Scaling** – Firebase performance degrades beyond ~10K items per session.
- **Maintenance** – changes require touching three codebases (PHP, Swift, and Firebase rules).

---

## What Works Well (To Preserve)

- **Offline‑first inventory** – bartenders can count without internet.
- **BLE scale integration** – accurate weight‑based counting.
- **Product catalog moderation** – human‑in‑the‑loop quality control.
- **Core UX for inventory** – fast, minimal taps.

---

## Why Rewrite (Very Briefly)

The current system cannot support:
- Procurement (integrated orders)
- Recipe‑based inventory breakdown
- Immutable historical snapshots
- API for third‑party integrations (POS, delivery apps)
- Multi-organisation analytics

*For the full list of problems, see [`KNOWN_PROBLEMS.md`](./KNOWN_PROBLEMS.md).*

---

## Migration Strategy (Overview)

We are **not** building from scratch – we are **extracting** bounded contexts:

1. **Phase 1** – New PostgreSQL schema + Node.js API (products, inventory, users).
2. **Phase 2** – Recipes & procurement.
3. **Phase 3** – Retire Firebase (keep only auth/push temporarily).

*Detailed roadmap is in [`ROADMAP.md`](./ROADMAP.md).*
