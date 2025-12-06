---
title: Frontend Architecture
nav_order: 5
parent: Architecture Overview
---

# Frontend Architecture

The GigSta frontend is a **Single Page Application (SPA)** built with **Vue 3**, designed to provide a responsive, interactive, and maintainable user interface for both customers and GigStas (service providers).

This document outlines the main structure, state management, component organization, and interactions with the backend.

---

## Project Structure

src/
├── assets/ # Images, icons, styles
├── components/ # Reusable Vue components
├── layouts/ # Page layouts (TopNavBar, Sidebar, etc.)
├── pages/ # Route-level pages
├── router/ # Vue Router configuration
├── store/ # Vuex or Pinia stores
├── composables/ # Reusable composition functions (hooks)
└── utils/ # Client-side helper functions


---

## Routing

- Uses **Vue Router 4**
- Each route maps to a page component  
- Supports **dynamic routes** (e.g., `/service/:id`)  
- Route guards handle authentication and role-based access

---

## State Management

- **Vuex** (or Pinia) manages global state  
- Stores include:
  - `user` (auth info, profile)
  - `bookings`
  - `providers`
  - `filters` (search, location, category)
- Actions handle API calls and mutations update state

---

## Component Organization

- **UI Components**: Buttons, modals, cards, tables  
- **Feature Components**: Booking forms, provider listings, search filters  
- **Layout Components**: Header, Sidebar, Footer, Hero section  
- **Composables**: Hooks for repeated logic (e.g., `useBookingActions`, `useGeolocation`)  

---

## Styling & Design

- **Tailwind CSS** for utility-first styling  
- **DaisyUI** for component-based UI elements  
- Consistent theming and responsive design principles  
- Dark/light mode support where applicable

---

## Backend Integration

- REST API calls handled through Axios  
- Auth token management (localStorage + Vuex store)  
- Actions like booking, payment, provider management interact with the backend services  
- Error handling and loading states standardized in API modules

---

## Authentication Flow

- Login/Signup handled via traditional and social OAuth (Google, Facebook)  
- Tokens stored securely and refreshed when needed  
- Route guards enforce protected pages

---

## Booking Action System

- Uses `useBookingActions()` composable to map backend action metadata to UI  
- Supports icons, button styles, modals, and warnings  
- Keeps frontend behavior decoupled from backend business rules

---

## Extensibility

- Modular structure supports adding new pages, components, or stores easily  
- Composables and utility functions reduce duplicated logic  
- Designed for maintainability by a small team

---
