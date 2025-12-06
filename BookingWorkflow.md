---
title: Booking Workflow
nav_order: 7
parent: Architecture Overview
---

# Booking Workflow

This document outlines the high-level lifecycle of a GigSta booking, describing the state transitions, rules, and systems that drive the booking process.

---

## Overview

GigSta uses a state-machine-style workflow where each booking moves through a series of well-defined states.  
Actions available to users (both customers and GigStas) depend on the current state of the booking.

This ensures:

- Clarity around what actions are allowed  
- Consistent handling of cancellations, updates, and confirmations  
- Predictable logic between backend and frontend behavior  

---

## Booking Lifecycle (High-Level States)

A typical lifecycle includes:

1. **Pending / Requested**  
2. **Accepted or Declined**  
3. **In Progress**  
4. **Completed**  
5. **Cancelled (by customer or GigSta)**  

You can expand these later with your exact states once finalized.

---

## Backend Rules (Authoritative Logic)

The backend determines which actions are valid at each state.

Examples:
- Accept booking  
- Decline booking  
- Mark as in-progress  
- Mark as completed  
- Cancel (critical action)

For each state, the backend returns the allowed actions through:

get_available_booking_actions()


This ensures UI cannot bypass business rules.

---

## Frontend Behaviour (User Interface Logic)

The frontend receives the action list and adds UI-specific details:

- Icons  
- Button styles  
- Modal confirmation requirements  
- Warnings or safety checks  

This approach keeps business rules centralized while still allowing full UI control.

---

## Action Metadata

Each action includes metadata such as:

- `action_name` – used internally  
- `button_label` – visible to the user  
- `note` – added to booking audit trail/history  
- `is_critical` – shown with destructive/confirm UI  

---

## Booking Audit Trail

Every booking change is recorded with:

- Action name  
- Actor (customer or GigSta)  
- Timestamp  
- Notes provided by the system  

This supports accountability and clear history tracking.

---

## Payment Considerations (General)

Depending on your final requirements:

- Payment may be required before confirmation  
- Stripe metadata can be attached to booking IDs  
- Refund or cancellation flows depend on final rules  

These can be expanded once Stripe integration is finalised.

---

