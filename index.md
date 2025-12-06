---
title: Home
nav_order: 1
---

# GigSta Documentation

Welcome to the official documentation for the **GigSta Platform** — a service marketplace connecting customers with gig workers across New Zealand and Australia.

This documentation provides an overview of the system architecture, backend implementation, and supporting components that power the GigSta application.

---

## Platform Overview

GigSta is built as a modern, service-oriented application using:

- **FastAPI** for backend APIs
- **MongoDB + Beanie** for data storage and document modeling
- **Vue 3** for the Presentation Layer
- **Stripe** for payments
- **Cloudflare R2** for S3-compatible file storage
- **Third-party APIs** such as Addressable.dev for address validation and geolocation

The project is structured to remain maintainable and scalable, with clean separation between the Presentation, Application, and Data layers.

---

## Backend Project Structure

The backend follows a simple, modular layout suitable for small to mid-sized applications:

gigsta-backend/
├── routes/ # API endpoints (public HTTP interface)
├── schemas/ # Pydantic request/response models
├── services/ # Business logic and workflows
├── crud/ # Database operations and query layer
├── models/ # Beanie document models (MongoDB)
├── utils/ # Shared helpers, transformations, validators
└── migrations/ # Database migration scripts


### Why This Structure Works

- Keeps components logically separated  
- Easy to extend as new features are added  
- Avoids over-engineering for a solo or small team  
- Maintains clarity without complex domain-driven abstractions  
- Supports fast development and onboarding  

---

## Documentation Structure

Use the left sidebar to explore each major layer of the system:

- **Architecture Overview** — high-level system design  
- **Presentation Layer** — frontend structure, components, flows, and styling  
- **Application Layer** — backend business logic, services, and integrations  
- **Data Layer** — database models, migrations, and query patterns  

Each section breaks down the relevant components in detail.

---

If you need adjustments or want to add deployment, CI/CD, or environment configuration sections later, just let me know and we can extend the documentation.


