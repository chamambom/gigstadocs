---
layout: default
title: Home
nav_order: 1
---

# GigSta API Documentation

Welcome to the GigSta API documentation. This documentation covers the architecture and implementation details of the GigSta backend system.

## 🏗️ Project Structure

#### 🧱 Best Practices for Small-to-Mid Projects

✅ **GigSta Backend Project Structure** - Structure that supports clarity, modularity, and maintainability as app grows


gigsta-backend/
├── routes/ # API endpoints (path operations) are defined
├── schemas/ # Pydantic models (used for request/response validation) live
├── crud/ # Contains business logic and database queries (via Beanie)
└── models/ # Your database models (Beanie documents)


🔍 **Why this Simple Structure**

- The project isn't growing too fast 
- Maintainability without over-engineering
- My team is small (just me)
- I am not dealing with deeply nested domain logic or massive APIs

