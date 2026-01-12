# Procurement Domain — Domain Design Draft

## Purpose

The Procurement Domain defines:
- how goods are ordered,
- how suppliers are managed,
- how procurement integrates with inventory.

Its goals:
- fast ordering based on inventory data
- supplier-friendly, email-first workflows
- no forced supplier onboarding
- natural integration with Products and Inventory
- foundation for future automation and analytics

Procurement is an **operational workflow**, not a financial system.  
Pricing, billing, payments, and accounting are explicitly out of scope.

---

## Core Concepts Overview

1. Supplier
2. Supplier Relationship (Pairing)
3. Procurement Order
4. Order Item
5. Supplier Communication
6. Procurement Lifecycle

---

## 1. Supplier

### Definition

A Supplier is an external entity providing goods
to a Team or Organization.

Suppliers are **not system users by default**.

### Examples
- alcohol distributor
- beverage wholesaler
- tobacco supplier
- local food vendor

### Characteristics
- user-created
- inactive / pending / active
- no login required
- email-based interaction

### Key Fields (conceptual)
- id
- name
- contact_email
- contact_name (optional)
- phone (optional)
- legal_name (optional)
- notes (optional)
- created_by_team_id
- created_at
- status

---

## 2. Supplier Relationship (Pairing)

### Definition

Operational link between a Team / Organization
and a Supplier.

### Pairing Flow

1. Supplier created (inactive)
2. Pairing request sent
3. Supplier receives email
4. Response:
   - accepted
   - rejected
   - no response

### States

- inactive
- pending
- accepted
- rejected

### Key Fields
- supplier_id
- owner_type (Team | Organization)
- owner_id
- status
- pairing_requested_at
- pairing_resolved_at

---

## 3. Procurement Order

### Definition

A request to a Supplier for goods.

### Characteristics
- single ownership scope
- single supplier
- manual or inventory-driven
- immutable after submission

### Key Fields
- id
- supplier_id
- owner_type
- owner_id
- created_by_user_id
- created_at
- submitted_at
- status
- notes (optional)

---

## 4. Order Item

### Definition

A single product line in an order.

### Characteristics
- references Product Variant
- explicit quantity and unit
- optional informational price

### Fields
- procurement_order_id
- product_variant_id
- quantity
- unit
- price (optional)

---

## 5. Email-first Communication

Suppliers interact via:
- email
- read-only web pages
- optional comment forms

Suppliers are never required to:
- create accounts
- install software

---

## 6. Procurement Lifecycle

### States

1. Draft
2. Submitted
3. Acknowledged (optional)
4. Completed (manual)
5. Cancelled

### Rules
- submitted orders are immutable
- procurement never auto-updates inventory

---

## 7. Inventory Integration

### Flow

1. Inventory completed
2. Missing stock identified
3. Order created
4. References Product Variants
5. Fulfillment outside system
6. Inventory updated manually

### Principle

> Procurement does NOT mutate inventory automatically

---

## 8. Ownership & Aggregation

Supported scopes:
- Team
- Organization

Ownership defines:
- visibility
- aggregation
- analytics

---

## 9. Relation to Other Domains

- Products → Product Variant
- Inventory → demand source
- Users / Teams → permissions
- Analytics → aggregation
- Notifications → email

---

## Explicitly Out of Scope

- payments
- invoices
- accounting
- pricing enforcement
- supplier authentication

---

## Key Architectural Principles

> Procurement is operational  
> Suppliers are external  
> Email is a feature  
> Inventory drives orders  
> Ownership is explicit