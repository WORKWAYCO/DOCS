# Pricing Update Impact Analysis: /workflows Routes

> **Ultrathink Analysis**: Comprehensive review of how the new two-sided marketplace pricing model affects workflow metadata, types, and display throughout the application.

**Date**: 2025-11-12
**Scope**: Major pricing model update affecting workflows, integrations, and marketplace pages
**Status**: 🔴 **CRITICAL** - Breaking changes to core data structures

---

## Executive Summary

The new pricing model introduces significant changes that affect:
- ✅ **Pricing page**: Updated successfully
- 🔴 **Workflow detail pages**: Requires updates
- 🔴 **TypeScript interfaces**: Missing fields
- 🔴 **API client**: Incorrect mapping
- 🔴 **Database schema**: Needs migration applied

**Impact Level**: **HIGH** - Affects multiple layers of the application stack

---

## Table of Contents

1. [New Pricing Model Overview](#new-pricing-model-overview)
2. [Current vs New Structure Comparison](#current-vs-new-structure-comparison)
3. [Affected Files and Interfaces](#affected-files-and-interfaces)
4. [Breaking Changes](#breaking-changes)
5. [Required Updates](#required-updates)
6. [Migration Strategy](#migration-strategy)
7. [Testing Checklist](#testing-checklist)

---

## 1. New Pricing Model Overview

### Business Logic Changes

**Old Model (Platform Subscription)**:
```
User → Pays monthly platform fee → Gets X executions
```

**New Model (Marketplace + Usage)**:
```
User → Pays developer upfront (developer keeps 100%)
     → Gets 20 FREE trial executions
     → Pays platform 5¢ or 25¢ per execution after trial
```

### Key Pricing Components

| Component | Description | Owner | Storage |
|-----------|-------------|-------|---------|
| **Upfront Fee** | Developer-set price (one-time or subscription) | Developer | `base_price` (cents) |
| **Trial Executions** | Fixed 20 free runs per workflow | Platform | `trial_executions_remaining` |
| **Complexity Tier** | Light (5¢) or Heavy (25¢) per execution | Developer | `complexity_tier` |
| **Usage Charges** | Post-trial execution pricing | Platform | `workflow_usage_charges` |

---

## 2. Current vs New Structure Comparison

### 2.1 Database Schema

#### BEFORE (Current)
```sql
marketplace_integrations:
  - pricing_model TEXT  -- 'free', 'freemium', 'paid', 'usage_based'
  - base_price INTEGER  -- In cents
  - pricing_details TEXT -- JSON (loosely defined)
```

#### AFTER (Migration 0009)
```sql
marketplace_integrations:
  - pricing_model TEXT
  - base_price INTEGER  -- Developer's upfront fee
  - complexity_tier TEXT -- 'light' or 'heavy' (NEW)
  - pricing_details TEXT -- Structured: { upfront_price, trial_executions, usage_price_per_run }

user_installations:
  - trial_executions_remaining INTEGER DEFAULT 20 (NEW)
  - trial_started_at INTEGER (NEW)
  - trial_expired INTEGER DEFAULT 0 (NEW)
  - total_paid_executions INTEGER DEFAULT 0 (NEW)
  - last_billed_at INTEGER (NEW)

workflow_usage_charges: (NEW TABLE)
  - Tracks post-trial billing
```

### 2.2 TypeScript Interfaces

#### BEFORE (api-client.ts:77-90)
```typescript
export interface Integration {
  id: string;
  name: string;
  pricing_model: 'free' | 'freemium' | 'paid' | 'usage_based';
  base_price?: number;
  // Missing: complexity_tier
  // Missing: trial info
  // Missing: usage pricing
}
```

#### AFTER (Required)
```typescript
export interface Integration {
  id: string;
  name: string;
  pricing_model: 'free' | 'freemium' | 'paid' | 'usage_based';
  base_price?: number; // Cents - developer's upfront fee
  complexity_tier?: 'light' | 'heavy'; // NEW
  pricing_details?: PricingDetails; // NEW - structured
}

interface PricingDetails {
  upfront_price: number; // Cents
  trial_executions: number; // Always 20
  usage_price_per_run: number; // 5 or 25 cents
  pricing_model_type?: 'one_time' | 'subscription';
}
```

### 2.3 Workflow Detail Page (workflow.$workflowId.tsx)

#### BEFORE (Lines 37-45)
```typescript
pricing: {
  model: 'subscription' | 'usage' | 'one-time'
  price: number
  executions: number | 'unlimited'
  trial?: {
    days: number      // Variable days
    executions: number // Variable executions
  }
}
```

#### AFTER (Required)
```typescript
pricing: {
  model: 'free' | 'one_time' | 'subscription'
  upfrontPrice: number // Developer's price in cents
  complexityTier: 'light' | 'heavy' // NEW
  usagePricePerRun: number // 5 or 25 cents
  trial: {
    executions: 20 // Fixed
  }
  estimatedMonthlyCost?: {
    upfront: number
    usage: number // Based on typical usage
    total: number
  }
}
```

---

## 3. Affected Files and Interfaces

### 3.1 Frontend Files

| File | Lines | Impact | Priority |
|------|-------|--------|----------|
| `apps/web/src/lib/api-client.ts` | 77-90 | Add `complexity_tier`, `pricing_details` | 🔴 HIGH |
| `apps/web/src/routes/workflow.$workflowId.tsx` | 29-70, 172-209, 472-505 | Update interface, mapping, display | 🔴 HIGH |
| `apps/web/src/routes/workflows.tsx` | N/A | No direct impact (lists only) | 🟢 LOW |
| `apps/web/src/routes/marketplace.tsx` | TBD | Needs pricing display update | 🟡 MEDIUM |
| `packages/cli/src/commands/workflow/publish.ts` | TBD | Add complexity tier prompt | 🔴 HIGH |

### 3.2 Backend Files

| File | Lines | Impact | Priority |
|------|-------|--------|----------|
| `apps/api/migrations/0009_usage_based_billing.sql` | ALL | Apply migration | 🔴 HIGH |
| `apps/api/src/lib/stripe.ts` | 156-295 | Already updated ✅ | 🟢 DONE |
| `apps/api/src/routes/integrations.ts` | TBD | Return `complexity_tier` in API | 🔴 HIGH |
| `apps/api/src/sdk/workflow-sdk.ts` | 103-119 | Update `WorkflowPricing` interface | 🟡 MEDIUM |

### 3.3 Documentation Files

| File | Status | Priority |
|------|--------|----------|
| `docs/DEVELOPER_PRICING_GUIDE.md` | ✅ Created | 🟢 DONE |
| `docs/PRICING_UPDATE_IMPACT_ANALYSIS.md` | ✅ This file | 🟢 DONE |
| `.claude/skills/workway-pricing-config/SKILL.md` | ✅ Created | 🟢 DONE |
| `.claude/skills/workway-usage-billing/SKILL.md` | ✅ Created | 🟢 DONE |

---

## 4. Breaking Changes

### 4.1 Data Structure Changes

**BREAKING**: Trial structure changed
```typescript
// ❌ OLD - Variable days and executions
trial?: { days: number; executions: number }

// ✅ NEW - Fixed 20 executions, no day limit
trial: { executions: 20 }
```

**BREAKING**: Price semantics changed
```typescript
// ❌ OLD - Platform fee (user pays platform)
price: 29 // $29/month platform subscription

// ✅ NEW - Developer fee (user pays developer, keeps 100%)
upfrontPrice: 4900 // $49 to developer
usagePricePerRun: 5 // 5¢ to platform per execution
```

**BREAKING**: New required field
```typescript
// ✅ NEW - Must classify all workflows
complexityTier: 'light' | 'heavy'
```

### 4.2 API Response Changes

**Endpoint**: `GET /integrations/:id`

**OLD Response**:
```json
{
  "integration": {
    "id": "int_123",
    "name": "Gmail to Notion",
    "pricing_model": "paid",
    "base_price": 4900
  }
}
```

**NEW Response** (Required):
```json
{
  "integration": {
    "id": "int_123",
    "name": "Gmail to Notion",
    "pricing_model": "paid",
    "base_price": 4900,
    "complexity_tier": "light",
    "pricing_details": {
      "upfront_price": 4900,
      "trial_executions": 20,
      "usage_price_per_run": 5,
      "pricing_model_type": "subscription"
    }
  }
}
```

### 4.3 Display Logic Changes

**OLD Pricing Display** (workflow.$workflowId.tsx:472-505):
```typescript
<div className="text-center mb-6">
  <div className="text-4xl">${workflow.pricing.price}</div>
  <div>per month</div>
  <div>{workflow.pricing.executions} executions/mo</div>
</div>

{workflow.pricing.trial && (
  <div className="border p-3 mb-4">
    {workflow.pricing.trial.days}-day free trial
    {workflow.pricing.trial.executions} executions included
  </div>
)}
```

**NEW Pricing Display** (Required):
```typescript
<div className="space-y-4">
  {/* Upfront Price */}
  <div className="text-center">
    <div className="text-4xl">${(workflow.pricing.upfrontPrice / 100).toFixed(2)}</div>
    <div>{workflow.pricing.model === 'subscription' ? 'per month' : 'one-time'}</div>
  </div>

  {/* Trial Badge */}
  <div className="border border-green-400 bg-green-900/20 p-3 text-center">
    <div className="text-green-400 font-medium">
      20 Free Trial Executions
    </div>
    <div className="text-xs text-green-300">
      Try before you're billed
    </div>
  </div>

  {/* Usage Pricing */}
  <div className="border border-neutral-700 p-4">
    <div className="text-sm text-neutral-400 mb-2">After trial:</div>
    <div className="flex items-baseline gap-2">
      <span className="text-2xl font-semibold text-neutral-100">
        {workflow.pricing.usagePricePerRun}¢
      </span>
      <span className="text-neutral-400">per execution</span>
    </div>
    <div className="text-xs text-neutral-500 mt-1">
      {workflow.pricing.complexityTier === 'heavy' ? 'Heavy' : 'Light'} workflow
    </div>
  </div>

  {/* Estimated Monthly Cost */}
  {workflow.pricing.estimatedMonthlyCost && (
    <div className="border-t border-neutral-700 pt-3 text-sm">
      <div className="text-neutral-400 mb-2">Estimated monthly cost:</div>
      <div className="space-y-1">
        <div className="flex justify-between">
          <span className="text-neutral-500">Upfront:</span>
          <span className="text-neutral-300">${workflow.pricing.estimatedMonthlyCost.upfront.toFixed(2)}</span>
        </div>
        <div className="flex justify-between">
          <span className="text-neutral-500">Usage (typical):</span>
          <span className="text-neutral-300">${workflow.pricing.estimatedMonthlyCost.usage.toFixed(2)}</span>
        </div>
        <div className="flex justify-between font-medium border-t border-neutral-700 pt-1 mt-1">
          <span className="text-neutral-300">Total:</span>
          <span className="text-neutral-100">${workflow.pricing.estimatedMonthlyCost.total.toFixed(2)}</span>
        </div>
      </div>
    </div>
  )}
</div>
```

---

## 5. Required Updates

### 5.1 Immediate (Pre-Launch - Before Dec 5)

#### ✅ COMPLETED
- [x] Pricing page updated (`apps/web/src/routes/pricing.tsx`)
- [x] Database migration created (`apps/api/migrations/0009_usage_based_billing.sql`)
- [x] Stripe utilities updated (`apps/api/src/lib/stripe.ts`)
- [x] Developer guide created (`docs/DEVELOPER_PRICING_GUIDE.md`)
- [x] Skills created (pricing-config, usage-billing)

#### 🔴 PENDING (High Priority)
- [ ] **Apply database migration** to D1 database
- [ ] **Update Integration interface** in `api-client.ts` (add `complexity_tier`, `pricing_details`)
- [ ] **Update WorkflowDetail interface** in `workflow.$workflowId.tsx` (new pricing structure)
- [ ] **Update API integration mapping** (lines 172-209) to use new fields
- [ ] **Update pricing display UI** (lines 472-505) to show complexity tier and usage pricing
- [ ] **Update integrations API route** to return `complexity_tier` and `pricing_details`
- [ ] **Update CLI `workway publish` command** to prompt for complexity tier selection

### 5.2 Follow-Up (Post-Migration)

#### 🟡 Medium Priority
- [ ] Update marketplace.tsx pricing cards
- [ ] Add usage dashboard for users
- [ ] Implement trial tracking UI
- [ ] Create admin tools for pricing management
- [ ] Update WorkflowPricing interface in SDK
- [ ] Update CLI with interactive pricing configuration

#### 🟢 Low Priority
- [ ] Add pricing calculator component
- [ ] Create pricing comparison tools
- [ ] SEO updates for new pricing model
- [ ] Analytics tracking for pricing events

---

## 6. CLI Integration

### 6.1 Current CLI Status

Developers submit workflows via **WORKWAY CLI** (`@workway/cli`), not a web form.

**CLI Command**: `workway workflow publish`

**Current Implementation**: ✅ Basic authentication and workflow submission
**Status**: 🟡 Needs pricing configuration prompts

### 6.2 Required CLI Updates

#### Update: `workway workflow publish` Command

**Location**: `packages/cli/src/commands/workflow/publish.ts` (to be created/updated)

**Add Interactive Prompts**:

```typescript
// packages/cli/src/commands/workflow/publish.ts

import inquirer from 'inquirer';

export async function publishWorkflow(options: PublishOptions) {
  // ... existing authentication logic

  // NEW: Pricing configuration prompts
  const pricingAnswers = await inquirer.prompt([
    {
      type: 'list',
      name: 'pricingModel',
      message: 'Select pricing model:',
      choices: [
        { name: 'Free (no upfront fee)', value: 'free' },
        { name: 'One-time payment', value: 'one_time' },
        { name: 'Monthly subscription', value: 'subscription' },
      ],
    },
    {
      type: 'number',
      name: 'upfrontPrice',
      message: 'Set upfront price (USD):',
      default: 0,
      when: (answers) => answers.pricingModel !== 'free',
      validate: (value) => {
        if (value < 0) return 'Price must be positive';
        if (value > 0 && value < 5) return 'Minimum price is $5';
        return true;
      },
    },
    {
      type: 'list',
      name: 'complexityTier',
      message: 'Classify workflow complexity:',
      choices: [
        {
          name: 'Light (5¢/run) - Simple integrations, 1-3 API calls, <5s execution',
          value: 'light',
        },
        {
          name: 'Heavy (25¢/run) - AI processing, 4+ API calls, complex logic',
          value: 'heavy',
        },
      ],
      default: 'light',
    },
    {
      type: 'confirm',
      name: 'confirmPricing',
      message: (answers) => {
        const upfront = answers.upfrontPrice || 0;
        const usage = answers.complexityTier === 'heavy' ? 25 : 5;
        const model = answers.pricingModel;

        let msg = `\nPricing Summary:\n`;
        msg += `  Upfront: $${upfront} ${model === 'subscription' ? '/month' : model === 'one_time' ? 'one-time' : ''}\n`;
        msg += `  Trial: 20 free executions\n`;
        msg += `  Usage: ${usage}¢ per execution after trial\n`;
        msg += `  You keep: 100% of upfront fees (minus ~3% Stripe fees)\n`;
        msg += `  Platform: Collects ${usage}¢ per execution for infrastructure\n\n`;
        msg += `Confirm this pricing configuration?`;

        return msg;
      },
      default: true,
    },
  ]);

  if (!pricingAnswers.confirmPricing) {
    console.log('❌ Publish cancelled');
    return;
  }

  // Convert to cents for API
  const basePrice = Math.round((pricingAnswers.upfrontPrice || 0) * 100);
  const usagePricePerRun = pricingAnswers.complexityTier === 'heavy' ? 25 : 5;

  const pricingDetails = {
    upfront_price: basePrice,
    trial_executions: 20,
    usage_price_per_run: usagePricePerRun,
    pricing_model_type: pricingAnswers.pricingModel,
  };

  // Submit to API
  await apiClient.publishWorkflow({
    ...workflowData,
    pricing_model: pricingAnswers.pricingModel,
    base_price: basePrice,
    complexity_tier: pricingAnswers.complexityTier,
    pricing_details: JSON.stringify(pricingDetails),
  });

  console.log('✅ Workflow published successfully!');
  console.log(`   Upfront: $${pricingAnswers.upfrontPrice} ${pricingAnswers.pricingModel === 'subscription' ? '/month' : pricingAnswers.pricingModel === 'one_time' ? 'one-time' : ''}`);
  console.log(`   Usage: ${usagePricePerRun}¢ per execution (${pricingAnswers.complexityTier})`);
  console.log(`   Marketplace: https://workway.com/marketplace/${workflowId}`);
}
```

### 6.3 CLI User Experience

**Example Session**:

```bash
$ workway workflow publish

✓ Authenticated as @developer_name
✓ Workflow validated successfully

? Select pricing model: › Monthly subscription
? Set upfront price (USD): › 49
? Classify workflow complexity: › Light (5¢/run) - Simple integrations

Pricing Summary:
  Upfront: $49 /month
  Trial: 20 free executions
  Usage: 5¢ per execution after trial
  You keep: 100% of upfront fees (minus ~3% Stripe fees)
  Platform: Collects 5¢ per execution for infrastructure

? Confirm this pricing configuration? › Yes

📤 Publishing workflow to marketplace...
✅ Workflow published successfully!
   Upfront: $49 /month
   Usage: 5¢ per execution (light)
   Marketplace: https://workway.com/marketplace/wf_abc123
```

### 6.4 CLI Help Documentation

**Add to CLI README**:

```markdown
### Pricing Configuration

When publishing a workflow, you'll configure pricing in three steps:

1. **Pricing Model**:
   - Free: No upfront fee
   - One-time: Single payment
   - Subscription: Monthly recurring

2. **Upfront Price** (if not free):
   - Set in USD
   - You keep 100% (minus ~3% Stripe fees)
   - Minimum $5 for paid workflows

3. **Complexity Tier**:
   - **Light (5¢/run)**: 1-3 API calls, simple logic, <5s execution
   - **Heavy (25¢/run)**: AI processing, 4+ API calls, complex workflows

#### Examples

**Simple Email Workflow**:
- Upfront: $29/month
- Complexity: Light (5¢/run)
- Target: 200 emails/mo = ~$10 usage

**AI Content Generator**:
- Upfront: $99/month
- Complexity: Heavy (25¢/run)
- Target: 100 generations/mo = ~$25 usage

See [Developer Pricing Guide](../../docs/DEVELOPER_PRICING_GUIDE.md) for strategies.
```

### 6.5 API Client Updates

**Location**: `packages/cli/src/lib/api-client.ts`

Add complexity_tier to workflow submission:

```typescript
export interface WorkflowSubmission {
  name: string;
  description: string;
  // ... existing fields

  // NEW FIELDS
  pricing_model: 'free' | 'one_time' | 'subscription';
  base_price: number; // cents
  complexity_tier: 'light' | 'heavy';
  pricing_details: string; // JSON
}

export async function publishWorkflow(data: WorkflowSubmission) {
  return fetch(`${API_BASE_URL}/workflows`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${getAuthToken()}`,
    },
    body: JSON.stringify(data),
  });
}
```

---

## 7. Migration Strategy

### Phase 1: Database & Backend (Day 1-2)

```bash
# 1. Apply migration to D1
cd apps/api
wrangler d1 execute workway --file=migrations/0009_usage_based_billing.sql

# 2. Verify migration
wrangler d1 execute workway --command="SELECT * FROM marketplace_integrations LIMIT 1"

# 3. Check new columns
wrangler d1 execute workway --command="PRAGMA table_info(marketplace_integrations)"
```

**Validation**:
- [ ] All existing integrations have `complexity_tier = 'light'`
- [ ] New tables created: `workflow_usage_charges`, `user_payment_methods`, `billing_notifications`
- [ ] Existing installations have `trial_executions_remaining = 20`

### Phase 2: Backend API Updates (Day 2-3)

**File**: `apps/api/src/routes/integrations.ts`

```typescript
// Update GET /integrations/:id response
app.get('/integrations/:id', async (c) => {
  const integration = await db.prepare(`
    SELECT
      id, name, tagline, description,
      pricing_model, base_price,
      complexity_tier, -- NEW
      pricing_details  -- NEW
    FROM marketplace_integrations
    WHERE id = ?
  `).bind(integrationId).first();

  // Parse pricing_details JSON
  const pricingDetails = integration.pricing_details
    ? JSON.parse(integration.pricing_details)
    : {
        upfront_price: integration.base_price || 0,
        trial_executions: 20,
        usage_price_per_run: integration.complexity_tier === 'heavy' ? 25 : 5
      };

  return c.json({
    integration: {
      ...integration,
      pricing_details: pricingDetails
    }
  });
});
```

**Testing**:
```bash
# Test integration endpoint
curl http://localhost:8787/integrations/int_123 | jq '.integration.complexity_tier'
# Expected: "light" or "heavy"

curl http://localhost:8787/integrations/int_123 | jq '.integration.pricing_details'
# Expected: { upfront_price: 4900, trial_executions: 20, usage_price_per_run: 5 }
```

### Phase 3: Frontend Type Updates (Day 3-4)

**File**: `apps/web/src/lib/api-client.ts`

```typescript
// Add new fields to Integration interface
export interface Integration {
  id: string;
  name: string;
  tagline: string;
  description: string;
  icon_url: string;
  category: string;
  rating: number;
  installs: string;
  trending: boolean;
  pricing_model: 'free' | 'freemium' | 'paid' | 'usage_based';
  base_price?: number;
  required_oauth_providers?: string[];

  // NEW FIELDS
  complexity_tier?: 'light' | 'heavy';
  pricing_details?: {
    upfront_price: number;
    trial_executions: number;
    usage_price_per_run: number;
    pricing_model_type?: 'one_time' | 'subscription';
  };
}
```

### Phase 4: Frontend UI Updates (Day 4-5)

**Files to Update**:
1. `apps/web/src/routes/workflow.$workflowId.tsx` - Detail page pricing section
2. `apps/web/src/routes/marketplace.tsx` - Workflow cards
3. Create `apps/web/src/components/PricingBreakdown.tsx` - Reusable pricing component

**Key Changes**:
- Show complexity tier badge (Light/Heavy)
- Display usage pricing (5¢ or 25¢ per execution)
- Show trial information (20 free executions)
- Calculate and display estimated monthly cost
- Add tooltip explaining pricing model

### Phase 5: Testing & Validation (Day 5-6)

**Test Cases**:
```typescript
// Test 1: Free workflow
{
  base_price: 0,
  complexity_tier: 'light',
  expected_display: "Free + 5¢ per execution after 20 free trials"
}

// Test 2: Paid light workflow
{
  base_price: 2900, // $29
  complexity_tier: 'light',
  expected_display: "$29/mo + 5¢ per execution after 20 free trials"
}

// Test 3: Paid heavy workflow
{
  base_price: 9900, // $99
  complexity_tier: 'heavy',
  expected_display: "$99/mo + 25¢ per execution after 20 free trials"
}

// Test 4: One-time payment
{
  base_price: 4900, // $49
  complexity_tier: 'light',
  pricing_model_type: 'one_time',
  expected_display: "$49 one-time + 5¢ per execution after 20 free trials"
}
```

**Manual Testing Checklist**:
- [ ] Pricing page displays correctly
- [ ] Workflow detail page shows all pricing components
- [ ] Marketplace cards show pricing accurately
- [ ] Trial badge displays "20 free executions"
- [ ] Complexity tier badge shows (Light/Heavy)
- [ ] Usage pricing displays (5¢ or 25¢)
- [ ] Estimated monthly cost calculates correctly
- [ ] Mobile responsive layout works
- [ ] Tooltips explain pricing model

---

## 7. Testing Checklist

### 7.1 Unit Tests

```typescript
// Test: Parse pricing details correctly
describe('Pricing Details Parsing', () => {
  it('should parse pricing_details JSON from API', () => {
    const integration = {
      base_price: 4900,
      complexity_tier: 'light',
      pricing_details: '{"upfront_price":4900,"trial_executions":20,"usage_price_per_run":5}'
    };

    const parsed = JSON.parse(integration.pricing_details);
    expect(parsed.upfront_price).toBe(4900);
    expect(parsed.trial_executions).toBe(20);
    expect(parsed.usage_price_per_run).toBe(5);
  });

  it('should calculate usage price from complexity tier', () => {
    expect(getUsagePrice('light')).toBe(5);
    expect(getUsagePrice('heavy')).toBe(25);
  });

  it('should format pricing for display', () => {
    const pricing = {
      upfrontPrice: 4900,
      complexityTier: 'light',
      usagePricePerRun: 5
    };

    expect(formatUpfrontPrice(pricing.upfrontPrice)).toBe('$49.00');
    expect(formatUsagePrice(pricing.usagePricePerRun)).toBe('5¢');
  });
});
```

### 7.2 Integration Tests

```typescript
// Test: Workflow detail page data flow
describe('Workflow Detail Page', () => {
  it('should fetch integration with pricing details', async () => {
    const { integration } = await integrationsAPI.get('int_123');

    expect(integration.complexity_tier).toBeDefined();
    expect(integration.pricing_details).toBeDefined();
    expect(integration.pricing_details.trial_executions).toBe(20);
  });

  it('should display pricing correctly', () => {
    render(<WorkflowDetailPage />);

    expect(screen.getByText('20 Free Trial Executions')).toBeInTheDocument();
    expect(screen.getByText(/per execution/i)).toBeInTheDocument();
  });
});
```

### 7.3 E2E Tests

```typescript
// Test: User pricing journey
describe('Pricing User Journey', () => {
  it('should show pricing info from browse to purchase', async () => {
    // 1. Browse marketplace
    await page.goto('/marketplace');
    const workflow = await page.locator('[data-workflow-id="int_123"]');
    expect(await workflow.getByText('5¢ per run')).toBeVisible();

    // 2. View detail page
    await workflow.click();
    expect(await page.getByText('20 Free Trial Executions')).toBeVisible();
    expect(await page.getByText('Light workflow')).toBeVisible();

    // 3. Enable workflow
    await page.getByText('Enable Workflow').click();

    // 4. Verify pricing in modal
    expect(await page.getByText('$49.00/month')).toBeVisible();
    expect(await page.getByText('5¢ per execution after trial')).toBeVisible();
  });
});
```

---

## 8. Rollback Plan

If issues arise after deployment:

### Option 1: Quick Rollback (Frontend Only)

```bash
# Revert frontend to previous pricing display
git revert <commit-hash>
git push origin main
```

**Use when**: Display issues only, no database corruption

### Option 2: Full Rollback (Database + Frontend)

```sql
-- Rollback migration 0009
ALTER TABLE marketplace_integrations DROP COLUMN complexity_tier;
ALTER TABLE user_installations DROP COLUMN trial_executions_remaining;
-- ... (full rollback script)
```

**Use when**: Data integrity issues or critical bugs

### Option 3: Forward Fix (Preferred)

Fix issues without rolling back:
1. Identify specific bug
2. Create hotfix branch
3. Test fix in staging
4. Deploy fix to production
5. Monitor metrics

---

## 9. Success Metrics

### Pre-Launch Validation

- [ ] All API endpoints return new pricing structure
- [ ] Frontend displays pricing correctly on all pages
- [ ] Trial tracking works (20 executions)
- [ ] Usage billing logic tested
- [ ] No TypeScript errors
- [ ] No console errors in browser
- [ ] Mobile layout works correctly

### Post-Launch Monitoring

**Week 1**:
- Monitor API error rates for /integrations/:id
- Track pricing display render errors
- Check database query performance
- Validate trial execution tracking

**Week 2-4**:
- User feedback on pricing clarity
- Conversion rates on workflow installs
- Trial to paid conversion rates
- Support tickets related to pricing

---

## 10. Known Issues & Limitations

### Current Limitations

1. **No Price History**: Can't track pricing changes over time
2. **No Grandfathering Logic**: Existing users will need manual migration
3. **No A/B Testing**: Can't test different pricing models simultaneously
4. **Hard-Coded Trial Count**: 20 executions is fixed, can't customize per workflow

### Future Enhancements

1. **Dynamic Trial Periods**: Allow developers to set trial execution count
2. **Tiered Usage Pricing**: Multiple pricing tiers beyond light/heavy
3. **Volume Discounts**: Bulk execution pricing for high-volume users
4. **Enterprise Pricing**: Custom pricing for large organizations
5. **Promotional Pricing**: Temporary discounts and sales

---

## 11. Resources

### Documentation
- [Developer Pricing Guide](./DEVELOPER_PRICING_GUIDE.md)
- [Database Migration 0009](../apps/api/migrations/0009_usage_based_billing.sql)
- [Pricing Page Update](../apps/web/src/routes/pricing.tsx)

### Skills
- [Pricing Configuration Skill](../.claude/skills/workway-pricing-config/SKILL.md)
- [Usage Billing Skill](../.claude/skills/workway-usage-billing/SKILL.md)

### API Documentation
- Stripe Integration: `apps/api/src/lib/stripe.ts`
- Workflow SDK: `apps/api/src/sdk/workflow-sdk.ts`
- API Client: `apps/web/src/lib/api-client.ts`

---

## Appendix A: Quick Reference

### Pricing Field Mapping

| UI Display | Database Field | TypeScript Property | Unit |
|------------|----------------|---------------------|------|
| "$49/mo" | `base_price` | `upfrontPrice` | Cents |
| "5¢ per run" | calculated from `complexity_tier` | `usagePricePerRun` | Cents |
| "Light workflow" | `complexity_tier` | `complexityTier` | String |
| "20 free trials" | `trial_executions_remaining` | `trial.executions` | Count |

### Color Coding

- 🔴 **High Priority**: Blocking launch, must fix immediately
- 🟡 **Medium Priority**: Important but not blocking
- 🟢 **Low Priority**: Nice to have, can defer
- ✅ **Completed**: Already done

---

## Change Log

| Date | Author | Changes |
|------|--------|---------|
| 2025-11-12 | Claude | Initial analysis created |
| TBD | | Database migration applied |
| TBD | | Frontend updates completed |
| TBD | | Testing completed |
| TBD | | Production deployment |

---

**Next Actions**:
1. Review this analysis with team
2. Prioritize required updates
3. Apply database migration
4. Update TypeScript interfaces
5. Update UI components
6. Test thoroughly
7. Deploy to production

**Questions?** Contact: developers@workway.com
