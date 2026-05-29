# Data Assets of the GETTI Platform

*This document is a **brief summary**. For a full description, refer to the [`README.md`](./README.md) and domain‑specific documents in [`DDD/`](./DDD).*

The system accumulates several types of data that have intrinsic value – both for current operations and for future monetisation (AI, marketplace, licensing). Below are the core assets, already present or under construction.

---

## 1. Global Product Catalogue (GPC)

- **What:** A moderated, category‑agnostic database of product labels, variants, barcodes, attributes (region, vintage, etc.), and tare weights/volumes.
- **Value:** 
  - Single source of truth for thousands of beverages and ingredients.
  - Reduces duplication across bars and suppliers.
  - Essential for any inventory or order system in hospitality.
- **Status:** Live, growing via user submissions and internal moderation.

---

## 2. Inventory Snapshots (Immutable History)

- **What:** Frozen records of every counted product – quantity, weight, and physical attributes at the moment of counting.
- **Value:**
  - **Auditability** – historical reports never change, even if catalogue data is updated later.
  - **Trust** – bars can prove stock levels to investors or tax authorities.
  - **Training data** for consumption forecasting and anomaly detection (AI).
- **Status:** Fully implemented in the current system; to be preserved and extended in the rewrite.

---

## 3. Structured Recipes (with Author & Versioning)

- **What:** Recipe definitions (cocktails, pre‑batches) that include ingredient lists, quantities, version history, and author/owner attribution.
- **Value:**
  - **Operational** – automatic breakdown of pre‑batches in inventory (planned).
  - **Creator economy** – recipes become licensable assets (NFTs, royalties) with clear provenance.
  - **Brand collaborations** – sponsored recipes placed directly in bars’ menus.
- **Status:** Data model ready; inventory integration planned for Phase 2.

---

## 4. Procurement Order History (Negotiations)

- **What:** Complete, append‑only log of order revisions – supplier proposals, counter‑proposals, accept/reject decisions, and comments.
- **Value:**
  - **Negotiation intelligence** – patterns of accepted vs rejected offers can train AI agents.
  - **Supplier performance** – reliability, response times, price changes over time.
  - **Legal protection** – full history of what was agreed.
- **Status:** Partially implemented in legacy; full revision model part of the target architecture (Procurement domain).

---

## 5. Operational Data (Inventory + Orders)

- **What:** Real‑world counts, consumption, variance, and order quantities per team/location/time.
- **Value:**
  - **Benchmarking** – anonymised industry insights (e.g., average waste per category).
  - **Predictive reordering** – AI that suggests optimal order quantities.
  - **Shrinkage detection** – anomaly alerts for potential theft or spoilage.
- **Status:** Live, but currently siloed. The target architecture will enable safe aggregation.

---

## 6. Normalised Product Semantics

- **What:** Consistent categories, units, densities, and tare weights – the “language” of products.
- **Value:**
  - **Integration** – any POS or delivery app can consume our data without guesswork.
  - **AI input** – clean, typed attributes (e.g., “region: Islay”, “style: single malt”) are ready for ML models.
- **Status:** Existing but being hardened in the Products domain rewrite.

---

## Conclusion

GETTI’s data assets go beyond simple stock levels. They include **authoritative product knowledge**, **immutable audit trails**, **creator‑ready recipes**, **negotiation histories**, and **clean operational data** – all of which can be leveraged for AI, marketplace, and licensing revenue streams.

*For implementation details, see the respective domain files in [`DDD/`](./DDD) and the [`BUSINESS_PROCESSES.md`](./BUSINESS_PROCESSES.md).*
