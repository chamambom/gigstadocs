---
title: Architecture Overview
nav_order: 1
has_children: true
---

# Architecture Overview

This document provides a high-level overview of the GigSta system architecture, including backend structure, frontend deployment, hosting environment, database design, booking action architecture, and recent platform changes. It serves as a single source of truth for how the system is built and how all components interact.

---

## System Hosting & Deployment

### Frontend (Vue.js)
- Hosted on: **Vercel** (SPA build served over CDN)
- Uses Vue 3 + Composition API + Tailwind/DaisyUI
- Communicates with backend through REST over HTTPS
- Deployment: Automated build-and-deploy pipeline

### Backend (FastAPI)
- Hosted on: **Render**
- Provides REST APIs for authentication, bookings, services, ratings, and geo-queries
- Environment variables used for secrets and connection strings
- Deploys from `main` branch via Render’s auto-deploy

### Database
- Hosted on: **MongoDB Atlas**
- ORM: **Beanie** (with ongoing migration toward PyMongo Async)
- Includes migration scripts for schema evolution
- Access controlled through IP allowlists and app-level roles

### DNS & Networking
- Domain registered with: **CrazyDomains**
- DNS delegated to: **Cloudflare**
- Cloudflare handles:
  - DNS records  
  - SSL/TLS termination  
  - Caching & security proxy  
- Traffic flow:

User → Cloudflare → Frontend (Vue) → Backend (Render) → MongoDB Atlas


---

## Backend Project Structure

GigSta uses a lightweight, modular backend structure that keeps logic clear and easy to maintain:

/routes → API endpoints (FastAPI path operations)
/schemas → Pydantic request/response models
/crud → Business logic and Beanie database operations
/models → Beanie document models


### Why This Structure Works
- Maintains clean boundaries between layers  
- Easy to scale without unnecessary complexity  
- Supports new features without architectural churn  

---

## Booking Action System Architecture

GigSta uses a shared action model where the **backend defines rules** and the **frontend defines presentation**.

### Backend Responsibilities
The backend exposes action metadata through `get_available_booking_actions()`:

- `action_name` – internal identifier  
- `button_label` – user-facing label  
- `note` – audit log text  
- `is_critical` – marks destructive actions  

These are tied to state-machine transitions that enforce booking lifecycle rules.

### Frontend Responsibilities
The frontend uses `useBookingActions()` to apply UI metadata:

- Icons  
- Button styles  
- Confirmation modals  
- UX behaviour  

### Adding a New Action
1. **Backend:** Add new action and mapping to the state machine.  
2. **Frontend:** Optionally extend `ACTION_UI_METADATA`.  
   Defaults are applied if no UI metadata is provided.

**Backend = business rules.  
Frontend = UX.**

---

## Platform Change: Motor → PyMongo Async

MongoDB is deprecating the Motor driver. GigSta is migrating to **PyMongo Async**, which is now fully async-native.

### Benefits
- Better async performance  
- Simpler integration  
- No separate async driver layer to maintain  

**Example change:**

```python
# Old
from motor.motor_asyncio import AsyncIOMotorClient

# New
from pymongo import AsyncMongoClient

This improves long-term compatibility and reduces driver fragmentation.

Working Efficiently with GenAI Tools

Share real code or logs, not summaries

Break problems into small, isolated steps

Request structured output when debugging

This reduces iteration time and improves solution accuracy.
