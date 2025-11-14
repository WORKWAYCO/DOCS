# WORKWAY Onboarding - Quick Reference Guide

## Critical Issues (Must Fix)

### 1. Email Verification TODO
- Status: Not implemented
- Impact: Users can bypass security verification
- Fix: Implement email verification flow with resend option
- Files: `/apps/api/src/routes/auth.ts` (line 71 - TODO comment)

### 2. Two-Step Developer Signup
- Status: Current flow requires signup → then developer signup
- Impact: 75% drop-off in developer conversion
- Fix: Add role selection to main signup form
- Files: `/apps/web/src/routes/auth/signup.tsx` + `/apps/web/src/routes/developer.signup.tsx`

### 3. Stripe Onboarding Dead-End
- Status: "Do This Later" button allows skipping
- Impact: Users create integrations they can't publish
- Fix: Make Stripe mandatory or block publishing without it
- Files: `/apps/web/src/routes/developer.onboard.stripe.tsx`

### 4. Missing Onboarding Checklist
- Status: No progress tracking component
- Impact: Users don't see what they've completed
- Fix: Build reusable checklist component for sidebar
- Files: Create `/apps/web/src/components/OnboardingChecklist.tsx`

### 5. Inconsistent Session Handling
- Status: localStorage ID + cookie session
- Impact: Session management complexity
- Fix: Consolidate to cookie-only or localStorage-only
- Files: `/apps/web/src/context/AuthContext.tsx`

---

## Implementation Priority

```
Week 1 (High Priority):
□ Email verification flow
□ Add role selection to signup
□ Build onboarding checklist component
□ Implement route guards for incomplete onboarding

Week 2 (High Priority):
□ Make Stripe onboarding mandatory for developers
□ Add OAuth setup to welcome modal
□ Improve empty states
□ Add role visibility to header

Week 3 (Medium Priority):
□ Password reset / account recovery
□ Better Stripe status tracking
□ Developer feature highlights
□ Analytics/tracking for onboarding

Week 4 (Low Priority):
□ Celebration modals for milestones
□ Advanced role management
□ Tutorial videos/tooltips
□ Graduation flows
```

---

## File Changes Required

### Frontend Changes

#### `/apps/web/src/routes/auth/signup.tsx`
- Add role selection radio buttons
- Add terms acceptance checkbox
- Remove hardcoded redirect (based on role now)
- Add email verification explanation

#### `/apps/web/src/routes/auth/login.tsx`
- Add post-login redirect logic based on user role & onboarding status
- Add forgot password implementation (currently non-functional)

#### `/apps/web/src/context/AuthContext.tsx`
- Add onboarding status tracking
- Add role/email verification info
- Remove localStorage session ID (use cookies only)
- Add session refresh mechanism

#### `/apps/web/src/routes/developer.signup.tsx`
- Make this optional (triggered from main signup if role selected)
- Remove authentication guard (should be authenticated from signup)
- Add success confirmation

#### `/apps/web/src/routes/developer.onboard.stripe.tsx`
- Track individual step completion (not all-or-none)
- Remove "Do This Later" button (make mandatory)
- Show step progress indicator
- Add estimated time

#### `/apps/web/src/components/Header.tsx`
- Add role badge next to user name
- Add onboarding status indicator
- Show "Getting Started" link for incomplete onboarding

#### `/apps/web/src/routes/workflows.tsx`
- Replace welcome modal with onboarding checklist
- Add distinction for returning users vs first-timers
- Better empty state with next steps

#### Create `/apps/web/src/components/OnboardingChecklist.tsx`
- New component for persistent onboarding tracking
- Sidebar component, collapsible
- Show progress percentage
- Link current step actions

#### Create `/apps/web/src/routes/onboarding/verify-email.tsx`
- Email verification page
- Resend email button
- Status indicator
- Link to continue

#### Create `/apps/web/src/routes/onboarding/connect-oauth.tsx`
- OAuth setup guidance
- Inline OAuth connections
- Show benefits of connecting
- Link to settings if skipped

#### Create route guards middleware
- Check email verification status
- Check developer profile existence
- Check Stripe setup status
- Redirect with appropriate message

### Backend Changes

#### `/apps/api/src/routes/auth.ts`
- Implement email verification flow
- Send verification email after signup
- Add role parameter to register endpoint
- Implement password reset
- Add email verification check endpoint

#### `/apps/api/src/lib/sessions.ts`
- No changes needed (already good)

#### `/apps/api/src/routes/developers.ts`
- Add onboarding status endpoint
- Track profile creation date
- Add Stripe completion check
- Return onboarding steps status

#### Create `/apps/api/src/routes/onboarding.ts`
- GET /onboarding/status - Returns user's onboarding progress
- POST /onboarding/verify-email - Check verification status
- POST /onboarding/resend-verification - Resend verification email
- GET /onboarding/checklist - Returns checklist items with status

---

## Testing Checklist

### Happy Paths to Test
- [ ] Signup → First workflow installed (5 min)
- [ ] Signup → Developer → Stripe → Build integration (10 min)
- [ ] Email verification → Try workflow (should work)
- [ ] Email unverified → Try certain actions (should block)
- [ ] OAuth missing → Try workflow (should prompt)
- [ ] Stripe incomplete → Try publish (should block)

### Edge Cases
- [ ] Already developer? Don't show dev signup again
- [ ] Returning user after 30 days? Show simplified onboarding
- [ ] Session expiration? Show clear message + re-login prompt
- [ ] Incomplete dev onboarding + revisit? Show progress

### Regression Testing
- [ ] Regular login/logout still works
- [ ] Remember me still works
- [ ] OAuth still works
- [ ] Developer dashboard loads
- [ ] Marketplace still shows
- [ ] Workflow installation unaffected

---

## Success Metrics

After implementation, track:

| Metric | Target | Current (Est.) |
|--------|--------|---|
| Signup completion | 90% | 85% |
| Email verification | 80% | 0% (not implemented) |
| First workflow < 5min | 60% | 20% |
| Developer profile creation | 25% | 25% |
| Stripe setup completion | 15% | 8% |
| Publish rate | 5% | 2% |

---

## Code Snippets for Reference

### Example: Onboarding Router
```typescript
export function useOnboardingRouter(user: User) {
  if (!user) return '/'
  
  if (user.role === 'DEVELOPER') {
    if (!user.emailVerified) return '/onboarding/verify-email'
    const profile = useDeveloperProfile()
    if (!profile?.stripeConnected) return '/developer/onboard/stripe'
    return '/developer/dashboard'
  }
  
  // Regular user
  if (!user.emailVerified) return '/onboarding/verify-email'
  return '/workflows'
}
```

### Example: Checklist Component
```typescript
interface OnboardingStep {
  id: string
  label: string
  completed: boolean
  action?: {
    label: string
    href: string
  }
}

function OnboardingChecklist({ steps }: { steps: OnboardingStep[] }) {
  const completed = steps.filter(s => s.completed).length
  const percentage = Math.round((completed / steps.length) * 100)
  
  return (
    <div className="...">
      <div>Your Progress: {percentage}%</div>
      <ProgressBar value={percentage} />
      {steps.map(step => (
        <ChecklistItem key={step.id} step={step} />
      ))}
    </div>
  )
}
```

---

## References

Full Documentation:
- See `ONBOARDING_FLOW_ANALYSIS.md` for detailed analysis
- See `ONBOARDING_FLOW_DIAGRAMS.md` for visual flows

Related Files:
- Auth Context: `/apps/web/src/context/AuthContext.tsx`
- Auth Routes: `/apps/api/src/routes/auth.ts`
- Sessions: `/apps/api/src/lib/sessions.ts`
- Header: `/apps/web/src/components/Header.tsx`
