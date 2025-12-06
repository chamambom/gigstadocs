---
title: Data Layer
nav_order: 4
parent: Architecture Overview
---

# Data Layer (Database Management)

The Data Layer manages how GigSta stores, retrieves, and structures data.  
It provides a consistent interface for interacting with MongoDB through the Beanie ODM, enforces data integrity, and supports migrations as the schema evolves.

This layer underpins all core operations—including bookings, users, services, providers, ratings, and payment metadata.

---

## Database Architecture

GigSta uses **MongoDB** as the primary datastore, chosen for:

- Flexible document modeling  
- Fast geolocation queries  
- Easy scaling for search-heavy workloads  
- Native JSON-like document structure  

Database operations are performed through **Beanie**, an asynchronous ODM built on top of Motor/PyMongo’s async API.

---

## Document Models (Schema Definitions)

In place of relational tables, MongoDB uses collections of flexible documents.

Each GigSta entity is represented as a **Beanie Document**, defining:

- Field names and types  
- Indexes (including `text`, `unique`, and `2dsphere`)  
- Default values  
- Validation rules  
- Optional document-level methods  

Examples of domain models include:

- `User`
- `Service`
- `Provider`
- `Booking`
- `Rating`
- `Category`

### Responsibilities of Models

- Enforcing structure and validation  
- Generating database queries  
- Providing typed Python objects for easy manipulation  
- Supporting relationships using references or embedded models  

Models represent **data**, not business logic.

---

## Migrations

Even though MongoDB is schemaless, GigSta maintains **structured migration scripts** to ensure consistent environments across development, staging, and production.

Typical migration tasks:

- Creating or updating indexes  
- Adding new fields with default values  
- Transforming or renaming existing fields  
- Backfilling computed fields  
- Performing data cleanup or normalization  
- Managing collection creation/deletion  

Migrations provide version control for database evolution, especially important when deploying new features.

---

## Data Access Layer (CRUD)

All direct interactions with MongoDB are routed through a dedicated CRUD layer containing repository-style functions.

### Responsibilities of the CRUD Layer

- Running queries and aggregations  
- Performing inserts, updates, deletes  
- Handling pagination  
- Implementing `$geoNear` distance lookup queries  
- Joining related documents using aggregation pipelines  
- Executing search filters (category, rating, location, provider name, etc.)  

This layer ensures that business logic modules don’t need to interact with the database directly.  
It also promotes consistency and encapsulates data-access complexity.

---

## Query Patterns & Indexing

GigSta uses several common query patterns optimized with appropriate indexes:

### **Geospatial Queries**
Used for matching users to providers based on proximity:

- `2dsphere` indexes on provider coordinates  
- `$geoNear` aggregation for ranking by distance  

### **Search & Filtering**
Support for:
- Category filtering  
- Rating aggregation  
- Provider name search  
- Location-based filtering  

### **Aggregations**
Common use cases:
- Popular category counts  
- Average ratings per service/provider  
- Booking status metrics  
- Search result ranking  

### **Performance Considerations**
Indexes are created for:
- Frequent query fields  
- Geospatial lookups  
- Text search fields  
- Provider/service relationships  

This ensures fast, predictable query performance.

---

## ODM Configuration

Beanie integrates tightly with FastAPI via asynchronous initialization:

- Models registered on app startup  
- Collections created automatically (if configured)  
- Index creation handled during initialization  
- Async execution ensures scalable I/O performance  

This configuration ensures reliable schema loading across environments.

---

