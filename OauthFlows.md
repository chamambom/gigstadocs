---
title: OauthUserLoginFlows
nav_order: 10
parent: Architecture Overview
---

# Oauth User Login Flows

- User clicks "Sign in with Google"
- Backend generates authorization URL with state
- Google redirects to frontend (/google-oauth-callback)
- Frontend component extracts code and state
- Frontend sends them to backend /auth/google/callback
- Backend validates and returns token

- This is a valid pattern. The issue is that FastAPI-Users is detecting this as cross-origin and adding CSRF protection in production but not localhost.