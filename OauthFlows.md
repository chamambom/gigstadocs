---
title: OauthUserLoginFlows
nav_order: 10
parent: Architecture Overview
---

# Oauth User Login Flows

I am using FastAPIUsers to implement Oauth flows

- User clicks "Sign in with Google"
- Backend generates authorization URL with state
- Google redirects to frontend (/google-oauth-callback)
- Frontend component extracts code and state
- Frontend sends them to backend /auth/google/callback
- Backend validates and returns token

- This is a valid pattern. The issue is that FastAPI-Users is detecting this as cross-origin and adding CSRF protection in production but not localhost.

Issues I needed to fix 

I suspect one of these is different between localhost and production:

Origin header - Production might be sending https://gigsta.co.nz but your CORS has https://www.gigsta.co.nz
Referer header - Different format between environments
X-Forwarded headers - Render's proxy might be adding headers that trigger CSRF

The csrftoken is added by FastAPI-Users when it thinks the OAuth flow is "unsafe" due to cross-origin detection.

I was using gigsta.co.nz on the frontend sending to gigsta.render.com backend. I had to update my DNS to api.gigsta.co.nz.


