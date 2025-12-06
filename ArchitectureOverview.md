---
title: Architecture Overview
nav_order: 1
has_children: true
---

# Architecture Overview

This document provides a high-level overview of the GigSta system architecture, focusing on backend structure, action system design, and recent platform changes. The goal is to maintain a clean, scalable, and easy-to-extend codebase suitable for a small-to-mid sized application.

---

## Backend Project Structure

GigSta follows a lightweight modular structure that supports clarity and maintainability without over-engineering.

/routes → API endpoints (FastAPI path operations)
/schemas → Pydantic request/response models
/crud → Business logic and Beanie database operations
/models → Beanie document models


### Why This Structure Works

- Scales well for small teams.
- Keeps business logic separate from API definitions.
- Avoids unnecessary architectural complexity.
- Supports gradual expansion as features grow.

---

## Action System Architecture

GigSta uses a split responsibility model for booking workflow actions.  
The backend defines authoritative action rules, while the frontend controls presentation and UI behavior.

### Backend Responsibilities

The backend exposes action metadata through `get_available_booking_actions()`:

- `action_name` – internal action identifier  
- `button_label` – user-facing label  
- `note` – text used for audit logs/history  
- `is_critical` – marks destructive actions (e.g., cancellations)

These actions map to state machine transitions that determine valid next steps in the booking lifecycle.

### Frontend Responsibilities

The frontend enhances backend action data with UI-specific metadata provided through `useBookingActions`:

- Icons (Lucide or custom SVG)
- Button styles/variants (success, error, primary, etc.)
- Interaction rules (confirmation modals, prompts)

### Adding a New Action

1. **Backend:**  
   Add the action to the booking state machine and return it through the action service.

2. **Frontend:**  
   Optionally add UI metadata to `ACTION_UI_METADATA`.  
   If omitted, a default icon and style are applied automatically.

This separation ensures clean boundaries:  
**backend = rules**, **frontend = presentation**.

---

## Platform Change: Migration from Motor to PyMongo Async

MongoDB is deprecating the **Motor** driver.  
GigSta is migrating to the **PyMongo Async API**, which now provides first-class async support.

### Benefits of PyMongo Async

- Faster and more consistent async performance.
- No separate async driver to maintain.
- Cleaner integration with modern Python async patterns.

Example import change:

```python
# Old
from motor.motor_asyncio import AsyncIOMotorClient

# New
from pymongo import AsyncMongoClient


This migration simplifies database operations and ensures long-term compatibility with MongoDB updates.

Working Efficiently with GenAI Tools

To get the most value when using AI for debugging or development:

Ask for structured debug output (console logs, formatted traces).

Break large problems into focused, isolated steps.

Provide real code snippets rather than summaries when possible.

These practices reduce iteration time and improve the accuracy of AI-assisted troubleshooting.

