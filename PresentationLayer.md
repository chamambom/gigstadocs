---
title: Presentation Layer
nav_order: 2
parent: Architecture Overview
---

# Presentation Layer (User Interface)

The Presentation Layer is responsible for rendering the GigSta user interface and managing all client-side interactions. It provides the bridge between the user and the underlying application logic exposed through the backend API.

This layer is implemented using **Vue 3**, the **Composition API**, and `<script setup>`, with Tailwind CSS and DaisyUI handling styling and component structure.

---

## UI Architecture Overview

The GigSta front-end follows a modular and component-driven architecture:

- **Pages/Views:** High-level route-based screens (Home, Search, Booking, Dashboard)
- **Components:** Reusable UI elements such as cards, forms, filters, and modals
- **Composable Functions:** Shared logic extracted into `useXYZ` modules (e.g., `useBookingActions`, `useAuth`)
- **Stores:** Vuex modules managing authentication, user state, categories, providers, and filters

This approach supports clean separation of concerns, reduces duplication, and makes components easy to maintain and extend.

---

## UI Components

Components are grouped by purpose:

### **Structural Components**
- Navigation bar  
- Sidebar panels  
- Footer  
- Layout wrappers (dynamic layout support)

### **Interactive Components**
- Search bar  
- Category and provider filters  
- Booking action buttons  
- Ratings widgets  
- Modals and confirmation dialogs  

### **Reusable Elements**
- Cards  
- Buttons  
- Input fields  
- Pagination controls  

Using small, composable units ensures consistent UI behavior across the platform.

---

## User Flows

The UI implements several key user flows that connect directly to backend processes:

### **1. Service Discovery**
- User selects a category or searches for providers  
- Optional filters: rating, location, distance, availability  
- Results update dynamically based on applied filters  

### **2. Booking Workflow**
- User selects a provider  
- Available actions come from backend metadata  
- UI maps actions to:
  - Icons  
  - Button styles  
  - Confirmation prompts  
  - Modal dialogs  

### **3. Authentication & Identity**
- Email/password login  
- Google and Facebook OAuth via backend FastAPI Users integration  
- Token handling via Vuex and localStorage  

These flows are designed to be intuitive, responsive, and mobile-friendly.

---

## Styling & Design System

GigSta uses a modern, utility-first styling approach:

### **Tailwind CSS**
- Provides low-level utility classes  
- Allows fast, consistent styling without writing custom CSS  

### **DaisyUI 5**
- Supplies accessible, themeable components  
- Forms the base for buttons, cards, modals, alerts, and navigation elements  

### **Brand System**
- Primary color: `#0d6efd`  
- Accent color: `rgb(255, 152, 67)`  
- Layout and components styled to reflect the GigSta identity  

This combination delivers speed, consistency, and a professional UI framework that scales as the product grows.

---
