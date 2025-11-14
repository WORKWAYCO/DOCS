# Developer Submission Process

This document outlines how developers submit, test, and deploy integrations and workflows to the WORKWAY marketplace.

---

## Two Types of Developers

### TYPE 1: Integration Developers
**Who:** Technical developers building service connectors (Gmail, Slack, Stripe, etc.)
**What:** Create low-level integrations that provide actions/triggers
**Approval:** Required (security review by WORKWAY team)
**Revenue:** Salary or contractor payment (not marketplace revenue)

### TYPE 2: Workflow Developers
**Who:** Product-focused developers building automation workflows
**What:** Compose existing integrations into sellable products
**Approval:** Automated (if using approved integrations)
**Revenue:** 70% of subscription revenue

---

## Integration Developer Flow

### Overview

```
┌─────────────────────────────────────────────────────────────┐
│ INTEGRATION DEVELOPER JOURNEY                               │
└─────────────────────────────────────────────────────────────┘

1. Apply for Integration Developer Access
   └─> Background check, NDA, technical assessment

2. Get Development Environment
   └─> Access to dev API, test database, OAuth sandbox

3. Build Integration Locally
   └─> Use Integration SDK, write actions, test locally

4. Submit for Review (Pull Request)
   └─> Code review, security audit, OAuth verification

5. Approved & Merged
   └─> Integration added to registry, deployed to production

6. Published to Marketplace
   └─> Available for workflow developers to use
```

---

## Step-by-Step: Integration Developer

### Phase 1: Application & Onboarding

#### 1. Apply for Access

**Requirements:**
- Portfolio of technical work
- Understanding of OAuth 2.0
- Agreement to WORKWAY integration guidelines

**Application Form:**
```
Name: _______________________
Email: ______________________
GitHub: _____________________
LinkedIn: ___________________
Integration Proposal: ________
Experience with OAuth: _______
Code samples: _______________
```

**Review Time:** 3-5 business days

#### 2. Onboarding (If Approved)

```bash
# Receive email with:
- Developer portal credentials
- GitHub repo access (workway/integrations)
- Development API key
- OAuth sandbox access
- Slack workspace invite
```

**Sign Documents:**
- Non-Disclosure Agreement (NDA)
- Integration Developer Agreement
- Security & Privacy Policy

---

### Phase 2: Local Development

#### 1. Clone Repository

```bash
# Clone main WORKWAY repo (or integrations-only repo)
git clone git@github.com:workway/workway.git
cd workway/apps/api

# Install dependencies
npm install

# Copy environment template
cp .dev.vars.example .dev.vars
```

#### 2. Set Up OAuth App

For your integration (e.g., Slack):

```bash
# Create OAuth app on provider's platform
# Example: https://api.slack.com/apps

# Add credentials to .dev.vars
echo "SLACK_CLIENT_ID=your_client_id" >> .dev.vars
echo "SLACK_CLIENT_SECRET=your_secret" >> .dev.vars
```

**Redirect URL (Development):**
```
http://localhost:8787/oauth/slack/callback
```

#### 3. Create Integration Structure

```bash
# Use CLI (future) or manual creation
npm run workway integration init slack

# Or manually:
mkdir -p src/integrations/slack/actions
touch src/integrations/slack/index.ts
touch src/integrations/slack/actions/send-message.ts
```

#### 4. Write Integration Code

**Example: `src/integrations/slack/actions/send-message.ts`**

```typescript
import { z } from 'zod';
import type { ActionDefinition } from '../../../sdk/integration-sdk';

const SendMessageInputSchema = z.object({
  channel: z.string(),
  text: z.string(),
});

export const sendMessageAction: ActionDefinition = {
  id: 'slack.send-message',
  name: 'Send Message',
  description: 'Send a message to a Slack channel',
  integration: 'slack',
  requiredAuth: ['slack'],
  inputSchema: SendMessageInputSchema,

  async execute(input, context) {
    const { channel, text } = SendMessageInputSchema.parse(input);

    const tokens = await context.oauth.getTokens(context.userId, 'slack');
    if (!tokens) {
      throw new Error('Slack not connected');
    }

    const response = await fetch('https://slack.com/api/chat.postMessage', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${tokens.accessToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ channel, text }),
    });

    const result = await response.json();
    if (!result.ok) {
      throw new Error(`Slack API error: ${result.error}`);
    }

    return result;
  },
};
```

---

### Phase 3: Local Testing

#### 1. Register Integration Locally

**Edit `src/integrations/registry.ts`:**

```typescript
import slackIntegration from './slack';

export function registerAllIntegrations(env?: any): void {
  integrationRegistry.registerIntegration(gmailIntegration);
  integrationRegistry.registerIntegration(notionIntegration);
  integrationRegistry.registerIntegration(slackIntegration); // Add this
}
```

#### 2. Run Local Dev Server

```bash
# Start local Cloudflare Worker
npm run dev

# Server starts at http://localhost:8787
```

#### 3. Test Registry Endpoints

```bash
# Check integration registered
curl http://localhost:8787/registry/integrations | jq .

# Should show:
# {
#   "count": 3,
#   "integrations": [
#     { "id": "gmail", ... },
#     { "id": "notion", ... },
#     { "id": "slack", ... }  ← Your integration
#   ]
# }

# Check action registered
curl http://localhost:8787/registry/actions/slack.send-message | jq .
```

#### 4. Test OAuth Flow

**Start OAuth Flow:**
```bash
# In browser, visit:
http://localhost:8787/oauth/slack/authorize

# You'll be redirected to Slack
# After authorizing, redirected to callback
# Check database for stored tokens
```

**Verify Tokens:**
```bash
npx wrangler d1 execute marketplace-development --local \
  --command "SELECT * FROM oauth_credentials WHERE provider = 'slack'"
```

#### 5. Test Action Execution

Create a test workflow:

**`test-slack-workflow.json`:**
```json
{
  "id": "test-slack",
  "name": "Test Slack Integration",
  "trigger": { "type": "manual" },
  "steps": [
    {
      "id": "send_msg",
      "name": "Send Test Message",
      "type": "slack.send-message",
      "config": {
        "channel": "#test",
        "text": "Hello from WORKWAY!"
      }
    }
  ],
  "config": {}
}
```

**Trigger Test Workflow:**
```bash
# Use API or create test script
curl -X POST http://localhost:8787/workflows/trigger \
  -H "Content-Type: application/json" \
  -d @test-slack-workflow.json
```

#### 6. Write Unit Tests (Required)

**`src/integrations/slack/__tests__/send-message.test.ts`:**

```typescript
import { describe, it, expect, vi } from 'vitest';
import { sendMessageAction } from '../actions/send-message';

describe('slack.send-message', () => {
  it('should send message successfully', async () => {
    const mockContext = {
      userId: 'test-user',
      oauth: {
        getTokens: vi.fn().mockResolvedValue({
          accessToken: 'test-token',
          expiresAt: Date.now() + 3600000,
        }),
      },
    };

    const input = {
      channel: '#test',
      text: 'Hello!',
    };

    // Mock fetch
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ ok: true, ts: '1234567890.123' }),
    });

    const result = await sendMessageAction.execute(input, mockContext as any);

    expect(result.ok).toBe(true);
    expect(fetch).toHaveBeenCalledWith(
      'https://slack.com/api/chat.postMessage',
      expect.objectContaining({
        method: 'POST',
        headers: expect.objectContaining({
          'Authorization': 'Bearer test-token',
        }),
      })
    );
  });

  it('should throw error if Slack returns error', async () => {
    const mockContext = {
      userId: 'test-user',
      oauth: {
        getTokens: vi.fn().mockResolvedValue({
          accessToken: 'test-token',
        }),
      },
    };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ ok: false, error: 'channel_not_found' }),
    });

    await expect(
      sendMessageAction.execute({ channel: '#invalid', text: 'Hi' }, mockContext as any)
    ).rejects.toThrow('Slack API error: channel_not_found');
  });
});
```

**Run Tests:**
```bash
npm test src/integrations/slack
```

---

### Phase 4: Submission

#### 1. Create Pull Request

```bash
# Create feature branch
git checkout -b integration/slack

# Commit changes
git add src/integrations/slack
git commit -m "feat: Add Slack integration

- Add send-message action
- Configure OAuth with chat:write scope
- Include unit tests
- Update registry"

# Push to GitHub
git push origin integration/slack
```

#### 2. PR Template (Auto-filled)

```markdown
## Integration Submission: Slack

### Checklist
- [x] OAuth app created and credentials added
- [x] Integration definition created
- [x] All actions have Zod schemas
- [x] Unit tests written (100% coverage)
- [x] Local testing completed
- [x] OAuth flow tested
- [x] Documentation added
- [x] No secrets committed

### Integration Details
**Name:** Slack
**Actions:** 1 (send-message)
**Triggers:** 0
**OAuth Scopes:** chat:write, channels:read

### Test Results
```
✅ Unit tests: 5/5 passed
✅ Local OAuth flow: Working
✅ Action execution: Working
✅ Registry loading: Working
```

### Security Considerations
- OAuth tokens stored in Durable Object (encrypted)
- No user data logged
- Rate limiting respected
- Error messages sanitized

### Documentation
- [x] Action JSDoc comments
- [x] README added to integration folder
- [x] Example workflow provided
```

#### 3. Automated Checks (CI/CD)

**GitHub Actions runs:**
```yaml
name: Integration Review

on:
  pull_request:
    paths:
      - 'apps/api/src/integrations/**'

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Security scan
        run: npm audit

      - name: Secret scan
        uses: trufflesecurity/trufflehog@main

  tests:
    runs-on: ubuntu-latest
    steps:
      - name: Run unit tests
        run: npm test

      - name: Check coverage
        run: npm run test:coverage

  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build
        run: npm run build

      - name: Type check
        run: npm run typecheck
```

**Status Checks:**
✅ Security scan passed
✅ No secrets found
✅ Unit tests passed (5/5)
✅ Coverage 100%
✅ Build successful
✅ TypeScript checks passed

---

### Phase 5: Review & Approval

#### 1. Code Review (WORKWAY Team)

**Reviewers check:**
- [ ] Code quality (TypeScript best practices)
- [ ] Security (no XSS, injection, secrets)
- [ ] OAuth implementation (proper token handling)
- [ ] Error handling (all edge cases)
- [ ] Tests (comprehensive coverage)
- [ ] Documentation (clear and complete)
- [ ] Performance (no blocking operations)

**Review Time:** 2-3 business days

#### 2. Security Audit (Automated + Manual)

**Automated:**
- Dependency vulnerabilities (npm audit)
- Secret scanning (TruffleHog)
- Static analysis (ESLint, TypeScript)

**Manual:**
- OAuth flow security
- Token storage review
- API key handling
- Rate limiting implementation

#### 3. Approval & Merge

**If approved:**
```bash
# WORKWAY team merges PR
git checkout main
git merge integration/slack
git push origin main

# Automatically triggers deployment
```

**If changes requested:**
- Developer addresses feedback
- Updates PR
- Re-review cycle

---

### Phase 6: Deployment

#### 1. Automatic Deployment

**GitHub Actions (on merge to main):**
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Cloudflare Workers
        run: npm run deploy
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CF_API_TOKEN }}
          SLACK_CLIENT_ID: ${{ secrets.SLACK_CLIENT_ID }}
          SLACK_CLIENT_SECRET: ${{ secrets.SLACK_CLIENT_SECRET }}
```

**Deployment automatically:**
1. Builds worker
2. Runs final tests
3. Deploys to production
4. Updates registry
5. Notifies team

#### 2. Registry Update

**Registry is updated automatically** because:
- Integration is in `src/integrations/slack/`
- `registry.ts` imports and registers it
- `registerAllIntegrations()` is called at startup
- No manual database updates needed

#### 3. Verification

```bash
# Check production registry
curl https://marketplace-api.half-dozen.workers.dev/registry/integrations | jq .

# Should show Slack integration
# {
#   "integrations": [
#     { "id": "slack", "name": "Slack", "actionsCount": 1 }
#   ]
# }
```

---

## Workflow Developer Flow

### Overview

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW DEVELOPER JOURNEY                                  │
└─────────────────────────────────────────────────────────────┘

1. Sign Up for Developer Account
   └─> Self-service, no approval needed

2. Create Workflow Locally (or via Web UI)
   └─> Use Workflow SDK, compose integrations

3. Test in Development Environment
   └─> Connect test OAuth accounts, trigger workflows

4. Publish to Marketplace (Automatic Approval)
   └─> If using approved integrations only

5. Users Enable Workflow
   └─> Start earning 70% revenue share
```

---

## Step-by-Step: Workflow Developer

### Phase 1: Sign Up

#### 1. Create Developer Account

**Visit:** `https://workway.dev/developers/signup`

**Form:**
```
Name: _______________________
Email: ______________________
Company (optional): _________
Stripe Connect: [Link Account]
```

**Instant approval** - No review needed

#### 2. Set Up Development Environment

**Option A: Web-based Builder (Easiest)**
```
1. Go to https://workway.dev/developer/builder
2. Use drag-and-drop interface
3. Test inline
4. Publish with one click
```

**Option B: Local Development (Advanced)**
```bash
# Install CLI
npm install -g @workway/cli

# Login
workway login

# Create new workflow
workway workflow init stripe-to-notion
```

---

### Phase 2: Build Workflow

#### Option A: Web Builder (No Code)

**Step 1: Create Workflow**
```
Name: Stripe to Notion Invoice Tracker
Category: Finance
Icon: 💰
Price: $5/month
```

**Step 2: Configure Trigger**
```
Trigger: stripe.payment-succeeded
Test data: [Load sample payment]
```

**Step 3: Add Steps**
```
Step 1: notion.create-page
  Database ID: [User will configure]
  Properties:
    - Invoice ID: {{trigger.payment.id}}
    - Amount: {{trigger.payment.amount}}
    - Customer: {{trigger.payment.customer}}
```

**Step 4: Test**
```
[Test with Sample Data] → ✅ Success
```

**Step 5: Publish**
```
[Publish to Marketplace] → Automatic approval ✅
```

#### Option B: Code-based (Advanced)

**Create `stripe-to-notion/workflow.ts`:**

```typescript
import { defineWorkflow } from '@workway/sdk';
import { z } from 'zod';

export default defineWorkflow({
  metadata: {
    id: 'stripe-to-notion-invoice',
    name: 'Stripe to Notion Invoice Tracker',
    tagline: 'Automatically track Stripe invoices in Notion',
    description: 'Every time you receive a Stripe payment, this workflow creates a new page in your Notion database with invoice details.',
    category: 'finance',
    icon: '💰',
    version: '1.0.0',
    author: {
      name: 'Your Name',
      email: 'you@example.com',
    },
    tags: ['stripe', 'notion', 'invoicing'],
  },

  pricing: {
    model: 'subscription',
    price: 5,
    trialDays: 7,
  },

  integrations: ['stripe', 'notion'],

  trigger: {
    type: 'stripe.payment-succeeded',
  },

  configSchema: z.object({
    notionDatabaseId: z.string().describe('Notion database ID'),
  }),

  configFields: [
    {
      key: 'notionDatabaseId',
      label: 'Notion Database',
      description: 'Select the Notion database to track invoices',
      type: 'integration',
      required: true,
      schema: z.string(),
    },
  ],

  async execute({ trigger, config, actions }) {
    const payment = trigger.data;

    await actions.execute('notion.create-page', {
      database_id: config.notionDatabaseId,
      properties: {
        'Invoice ID': {
          title: [{ text: { content: payment.id } }],
        },
        'Amount': {
          number: payment.amount / 100,
        },
        'Customer': {
          rich_text: [{ text: { content: payment.customer_email } }],
        },
        'Date': {
          date: { start: new Date(payment.created * 1000).toISOString() },
        },
      },
    });

    return { success: true };
  },
});
```

---

### Phase 3: Testing

#### 1. Test with Sample Data

**Web Builder:**
```
[Load Sample Data] → [Test Run] → See results
```

**CLI:**
```bash
# Test locally
workway workflow test --data sample-payment.json

# Output:
# ✅ Workflow executed successfully
# ✅ notion.create-page completed
# ⏱️  Execution time: 1.2s
```

#### 2. Test with Real OAuth

```bash
# Connect test accounts
workway oauth connect stripe
workway oauth connect notion

# Test with real API calls
workway workflow test --live

# Output:
# ✅ Stripe webhook received
# ✅ Notion page created: https://notion.so/...
```

#### 3. Preview in Marketplace (Draft Mode)

```bash
# Publish as draft (visible only to you)
workway workflow publish --draft

# Visit: https://workway.dev/marketplace/your-workflow-id?preview=true
```

---

### Phase 4: Publish

#### 1. Submit to Marketplace

**Web Builder:**
```
[Publish to Marketplace]

Review:
- Name: Stripe to Notion Invoice Tracker ✓
- Price: $5/month ✓
- Description: Complete ✓
- Screenshots: 2 uploaded ✓
- Test results: All passed ✓

[Confirm & Publish]
```

**CLI:**
```bash
workway workflow publish

# Checks:
# ✅ Valid workflow definition
# ✅ All integrations approved
# ✅ Tests passed
# ✅ Metadata complete
# ✅ Screenshots uploaded

# Publishing...
# ✅ Published!
# 🎉 Your workflow is now live at:
#    https://workway.dev/marketplace/stripe-to-notion-invoice
```

#### 2. Automatic Approval

**Workflows get automatic approval if:**
- ✅ Use only approved integrations (Stripe, Notion)
- ✅ Pass automated tests
- ✅ Have valid metadata
- ✅ No custom code (web builder) OR code review passed (CLI)

**Manual review required if:**
- ❌ Use of custom/unapproved integrations
- ❌ External API calls in workflow code
- ❌ Potential security issues detected

#### 3. Listing Goes Live

**Immediately after approval:**
```
Your workflow is now live!

📊 Dashboard: https://workway.dev/developer/dashboard
📈 Analytics: 0 installs, 0 revenue (so far)
🎯 Optimize: Add screenshots, improve description
```

---

## Registry Update Process

### How Registry Updates Work

```
┌───────────────────────────────────────────────────┐
│ REGISTRY UPDATE FLOW                              │
└───────────────────────────────────────────────────┘

1. Developer submits integration (PR)
   └─> Code merged to main branch

2. GitHub Actions triggers deployment
   └─> npm run deploy

3. Wrangler builds and uploads Worker
   └─> New code includes integration

4. Worker starts up
   └─> index.ts runs registerAllIntegrations()

5. Registry populated at runtime
   └─> Integration now available

6. No manual database updates needed!
   └─> Registry is code-driven
```

### Key Point: Registry is Code-Driven

**There is NO database table for the registry.**

The registry exists in memory, populated at startup from the codebase:

**`apps/api/src/integrations/registry.ts`:**
```typescript
export function registerAllIntegrations(env?: any): void {
  // These imports come from the codebase
  integrationRegistry.registerIntegration(gmailIntegration);
  integrationRegistry.registerIntegration(notionIntegration);
  integrationRegistry.registerIntegration(slackIntegration);
  // As soon as Slack code is deployed, it's registered!
}
```

**`apps/api/src/index.ts`:**
```typescript
// Called at Worker startup
registerAllIntegrations();
console.log('🔌 Integration registry initialized');
```

### Adding Integration = Auto-Registry Update

**Before (Monolithic):**
```
1. Write integration code
2. Manually update database table
3. Manually update route handlers
4. Deploy
```

**After (Plugin Architecture):**
```
1. Write integration code
2. Deploy
   └─> Registry auto-updates! ✅
```

### Workflow Registry (Future)

For workflows, we WILL use a database:

**`marketplace_workflows` table:**
```sql
CREATE TABLE marketplace_workflows (
  id TEXT PRIMARY KEY,
  developer_id TEXT NOT NULL,
  name TEXT NOT NULL,
  definition TEXT NOT NULL,  -- JSON workflow definition
  status TEXT DEFAULT 'published',  -- published, draft, unlisted
  price REAL NOT NULL,
  installs INTEGER DEFAULT 0,
  rating REAL,
  created_at INTEGER DEFAULT (unixepoch()),
  FOREIGN KEY (developer_id) REFERENCES developers(id)
);
```

**Why database for workflows?**
- Workflows are user-generated content
- Need versioning, draft/published states
- Analytics (installs, revenue, ratings)
- Can't bundle every workflow in code

**Why NOT database for integrations?**
- Integrations are platform-maintained code
- Need to be loaded at startup
- Part of the Worker bundle
- Security-reviewed before deployment

---

## Summary

### Integration Developer (Platform Team / Trusted Partners)

**Submission:**
1. Apply → get approved → get repo access
2. Clone repo, create integration, write tests
3. Submit PR with tests and docs
4. Code review + security audit
5. Merge → auto-deploy → registry updates

**Registry Update:** Automatic (code-driven)

**Timeline:** 1-2 weeks from start to production

---

### Workflow Developer (Marketplace Ecosystem)

**Submission:**
1. Sign up (instant)
2. Build workflow (web or CLI)
3. Test with sample + real data
4. Publish → automatic approval (if using approved integrations)
5. Start earning 70% revenue

**Registry Update:** Database insert (on publish)

**Timeline:** 30 minutes to 2 hours from idea to live

---

## Developer Support

**Integration Developers:**
- Slack channel: `#integration-developers`
- Email: integrations@workway.dev
- Documentation: https://docs.workway.dev/integrations

**Workflow Developers:**
- Dashboard: https://workway.dev/developer/dashboard
- Community: https://community.workway.dev
- Support: https://workway.dev/developer/support

---

## Next: Build the Developer Portal

To make this all real, we need:

1. **Developer Portal** (web app)
   - Developer signup
   - Workflow builder
   - Testing sandbox
   - Analytics dashboard

2. **CLI Tool** (`@workway/cli`)
   - `workway login`
   - `workway workflow init`
   - `workway workflow test`
   - `workway workflow publish`

3. **Review System**
   - Automated checks (security, tests)
   - Manual review UI for WORKWAY team
   - Approval workflow

4. **Marketplace Workflows Table**
   - Store user-generated workflows
   - Version control
   - Analytics tracking

Want me to design any of these next?
