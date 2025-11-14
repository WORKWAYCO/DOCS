# WORKWAY Comprehensive Onboarding & User Flow Analysis

## Executive Summary

The WORKWAY platform has implemented a dual-track user flow system supporting both regular users and developers. The authentication and onboarding flows are partially complete but contain critical gaps, inconsistent messaging, and missing steps that create friction in the user experience. Below is a detailed analysis with findings and recommendations.

---

## 1. AUTHENTICATION FLOW ANALYSIS

### 1.1 Login Flow
**File:** `/apps/web/src/routes/auth/login.tsx`

Current Implementation:
- Email/Password authentication
- Remember me option
- Basic error handling
- Post-login redirect: `/workflows` (hardcoded)

Issues:
- No forgot password functionality (link present but non-functional)
- No account recovery mechanisms
- No session persistence verification on app load
- Single redirect target regardless of user role

### 1.2 Signup Flow
**File:** `/apps/web/src/routes/auth/signup.tsx`

Current Implementation:
- Email, password, display name fields
- Password validation (8 character minimum)
- Success message with 1.5s delay before redirect
- Redirect to `/workflows` with `welcome=true` parameter
- Proper error handling

Issues:
- No role selection during signup (all users default to 'USER' role)
- No email verification step (noted as TODO in backend)
- No terms of service acceptance
- No distinction between regular user and developer signup paths at entry point
- Success state shown but user still sees form briefly after redirect

### 1.3 Session Management
**File:** `/apps/web/src/context/AuthContext.tsx` & `/apps/api/src/lib/sessions.ts`

Current Implementation:
- localStorage stores `workway_session_id`
- Cookie-based sessions with httpOnly, Secure, SameSite=None
- Session duration: LONG (default) and REMEMBER_ME (longer duration)
- Backend validates sessions via KV store
- Auto-refresh on app mount

Issues:
- Inconsistent session handling: localStorage ID + cookie session
- No session refresh mechanism for long-lived apps
- No clear UX feedback for session expiration
- No logout confirmation or in-progress indicator

---

## 2. FIRST-TIME USER EXPERIENCE (FTU) FLOW

### 2.1 Post-Signup Experience
**File:** `/apps/web/src/routes/workflows.tsx`

Current Implementation:
- Welcome modal appears when `welcome=true` search param exists
- Modal shows "Getting Started (3 Steps)"
- Links to: Browse Marketplace → Connect Accounts → Install Workflow
- Closes on button click or backdrop click

Issues:
- Welcome modal is optional/dismissible without completion indication
- 3-step process suggests completing all, but each is independent
- No progress tracking between steps
- No "onboarding checklist" or completion status
- Modal doesn't prevent user from skipping to other pages

### 2.2 Post-Login Experience
**File:** `/apps/web/src/routes/workflows.tsx`

Current Implementation:
- Loads active installations (workflow instances)
- Shows empty state if no workflows installed
- Displays "Create New Workflow" button

Issues:
- No distinction between "new account" and "returning user"
- Empty state could be more helpful (no suggested next steps)
- No "getting started" equivalent for returning users
- No quick-start or tutorial options

### 2.3 Navigation After Auth
**File:** `/apps/web/src/components/Header.tsx`

Navigation Changes by Auth State:

Anonymous Users See:
- Marketplace, Developers, Docs, Pricing
- Login/Signup buttons

Logged-In Users (USER role) See:
- Marketplace, Workflows, Settings
- User menu with Dashboard, OAuth Settings, Logout

Logged-In Users (DEVELOPER role) See:
- Marketplace, Workflows, Settings/Dashboard (developer-specific)
- User menu with Developer Dashboard, OAuth Settings, Developer Settings

Issues:
- No progressive disclosure (all options shown to all logged-in users)
- No clear "onboarding" or "getting started" link
- Developer features not highlighted until onboard complete
- Settings navigation is not intuitive (OAuth vs Developer Settings)

---

## 3. DEVELOPER ONBOARDING FLOW

### 3.1 Developer Signup Path
**File:** `/apps/web/src/routes/developer.signup.tsx`

Current Implementation:
- Requires authentication first (guards against unauthenticated access)
- Form fields: Company Name (optional), Bio (optional), Terms acceptance (required)
- Benefits section showing developer value propositions
- "What Happens Next?" section with 4 numbered steps
- Creates developer profile in backend
- Conditional redirect based on Stripe setup status

Flow:
1. User clicks "Start Building" (from home)
2. Redirected to `/developer/signup` 
3. If not authenticated → shows login/signup prompts
4. If authenticated → shows developer signup form
5. On submit → creates developer profile
6. If Stripe exists → redirects to `/developer/dashboard`
7. If no Stripe → redirects to `/developer/onboard/stripe`

Issues:
- No clear distinction from regular signup
- Terms link points to `/terms/developer` and `/terms/marketplace` (likely don't exist)
- Doesn't check if user is already a developer
- "What Happens Next" shows 4 steps but only 2 are actually implemented
- No back button to cancel workflow
- Step 2 (Stripe Connect) makes the process feel incomplete

### 3.2 Stripe Onboarding
**File:** `/apps/web/src/routes/developer.onboard.stripe.tsx`

Current Implementation:
- Loads developer profile on mount
- Shows 4 onboarding steps (Business, Personal, Banking, Verification)
- Displays status: completed, pending, in-progress
- Shows important information about Stripe
- Shows revenue split breakdown (70% developer, 30% platform)
- "Do This Later" or "Continue to Stripe" buttons
- Redirects to Stripe's hosted onboarding flow

Issues:
- All 4 steps show same completion status (all or nothing)
- Can't distinguish which step user is on if revisiting
- "Do This Later" button doesn't prevent payment acceptance (no blocking)
- No estimated time duration mentioned
- No status checking mechanism (must manually reload)
- Revenue breakdown shown here, not during signup decision

### 3.3 Developer Dashboard
**File:** `/apps/web/src/routes/developer.dashboard.tsx`

Current Implementation (partial - first 100 lines):
- Loads developer profile
- Loads integrations (transformed to workflow format)
- Shows loading state and error handling

Issues:
- Not fully visible in analysis, but likely missing:
  - Onboarding status indicators
  - Next steps for incomplete onboards
  - Call-to-action for Stripe if not completed

---

## 4. USER ROLE DIFFERENTIATION

### 4.1 Role Assignment
**Backend:** `/apps/api/src/routes/auth.ts` (line 47)

Current Implementation:
- Default role: 'USER'
- Developer role: Only via `/developer/signup` after account creation
- Admin role: Not visible in onboarding flows

Navigation Implementation:
**Header Component** checks `user.role === 'DEVELOPER'` for conditional nav items

Issues:
- Two-step process to become developer (signup, then developer signup)
- No explicit role selection during main signup
- No role conversion pathway (USER → DEVELOPER)
- Role is used but not prominently displayed

### 4.2 Role-Based Content
**Implemented:**
- Header nav shows "Dashboard" only for DEVELOPER role
- User menu shows "Developer Settings" only for DEVELOPER role
- `/developer/*` routes guard access

**Missing:**
- Role indicator in UI (users don't see their own role)
- Role upgrade prompts or suggestions
- Role-specific onboarding experiences
- Role-based home page customization

---

## 5. SUCCESS & COMPLETION STATES

### 5.1 Signup Success
**File:** `/apps/web/src/routes/auth/signup.tsx` (line 76-81)

Current Implementation:
- Green success message: "Account created successfully! Redirecting..."
- 1.5 second delay before redirect
- Redirect to `/workflows?welcome=true`

Issues:
- Success message brief and easy to miss
- Welcome modal optional, not required
- No confirmation checklist
- Form still visible while "redirecting" message shows

### 5.2 Developer Signup Success
**File:** `/apps/web/src/routes/developer.signup.tsx`

Current Implementation:
- Button state changes to "Creating Profile..." with spinner
- Redirects on success
- Error displayed in alert box

Issues:
- No success message/confirmation
- Immediate redirect feels abrupt
- Success state not reinforced

### 5.3 Onboarding Completion Indicators
**Missing:**
- No onboarding checklist/progress bar
- No completion badges or certificates
- No "next steps" after each stage
- No email confirmation flows
- No achievement/milestone notifications

---

## 6. GAPS & INCONSISTENCIES

### Critical Gaps

1. **Email Verification Missing**
   - Signup creates users with `email_verified: false`
   - No verification email sent (TODO comment in code)
   - Users can use unverified emails for integrations
   - No resend verification mechanism

2. **Two-Step Developer Signup**
   - Confusing UX: Signup → then Developer Signup
   - No role selection during initial signup
   - Separation creates friction

3. **Incomplete Onboarding Flows**
   - No checkpoints to prevent skipping critical steps
   - Users can access features without setup
   - Stripe onboarding is optional but necessary

4. **Missing OAuth Setup**
   - Users must manually navigate to Settings → OAuth
   - No onboarding prompt for OAuth connections
   - Not integrated into first-time workflow creation

5. **No Account Recovery**
   - Forgot password link exists but non-functional
   - No password reset implementation visible
   - No account deletion/management

### Inconsistent Messaging

1. **Developer Onboarding Steps**
   - Developer Signup: Shows 4 "next steps"
   - Only Stripe onboarding fully implemented
   - Discrepancy between promised and delivered

2. **Empty States**
   - No workflows: Generic message
   - No OAuth connections: No guidance
   - No developer integrations: No creation prompt

3. **Navigation Labeling**
   - "Settings" vs "OAuth Settings" vs "Developer Settings"
   - Unclear which settings apply where

---

## 7. DETAILED ONBOARDING FLOW MAP

### Standard User (Regular Signup)
```
1. Anonymous User
   ↓
2. Click "Sign Up" (homepage or header)
   ↓
3. `/auth/signup` form
   - Email, Password, Display Name
   - Terms NOT required
   ↓
4. Success message → Auto-redirect after 1.5s
   ↓
5. `/workflows?welcome=true`
   ↓
6. Welcome modal (optional, dismissible)
   ├─ Step 1: Browse Workflows → `/integrations`
   ├─ Step 2: Connect Accounts → `/settings/oauth`
   └─ Step 3: Install Workflow → `/marketplace`
   ↓
7. User can dismiss and navigate freely
   ↓
8. First workflow installation
```

### Developer User (Two-Path Signup)
```
Path A: Signup First, Then Developer
1. Signup via standard `/auth/signup`
   ↓
2. `/workflows?welcome=true` (same as above)
   ↓
3. User manually navigates to `/developer/signup` (not prompted)
   ↓
4. Form: Company Name, Bio, Terms
   ↓
5. Success → `/developer/onboard/stripe`
   ↓
6. Stripe setup (4 steps)
   ├─ Can click "Do This Later"
   └─ Redirects to `/developer/dashboard`

Path B: Signup Page Links Directly (Not Fully Implemented)
1. From homepage: "Start Building" → `/developer/signup`
   ├─ If not authenticated → Prompts login/signup
   └─ If authenticated → Shows developer form
```

### Post-Onboarding Journeys

**For Regular Users:**
- Browse marketplace integrations
- Click integration → Redirects to integration details
- Can install workflows without OAuth setup (integration will fail)
- Setup OAuth at `/settings/oauth`

**For Developers:**
- Receive developer profile creation confirmation
- Must complete Stripe Connect
- Access dev dashboard at `/developer/dashboard`
- Navigate to `/developer/builder` (if exists)
- Publish integrations to marketplace

---

## 8. ISSUES WITH SPECIFIC USER FLOWS

### Issue: No OAuth Setup Guidance
- Welcome modal suggests connecting accounts
- But provides direct link to `/settings/oauth`
- No inline OAuth connection in workflow creation
- Users frustrated if they install workflow without OAuth

### Issue: Developer Onboarding Dead-ends
- After `/developer/onboard/stripe`, user can click "Do This Later"
- This allows users to create integrations without payment setup
- Later publishing to marketplace fails due to missing Stripe

### Issue: Empty State Inconsistencies
- `/workflows` with no installations: Generic "Browse Workflows"
- `/developer/dashboard` with no integrations: Unknown (partially visible)
- No contextual help or suggested actions

### Issue: Navigation Confusion
- Users don't know their role (USER vs DEVELOPER)
- "Dashboard" link changes based on role but no indication why
- "Settings" options fragmented across multiple pages

---

## 9. RECOMMENDATIONS FOR UNIFIED ONBOARDING

### Phase 1: Consolidate Signup Flows
```
RECOMMENDATION 1.1: Role Selection During Signup
- Add optional "I'm a developer" checkbox to `/auth/signup`
- If checked: After account creation → `/developer/signup` automatically
- If unchecked: Default USER role → `/workflows?welcome=true`
- Single signup experience instead of two
```

### Phase 2: Email Verification
```
RECOMMENDATION 2.1: Implement Verification Flow
- Send verification email after signup
- Require verification before OAuth connections
- Show "email pending verification" banner
- Provide resend verification option
- Store verification status and prompt users
```

### Phase 3: Guided Onboarding Checklists
```
RECOMMENDATION 3.1: Create Onboarding Checklist Component
- For Regular Users:
  ✓ Account Created
  ☐ Email Verified
  ☐ Connect OAuth (Gmail, Notion)
  ☐ Install First Workflow
  ☐ Configure & Run
  
- For Developers:
  ✓ Account Created
  ✓ Developer Profile Created
  ☐ Email Verified
  ☐ Stripe Connect Complete
  ☐ Create First Integration
  ☐ Publish to Marketplace

RECOMMENDATION 3.2: Checklist Placement
- Show in sidebar until 80% complete
- Highlight current step
- Allow collapsing but not dismissing
- Show progress percentage
```

### Phase 4: Intelligent Routing
```
RECOMMENDATION 4.1: Post-Signup Router
export function useOnboardingRouter() {
  const { user } = useAuth()
  const { completedSteps } = useOnboarding()
  
  // Regular users
  if (user.role === 'USER') {
    if (!user.emailVerified) return '/onboarding/verify-email'
    if (completedSteps.emailVerified < 100) return '/onboarding/connect-oauth'
    if (!hasInstalledWorkflow) return '/marketplace'
    return '/workflows'
  }
  
  // Developer users
  if (user.role === 'DEVELOPER') {
    if (!user.emailVerified) return '/onboarding/verify-email'
    if (!stripeConnected) return '/developer/onboard/stripe'
    if (!createdIntegration) return '/developer/builder?onboarding=true'
    return '/developer/dashboard'
  }
}

RECOMMENDATION 4.2: Route Guards
- Prevent accessing `/developer/dashboard` without profile
- Prevent publishing integrations without Stripe
- Prevent running workflows without OAuth setup
```

### Phase 5: Improved Empty States
```
RECOMMENDATION 5.1: Contextual Empty States
/workflows (no installations):
- Icon + Title: "No Workflows Yet"
- Description: "Choose from our marketplace to get started"
- CTA: "Browse Workflows" (primary)
- Secondary: "View Popular Integrations" or "See Trending"

/developer/dashboard (no integrations):
- Icon + Title: "Build Your First Integration"
- Description: "Create powerful workflow automations"
- CTA: "Create Integration" (primary)
- Secondary: "View Documentation" or "See Examples"

/settings/oauth (no connections):
- Icon + Title: "Connect Your Accounts"
- Description: "Link Gmail, Notion, and other services"
- Cards for each provider with connect buttons
- Note: "Required to use workflows"
```

### Phase 6: Enhanced User Profile Visibility
```
RECOMMENDATION 6.1: User Profile Display
- Show role badge in header ("Developer" or "User")
- Show account status (verified/unverified)
- Show onboarding progress in account menu
- Allow role upgrade from settings

RECOMMENDATION 6.2: Account Status Indicators
- Email verification: Yellow banner with "Verify Email" button
- OAuth missing: Inline banner in workflows section
- Stripe incomplete: Warning in dev dashboard
```

### Phase 7: Onboarding Modals & Tooltips
```
RECOMMENDATION 7.1: Smart Onboarding Modals
- First workflow installation: Show OAuth requirement if not connected
- First developer integration: Show Stripe requirement
- Each modal: Explain why step is needed, show time estimate

RECOMMENDATION 7.2: Progressive Disclosure
- Hide developer features from USER role users
- Show "Upgrade to Developer" CTA in main nav
- Provide explanation of developer benefits
```

### Phase 8: Completion & Graduation
```
RECOMMENDATION 8.1: Milestone Celebrations
- Email verification complete: Celebration modal
- First workflow installed: "You're all set!" message
- Stripe onboarding complete: "Ready to earn" message
- First integration published: Achievement notification

RECOMMENDATION 8.2: Graduation Flows
- After completing mandatory steps:
  - Close onboarding checklist
  - Show "Welcome to WORKWAY" screen
  - Provide next learning steps (docs, examples)
```

---

## 10. IMPLEMENTATION PRIORITY

### High Priority (Blocks User Experience)
1. Email verification flow (currently TODO)
2. Role selection during signup (prevents confusion)
3. Onboarding checklist (provides clarity)
4. Post-signup success confirmation (users need reassurance)
5. OAuth setup prompts (prevents workflow failures)

### Medium Priority (Improves Experience)
1. Empty state improvements (better guidance)
2. User role visibility (less confusion)
3. Stripe onboarding status (prevents dead-ends)
4. Developer feature highlights (encourages upgrade)
5. Account recovery (password reset, etc.)

### Low Priority (Nice-to-Have)
1. Onboarding celebrations/tooltips (engagement)
2. Graduation flows (retention)
3. Advanced role management (future feature)
4. Onboarding analytics tracking (optimization)

---

## 11. TESTING RECOMMENDATIONS

### User Flows to Test
1. Fresh signup → Complete first workflow installation (full funnel)
2. Fresh signup → Become developer → Publish integration (developer flow)
3. Email verification workflow
4. OAuth connection during workflow creation
5. Session expiration handling
6. Returning user experience (dashboard load)

### Metrics to Track
- Signup completion rate
- Time to first workflow installation
- Time to Stripe setup completion
- OAuth connection rate
- Developer profile creation rate
- Churn at each onboarding step

---

## 12. SUMMARY TABLE

| Component | Status | Issues | Recommendation |
|-----------|--------|--------|-----------------|
| Auth Login | Complete | Weak recovery, no forgot password | Add password reset |
| Auth Signup | Complete | No role selection, email verify TODO | Add role choice, implement email |
| Sessions | Complete | Inconsistent storage method | Consolidate to cookies only |
| Welcome Modal | Partial | Optional, too dismissible | Make required until steps done |
| Onboarding Checklist | Missing | N/A | Build and integrate |
| Role-Based Nav | Partial | No visibility, confusing | Add badges, explain features |
| Empty States | Weak | Generic messages | Add contextual guidance |
| Dev Signup | Partial | Two-step process confusing | Consolidate with main signup |
| Stripe Onboarding | Partial | All-or-nothing status | Track individual steps |
| OAuth Setup | Missing | Not in onboarding flow | Integrate into workflows |
| Email Verification | Missing | Users can skip | Implement flow with blocks |
| First-Time UX | Weak | No clear next steps | Build guided paths |

