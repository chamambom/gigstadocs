---
title: Application Layer
nav_order: 3
parent: Architecture Overview
---

# Application Layer (Business Logic)

The Application Layer contains all business logic that powers GigSta.  
It orchestrates workflows, processes incoming requests, interacts with external services, and coordinates data operations between the Presentation and Data layers.

This layer is implemented using **FastAPI**, with a modular structure that keeps concerns separated and makes the system easier to extend as functionality grows.

---

## Architecture Overview

The Application Layer is composed of the following core modules:

/routes → API endpoints (HTTP interfaces)
/schemas → Request/response validation (Pydantic)
/services → Business workflows and integrations
/utils → Helper functions, shared logic, transformations
/models → Beanie document models (domain entities)
/crud → Database operations and repository-style logic


Each part contributes a specific responsibility to maintain code clarity and prevent tightly coupled modules.

---

## Module Breakdown

### **Routes (API Endpoints)**

Routes define the public HTTP interface of the backend.  
They map incoming requests to business logic methods and return structured responses.

Responsibilities:
- Request parsing  
- Input validation  
- Authentication / authorization checks  
- Triggering appropriate service calls  
- Returning Pydantic-validated responses  

Example modules include:
- `/auth`
- `/bookings`
- `/providers`
- `/categories`
- `/payments`

FastAPI’s dependency injection ensures clean, testable endpoints.

---

### **Schemas (Pydantic Models)**

Schemas provide strict definitions for:
- Incoming request bodies  
- Query and path parameters  
- Response formats  

Key roles:
- Validate client input  
- Prevent malformed data  
- Ensure consistent API contracts  
- Serialize/deserialize between Python and JSON  

Schemas act as the **shared contract** between frontend and backend.

---

### **Services (Business Logic)**

Services implement the core workflows of the GigSta platform.

Responsibilities:
- Booking state machine transitions  
- Payment orchestration via Stripe  
- Filtering & search logic  
- User registration flows  
- Integrations with external APIs  
- Generating action metadata (e.g., booking actions)

Services are where business rules live—not in routes or models.

---

### **Utils (Utility Functions)**

Small, reusable helpers used across multiple modules.

Common examples:
- Date/time formatting  
- Token generation  
- Email utilities  
- Logging helpers  
- Geo-distance calculations  
- Validation utilities  

Utilities ensure repeated logic is shared and consistent.

---

### **Models (Beanie Document Models)**

Models represent the domain entities stored in MongoDB.

These define:
- Field structure  
- Indexes  
- Default values  
- Relationships  
- Validation rules  
- Document methods (if needed)

Models are not responsible for business logic—only data structure.

---

### **CRUD Modules (Repository Layer)**

CRUD modules encapsulate all read/write operations against MongoDB.

Responsibilities:
- Fetching documents  
- Running aggregations (e.g., popular categories, ratings summaries)  
- Handling filters and search queries  
- Creating, updating, deleting records  
- Running `$geoNear` distance queries  

This structure prevents business logic from being mixed with database calls.

---

## External Integrations

The Application Layer also manages connectivity with external systems that support GigSta.

### **Stripe (Payments)**

Used for:
- Provider subscription billing  
- Customer payments  
- Checkout sessions  
- Webhook-driven updates  

Integration handled through a dedicated payment service, ensuring:
- Secure API interactions  
- Idempotent event handling  
- Safe state updates after webhook validation  

---

### **Addressable.dev (Address & Geolocation API)**

Used to:
- Convert user-provided addresses into structured metadata  
- Validate addresses  
- Enhance search and filtering  
- Support location accuracy for distance-based provider matching  

This API is wrapped inside a service module to standardize interaction.

---

### **Cloudflare R2 (S3-Compatible Storage)**

Used for:
- Provider images  
- User avatars  
- Service listing images  
- Document uploads  

Implementation follows the S3 API pattern via `boto3`-compatible SDKs.

Responsibilities include:
- Upload  
- Secure access token generation  
- File deletion  
- Public/private file handling  

---

## Business Rules & Workflows

Typical business logic implemented in this layer includes:

- Booking lifecycle management  
- Provider onboarding rules  
- Rating and review validation  
- Distance-based provider recommendations  
- Payment verification and post-payment workflows  
- Email verification and account security flows  

These rules ensure GigSta behaves consistently and predictably across all user interactions.

---

