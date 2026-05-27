# Development Roadmap (Final)

This document outlines the approximate development plan
based on the current domain model
(Products, Inventory, Recipes, Users, Procurement, Moderation).

The goal is to build a **stable core product**
that can later be extended to enterprise features,
marketplaces, and creator economy **without rewriting the architecture**.

---

## 0. Planning Principles

### What is considered MVP

MVP = a closed operational loop:

**Products → Inventory → Orders → Basic analytics**

**Not in MVP:**
- Marketplace
- Creator economy
- NFT / licensing
- Billing and payments
- Complex enterprise hierarchies (parent/child orgs)

### Key assumptions

- Small team
- Backend + iPad / Web
- Offline‑first (iPad) – mandatory
- Historical data integrity > UX polish
- Domain contracts > UI speed

---

## Phase 0 — Architecture & Domain Specification

**Goal:** Freeze the domain model so that database and API changes do not break later.

**Tasks:**
- Products Domain (EN)
- Inventory Domain (EN)
- Recipes Domain (EN)
- Users / Teams / Organisations Domain (EN)
- Procurement Domain (EN)
- Business Processes AS‑IS → TO‑BE (EN)
- `00_core_ownership.md` (ownership & identity layer)

**Deliverable:** Architectural constitution – ownership, snapshot, lifecycle rules.

**Estimated time:** 1–2 weeks (already completed)

---

## Phase 1 — Core Backend & Data Model

### 1.1 Products + Moderation

**Tasks:**
- Product Label / Product Variant
- Category and Category Attributes
- `Other` category (local products, no moderation)
- Moderation queue
- Approve / Assign / Ignore logic
- Global vs local products
- Snapshot‑safe references

**Time:** 6–8 weeks

### 1.2 Users / Teams / Organisations

**Tasks:**
- User
- Organisation (flat, no hierarchy)
- Team
- Membership & Roles
- Ownership resolution

**Time:** ~2 weeks

### 1.3 Inventory Core (without recipes)

**Tasks:**
- Inventory Session lifecycle (`active` → `pending_finalization` → `finished`)
- Ownership (Team) + optional Location
- Inventory Items
- Inventory Snapshots (frozen product attributes)
- Previous quantity, diff, usage calculation
- `Save & Send` with idempotent retry
- Read‑only after finalisation

**Time:** 4–5 weeks

### 1.4 Backend Re‑platforming (migration from Firebase)

**Tasks:**
- Move away from Firebase as a source of truth
- New Node.js API + PostgreSQL schema
- **Data migration script with verification** (count parity between Firebase and PostgreSQL)
- Keep Firebase temporarily for auth/push (to be replaced later)

**Time:** 5–6 weeks (including migration script)

### 1.5 Mobile App Refactoring (parallel)

**Tasks:**
- Rewrite sync logic: from Firebase listeners to explicit API calls
- Preserve offline-first (local storage, retry queue)
- BLE scale integration re-test
- iPad UI adjustments for new API responses

**Time:** 3–4 weeks (can start mid-Phase 1)

### 1.6 Automated API Tests (snapshot integrity)

**Tasks:**
- Test suite for inventory lifecycle (start, save, finish, export)
- Snapshot integrity tests (change product after session → old sessions unchanged)
- Idempotency tests for `Save & Send`

**Time:** 1–2 weeks (parallel, can overlap with 1.3)

**Phase 1 total:** **12–14 weeks** (realistic, including all subtasks)

**Checkpoint after Phase 1:**  
- No Firebase writes in new code (reads allowed temporarily)
- Inventory session can be started, saved, finished, and exported to PDF
- Snapshot test: product weight changed in catalogue → old sessions still show original weight

---

## Phase 2 — Recipes & Inventory Breakdown (TO‑BE)

**Note:** This phase is **not yet implemented** (legacy limitations). It is the **highest priority near‑future feature**.

### 2.1 Recipes Core

**Tasks:**
- Recipe (abstract)
- Recipe Version (immutable, auto‑increment)
- Ingredients (Product Variant only)
- Ownership & visibility (Team / Organisation / User)

**Time:** ~2 weeks

### 2.2 Inventory → Recipe Breakdown

**Tasks:**
- Linear scaling for volumetric ingredients (ml → ml)
- Support for piece‑based ingredients (e.g., eggs, limes) using simple integer rounding
- Store `inventory_item_recipe_components` for audit
- Handle case where recipe version changes after session start (use version at count time)
- Validation: count the same pre‑batch twice → identical breakdown

**Time:** 4–5 weeks

**Phase 2 total:** 6–7 weeks

### 2.3 Pilot Validation (with real bars)

**Tasks:**
- Deploy to 2–3 pilot venues
- Collect feedback on breakdown accuracy
- Fix edge cases (non‑linear scaling for heterogeneous units)

**Time:** 2 weeks (overlaps with 2.2)

**Checkpoint after Phase 2:**  
- At least 3 real bar recipes entered and counted in inventory
- Breakdown results match manual calculation within acceptable rounding error (e.g., 1 ml or 1 g)

---

## Phase 3 — Procurement (Orders)

### 3.1 Supplier Management

**Tasks:**
- Supplier entity
- Pairing lifecycle (`inactive` → `pending` → `accepted`/`rejected`)
- No‑reply detection, cooldown after reject
- Email‑first with token links

**Time:** 1–2 weeks

### 3.2 Procurement Orders

**Tasks:**
- Procurement Order (draft → submitted → in_negotiation → agreed → completed)
- Order Items (Product Variant references)
- Create order from inventory data (low stock)
- Email to supplier with token link
- Append‑only revisions (negotiation history)

**Time:** 2–3 weeks

**Phase 3 total:** 3–5 weeks

**Checkpoint:** Can a manager close the loop (inventory → order → inventory) daily?

---

## Phase 4 — Basic Analytics (two‑step)

### 4.1 Core analytics (MVP)

**Tasks:**
- Difference (current vs previous) per product variant
- Basic consumption (usage = previous + received - current)
- CSV / XLSX export
- Aggregation by Team / Location

**Time:** 2 weeks

### 4.2 Extended analytics (optional near‑future)

**Tasks:**
- Waste / variance reports
- Filters by date range, category, brand
- Trends (weekly/monthly comparison)

**Time:** additional 2–3 weeks (TO‑BE, may be deferred)

**Phase 4 total (MVP):** 2 weeks  
**Extended version:** +2–3 weeks

**Checkpoint:** Customers show willingness to pay (or upgrade) based on core analytics.

---

## Phase 5 — Integrations & Stabilisation

### 5.1 POS read‑only (CompuCash)

**Tasks:**
- Reverse‑engineer CompuCash export format (CSV/XML)
- Build Anti‑Corruption Layer (ACL)
- Map external product codes to `product_variant_id`
- **Time:** 4–6 weeks (complexity was underestimated)

### 5.2 DevOps & Infrastructure

**Tasks:**
- Staging and production environments (Zone.ee)
- CI/CD pipeline (GitHub Actions or similar)
- Automated backups (3 months retention)
- Monitoring & alerting (basic)

**Time:** 1–2 weeks

### 5.3 API Documentation & Quality

**Tasks:**
- Generate OpenAPI spec from code
- Internal audit tools (view logs, failed notifications)

**Time:** 1 week

**Phase 5 total:** 6–9 weeks

---

## Total Estimated Time (Phases 1–5)

| Phase | Duration (weeks) |
|-------|------------------|
| Phase 0 | 1–2 (done) |
| Phase 1 | 12–14 |
| Phase 2 | 8–9 (including pilot) |
| Phase 3 | 3–5 |
| Phase 4 (MVP) | 2 |
| Phase 5 | 6–9 |

**Total:** **~31–39 weeks (8–10 months)** – realistic with buffer.

---

## Definition of Done (Checkpoint Criteria)

**After Phase 1:**
- [ ] No Firebase writes in new code (reads allowed temporarily)
- [ ] Inventory session can be started, saved, finished, and exported to PDF
- [ ] Snapshot test: product weight changed in catalogue → old sessions still show original weight

**After Phase 2:**
- [ ] At least 3 real bar recipes entered and counted in inventory
- [ ] Breakdown results match manual calculation within rounding error (1 ml or 1 g)

**After Phase 3:**
- [ ] Orders created from inventory low‑stock suggestions are used daily by at least 3 teams

**After Phase 4:**
- [ ] Core analytics (difference, basic usage) available for all teams

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-------------|
| Migration from Firebase takes longer than expected | High | Keep Firebase reads as fallback; migrate one domain at a time |
| Mobile sync breaks with new API | High | Run parallel tests with legacy and new API before cutover |
| Recipe breakdown logic becomes complex for piece‑based ingredients | Medium | Limit v1 to linear scaling; add unit conversion later |
| QA effort underestimated | Medium | Automate snapshot integrity tests (Phase 1.6) |
| POS integration (CompuCash) has undocumented format | High | Allocate 4–6 weeks; consider outsourcing if blocked |
| Infrastructure setup delays | Low | Use Zone.ee managed services; keep it simple at start |

---

## Minimal Team

- Product / Domain Architect
- Senior Backend Engineer
- Frontend / iPad Developer
- QA (part‑time, but dedicated during Phase 1 and 2)
- Moderator (part‑time, internal)
- DevOps (can be shared with backend in early phases)

---

## Critical Risks (repeated for emphasis)

- Simplifying the domain for speed
- Breaking snapshot / immutability rules
- Adding marketplace or billing too early
- Treating POS as a dependency instead of an integration

---

## Future Horizons (Strategic, no near‑term commitment)

The following are **not part of the core roadmap** but represent long‑term opportunities. They will be implemented as **separate bounded contexts** when the time comes.

- **POS integrations (full)** – CompuCash, Poster, Micros
- **Delivery merchant apps** – embedding inventory & recipes into Wolt, Bolt, Uber Eats vendor apps
- **NFT / RWA for recipes** – creator economy: licensing, royalties, tokenised ownership
- **AI agents** – automatic reordering, supplier negotiation, anomaly detection (requires agent‑ready APIs – we are designing for it)

These directions are documented to show architectural foresight, not immediate delivery.

---

## Key Milestones (with realistic dates)

| Milestone | Success Criteria | Estimated completion (from start of Phase 1) |
|-----------|------------------|-----------------------------------------------|
| After Phase 1 | Inventory works offline, reports exportable, no schema changes expected for 1 year | 3–3.5 months |
| After Phase 2 | Recipe breakdown validated with pilot bars | 5–6 months |
| After Phase 3 | Orders used daily by multiple teams | 6.5–7.5 months |
| After Phase 4 | Core analytics live | 7–8 months |
| After Phase 5 | Production‑ready with POS and DevOps | 8–10 months |
