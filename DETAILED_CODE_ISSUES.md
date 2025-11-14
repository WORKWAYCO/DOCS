# Detailed Technical Issues - Code Evidence

## Issue 1: /integrations Dead Code Route

### File: `apps/web/src/routes/integrations.tsx`

**Current Implementation (BROKEN):**
```tsx
// Lines 1-5: Route definition
export const Route = createFileRoute('/integrations')({
  component: () => <Navigate to="/marketplace" replace />
});

// Lines 7-215: Dead code - never executes
function Integrations() {
  const [searchQuery, setSearchQuery] = useState('');
  // ... entire component defined but unused
}
```

**Problem:**
- Route silently redirects without user feedback
- Component code (lines 7-215) is dead code - never used
- Imports like `useState` are used in dead code
- Multiple navigation paths point here expecting real content

**Where It's Broken:**
1. Homepage: `apps/web/src/routes/index.tsx:72` - "Browse Integrations" button
2. Welcome Modal: `apps/web/src/routes/workflows.tsx:136` - Step 1 "Browse Workflows"
3. Header: `apps/web/src/components/Header.tsx:62` - Desktop navigation link
4. Mobile nav: `apps/web/src/components/Header.tsx:240` - Mobile marketplace link

**Impact:**
- SEO: Redirect chains look broken to crawlers
- UX: Users click "Browse" and mysteriously land on different page
- Maintenance: Dead code confuses developers

---

## Issue 2: Marketplace API/UI Mismatch

### File: `apps/web/src/routes/marketplace.tsx`

**Current Implementation (MIXED):**
```tsx
// Lines 198-210: Loading integrations API but calling them workflows
useEffect(() => {
  const loadData = async () => {
    // Load integrations with filters
    const integrationsResult = await integrationsAPI.list({
      page: currentPage,
      limit: 50,
      search: searchQuery || undefined,
      category: category !== 'all' ? category : undefined,
    })

    // Load categories
    const categoriesData = await categoriesAPI.list()
    const categoriesArray = Array.isArray(categoriesData) 
      ? categoriesData 
      : (categoriesData as any).categories || []

    setIntegrations(integrationsResult.integrations)
    setTotalCount(integrationsResult.pagination?.total || integrationsResult.integrations.length)
  }
}, [searchQuery, category, currentPage])

// Lines 230-256: Converting Integration to Workflow with hardcoded data
const convertToWorkflow = (integration: Integration): Workflow => {
  return {
    id: integration.id,
    name: integration.name,
    description: integration.description || integration.tagline,
    icon: CreditCard, // HARDCODED - should be integration.icon_url
    category: integration.category as CategoryType || 'productivity',
    type: 'integration', // HARDCODED - should be based on integration.type
    pricing: {
      model: integration.pricing_model === 'paid' ? 'subscription' : 
             integration.pricing_model === 'usage_based' ? 'usage' : 'one-time',
      price: integration.base_price || 0,
      executions: 'unlimited' // HARDCODED - should be from integration
    },
    rating: integration.rating || 0,
    totalRatings: 0, // NOT AVAILABLE in API
    activeUsers: parseInt(integration.installs?.replace(/[^0-9]/g, '') || '0'),
    developer: {
      name: '@developer', // HARDCODED - not from API
      verified: true // HARDCODED
    },
    features: [], // ALWAYS EMPTY - not from integration object
    setupTime: '5 min', // HARDCODED
    trending: integration.trending,
    popular: false // HARDCODED
  }
}
```

**Problems:**
1. **API Mismatch**: Loading `integrationsAPI` but displaying as `Workflow` type
2. **Hardcoded Icon**: All workflows show credit card icon (line 236)
3. **Hardcoded Type**: All workflows marked as 'integration' (line 238)
4. **Missing Data**: Features always empty (line 251)
5. **Dummy Developer**: Developer name is '@developer' for all (line 248)
6. **Fixed Execution**: All show 'unlimited' even for usage-based pricing (line 242)

**Impact:**
- Users see incorrect information
- Can't compare pricing models effectively
- Marketplace looks unfinished
- Actual integration data is hidden

---

## Issue 3: Hardcoded Workflow Configuration

### File: `apps/web/src/routes/marketplace.$workflowId.tsx`

**Current Implementation (BROKEN):**
```tsx
// Lines 761-855: SmartConfigurationStep assumes Notion databases
function SmartConfigurationStep({
  workflowId,
  onConfigured,
  isInstalling,
  installError
}: {
  workflowId: string
  onConfigured: () => void
  isInstalling: boolean
  installError: string | null
}) {
  // ... state for databases, selectedDatabase, etc.

  const loadConfiguration = async () => {
    try {
      setIsLoadingConfig(true)
      setConfigError(null)

      const response = await fetch(
        `${import.meta.env.VITE_API_URL}/installations/configure`,
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          credentials: 'include',
          body: JSON.stringify({
            integrationId: workflowId
          })
        }
      )

      if (!response.ok) {
        const error = await response.json()
        throw new Error(error.error || 'Failed to load configuration')
      }

      const data = await response.json()

      // Lines 806-817: ASSUMES Notion structure
      if (data.recommendation === 'create_database') {
        // Error message assumes Notion
        setConfigError('No Notion databases found. Please create a database in Notion first.')
        return
      }

      if (data.recommendation === 'select_database') {
        // Assumes data.databases is Notion structure
        setDatabases(data.databases)
      } else if (data.autoSelected) {
        // Assumes data.database and data.configuration are Notion
        setSelectedDatabase(data.database.id)
        setConfiguration(data.configuration)
        setAnalysis(data.analysis)
      }
    } catch (err: any) {
      console.error('Failed to load configuration:', err)
      setConfigError(err.message || 'Failed to load Notion databases')
    } finally {
      setIsLoadingConfig(false)
    }
  }

  // ... more Notion-specific logic ...

  return (
    <div className="space-y-6">
      <div>
        {/* LINE 890: HARDCODED WORKFLOW NAME */}
        <h3 className="font-inter-tight text-lg font-semibold text-neutral-100 mb-4">
          Configure Gmail to Notion Sync
        </h3>

        {databases.length > 0 && !selectedDatabase ? (
          <div className="space-y-3">
            {/* LINE 896: HARDCODED DESCRIPTION */}
            <p className="text-neutral-400 text-sm mb-4">
              Select which Notion database to sync emails to:
            </p>
            {databases.map((db) => (
              // ... Notion database UI
            ))}
          </div>
        ) : analysis ? (
          <div className="space-y-4">
            {/* LINES 934-970: Notion-specific analysis display */}
          </div>
        ) : null}
      </div>

      {/* ... rest of configuration UI ... */}
    </div>
  )
}
```

**Problems:**
1. **Hardcoded Title** (line 890): "Configure Gmail to Notion Sync" for ALL workflows
2. **Hardcoded Description** (line 896): "Select which Notion database..."
3. **Assumes Notion** (lines 806-855): All logic assumes Notion database structure
4. **No Workflow Context**: Doesn't check what type of workflow is being configured
5. **Error Messages Assume Notion** (line 807): "No Notion databases found"
6. **Brittle API Assumptions**: Expects specific structure from `/installations/configure`

**What Happens:**
- User installs "Stripe to Slack" workflow
- Sees "Configure Gmail to Notion Sync"
- Tries to select Notion database (doesn't exist)
- Configuration fails with Notion-specific error
- Installation never completes

---

## Issue 4: Enable Modal - Fake OAuth Integration

### File: `apps/web/src/routes/marketplace.$workflowId.tsx`

**Current Implementation (BROKEN):**
```tsx
// Lines 555-754: EnableWorkflowModal component
function EnableWorkflowModal({
  workflow,
  onClose
}: {
  workflow: WorkflowDetail
  onClose: () => void
}) {
  const [connectedIntegrations, setConnectedIntegrations] = useState<string[]>([])

  // LINE 568-570: FAKE CONNECTION LOGIC
  const handleConnectIntegration = (name: string) => {
    // Just adds to local state, doesn't actually authenticate!
    setConnectedIntegrations([...connectedIntegrations, name])
  }

  // LINE 572-574: Checks LOCAL state, not real OAuth
  const allIntegrationsConnected = workflow.requirements.integrations.every(
    integration => connectedIntegrations.includes(integration.name)
  )

  const handleEnableWorkflow = async () => {
    try {
      setIsInstalling(true)
      setInstallError(null)

      // LINE 582: Calls API without verifying OAuth is real
      await installationsAPI.install(workflow.id, {
        // Configuration can be added here if needed
      })

      setStep('done')
    } catch (err: any) {
      console.error('Failed to install workflow:', err)
      setInstallError(err.message || 'Failed to enable workflow')
    } finally {
      setIsInstalling(false)
    }
  }

  // ... JSX for modal steps ...

  // LINES 664-694: Show connection status but don't verify
  {step === 'connect' && (
    <div className="space-y-6">
      <div>
        <h3 className="font-inter-tight text-lg font-semibold text-neutral-100 mb-4">
          Connect Integrations
        </h3>
        <div className="space-y-3">
          {workflow.requirements.integrations.map((integration, idx) => {
            // LINE 665: Check local state, NOT real OAuth
            const isConnected = connectedIntegrations.includes(integration.name)
            return (
              <div key={idx} className="border border-neutral-700 p-4">
                <div className="flex items-center justify-between">
                  <div className="flex items-center gap-3">
                    <Shield size={20} className="text-neutral-400" />
                    <div>
                      <div className="font-medium text-neutral-100">{integration.name}</div>
                      <div className="text-xs text-neutral-500">
                        {integration.scopes.join(', ')}
                      </div>
                    </div>
                  </div>
                  {isConnected ? (
                    <div className="flex items-center gap-2 text-green-400">
                      <Check size={16} />
                      Connected
                    </div>
                  ) : (
                    <button
                      onClick={() => handleConnectIntegration(integration.name)}
                      className="px-4 py-2 bg-neutral-800 border border-neutral-400 text-neutral-100 hover:bg-neutral-700 transition-colors duration-300"
                    >
                      Connect
                    </button>
                  )}
                </div>
              </div>
            )
          })}
        </div>
      </div>
    </div>
  )}
}
```

**Problems:**
1. **No Real OAuth** (line 569): `handleConnectIntegration` just adds to local state
2. **No API Call**: Doesn't call OAuth endpoint
3. **Fake Status Check** (line 665): Checks local state, not actual OAuth connections
4. **Installation Proceeds Without OAuth** (line 582): Calls install API without verifying
5. **No Scope Verification**: Doesn't check if connected OAuth has right permissions

**How It Breaks:**
1. User clicks "Connect Gmail"
2. Modal just adds "Gmail" to local state
3. No OAuth flow happens
4. User clicks "Enable Workflow"
5. Modal shows "Connected" but no OAuth token exists
6. Installation API call fails when trying to use Gmail (no token)

**What It Should Do:**
```tsx
// Should check real OAuth like settings.oauth.tsx does:
const isConnected = connections.some((c) => c.provider === providerId)

// Should handle missing OAuth:
if (!isConnected) {
  // Trigger OAuth flow via oauthAPI.authorize(provider)
}

// Should verify scopes:
const hasRequiredScopes = connection.scopes.includes(...workflow.requires)
```

---

## Issue 5: Developer Signup Missing Stripe Onboarding

### File: `apps/web/src/routes/developer.signup.tsx`

**Current Implementation (INCOMPLETE):**
```tsx
// LINE 54: Wrong redirect after signup
async function handleSubmit(e: React.FormEvent) {
  e.preventDefault()
  
  if (!formData.acceptTerms) {
    setError('Please accept the Developer Terms of Service')
    return
  }

  try {
    setIsLoading(true)
    setError(null)

    // Call the API to create developer profile
    const response = await fetch('https://marketplace-api.half-dozen.workers.dev/developers', {
      method: 'POST',
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        companyName: formData.companyName || undefined,
        bio: formData.bio || undefined,
      }),
    })

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({ 
        error: 'Failed to create developer profile' 
      }))
      throw new Error(errorData.error || 'Failed to create developer profile')
    }

    // LINE 54: WRONG! Should check Stripe status
    navigate({ to: '/developer/dashboard' })
  } catch (err: any) {
    console.error('Failed to create developer profile:', err)
    setError(err.message || 'Failed to create developer profile')
  } finally {
    setIsLoading(false)
  }
}
```

**Problem:**
- Routes directly to `/developer/dashboard` after signup
- Skips `developer.onboard.stripe.tsx` (which exists!)
- No Stripe account → developer can't receive payments
- Missing onboarding steps

**Expected Flow:**
1. `/auth/signup` → Create account
2. `/developer/signup` → Create developer profile
3. `/developer/onboard.stripe` → Connect Stripe (MISSING)
4. `/developer/builder` → Build workflow (optional)
5. `/developer/dashboard` → See stats

**What Code Should Do:**
```tsx
// After successful signup:
navigate({ to: '/developer/onboard.stripe' })

// Or check if Stripe already connected:
const developerProfile = await developerAPI.getProfile()
if (developerProfile.stripeConnected) {
  navigate({ to: '/developer/dashboard' })
} else {
  navigate({ to: '/developer/onboard.stripe' })
}
```

---

## Issue 6: OAuth Settings - Hardcoded Suggestion

### File: `apps/web/src/routes/settings.oauth.tsx`

**Current Implementation (STATIC):**
```tsx
// LINE 73-74: Only checks Gmail AND Notion
const hasGmailAndNotion = isConnected('gmail') && isConnected('notion')

// LINE 134-164: Only suggestion when both connected
{hasGmailAndNotion && !loading && (
  <section className="px-6 pb-6">
    <div className="max-w-7xl mx-auto w-full">
      <motion.div
        className="border border-green-400 bg-green-900/20 p-8"
      >
        <div className="flex items-start gap-4">
          <Sparkles className="w-6 h-6 text-green-400 flex-shrink-0 mt-0.5" />
          <div className="flex-1">
            <h3 className="font-inter-tight text-xl font-semibold text-green-100 mb-2">
              Ready to Create Your First Workflow!
            </h3>
            <p className="text-green-200 mb-6">
              You've connected both Gmail and Notion. Enable the 
              <strong>Gmail to Notion Email Sync</strong> workflow to automatically 
              save your emails to Notion.
            </p>
            <button
              onClick={() => navigate({ 
                // LINE 153: HARDCODED workflow ID
                to: '/marketplace/int_gmail_notion_sync' 
              })}
              className="px-6 py-3 bg-green-400 hover:bg-green-300 text-neutral-900 font-inter-tight font-semibold transition-colors duration-300 flex items-center gap-2"
            >
              Set Up Workflow
              <ArrowRight className="w-4 h-4" />
            </button>
          </div>
        </div>
      </motion.div>
    </div>
  </section>
)}
```

**Problems:**
1. **Hardcoded Check** (line 74): Only checks Gmail + Notion
2. **Hardcoded Suggestion** (line 150): Only suggests "Gmail to Notion Email Sync"
3. **Hardcoded ID** (line 153): Workflow ID is hardcoded string
4. **No Other Combinations**: Gmail + Slack? No suggestion
5. **Brittle Routing**: Relies on workflow having specific ID format

**What Should Happen:**
```tsx
// Should query for workflows that match connected integrations:
async function loadSuggestions() {
  const connectedProviders = connections.map(c => c.provider)
  
  // Find workflows that use these providers
  const suggestions = await integrationsAPI.list({
    providers: connectedProviders,
    limit: 3
  })
  
  // Show relevant suggestions
  return suggestions.integrations.map(w => ({
    id: w.id,
    name: w.name,
    description: w.description,
    providers: w.required_oauth_providers
  }))
}
```

---

## Issue 7: Marketplace Pricing Display

### File: `apps/web/src/routes/marketplace.tsx`

**Current Implementation (INCOMPLETE):**
```tsx
// LINE 242: All workflows get 'unlimited' executions
const convertToWorkflow = (integration: Integration): Workflow => {
  return {
    // ...
    pricing: {
      model: integration.pricing_model === 'paid' ? 'subscription' : 
             integration.pricing_model === 'usage_based' ? 'usage' : 'one-time',
      price: integration.base_price || 0,
      executions: 'unlimited' // HARDCODED!
    },
    // ...
  }
}

// LINES 508-514: Display doesn't show execution limits
<div className="text-lg font-medium text-neutral-100">
  {workflow.pricing.price === 0 ? 'Free' : `$${workflow.pricing.price}`}
</div>
{workflow.pricing.price > 0 && (
  <div className="text-xs text-neutral-500">/month</div>
)}
```

**Problems:**
1. **Hardcoded Unlimited** (line 242): All workflows show unlimited
2. **Ignores pricing_model** (line 240-241): Converts model but doesn't use execution limits
3. **No Execution Display** (lines 508-514): Doesn't show monthly execution limits
4. **Misleading Pricing**: Usage-based pricing looks like fixed subscription

**Real Data Available:**
```tsx
// API integration objects should have:
integration.pricing_model: 'subscription' | 'usage_based' | 'one-time'
integration.base_price: number
integration.monthly_execution_limit?: number // For usage-based
integration.execution_cost?: number // Per execution
```

**Should Display:**
```tsx
// For subscription pricing:
$25/month - Unlimited executions

// For usage-based pricing:
$0.01 per execution - $500/month maximum

// For one-time:
$99 one-time fee
```

---

## Issue 8: Welcome Modal Navigation

### File: `apps/web/src/routes/workflows.tsx`

**Current Implementation (REDUNDANT):**
```tsx
// LINES 115-191: Welcome modal steps

{/* Step 1 */}
<div className="flex items-start gap-4">
  <div>...</div>
  <div>
    <h4 className="font-inter-tight font-semibold text-neutral-100">
      Browse Workflows
    </h4>
    <p className="text-neutral-400 text-sm mb-3">
      Explore our marketplace...
    </p>
    <Link
      to="/integrations"  {/* LINE 136: POINTS TO BROKEN PAGE */}
      onClick={closeWelcome}
      className="..."
    >
      Browse Marketplace →
    </Link>
  </div>
</div>

{/* Step 2 - This one is correct */}
<div className="flex items-start gap-4">
  <div>...</div>
  <div>
    <h4 className="font-inter-tight font-semibold text-neutral-100">
      Connect Your Accounts
    </h4>
    <p className="text-neutral-400 text-sm mb-3">
      Link your Gmail, Notion...
    </p>
    <Link
      to="/settings/oauth"  {/* CORRECT */}
      onClick={closeWelcome}
      className="..."
    >
      Connect Accounts →
    </Link>
  </div>
</div>

{/* Step 3 */}
<div className="flex items-start gap-4">
  <div>...</div>
  <div>
    <h4 className="font-inter-tight font-semibold text-neutral-100">
      Install Your First Workflow
    </h4>
    <p className="text-neutral-400 text-sm mb-3">
      Choose a workflow...
    </p>
    <Link
      to="/integrations"  {/* LINE 186: ALSO POINTS TO BROKEN PAGE */}
      onClick={closeWelcome}
      className="..."
    >
      Get Started →
    </Link>
  </div>
</div>
```

**Problems:**
1. **Step 1** (line 136): "Browse Workflows" → `/integrations` → redirects to `/marketplace`
2. **Step 3** (line 186): "Install Workflow" → `/integrations` → redirects to `/marketplace`
3. **Redundant**: Both point to same page, no progression
4. **Confusing**: User clicks button twice and ends up on same page

**What User Experiences:**
1. New user sees welcome modal
2. Clicks "Browse Workflows" (Step 1)
3. Page redirects silently to `/marketplace`
4. User learns about workflows
5. Clicks "Install Your First Workflow" (Step 3)
6. Page redirects to `/marketplace` AGAIN (confusing - thought we already went there)

**Better Approach:**
```tsx
// Step 1: Browse (go to marketplace)
<Link to="/marketplace" onClick={closeWelcome}>
  Browse Marketplace →
</Link>

// Step 2: Connect (correct)
<Link to="/settings/oauth" onClick={closeWelcome}>
  Connect Accounts →
</Link>

// Step 3: Install (marketplace again is fine, but maybe show specific category)
<Link to="/marketplace?category=popular" onClick={closeWelcome}>
  Install Your First →
</Link>
```

---

## Issue 9: Developer Dashboard Mock Data

### File: `apps/web/src/routes/developer.dashboard.tsx`

**Current Implementation (NOT FROM API):**
```tsx
// LINE 127-131: Hardcoded earnings breakdown
const earningsBreakdown: EarningsBreakdown[] = [
  { workflow: 'Daily AI Newsletter', amount: 287.00, percentage: 70 },
  { workflow: 'AI Research Assistant', amount: 41.12, percentage: 10 },
  { workflow: 'Referral Commissions', amount: 89.27, percentage: 22 }
]

// LINE 133-139: Hardcoded leaderboard
const leaderboard = [
  { rank: 1, name: '@automation_guru', earnings: 12847, badge: '🥇' },
  { rank: 2, name: '@ai_workflows', earnings: 8234, badge: '🥈' },
  { rank: 3, name: '@you', earnings: 2847, badge: '🥉', isYou: true },
  { rank: 4, name: '@workflow_wizard', earnings: 2156, badge: '' },
  { rank: 5, name: '@dev_master', earnings: 1843, badge: '' }
]
```

**Problems:**
1. **Not from API**: Data is hardcoded, same for every developer
2. **Mock Data in Production**: Every developer sees "Daily AI Newsletter" earning $287
3. **Fake Leaderboard**: Shows hardcoded users, not real top developers
4. **No Real Analytics**: Earnings breakdown not from actual revenue data

**What Shows:**
- Developer A sees: "Daily AI Newsletter $287"
- Developer B sees: "Daily AI Newsletter $287"
- Both see same fake leaderboard with same fake top developers

**Should Load From API:**
```tsx
// Load real earnings data
const analyticsData = await developerAPI.getAnalytics({
  period: 'month',
  metrics: ['revenue', 'executions', 'users']
})

// Load real leaderboard
const leaderboardData = await developerAPI.getLeaderboard({
  limit: 5,
  period: 'month'
})

// Display actual data
const earningsBreakdown = analyticsData.workflows.map(w => ({
  workflow: w.name,
  amount: w.revenue / 100, // Convert from cents
  percentage: (w.revenue / analyticsData.totalRevenue) * 100
}))
```

