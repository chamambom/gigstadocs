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

## ⚙️ Action System Architecture

### **Backend** (`get_available_booking_actions()`): Provides action metadata
- `action_name`: Internal identifier
- `button_label`: Human-readable text
- `note`: Description for logging/history
- `is_critical`: Destructive action flag

### **Frontend** (`useBookingActions`): Adds UI-specific metadata
- Icons (SVG paths)
- CSS variants (success/error/primary/ghost)
- Interaction behaviors (requiresModal, requiresConfirmation)

**To add a new action:**
1. Backend: Add to state machine transitions
2. Frontend: Add icon/variant to `ACTION_UI_METADATA` (optional, defaults to lightning icon)

## 🔄 Database Migration

### Document Changes to GigSta

```python
# from motor.motor_asyncio import AsyncIOMotorClient
from pymongo import AsyncMongoClient


Important: Users are not moving to PyMongo from Motor; rather, MongoDB is deprecating the Motor library and has built native asynchronous support directly into PyMongo (specifically the PyMongo Async API). The migration is to the new PyMongo Async API, which offers improved performance and a more direct integration with Python's asyncio library.


🤖 Gen AI Tips
Ask your AI to give you debug console logs

Break the problem into manageable parts

📚 Documentation Sections
Navigate through our documentation using the sidebar:

Presentation Layer (User Interface) - Frontend components, user interfaces, and client-side logic

Application Layer (Business Logic) - API endpoints, business rules, and application services

Data Layer (Database Management) - Database models, migrations, and data access patterns

🚀 Quick Start
For development setup and getting started, check our GitHub repository.

📞 Support
For questions or issues, please:

Check the relevant documentation section

Review existing GitHub issues

Create a new issue if needed

