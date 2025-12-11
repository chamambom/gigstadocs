---
title: GigSta Onboarding & Booking Workflows Documentation
nav_order: 9
parent: Architecture Overview
---

## Table of Contents

1. [Overview](#overview)
2. [User States & Flags](#user-states--flags)
3. [Booking Workflows](#booking-workflows)
   - [Public Booking Flow (Guest Checkout)](#public-booking-flow-guest-checkout)
   - [Private Booking Flow (Authenticated Users)](#private-booking-flow-authenticated-users)
4. [Onboarding Workflows](#onboarding-workflows)
   - [Seeker Onboarding Path](#seeker-onboarding-path)
   - [Provider Onboarding Path](#provider-onboarding-path)
   - [Seeker-to-Provider Transition](#seeker-to-provider-transition)
5. [Authentication Methods](#authentication-methods)
6. [Router Guard Logic](#router-guard-logic)
7. [Key Components & Composables](#key-components--composables)
8. [Database Schema Reference](#database-schema-reference)

---

## Overview

GigSta is a two-sided marketplace connecting service seekers with service providers. The platform supports:

- **Guest bookings** (users can book without creating an account upfront)
- **Traditional signup** (email/password)
- **OAuth2 signup** (Google, Facebook)
- **Dual roles** (users can be both seekers and providers)
- **Progressive onboarding** (users can abandon and resume provider setup)

---

## User States & Flags

### Core User Flags

| Flag | Type | Description |
|------|------|-------------|
| `is_verified` | Boolean | Email has been verified |
| `is_provisional` | Boolean | User has not chosen seeker/provider path yet |
| `is_active` | Boolean | Account is active (not banned) |
| `is_superuser` | Boolean | Admin privileges |
| `is_oauth_registered` | Boolean | Signed up via OAuth |
| `requires_password_setup` | Boolean | OAuth users need to set password |

### Role System

| Role | Description |
|------|-------------|
| `user` | Seeker role - can browse and book services |
| `provider` | Provider role - can offer services and earn |

**Important:** Users can have BOTH roles simultaneously.

### Onboarding Status Flags

```javascript
onboarding_status: {
  basic_complete: false,              // Name, phone, address collected
  provider_onboarding_complete: false, // Trading name, services defined
  billing_setup_complete: false        // Stripe Connect completed
}
```

### Provider Status

| Status | Description |
|--------|-------------|
| `not_applied` | User has not started provider onboarding |
| `pending` | Provider application submitted, awaiting admin approval |
| `approved` | Provider approved, can access full provider features |
| `rejected` | Provider application rejected |

---

## Booking Workflows

### Public Booking Flow (Guest Checkout)

**Use Case:** Unauthenticated user wants to book a service immediately without creating an account first.

#### Flow Diagram

```
Guest User → Browse Services → Select Service → Initiate Booking
    ↓
Guest Checkout Form
    ├─ Service Details (prefilled)
    ├─ Customer Details (email, name, phone, address)
    ├─ Date & Time Selection
    └─ Payment Method (NOT charged yet)
    ↓
Submit Booking (Backend creates provisional user)
    ↓
Email Verification Sent
    ↓
User Clicks Verification Link (auto-login)
    ↓
Router Guard Detects: is_provisional=true + pendingBookingIntent in localStorage
    ↓
Redirect to /complete-profile
    ↓
Complete Profile Form
    ├─ Set Password (required for provisional users)
    ├─ Confirm Details (prefilled from booking intent)
    └─ Confirm Address
    ↓
Submit Profile (Backend creates booking + charges payment)
    ↓
Redirect to /bookings/:id (Booking Detail Page)
```

#### Key Implementation Details

**Frontend:**
1. **Booking Intent Storage:**
   ```javascript
   localStorage.setItem('pendingBookingIntent', JSON.stringify({
     serviceId: 'service_123',
     gigId: 'gig_456',
     selectedDate: '2024-12-15',
     selectedTime: '14:00',
     customerDetails: {
       email: 'user@example.com',
       fullName: 'John Doe',
       phone: '+64212345678',
       address: { /* address object */ }
     },
     paymentMethodId: 'pm_xxx'
   }));
   ```

2. **Router Guard Detection (Guard D):**
   ```javascript
   if (isEmailVerified && isProvisional) {
     const pendingBookingIntent = localStorage.getItem('pendingBookingIntent');
     if (pendingBookingIntent) {
       // Redirect to complete-profile
       return next({name: 'complete-profile'});
     }
   }
   ```

3. **Profile Completion:**
   - Form auto-fills from `pendingBookingIntent`
   - User sets password
   - On submit, sends `booking_intent` to backend
   - Backend creates booking and returns `booking._id`
   - Frontend redirects to `/bookings/:id`

**Backend:**
```python
async def complete_basic_profile(
    user: User,
    profile_data: BasicProfileComplete,
    booking_intent: Optional[BookingIntent] = None
) -> dict:
    # Update user profile
    user.full_name = profile_data.full_name
    user.phone_number = profile_data.phone_number
    user.address = profile_data.address
    user.is_provisional = False
    user.onboarding_status.basic_complete = True
    
    # Add 'user' role if not present
    if 'user' not in user.roles:
        user.roles.append('user')
    
    await user.save()
    
    # Create booking if booking_intent provided
    booking = None
    if booking_intent:
        booking = await create_booking_from_intent(user, booking_intent)
    
    return {
        'user': user,
        'booking': booking,
        'access_token': create_access_token(user.id)
    }
```

**Cleanup:**
```javascript
// After successful profile completion and booking creation
localStorage.removeItem('pendingBookingIntent');
localStorage.removeItem('pendingBookingUser');
localStorage.removeItem('postOnboardingRedirect');
```

---

### Private Booking Flow (Authenticated Users)

**Use Case:** Logged-in user (seeker or provider) wants to book a service.

#### Flow Diagram

```
Authenticated User → Browse Services → Select Service → Initiate Booking
    ↓
Check: Has basic profile?
    ├─ NO → Redirect to /complete-profile (no booking intent)
    └─ YES → Continue
    ↓
Booking Form
    ├─ Service Details (prefilled)
    ├─ Customer Details (prefilled from user profile)
    ├─ Date & Time Selection
    └─ Payment Method
    ↓
Submit Booking (Backend creates booking immediately)
    ↓
Redirect to /bookings/:id (Booking Detail Page)
```

#### Key Implementation Details

**Frontend:**
- No `pendingBookingIntent` needed
- User already authenticated and verified
- Profile data prefilled from `currentUser`
- Direct booking creation via API call
- Immediate redirect to booking detail page

**Backend:**
```python
async def create_booking(
    user: User,
    booking_data: BookingCreate
) -> Booking:
    # User must have basic_complete = true
    if not user.onboarding_status.basic_complete:
        raise HTTPException(
            status_code=400,
            detail="Please complete your profile before booking"
        )
    
    # Create booking
    booking = Booking(
        user_id=user.id,
        gig_id=booking_data.gig_id,
        service_id=booking_data.service_id,
        scheduled_date=booking_data.scheduled_date,
        # ... other fields
    )
    
    await booking.save()
    return booking
```

---

## Onboarding Workflows

### Seeker Onboarding Path

**Use Case:** New user wants to browse and book services (no interest in providing services).

#### Traditional Signup Flow

```
User → /signup
    ↓
Enter Email + Password
    ↓
Backend Creates User:
    ├─ is_verified = false
    ├─ is_provisional = true
    ├─ roles = []
    └─ requires_password_setup = false
    ↓
Email Verification Sent
    ↓
User Clicks Verification Link (auto-login)
    ↓
Router Guard Detects: is_provisional=true, no bookingIntent
    ↓
Redirect to /onboarding-choice (Guard E)
    ↓
Onboarding Choice Page
    ├─ "Find Services" (Seeker)
    └─ "Become a GigSta" (Provider)
    ↓
User Clicks "Find Services"
    ↓
Frontend dispatches:
    roles: ['user']
    is_provisional: false
    ↓
Redirect to /seeker-dashboard
```

#### OAuth2 Signup Flow

```
User → /login → Click "Sign in with Google"
    ↓
OAuth2 Flow (Google/Facebook)
    ↓
Backend Creates/Updates User:
    ├─ is_verified = true (email auto-verified by OAuth)
    ├─ is_provisional = true
    ├─ is_oauth_registered = true
    ├─ requires_password_setup = true
    ├─ roles = []
    └─ oauth_accounts = [{ oauth_name: 'google', ... }]
    ↓
Auto-login with OAuth token
    ↓
Router Guard Detects: is_provisional=true, no bookingIntent
    ↓
Redirect to /onboarding-choice (Guard E)
    ↓
[Same as traditional signup from here]
```

**Note:** OAuth users don't need to verify email (already verified by OAuth provider) but must set a password if they want to use traditional login later.

---

### Provider Onboarding Path

**Use Case:** User wants to offer services and earn money on the platform.

#### Complete Flow (3 Stages)

```
User → /onboarding-choice → Click "Become a GigSta"
    ↓
Frontend dispatches:
    roles: ['user', 'provider']  ← BOTH roles assigned
    is_provisional: false
    onboarding_status: {
      provider_onboarding_complete: false,
      basic_complete: false
    }
    ↓
Redirect to /onboarding/provider-details
    ↓
┌─────────────────────────────────────────────────────────────┐
│ STAGE 1: Provider Details Form                              │
│ (/onboarding/provider-details)                              │
├─────────────────────────────────────────────────────────────┤
│ Fields Collected:                                            │
│   • Full Name                                                │
│   • Phone Number (+64 or +61)                                │
│   • Trading Name (business name)                             │
│   • Business Address (with geocoding)                        │
│                                                              │
│ Backend Sets:                                                │
│   • basic_complete = true                                    │
│   • provider_onboarding_complete = true                      │
│   • Adds 'provider' role if not present                      │
│                                                              │
│ Redirect: /onboarding/billing-setup                          │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│ STAGE 2: Billing Setup                                      │
│ (/onboarding/billing-setup)                                 │
├─────────────────────────────────────────────────────────────┤
│ Actions:                                                     │
│   • Create Stripe Connect Account                            │
│   • Complete Stripe onboarding                               │
│   • Link bank account for payouts                            │
│                                                              │
│ Backend Sets:                                                │
│   • billing_setup_complete = true                            │
│   • stripe_account_id = 'acct_xxx'                           │
│   • provider_status = 'pending'                              │
│                                                              │
│ Redirect: /onboarding/awaiting-verification                  │
└─────────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────────┐
│ STAGE 3: Awaiting Admin Approval                            │
│ (/onboarding/awaiting-verification)                         │
├─────────────────────────────────────────────────────────────┤
│ User State:                                                  │
│   • provider_status = 'pending'                              │
│   • Can access seeker features ✅                            │
│   • Cannot access provider-only features ❌                  │
│                                                              │
│ Admin Reviews Application:                                   │
│   • Approves → provider_status = 'approved'                  │
│   • Rejects → provider_status = 'rejected'                   │
│                                                              │
│ After Approval:                                              │
│   • Redirect: /provider-dashboard                            │
│   • Full provider access granted                             │
└─────────────────────────────────────────────────────────────┘
```

#### Key Backend Function

```python
async def complete_provider_onboarding(
    user: User,
    provider_data: ProviderOnboarding
) -> User:
    """
    Completes provider-specific onboarding.
    Sets BOTH basic_complete and provider_onboarding_complete flags.
    """
    # Update basic profile fields
    user.full_name = provider_data.full_name
    user.phone_number = provider_data.phone_number
    user.tradingName = provider_data.tradingName
    
    # Update address
    if provider_data.address:
        user.address = Address(**provider_data.address)
        user.location = compute_location(user.address.model_dump())
    
    # Set onboarding flags
    user.is_provisional = False
    user.onboarding_status.basic_complete = True
    user.onboarding_status.provider_onboarding_complete = True
    
    # Add provider role if not present
    if 'provider' not in user.roles:
        user.roles.append('provider')
    
    await user.save()
    return user
```

---

### Seeker-to-Provider Transition

**Use Case:** Existing seeker decides to become a provider later.

#### Flow Diagram

```
Seeker Dashboard → Click "Become a GigSta" Button
    ↓
Frontend: useProviderTransition.handleProviderAction()
    ↓
Dispatch: Add 'provider' role
    roles: ['user', 'provider']  ← Keeps existing 'user' role
    is_provisional: false
    onboarding_status: {
      provider_onboarding_complete: false,
      basic_complete: true  ← Already set from seeker profile
    }
    ↓
Redirect to /onboarding/provider-details
    ↓
[Continue with Provider Onboarding Stages 1-3]
```

#### Provider Abandonment & Resumption

**Abandonment:**
```
User on /onboarding/provider-details
    ↓
Click "Back to Dashboard" or Navigate Away
    ↓
Router Guard F: Detects !isProviderOnly route
    ↓
Allows navigation (Escape Hatch) ✅
    ↓
User lands on /seeker-dashboard
    ↓
Button changes to "Continue Provider Setup" with progress badge (1/3, 2/3, etc.)
```

**Resumption:**
```
User on /seeker-dashboard
    ↓
Click "Continue Provider Setup"
    ↓
Frontend: useProviderTransition.continueProviderSetup()
    ↓
Determines incomplete step:
    ├─ !provider_onboarding_complete → /onboarding/provider-details
    ├─ !billing_setup_complete → /onboarding/billing-setup
    └─ status='pending' → /onboarding/awaiting-verification
    ↓
Navigates to appropriate step
```

#### Key Composables

**useAuthFlags.js** (Detection)
```javascript
const showBecomeProviderCTA = computed(() =>
  isSeeker.value &&
  !isProvider.value &&  // Pure seeker
  isVerified.value &&
  !isProvisional.value
);

const showContinueProviderSetup = computed(() =>
  isProvider.value &&
  hasIncompleteProviderOnboarding.value  // Any stage incomplete
);
```

**useProviderTransition.js** (Actions)
```javascript
const handleProviderAction = async () => {
  if (showBecomeProviderCTA.value) {
    // New provider: Add role + navigate
    await transitionToProvider();
  } else if (showContinueProviderSetup.value) {
    // Resuming: Navigate to incomplete step
    await continueProviderSetup();
  }
};
```

---

## Authentication Methods

### Traditional Email/Password

**Signup:**
```javascript
POST /api/v1/auth/register
{
  email: "user@example.com",
  password: "securePassword123"
}

Response:
{
  user: {
    email: "user@example.com",
    is_verified: false,
    is_provisional: true,
    requires_password_setup: false,
    roles: []
  }
}
```

**Login:**
```javascript
POST /api/v1/auth/login
{
  email: "user@example.com",
  password: "securePassword123"
}

Response:
{
  access_token: "eyJhbGc...",
  token_type: "bearer",
  user: { /* user object */ }
}
```

### OAuth2 (Google/Facebook)

**Flow:**
```
1. Frontend: window.location.href = '/api/v1/auth/google/authorize'
2. Backend: Redirects to Google OAuth consent screen
3. User: Approves permissions
4. Google: Redirects to '/api/v1/auth/google/callback?code=xxx'
5. Backend: Exchanges code for tokens, creates/updates user
6. Backend: Redirects to frontend with token in URL
7. Frontend: Extracts token, stores in localStorage, auto-login
```

**User Creation (OAuth):**
```python
async def oauth_callback(code: str, provider: str):
    # Exchange code for OAuth tokens
    oauth_data = await exchange_code_for_token(code, provider)
    
    # Get or create user
    user = await get_or_create_oauth_user(
        email=oauth_data['email'],
        oauth_account_id=oauth_data['id'],
        provider=provider
    )
    
    # Set OAuth-specific flags
    user.is_verified = True  # Email auto-verified
    user.is_oauth_registered = True
    user.requires_password_setup = True
    user.is_provisional = True  # Must choose path
    
    await user.save()
    
    # Create JWT token
    access_token = create_access_token(user.id)
    
    # Redirect to frontend with token
    return RedirectResponse(
        url=f"{FRONTEND_URL}/auth/callback?token={access_token}"
    )
```

---

## Router Guard Logic

### Guard Hierarchy

```
A. Authentication Check
    ├─ requiresAuth meta → Redirect to /login
    └─ Pass → Continue
        ↓
B. Prevent Auth Pages for Logged In Users
    ├─ /login, /signup, /reset-password → Redirect to dashboard
    └─ Pass → Continue
        ↓
C. Email Verification Gate
    ├─ !is_verified → Redirect to /check-email
    └─ Pass → Continue
        ↓
D. Public Booking Flow (provisional + bookingIntent)
    ├─ pendingBookingIntent exists → /complete-profile
    └─ Pass → Continue
        ↓
E. General Signup Flow (provisional, no bookingIntent)
    ├─ is_provisional → /onboarding-choice
    └─ Pass → Continue
        ↓
F. Provider Onboarding Funnel
    ├─ isProvider && !isProviderOnly route → Allow (Escape Hatch)
    ├─ isProvider && isProviderOnly route → Check stages:
    │   ├─ !provider_onboarding_complete → /onboarding/provider-details
    │   ├─ !billing_setup_complete → /billing-setup
    │   └─ status='pending' → /awaiting-verification
    └─ Pass → Continue
        ↓
G. Seeker Access Control
    ├─ isSeeker && !isProvider && isProviderOnly route → Redirect
    └─ Pass → Continue
        ↓
H. Role-Based Access Control (RBAC)
    ├─ requiredRoles check → Redirect if missing
    └─ Pass → Allow Navigation
```

### Key Guard Behaviors

**Guard F - Provider Escape Hatch:**
```javascript
if (isEmailVerified && !isProvisional && isUserProvider) {
  const isProviderRouteAttempt = to.meta.isProviderOnly === true;
  
  // ✅ ESCAPE HATCH: Allow seeker features during provider onboarding
  if (!isProviderRouteAttempt) {
    console.log('Allowing non-provider route');
    return next();
  }
  
  // Enforce onboarding only for provider-specific routes
  if (!isProviderOnboardingComplete) {
    return next({name: 'ProviderOnboardingDetails'});
  }
  // ... billing and approval checks
}
```

**Route Meta Examples:**
```javascript
// Provider-only route
{
  path: '/provider-dashboard',
  meta: {
    requiresAuth: true,
    requiredRoles: ['provider'],
    isProviderOnly: true  // ← Enforces onboarding progression
  }
}

// Shared route (accessible to both)
{
  path: '/seeker-dashboard',
  meta: {
    requiresAuth: true,
    requiredRoles: ['user']
    // No isProviderOnly flag
  }
}
```

---

## Key Components & Composables

### Frontend Architecture

```
src/
├── composables/
│   ├── useAuthFlags.js          # State detection (read-only)
│   └── useProviderTransition.js # Actions & transitions
├── components/
│   ├── OnboardingRouter.vue     # Role selection page
│   ├── CompleteBasicProfile.vue # Seeker profile completion
│   ├── ProviderOnboarding.vue   # Provider details form
│   ├── BillingSetup.vue         # Stripe Connect integration
│   └── AwaitingVerification.vue # Pending approval page
├── router/
│   └── index.js                 # Route definitions & guards
└── store/
    └── modules/
        └── auth.js              # Vuex auth state management
```

### useAuthFlags.js

**Purpose:** Detect user state and determine UI visibility.

**Key Exports:**
```javascript
{
  // Roles
  isProvider,
  isSeeker,
  
  // Onboarding
  hasBasicProfile,
  hasProviderOnboarding,
  hasBillingSetup,
  hasIncompleteProviderOnboarding,
  
  // Provider Status
  isProviderApproved,
  isProviderPending,
  isProviderRejected,
  
  // UI Flags
  showBecomeProviderCTA,        // Pure seeker
  showContinueProviderSetup,    // Provider with incomplete onboarding
  
  // Access Control
  canAccessProviderDashboard,
  canAccessSeekerDashboard
}
```

### useProviderTransition.js

**Purpose:** Handle provider transitions and navigation.

**Key Exports:**
```javascript
{
  // State
  isTransitioning,
  
  // UI Helpers
  shouldShowButton,
  buttonText,
  progressText,
  
  // Actions
  transitionToProvider,      // Seeker → Provider (add role)
  continueProviderSetup,     // Navigate to incomplete step
  handleProviderAction       // Smart method (decides which action)
}
```

---

## Database Schema Reference

### User Model

```javascript
{
  _id: ObjectId,
  email: String,
  hashed_password: String,
  
  // Authentication
  is_active: Boolean,
  is_verified: Boolean,
  is_superuser: Boolean,
  is_oauth_registered: Boolean,
  requires_password_setup: Boolean,
  
  // Onboarding State
  is_provisional: Boolean,
  roles: ['user', 'provider'],  // Array of roles
  
  // Profile
  full_name: String,
  phone_number: String,
  tradingName: String,
  profile_picture: String,
  
  // Address (embedded)
  address: {
    formatted: String,
    street_number: String,
    street: String,
    locality: String,
    city: String,
    region: String,
    postcode: String,
    latitude: Number,
    longitude: Number
  },
  
  location: {
    type: 'Point',
    coordinates: [longitude, latitude]
  },
  
  // Onboarding Progress
  onboarding_status: {
    basic_complete: Boolean,
    provider_onboarding_complete: Boolean,
    billing_setup_complete: Boolean
  },
  
  // Provider Status
  provider_status: Enum['not_applied', 'pending', 'approved', 'rejected'],
  
  // Stripe
  stripe_customer_id: String,
  stripe_account_id: String,
  
  // OAuth
  oauth_accounts: [{
    id: ObjectId,
    oauth_name: String,
    account_id: String,
    account_email: String,
    access_token: String,
    refresh_token: String,
    expires_at: Number
  }],
  
  // Timestamps
  created_at: DateTime,
  last_verify_request: DateTime
}
```

### Booking Model

```javascript
{
  _id: ObjectId,
  user_id: ObjectId,           // Reference to User
  gig_id: ObjectId,            // Reference to Gig
  service_id: ObjectId,        // Reference to Service
  provider_id: ObjectId,       // Reference to Provider User
  
  // Booking Details
  scheduled_date: Date,
  scheduled_time: String,
  duration_minutes: Number,
  
  // Customer Info (denormalized for historical record)
  customer_name: String,
  customer_phone: String,
  customer_address: Object,
  
  // Pricing
  price: Number,
  currency: String,
  
  // Status
  status: Enum['pending', 'confirmed', 'completed', 'cancelled'],
  
  // Payment
  payment_method_id: String,
  payment_intent_id: String,
  paid: Boolean,
  
  // Timestamps
  created_at: DateTime,
  updated_at: DateTime
}
```

---

## Decision Log

### Why Both Roles for Providers?

**Decision:** Providers receive both `'user'` and `'provider'` roles.

**Rationale:**
- Providers may want to book services from other providers
- Avoids creating separate "provider-as-seeker" logic
- Allows seamless switching between provider and seeker features
- Escape hatch enables abandoning provider onboarding without losing seeker access

### Why Separate Profile Completion Forms?

**Decision:** `CompleteBasicProfile.vue` (seeker) vs `ProviderOnboarding.vue` (provider)

**Rationale:**
- **CompleteBasicProfile:** Optimized for guest checkout flow
  - Includes password setup for provisional users
  - Minimal fields (name, phone, address)
  - Handles booking intent creation
  - Used by seekers who will never be providers

- **ProviderOnboarding:** Includes business-specific fields
  - Trading name (business name)
  - Business address with geocoding
  - Sets both `basic_complete` and `provider_onboarding_complete`
  - More extensive form appropriate for business setup

### Why Set Both Flags in Provider Form?

**Decision:** `ProviderOnboarding.vue` sets both `basic_complete` and `provider_onboarding_complete`.

**Rationale:**
- Providers collect all basic profile info in their form
- Avoids forcing providers through seeker profile completion
- Streamlines provider onboarding (1 form vs 2)
- `basic_complete` is required for booking services later

### Why Provisional State?

**Decision:** New users start with `is_provisional: true`.

**Rationale:**
- Allows flexible onboarding (choose seeker or provider path)
- Supports guest checkout (user chooses path after booking)
- Prevents premature role assignment
- Clear distinction between "account created" and "onboarding complete"

---

## Common Edge Cases

### 1. OAuth User Books as Guest

**Scenario:** User with OAuth account books a service as guest using same email.

**Behavior:**
- Backend detects existing OAuth user by email
- Marks account as provisional
- User completes profile (including password setup)
- Can now login with email/password OR OAuth

### 2. Provider Abandons at Billing Step

**Scenario:** Provider completes details but abandons at Stripe Connect.

**State:**
```javascript
{
  roles: ['user', 'provider'],
  onboarding_status: {
    basic_complete: true,
    provider_onboarding_complete: true,
    billing_setup_complete: false  // ← Incomplete
  },
  provider_status: 'not_applied'
}
```

**Behavior:**
- "Continue Provider Setup (2/3)" button shows
- Can still browse and book services
- Clicking button navigates to `/billing-setup`
- Cannot access provider dashboard until complete

### 3. Seeker Books While Provider Onboarding Pending

**Scenario:** User has provider application pending but wants to book a service.

**State:**
```javascript
{
  roles: ['user', 'provider'],
  provider_status: 'pending',
  onboarding_status: { all: true }
}
```

**Behavior:**
- Guard F allows non-provider routes (escape hatch)
- Can successfully create bookings
- Provider features remain locked until approval

### 4. User Resets Password (OAuth User)

**Scenario:** OAuth user wants to set password for traditional login.

**Flow:**
```
1. User goes to /reset-password
2. Enters email (registered via OAuth)
3. Receives password reset email
4. Sets new password
5. Can now login with email/password
6. OAuth login still works
```

---

## Testing Checklist

### Public Booking Flow
- [ ] Guest can book without account
- [ ] Email verification works
- [ ] Profile completion prefills from booking intent
- [ ] Booking created after profile completion
- [ ] Redirects to booking detail page
- [ ] localStorage cleaned up after completion

### Seeker Onboarding
- [ ] Traditional signup works
- [ ] OAuth signup works
- [ ] Email verification required (traditional only)
- [ ] Onboarding choice page appears
- [ ] "Find Services" adds 'user' role
- [ ] Redirects to seeker dashboard

### Provider Onboarding
- [ ] "Become a Provider" adds both roles
- [ ] Provider details form works