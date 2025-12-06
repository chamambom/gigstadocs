---
title: Backend Architecture
nav_order: 6
parent: Architecture Overview
---

# Backend Architecture

The GigSta backend is a **FastAPI-based** system providing RESTful APIs for the platform.  
It manages all business logic, data persistence, authentication, and external integrations.

This document describes the high-level structure, module responsibilities, and request flow.

---

## Project Structure

gigsta-backend/
├── routes/ # API endpoints (FastAPI path operations)
├── schemas/ # Pydantic request/response models
├── services/ # Business workflows and integrations
├── crud/ # Database access and repository-style operations
├── models/ # Beanie document models
├── utils/ # Shared helpers and utilities
└── migrations/ # Database migration scripts


---

## Request Flow

1. **HTTP Request** → route handler  
2. **Route handler** → schema validation via Pydantic  
3. **Business logic** → services invoked  
4. **Data access** → CRUD layer interacts with MongoDB  
5. **Response** → serialized via Pydantic schema back to client  

This layered approach ensures separation of concerns and maintainable code.

---

## Business Logic Layer (Services)

- Orchestrates core workflows:
  - Booking lifecycle
  - Payment processing (Stripe)
  - Search & filtering
  - Provider onboarding
- Integrates with third-party APIs:
  - Addressable.dev for geolocation
  - Cloudflare R2 for S3-compatible file storage
- Adds metadata for frontend consumption (`useBookingActions`)  

---

## Database Layer

- MongoDB Atlas as the primary datastore  
- Beanie ODM for async Python models  
- CRUD modules encapsulate queries, aggregations, and geo-spatial lookups  
- Migration scripts ensure consistent schema evolution across environments  

---

## Authentication & Security

- JWT-based authentication with OAuth options (Google, Facebook)  
- Route-level guards and dependency injection for role-based access  
- Password hashing and token lifecycle management  
- CORS configured for frontend domains

---

## Async & Background Tasks

- FastAPI async routes for non-blocking I/O  
- Supports background tasks for email notifications, payment verification, and other workflows  
- Async database access via PyMongo Async API  

---

## Logging & Monitoring

- Structured logging for all request/response cycles  
- Error handling with consistent JSON responses  
- Metrics and alerts via hosting provider dashboards (Render, Atlas)

---

## Extensibility & Maintainability

- Modular services prevent tightly coupled code  
- Schema-driven endpoints reduce errors and improve frontend contracts  
- Well-documented CRUD and model layers simplify onboarding and feature expansion

---

