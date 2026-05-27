# Business Processes — AS-IS → TO-BE

This document describes real‑world business processes supported by the system **today (AS‑IS)** and those planned for the **near future (TO‑BE)**.  
Strategic long‑term directions (POS integrations, delivery merchant apps, NFT / creator economy) are listed separately at the end.

The focus is on **operational reality**, not UI details.  
Each process reflects how bars, restaurants, and enterprises actually work.

---

## 1. Business Onboarding

### AS‑IS

**Actors:** Business owner / manager, System.

**Process:**
1. User registers.
2. User creates an **Organisation**.
3. System automatically:
   - assigns the highest role (owner) to the creator within that Organisation,
   - creates the first **Team**.
4. User:
   - configures the Team,
   - creates **Locations** (optional),
   - invites staff via email.

**Key properties:**
- Zero‑friction start.
- No pre‑configuration required.
- Supports single‑bar and multi‑location businesses.

**Business value:** Fast adoption, low cognitive load, no sales assistance needed.

### TO‑BE (near future)
- Multiple Teams per Organisation.
- Cross‑team analytics.
- Enterprise‑level permissions.

---

## 2. Product Management & Moderation

### AS‑IS

**Actors:** Bar staff, managers, internal moderation team (Getti staff only).

**Process:**
1. User searches for a product.
2. If found → selects existing **Product Variant**.
3. If not found → creates a new product:
   - fills in label, variant, barcode, category, attributes.
4. Category decision:
   - If category is **standard** (e.g., Whisky, Wine, Soft Drink) → product is queued for moderation. It is saved locally (`source = 'team'`) until a decision is made.
   - If category is **`Other`** → product stays local, **never goes to moderation**, visible only within the same Team/Organization.
5. Moderator reviews the request (in Diabolics). May edit any field.
6. Moderator decisions:
   - **Accept** → product becomes global (`source = 'global'`). User notified.
   - **Assign to** existing product (by ID) → user’s product linked to existing one. User notified.
   - **Ignore** → product remains local. The category is automatically set to `Other` (if the user did not already set it manually). User notified.

**Business value:** Scalable product database, controlled data quality, foundation for analytics and integrations.

### TO‑BE (near future)
- Producer / supplier portals.
- Brand‑managed product catalogues.

---

## 3. Inventory Management (CORE PAID FEATURE)

### AS‑IS

**Actors:** Inventory staff (bartenders), managers, system.

**Process:**
1. User opens an **Inventory Session** (select Team and optional Location).
2. System works **offline‑first** (no internet required). BLE scale support is available.
3. User counts:
   - **Product Variants** (by units or weight).
4. User completes counting and taps **Save & Send** (requires internet). Session status becomes `pending_finalization`.
5. Manager reviews the session in the web app. May ask staff to continue (`is_continuation = TRUE`).
6. Manager clicks **Finish** → session becomes `finished`. Data frozen, read‑only.
7. Reports (PDF, XLSX) can be generated.

**Notifications:**
- 24 hours after `Save & Send` without `Finish` → push + email to manager (first reminder).
- 48 hours → second and final reminder.

**Key properties:**
- Offline‑first.
- Snapshot‑based (product attributes frozen at count time).
- Immutable historical data.

**Business value:** Time savings, reduced shrinkage, reliable stock visibility. This is the **primary monetised feature**.

### TO‑BE (near future)
- **Recipe breakdown** – user counts a pre‑batch volume, system automatically breaks it into ingredient items (planned, not yet live).
- Cross‑location aggregation.
- Automatic reorder suggestions.

---

## 4. Recipes & Pre‑batches

### AS‑IS

**Actors:** Bar staff, managers.

**Process:**
1. User creates a **Recipe** (e.g., Negroni pre‑batch).
2. Recipe consists of structured **Product Variants** with fixed quantities.
3. Recipe is **versioned** (versions are immutable).
4. **Current limitation:** Recipes are **not yet used in inventory** (due to legacy issues). This is planned for TO‑BE.

**Business value:** Structured data foundation for future inventory automation and creator economy.

### TO‑BE (near future)
- Inventory counts reference **Recipe Versions**.
- System breaks down recipe volume into component items automatically.
- Recipe sharing between Teams.
- Author attribution (ownership layer).

---

## 5. Orders & Procurement (Procurement Domain)

### AS‑IS (partial / legacy)

**Actors:** Manager, supplier, system.

**Process:**
1. User manually identifies missing products.
2. User creates an order (external system, not integrated).
3. Order sent via email.
4. Supplier responds manually.

**Limitations:** Not connected to inventory, no unified supplier state, and manual reconciliation.

### TO‑BE (planned, high priority)

**Actors:** Manager, supplier (email / web interface), system.

**Process:**
1. Inventory session finishes → system identifies low stock.
2. User creates **Procurement Order** directly from inventory data (or manually).
3. Order submitted → status `submitted`. Supplier receives an email with a token‑based link to view/accept/reject.
4. Supplier may propose changes → a new **revision** is created. Order status → `in_negotiation`.
5. User accepts or rejects the revision.
6. When agreed → status `agreed`.
7. After physical delivery, the user marks the order `completed`.

**Supplier lifecycle:**  
`inactive` → `pending` (pairing request sent) → `accepted` / `rejected`.

**Key properties:**
- Suppliers never need to log in – email + token links.
- Full revision history (append‑only).
- Order does **not** automatically update inventory – inventory is updated by the next physical count.

**Business value:** Closed operational loop (Inventory → Order → Inventory), high daily utility, free entry‑level feature, foundation for B2B integrations.

---

## 6. Supplier Management

### AS‑IS / TO‑BE (hybrid – mostly TO‑BE)

**Actors:** Manager, supplier, system.

**Process:**
1. User creates a **Supplier** profile (name, email, contact).
2. Supplier is `inactive` by default.
3. User sends **pairing request**.
4. System validates email (no‑reply detection, known autoresponders).
5. Supplier receives email with accept/reject buttons (token links).
6. Supplier clicks → status becomes `accepted` or `rejected`. User notified.
7. If rejected, a 24‑hour cooldown applies before re‑request.

**Business value:** Low friction for suppliers, higher response rate, better order reliability.

---

## 7. Analytics & Reporting

### AS‑IS

**For customers:**  
- No analytics available.

**For platform (internal):**  
- List of Teams and users.
- List of products used per Team.
- Logs from iPad devices.

### TO‑BE (near future)
- Inventory variance reports.
- Usage and shrinkage analytics.
- Cost analytics (after price data is available).
- Waste tracking.

---

## 8. Moderation (cross‑domain)

### AS‑IS

**Scope:** Products, attributes, brands.

**Process:** Queue‑based, rule‑driven, human‑in‑the‑loop (internal Getti staff only).

**Business value:** Data quality, marketplace readiness, regulatory safety.

**TO‑BE:** Extended moderation workflows (e.g., bulk actions, AI assistance).

---

## Strategic Future Directions (Separate Phase)

The following are **not part of the current or near‑term roadmap** but represent long‑term strategic opportunities. They are documented here for transparency.

- **POS integrations** (CompuCash, Poster, Micros) – legacy partial integration exists; full integration is a future phase.
- **Delivery merchant apps** (Wolt, Bolt, Uber Eats) – embedding inventory and recipes directly into vendor apps.
- **NFT / RWA for recipes** – creator economy: recipe licensing, royalties, tokenised ownership.

These will be implemented as **separate bounded contexts** when the time comes.

### AI Agents for Procurement & Inventory Decisions

**Context:**  
The market is moving toward AI‑driven decision layers. In a narrow B2B vertical like HoReCa, the first moat is **data + execution**. We already have both:
- Historical inventory data (consumption patterns, waste, variance).
- An append‑only procurement revision model (full negotiation history).
- A closed loop: Inventory → Order → Inventory.

**Strategic direction (no near‑term commitment):**  
We are designing the system to be **agent‑ready** without breaking the current architecture:
- Inventory and procurement APIs are explicit and idempotent.
- Domain data is structured (not free text) – suitable for fine‑tuning or prompting.
- Procurement revisions provide a clean supervised learning signal (what was proposed vs what was accepted).

**Potential agent capabilities (future):**
- Automatic reorder generation based on predicted consumption.
- Supplier selection and negotiation (within defined guardrails).
- Anomaly detection (e.g., unusual variance alerts).

**Current action:**  
We will not implement agents in the first release, but we will ensure that the **Procurement API** and **Inventory snapshots** are clean enough to add an agent layer later without rewriting core domains.

---

## Key Process Principles

> Inventory is the source of truth.  
> History is immutable.  
> Orders close the operational loop.  
> Users act, entities own.  
> Future monetisation must not break operations.

This document intentionally avoids UI details and focuses on real operational behaviour.
