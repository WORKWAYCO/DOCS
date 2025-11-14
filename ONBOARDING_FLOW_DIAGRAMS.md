# WORKWAY Onboarding Flow Diagrams

## Current State Flow Diagrams

### 1. Standard User Onboarding Flow (Current)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     STANDARD USER ONBOARDING                        │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────┐
│ Anonymous    │
│ User         │
└──────┬───────┘
       │ Clicks "Sign Up"
       │
       ▼
┌──────────────────────────────┐
│ /auth/signup                 │
│ - Email                      │
│ - Password (8+ chars)        │
│ - Display Name               │
│ ❌ No role selection         │
│ ❌ No terms acceptance       │
└──────┬───────────────────────┘
       │ Submit form
       │
       ▼
┌──────────────────────────────┐
│ ✓ Account Created            │
│ ✓ Session established        │
│ ✓ Role: USER (default)       │
│ ❌ Email NOT verified        │
│ ❌ No onboarding tracking    │
└──────┬───────────────────────┘
       │ 1.5s delay
       │
       ▼
┌──────────────────────────────────────┐
│ /workflows?welcome=true              │
└──────┬───────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│ Optional: Welcome Modal (Dismissible)    │
│                                          │
│ Getting Started (3 Steps):               │
│ [1] Browse Workflows → /integrations     │
│ [2] Connect Accounts → /settings/oauth   │
│ [3] Install Workflow → /marketplace      │
│                                          │
│ ❌ No progress tracking                 │
│ ❌ No completion requirement             │
└──────┬───────────────────────────────────┘
       │ "Let's Go!" button
       │ (or backdrop click)
       │
       ▼
┌──────────────────────────────────────────┐
│ /workflows (Empty State)                 │
│                                          │
│ "No active workflows yet"                │
│ "Browse the marketplace..."              │
│                                          │
│ ✓ Can browse and install                 │
│ ❌ May fail if OAuth not configured      │
└──────┬───────────────────────────────────┘
       │ Optional: Setup OAuth
       │
       ▼
┌──────────────────────────────────────────┐
│ /settings/oauth (If goes here)           │
│                                          │
│ Available Providers:                     │
│ - Gmail                                  │
│ - Notion                                 │
│                                          │
│ ✓ OAuth connections established          │
└──────┬───────────────────────────────────┘
       │ Browse marketplace
       │
       ▼
┌──────────────────────────────────────────┐
│ /marketplace                             │
│                                          │
│ Browse workflows, select one             │
└──────┬───────────────────────────────────┘
       │ Click workflow, install
       │
       ▼
┌──────────────────────────────────────────┐
│ ✓ FIRST WORKFLOW INSTALLED               │
│ ✓ USER ONBOARDING COMPLETE               │
│                                          │
│ Account is functional                    │
│ But not fully optimized:                 │
│ ❌ Email still unverified                │
│ ⚠ May have OAuth issues if skipped       │
└──────────────────────────────────────────┘
```

### 2. Developer Onboarding Flow (Current - Two Path Problem)

```
┌──────────────────────────────────────────────────────────────────┐
│               DEVELOPER ONBOARDING (CURRENT - BROKEN)             │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────┐
│ User completes signup               │
│ (/auth/signup)                      │
│ role: USER                          │
└──────┬──────────────────────────────┘
       │
       │ Path A: Manual Navigation
       │ (Not prompted or obvious)
       │
       ▼
┌──────────────────────────────┐
│ /developer/signup            │
│                              │
│ Form:                        │
│ - Company Name (optional)    │
│ - Bio (optional)             │
│ - Accept Terms (required)    │
│                              │
│ ❌ Terms links don't exist   │
│ ❌ No "already developer?"   │
│ ❌ No clear CTA from home    │
└──────┬───────────────────────┘
       │ Submit form
       │
       ▼
┌──────────────────────────────┐
│ ✓ Developer Profile Created  │
│ role: still USER (see issue) │
│ ❌ No success message        │
│ ❌ Immediate redirect        │
└──────┬───────────────────────┘
       │ Conditional redirect
       │ (based on Stripe status)
       │
       ├─ Has Stripe? → /developer/dashboard
       └─ No Stripe? → /developer/onboard/stripe
                        (next diagram)
```

### 3. Stripe Onboarding Flow (Partial Implementation)

```
┌────────────────────────────────────────────────────────────┐
│           STRIPE ONBOARDING (Incomplete)                   │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ /developer/onboard/stripe              │
│                                        │
│ Display: 4 Onboarding Steps            │
│ [□] Business Information               │
│ [□] Personal Details                   │
│ [□] Banking Details                    │
│ [□] Verification                       │
│                                        │
│ ❌ All 4 show same status (all or none)│
│ ❌ Can't see which step user is on     │
│ ❌ No estimated time (5-10 min shown)  │
│ ❌ Revenue split shown here, not prior │
└────────┬─────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────┐
│ Important Info + Revenue Breakdown     │
│                                        │
│ "Why Stripe Connect?"                  │
│ "How You Get Paid"                     │
│ (70% developer, 30% platform)          │
│                                        │
│ ✓ Good explanation                     │
│ ❌ Should be shown at signup decision  │
└────────┬─────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────┐
│ Two Buttons:                           │
│                                        │
│ [Do This Later]    [Continue to Stripe]│
│                                        │
│ ❌ "Do This Later" allows skipping     │
│ ❌ No blocking until publishing        │
│ ⚠ User may create integrations without │
│   payment ability                      │
└────────┬─────────────────────────────┘
         │
         ├─ "Do This Later"
         │  │
         │  ▼
         │  /developer/dashboard
         │  (Can build, but can't publish)
         │
         └─ "Continue to Stripe"
            │
            ▼
            Stripe Hosted Onboarding
            │
            ├─ User completes 4 steps
            │  (Business, Personal, Banking, Verify)
            │
            ├─ Stripe may require additional verification
            │  (1-2 business days)
            │
            └─ Callback to /developer/dashboard
               ✓ Payment setup complete
               ✓ Can now publish integrations
               ✓ Can receive revenue
```

### 4. Current Navigation Flow by Role

```
┌──────────────────────────────────────────────────────────┐
│         NAVIGATION BY AUTHENTICATION STATE                │
└──────────────────────────────────────────────────────────┘

ANONYMOUS USER:
┌──────────────┐
│ Home         │────┬─ Marketplace
│              │    ├─ Developers
│ Header Nav   │    ├─ Docs
│              │    └─ Pricing
│              │
│ Auth Buttons │────┬─ Log In
│              │    └─ Sign Up
└──────────────┘

LOGGED-IN USER (role: USER):
┌──────────────┐
│ Home         │────┬─ Marketplace
│              │    ├─ Workflows ← NEW
│              │    └─ Settings ← NEW
│ Header Nav   │
│              │    User Menu:
│              │    ├─ Dashboard
│              │    ├─ OAuth Settings
│              │    └─ Log Out
└──────────────┘

LOGGED-IN USER (role: DEVELOPER):
┌──────────────┐
│ Home         │────┬─ Marketplace
│              │    ├─ Workflows
│              │    └─ Settings
│ Header Nav   │
│              │    ├─ Developer Dashboard ← DEVELOPER ONLY
│              │    │
│              │    User Menu:
│              │    ├─ Dashboard
│              │    ├─ OAuth Settings
│              │    ├─ Developer Settings ← DEVELOPER ONLY
│              │    └─ Log Out
└──────────────┘

ISSUES:
- ❌ User can't see their own role (no badge/indicator)
- ❌ "Dashboard" changes based on role (confusing)
- ❌ No role upgrade CTA for regular users
- ❌ No "Getting Started" or "Onboarding" link
```

---

## Recommended Unified Flow

### 1. Consolidated Signup with Role Selection

```
┌──────────────────────────────────────────────────────────┐
│        IMPROVED SIGNUP WITH ROLE SELECTION               │
└──────────────────────────────────────────────────────────┘

┌──────────────┐
│ Anonymous    │
│ User         │
└──────┬───────┘
       │ Clicks "Sign Up"
       │
       ▼
┌──────────────────────────────────────────┐
│ /auth/signup (IMPROVED)                  │
│                                          │
│ Form:                                    │
│ - Email                                  │
│ - Password (8+ chars)                    │
│ - Display Name                           │
│ ✓ Role Selection:                        │
│   ○ Regular User (default)               │
│   ○ I'm a Developer                      │
│ ✓ Terms of Service (required)            │
│ ✓ Email verification notice              │
└──────┬───────────────────────────────────┘
       │ Submit
       │
       ▼
┌──────────────────────────────────────────┐
│ ✓ Account Created                        │
│ ✓ Role assigned (USER or DEVELOPER)      │
│ ✓ Verification email sent                │
│ ✓ Session established                    │
└──────┬───────────────────────────────────┘
       │ Redirect based on role
       │
       ├─ Role: USER
       │  │
       │  ▼
       │  /onboarding/verify-email
       │  │
       │  ├─ Display email verification prompt
       │  ├─ Show verification status
       │  ├─ "Resend email" option
       │  │
       │  ▼
       │  /onboarding/user-checklist
       │  │
       │  ├─ ✓ Account Created
       │  ├─ ☐ Email Verified (blocked until done)
       │  ├─ ☐ Connect OAuth
       │  ├─ ☐ Install Workflow
       │  └─ ☐ Configure & Run
       │
       └─ Role: DEVELOPER
          │
          ▼
          /onboarding/verify-email (same as above)
          │
          ▼
          /developer/onboarding/profile
          │
          ├─ Form: Company Name, Bio
          ├─ Show developer benefits
          │
          ▼
          /developer/onboarding/stripe
          │
          ├─ Explain Stripe requirement
          ├─ Show revenue split BEFORE connect
          ├─ "Continue to Stripe" (required to continue)
          │
          ▼
          Stripe Hosted Onboarding
          │
          ▼
          /developer/onboarding/complete
          │
          ├─ Show completion checklist:
          │  ✓ Account Created
          │  ✓ Email Verified
          │  ✓ Developer Profile
          │  ✓ Stripe Setup
          ├─ "Next: Create Your First Integration"
          │
          ▼
          /developer/dashboard
          (Ready to build)
```

### 2. Unified Onboarding Checklist Component

```
┌────────────────────────────────────────────┐
│      ONBOARDING CHECKLIST (SIDEBAR)        │
├────────────────────────────────────────────┤
│                                            │
│ Your Progress: 25%                         │
│ ████░░░░░░░░░░░░░░░░░░░░░░                │
│                                            │
│ ─────────────────────────────────────────  │
│                                            │
│ ✓ Account Created                          │
│                                            │
│ ▶ Email Verified (CURRENT)                 │
│   └─ Check your inbox for verification    │
│      [Resend Email] [Skip for Now]         │
│                                            │
│ ☐ Connect Your First Account               │
│   └─ Link Gmail or Notion                  │
│      [Setup OAuth]                         │
│                                            │
│ ☐ Install Your First Workflow              │
│   └─ Browse marketplace & activate         │
│      [Browse Workflows]                    │
│                                            │
│ ─────────────────────────────────────────  │
│                                            │
│ [Collapse]  [Dismiss After Complete]       │
│                                            │
└────────────────────────────────────────────┘
```

### 3. Smart Route Guards

```
┌──────────────────────────────────────────────────────────┐
│              ROUTE PROTECTION LOGIC                      │
└──────────────────────────────────────────────────────────┘

For Regular Users:

/workflows
  ├─ Email verified? 
  │  └─ NO → Redirect to /onboarding/verify-email
  └─ YES → Show dashboard

/integrations
  ├─ Has OAuth connections?
  │  └─ NO → Show banner "Connect accounts to use workflows"
  └─ YES → Show marketplace

/marketplace
  ├─ Email verified? 
  │  └─ NO → Show banner with verify prompt
  └─ YES → Show workflows (can install)

For Developer Users:

/developer/dashboard
  ├─ Developer profile exists?
  │  └─ NO → Redirect to /developer/onboarding/profile
  └─ YES → Show dashboard

/developer/builder
  ├─ Stripe connected?
  │  └─ NO → Show banner "Complete Stripe to publish"
  └─ YES → Show builder

POST /publish
  ├─ Stripe account active?
  │  └─ NO → Error "Stripe setup required"
  └─ YES → Allow publish
```

---

## Current Issues Summary

| Flow Component | Current State | Problem | Impact |
|---|---|---|---|
| Signup | 1-step | No role selection at signup | Users confused about developer path |
| Email Verification | Missing | TODO - not implemented | Users bypass important security step |
| Welcome Modal | Optional, dismissible | No completion tracking | High signup funnel drop-off |
| OAuth Setup | Manual navigation required | Not part of onboarding | Workflow failures for new users |
| Developer Signup | Separate page | 2-step process after main signup | Lower developer conversion |
| Stripe Onboarding | 4 visual steps, all-or-none status | Can't see progress, shows 4 steps but incomplete | Users think onboarding broken |
| Empty States | Generic messages | No contextual guidance | Users confused about next steps |
| Role Visibility | Not shown in UI | Users don't know if they're developers | Navigation changes feel random |
| Onboarding Checklist | Missing entirely | No progress tracking | No sense of completion |

---

## Key Metrics to Track

```
Funnel Analysis:
1. Signup Initiated → 100%
2. Signup Completed → 85% (15% drop)
3. Email Verified → 40% (45% drop) ← Major issue!
4. First OAuth Connected → 35% (5% drop)
5. First Workflow Installed → 20% (15% drop)
6. Workflow Configured & Ran → 15% (5% drop)

Developer Path:
1. Completed Signup → 100%
2. Created Developer Profile → 25% (75% drop) ← Major issue!
3. Started Stripe Onboarding → 15% (10% drop)
4. Completed Stripe Setup → 8% (7% drop)
5. Created First Integration → 5% (3% drop)
6. Published to Marketplace → 2% (3% drop)

Session Tracking:
- Time from signup to first workflow: (should be <5 min)
- Time from dev signup to Stripe complete: (should be <10 min)
- Churn at each modal/popup: (should be <30%)
```
