# WORKWAY Plugin Architecture

## Overview

WORKWAY now uses a **two-layer plugin architecture** that separates integration logic from workflow orchestration, enabling:

1. **Easy integration development** - Add new services without modifying core code
2. **Marketplace-ready workflows** - Compose integrations into sellable products
3. **Clean separation of concerns** - Orchestration vs integration logic
4. **Type-safe development** - Full TypeScript support with schemas

---

## Architecture Layers

```
┌─────────────────────────────────────────────┐
│   LAYER 2: Workflow SDK                     │
│   - Marketplace products                    │
│   - User-facing workflows                   │
│   - Compose integrations                    │
│   - Example: "Stripe→Notion Invoice"        │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│   LAYER 1: Integration SDK                  │
│   - Service connectors                      │
│   - OAuth management                        │
│   - Action definitions                      │
│   - Example: Gmail, Slack, Notion plugins   │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│   CORE: WorkflowExecutor                    │
│   - Pure orchestration                      │
│   - Step execution                          │
│   - State management                        │
│   - Retry logic                             │
└─────────────────────────────────────────────┘
```

---

## Directory Structure

```
apps/api/src/
├── sdk/                          # SDK Layer
│   ├── integration-sdk.ts        # Layer 1: Integration types
│   ├── workflow-sdk.ts           # Layer 2: Workflow types
│   └── index.ts                  # Exports
│
├── integrations/                 # Integration Plugins
│   ├── gmail/
│   │   ├── actions/
│   │   │   ├── fetch-email.ts
│   │   │   ├── add-label.ts
│   │   │   └── send-email.ts
│   │   └── index.ts              # Integration definition
│   │
│   ├── notion/
│   │   ├── actions/
│   │   │   ├── create-page.ts
│   │   │   ├── update-page.ts
│   │   │   └── query-database.ts
│   │   └── index.ts
│   │
│   └── registry.ts               # Central registry
│
├── workflows/                    # Workflow Products (Future)
│   ├── stripe-to-notion/
│   │   ├── workflow.ts
│   │   └── manifest.json
│   └── registry.ts
│
└── durable-objects/
    ├── WorkflowExecutor.ts           # Old (monolithic)
    └── WorkflowExecutor.refactored.ts # New (plugin-based)
```

---

## Layer 1: Integration SDK

### Creating an Integration

**Example: Gmail Integration**

```typescript
// apps/api/src/integrations/gmail/index.ts

import type { IntegrationDefinition } from '../../sdk/integration-sdk'
import { sendEmailAction } from './actions/send-email'
import { fetchEmailAction } from './actions/fetch-email'

export const gmailIntegration: IntegrationDefinition = {
  id: 'gmail',
  name: 'Gmail',
  description: 'Gmail integration for WORKWAY',
  version: '1.0.0',
  author: 'WORKWAY Team',
  icon: '📧',

  oauth: {
    provider: 'gmail',
    scopes: [
      'https://www.googleapis.com/auth/gmail.readonly',
      'https://www.googleapis.com/auth/gmail.send',
      'https://www.googleapis.com/auth/gmail.labels',
      'https://www.googleapis.com/auth/gmail.modify'
    ]
  },

  actions: [sendEmailAction, fetchEmailAction],
  triggers: []
}
```

### Creating an Action

**Example: Send Email Action**

```typescript
// apps/api/src/integrations/gmail/actions/send-email.ts

import { z } from 'zod'
import type { ActionDefinition } from '../../../sdk/integration-sdk'

const SendEmailInputSchema = z.object({
  to: z.string().email(),
  subject: z.string().min(1),
  body: z.string().min(1),
  html: z.boolean().optional()
})

export const sendEmailAction: ActionDefinition = {
  id: 'gmail.send-email',
  name: 'Send Email',
  description: 'Send an email via Gmail',
  integration: 'gmail',
  requiredAuth: ['gmail'],
  inputSchema: SendEmailInputSchema,

  async execute(input, context) {
    // Validate input
    const { to, subject, body, html } = SendEmailInputSchema.parse(input)

    // Get OAuth tokens
    const tokens = await context.oauth.getTokens(context.userId, 'gmail')
    if (!tokens) {
      throw new Error('Gmail not connected')
    }

    // Call Gmail API
    const response = await fetch('https://gmail.googleapis.com/gmail/v1/users/me/messages/send', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${tokens.accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        raw: encodeEmail({ to, subject, body, html })
      })
    })

    if (!response.ok) {
      throw new Error(`Gmail API error: ${response.status}`)
    }

    return await response.json()
  }
}
```

### Registering Integrations

```typescript
// apps/api/src/integrations/registry.ts

import { ActionRegistry } from '../sdk/integration-sdk'
import gmailIntegration from './gmail'
import notionIntegration from './notion'

export const integrationRegistry = new ActionRegistry()

export function registerAllIntegrations(env?: any) {
  integrationRegistry.registerIntegration(gmailIntegration)
  integrationRegistry.registerIntegration(notionIntegration)

  // Initialize
  gmailIntegration.initialize?.(env)
  notionIntegration.initialize?.(env)
}
```

---

## Layer 2: Workflow SDK

### Creating a Workflow Product

**Example: Stripe to Notion Invoice Tracker**

```typescript
// apps/api/src/workflows/stripe-to-notion/workflow.ts

import { defineWorkflow } from '../../sdk/workflow-sdk'
import { z } from 'zod'

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
      name: 'WORKWAY Team',
      email: 'team@workway.dev'
    },
    tags: ['stripe', 'notion', 'invoicing', 'finance']
  },

  pricing: {
    model: 'subscription',
    price: 5,
    trialDays: 7
  },

  integrations: ['stripe', 'notion'],

  trigger: {
    type: 'stripe.payment-succeeded'
  },

  configSchema: z.object({
    notionDatabaseId: z.string().min(1).describe('Notion database ID for invoices'),
    currencySymbol: z.string().default('$').describe('Currency symbol')
  }),

  configFields: [
    {
      key: 'notionDatabaseId',
      label: 'Notion Database',
      description: 'Select the Notion database to track invoices',
      type: 'integration',
      required: true,
      schema: z.string()
    },
    {
      key: 'currencySymbol',
      label: 'Currency Symbol',
      description: 'Symbol to display for amounts',
      type: 'text',
      required: false,
      default: '$',
      schema: z.string()
    }
  ],

  async execute({ trigger, config, actions }) {
    const payment = trigger.data

    // Create Notion page
    await actions.execute('notion.create-page', {
      database_id: config.notionDatabaseId,
      properties: {
        'Invoice ID': {
          title: [{ text: { content: payment.id } }]
        },
        'Amount': {
          number: payment.amount / 100
        },
        'Currency': {
          select: { name: payment.currency.toUpperCase() }
        },
        'Customer': {
          rich_text: [{ text: { content: payment.customer_email } }]
        },
        'Date': {
          date: { start: new Date(payment.created * 1000).toISOString() }
        },
        'Status': {
          select: { name: 'Paid' }
        }
      }
    })

    return {
      success: true,
      data: {
        invoiceId: payment.id,
        amount: payment.amount / 100
      }
    }
  }
})
```

---

## Using the Plugin Architecture

### In WorkflowExecutor

**OLD (Monolithic):**

```typescript
// Step type is hardcoded
switch (step.type) {
  case 'gmail':
    return await this.executeGmailStep(step)  // 100+ lines of Gmail logic
  case 'notion':
    return await this.executeNotionStep(step) // 100+ lines of Notion logic
  // Every new integration = modify this file
}
```

**NEW (Plugin-based):**

```typescript
// Step type references action ID from registry
const action = integrationRegistry.getAction(step.type) // e.g., 'gmail.send-email'
if (!action) {
  throw new Error(`Action not found: ${step.type}`)
}

// Execute via registry
const context = {
  userId: this.state.userId,
  oauth: this.createOAuthManager(),
  storage: this.ctx.storage,
  triggerData: this.state.triggerData,
  stepResults: this.getStepResultsMap(),
  env: this.env
}

return await action.execute(step.config, context)
// Adding new integration = ZERO changes to this file
```

### Workflow Definition (Updated)

**OLD Format:**

```json
{
  "steps": [
    {
      "type": "gmail_fetch",  // Hardcoded string
      "config": { ... }
    }
  ]
}
```

**NEW Format:**

```json
{
  "steps": [
    {
      "type": "gmail.fetch-email",  // Action ID from registry
      "config": { ... }
    }
  ]
}
```

---

## Benefits

### For Platform

✅ **Scalable** - Add unlimited integrations without core changes
✅ **Maintainable** - Each integration isolated in its own module
✅ **Testable** - Test integrations independently
✅ **Type-safe** - Full TypeScript support with Zod schemas
✅ **Documentation** - Auto-generate docs from action definitions

### For Integration Developers

✅ **Simple API** - Clear `ActionDefinition` interface
✅ **OAuth abstracted** - Just call `context.oauth.getTokens()`
✅ **No core changes** - Never touch `WorkflowExecutor.ts`
✅ **Hot-swappable** - Update integrations without redeploying core
✅ **Reusable** - Actions work across all workflows

### For Workflow Developers

✅ **Compose integrations** - Mix and match actions
✅ **Type-safe configs** - Zod schemas for validation
✅ **Sellable products** - Marketplace-ready workflows
✅ **Revenue share** - 70% of subscription goes to developer
✅ **User-friendly** - No code required for end users

---

## Migration Path

### Phase 1: ✅ COMPLETED
- [x] Create Integration SDK (`integration-sdk.ts`)
- [x] Create Workflow SDK (`workflow-sdk.ts`)
- [x] Extract Gmail integration to plugin format
- [x] Extract Notion integration to plugin format
- [x] Create ActionRegistry
- [x] Create refactored WorkflowExecutor

### Phase 2: In Progress
- [ ] Update `index.ts` to use refactored WorkflowExecutor
- [ ] Migrate existing workflow definitions to new format
- [ ] Deploy and test
- [ ] Monitor for issues

### Phase 3: Future
- [ ] Build Integration CLI (`workway integration init`)
- [ ] Build Workflow CLI (`workway workflow init`)
- [ ] Add more integrations (Slack, Stripe, Salesforce, etc.)
- [ ] Open marketplace to developers
- [ ] Implement revenue sharing

---

## Developer Experience

### Adding a New Integration (Future CLI)

```bash
$ workway integration init slack

✨ Creating new integration: slack

📁 Created directory: integrations/slack/
📝 Created index.ts
📝 Created actions/send-message.ts
📝 Created README.md

Next steps:
  cd integrations/slack
  workway integration test
  workway integration deploy
```

### Adding a New Workflow (Future CLI)

```bash
$ workway workflow init gmail-to-slack

✨ Creating new workflow: gmail-to-slack

? Workflow name: Gmail to Slack Alerts
? Category: productivity
? Trigger: gmail:new-email
? Price: $8/month

📁 Created workflows/gmail-to-slack/
📝 Created workflow.ts
📝 Created manifest.json

? Add first step: slack.send-message

✅ Workflow created!

Test: workway workflow test
Publish: workway workflow publish
```

---

## API Reference

### Integration SDK

```typescript
// Action Definition
interface ActionDefinition<TInput, TOutput> {
  id: string                    // e.g., 'gmail.send-email'
  name: string                  // Human-readable name
  description: string
  integration: string           // Parent integration ID
  inputSchema: ZodSchema        // Input validation
  requiredAuth?: string[]       // OAuth providers needed
  execute(input, context): Promise<TOutput>
}

// Action Context
interface ActionContext {
  userId: string                // User running the workflow
  oauth: OAuthManager           // OAuth token manager
  storage: DurableObjectStorage // Persistent storage
  triggerData: any              // Trigger event data
  stepResults: Record<string, any> // Previous step outputs
  env: Env                      // Environment variables
}
```

### Workflow SDK

```typescript
// Workflow Definition
interface WorkflowDefinition {
  metadata: WorkflowMetadata    // Name, description, icon
  pricing: WorkflowPricing      // Price, model, trial
  integrations: string[]        // Required integrations
  trigger: { type: string }     // Trigger type
  configSchema?: ZodSchema      // User config schema
  execute(context): Promise<WorkflowResult>
}

// Workflow Context
interface WorkflowContext {
  runId: string
  userId: string
  trigger: { type: string, data: any }
  config: Record<string, any>   // User configuration
  actions: ActionHelpers        // Integration helpers
  storage: WorkflowStorage      // State storage
}
```

---

## Examples

See:
- `apps/api/src/integrations/gmail/` - Complete Gmail integration
- `apps/api/src/integrations/notion/` - Complete Notion integration
- `apps/api/src/durable-objects/WorkflowExecutor.refactored.ts` - Refactored executor

---

## Questions?

- 📖 Docs: https://docs.workway.dev
- 💬 Discord: https://discord.gg/workway
- 📧 Email: developers@workway.dev

---

**Status:** Phase 1 Complete ✅ | Phase 2 In Progress 🚧 | Phase 3 Planned 📋
