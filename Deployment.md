---
title: Deployment
nav_order: 6
parent: Architecture Overview
---

# Deployment

This document outlines how GigSta is deployed across its hosting components, including the frontend, backend, database, CI/CD, and DNS infrastructure.

It provides a clear reference for where each part of the system runs and how updates move from development to production.

---

## Hosting Overview

GigSta uses a distributed hosting model:

- **Frontend (Vue.js SPA)**
  - Built and deployed as static assets
  - Served via CDN for low latency
  - Interacts with the backend via HTTPS REST APIs

- **Backend (FastAPI)**
  - Hosted on Render as a containerized or service-based deployment
  - Auto-deploys on pushes to the main branch
  - Handles authentication, bookings, payments, search/filtering, and integrations

- **Database (MongoDB Atlas)**
  - Managed cloud-hosted cluster
  - Secure network access controls and connection strings
  - Supports geospatial queries and horizontal scaling

- **DNS (Cloudflare)**
  - All DNS records managed through Cloudflare
  - Proxy enabled for performance, caching, and WAF protection
  - Domain still registered through CrazyDomains (delegated to Cloudflare)

---

## Deployment Flow

1. **Developer commits code → Git provider**
2. **CI pipeline runs tests and builds**
3. **Backend automatically deploys to Render**
4. **Frontend build artifacts shipped to hosting provider / CDN**
5. **Environment variables injected during deployment**
6. **Cloudflare updates propagate instantly for DNS or CDN configuration changes**

This ensures a fast, predictable deployment lifecycle with minimal manual effort.

---

## Environment Configuration

Environment variables are used for:

- Database URIs  
- Stripe keys  
- API secrets  
- JWT tokens  
- 3rd-party API keys (Addressable.dev, Cloudflare R2, etc.)  
- CORS and domain configuration  

Secrets are stored securely inside hosting providers and never committed to the repository.

---

## Monitoring & Logging

- Backend logs streamed through Render dashboard (or external log sink)
- MongoDB Atlas metrics and performance dashboards
- Cloudflare analytics for DNS, caching, and traffic
- Error alerts emailed or surfaced through monitoring services

---

## Backup & Recovery

- Atlas handles automated backups  
- Restore points available depending on the tier  
- Manual exports can be scheduled for long-term archiving  

---

## Zero-Downtime Considerations

- Render supports rolling restarts
- Static frontend hosting ensures seamless UI updates
- Database migrations designed to be backward compatible

---

