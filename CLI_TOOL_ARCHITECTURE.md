# WORKWAY CLI Tool Architecture

**Package:** `@workway/cli`
**NPM:** `npm install -g @workway/cli`
**Language:** TypeScript
**Runtime:** Node.js 18+

---

## Overview

The WORKWAY CLI is a developer tool that enables:
1. **Workflow Developers** to build, test, and publish marketplace workflows
2. **Integration Developers** to scaffold new integrations quickly
3. **Both** to authenticate, test OAuth flows, and manage their developer profiles

---

## Design Principles

1. **Developer Experience First**: Commands should be intuitive and well-documented
2. **TypeScript Native**: Full type safety and autocomplete
3. **Local-First Development**: Test workflows locally before publishing
4. **OAuth Made Easy**: Simple OAuth connection flows for testing
5. **Hot Reload**: Fast iteration with file watching
6. **Template-Driven**: Scaffolding based on proven patterns

---

## Architecture

```
@workway/cli
├── bin/
│   └── workway.ts              # CLI entry point
├── src/
│   ├── commands/
│   │   ├── auth/               # Authentication commands
│   │   │   ├── login.ts
│   │   │   ├── logout.ts
│   │   │   └── whoami.ts
│   │   ├── workflow/           # Workflow development
│   │   │   ├── init.ts
│   │   │   ├── dev.ts
│   │   │   ├── test.ts
│   │   │   ├── build.ts
│   │   │   ├── publish.ts
│   │   │   └── unpublish.ts
│   │   ├── integration/        # Integration development
│   │   │   ├── init.ts
│   │   │   ├── generate.ts     # Generate action/trigger
│   │   │   └── test.ts
│   │   ├── oauth/              # OAuth management
│   │   │   ├── connect.ts
│   │   │   ├── list.ts
│   │   │   └── disconnect.ts
│   │   └── developer/          # Developer profile
│   │       ├── register.ts
│   │       ├── profile.ts
│   │       └── earnings.ts
│   ├── lib/
│   │   ├── api-client.ts       # WORKWAY API client
│   │   ├── config.ts           # CLI configuration
│   │   ├── oauth-flow.ts       # Local OAuth server
│   │   ├── workflow-builder.ts # Workflow compilation
│   │   └── templates/          # Code templates
│   │       ├── workflow/
│   │       │   ├── basic.ts
│   │       │   ├── scheduled.ts
│   │       │   └── webhook.ts
│   │       └── integration/
│   │           ├── action.ts
│   │           └── trigger.ts
│   ├── types/
│   │   └── index.ts            # TypeScript types
│   └── utils/
│       ├── logger.ts           # Pretty logging
│       ├── prompts.ts          # Interactive prompts
│       └── validator.ts        # Validation utilities
└── package.json
```

---

## Commands

### Authentication

#### `workway login`
Authenticate with WORKWAY platform.

```bash
workway login

# Opens browser for OAuth flow
# Stores token in ~/.workway/credentials.json
```

**Implementation:**
1. Start local server on `http://localhost:3456`
2. Open browser to `https://marketplace-api.half-dozen.workers.dev/auth/cli/login?callback=http://localhost:3456`
3. User logs in via browser
4. API redirects back to local server with token
5. CLI saves token to config file
6. Display success message with user info

#### `workway logout`
Clear local authentication.

```bash
workway logout
```

#### `workway whoami`
Display current authenticated user.

```bash
workway whoami

# Output:
# Logged in as: john@example.com (Developer)
# Account ID: dev_abc123
# Rate Limit: 1000/hour
```

---

### Workflow Development

#### `workway workflow init [name]`
Create a new workflow project.

```bash
workway workflow init gmail-to-slack

# Interactive prompts:
? Workflow name: Gmail to Slack Alerts
? Category: productivity
? Description: Get Slack notifications for important emails
? Price (USD): 8
? Free trial days: 7

✅ Created workflow project at ./gmail-to-slack/
📝 Edit workflow.ts to define your workflow
🧪 Run 'workway workflow test' to test locally
📦 Run 'workway workflow publish' when ready
```

**Creates:**
```
gmail-to-slack/
├── workflow.ts           # Main workflow definition
├── workway.config.ts     # CLI configuration
├── test-data.json        # Sample trigger data
├── .env.example          # OAuth credentials template
└── README.md             # Generated documentation
```

**Template (`workflow.ts`):**
```typescript
import { defineWorkflow } from '@workway/sdk/workflow';
import { z } from 'zod';

export default defineWorkflow({
  metadata: {
    id: 'gmail-to-slack-alerts',
    name: 'Gmail to Slack Alerts',
    tagline: 'Get Slack notifications for important emails',
    description: 'Automatically send Slack messages when you receive emails from VIP contacts.',
    category: 'productivity',
    icon: '📧',
    version: '1.0.0',
    author: {
      name: 'Your Name',
      email: 'you@example.com',
    },
    tags: ['gmail', 'slack', 'notifications'],
  },

  pricing: {
    model: 'subscription',
    price: 8,
    trialDays: 7,
  },

  integrations: ['gmail', 'slack'],

  trigger: {
    type: 'gmail.new-email',
  },

  configSchema: z.object({
    slackChannel: z.string().describe('Slack channel for notifications'),
  }),

  configFields: [
    {
      key: 'slackChannel',
      label: 'Slack Channel',
      description: 'Where to send notifications',
      type: 'text',
      required: true,
      placeholder: '#inbox',
    },
  ],

  async execute({ trigger, config, actions }) {
    // Fetch email details
    const email = await actions.execute('gmail.fetch-email', {
      messageId: trigger.data.messageId
    });

    // Send Slack notification
    await actions.execute('slack.send-message', {
      channel: config.slackChannel,
      text: `📧 New email from ${email.from}: ${email.subject}`
    });

    return { success: true };
  },
});
```

#### `workway workflow dev`
Start local development server with hot reload.

```bash
cd gmail-to-slack
workway workflow dev

# Output:
# 🔧 Starting development server...
# 📡 Workflow API: http://localhost:3000
# 🔄 Watching for changes...
# 🔌 OAuth callback: http://localhost:3000/oauth/callback
#
# Available endpoints:
# - POST /trigger - Manually trigger workflow
# - GET /config - View workflow configuration
# - GET /health - Health check
```

**Features:**
- File watching with hot reload
- Local OAuth flow testing
- Mock integrations available
- Detailed execution logs

#### `workway workflow test`
Test workflow execution.

```bash
# Test with sample data (mocked actions)
workway workflow test --mock

# Test with live OAuth (real API calls)
workway workflow test --live

# Test with specific trigger data
workway workflow test --data ./test-email.json

# Output:
# 🧪 Testing workflow: Gmail to Slack Alerts
#
# ✅ gmail.fetch-email
#    Fetched email from customer@example.com
#    Subject: Invoice Payment Received
#
# ✅ slack.send-message
#    Sent to #inbox
#    Message: 📧 New email from customer@example.com: Invoice Payment Received
#
# ✅ Workflow completed in 1.2s
```

#### `workway workflow build`
Build workflow for production.

```bash
workway workflow build

# Output:
# 📦 Building workflow...
# ✅ Compiled workflow.ts
# ✅ Validated metadata
# ✅ Checked integrations
# ✅ Generated manifest
#
# Build output: ./dist/workflow.json
```

#### `workway workflow publish`
Publish workflow to marketplace.

```bash
workway workflow publish

# Interactive confirmation:
? Publish "Gmail to Slack Alerts" to marketplace? Yes
? Make it public immediately? Yes

# Output:
# 📤 Publishing workflow...
# ✅ Uploaded to marketplace
# ✅ Validation passed
# ✅ Status: Published
#
# 🎉 Your workflow is live!
# URL: https://workway.dev/marketplace/gmail-to-slack-alerts
# Start earning: 70% of $8/month per install
```

**Options:**
```bash
# Publish as draft (not public)
workway workflow publish --draft

# Update existing workflow
workway workflow publish --update

# Set pricing
workway workflow publish --price 12

# Add screenshots
workway workflow publish --screenshots ./screenshots/*.png
```

#### `workway workflow unpublish`
Remove workflow from marketplace.

```bash
workway workflow unpublish

# Confirmation prompt
? Are you sure? This will affect X active installations. Yes

# Output:
# ✅ Workflow unpublished
# ℹ️ Existing installations will continue to work
# ℹ️ No new users can install
```

---

### Integration Development

#### `workway integration init [name]`
Scaffold a new integration.

```bash
workway integration init slack

# Interactive prompts:
? Integration name: Slack
? Description: Slack messaging and notifications
? OAuth provider: slack
? OAuth scopes: chat:write,channels:read
? First action name: send-message

✅ Created integration at ./src/integrations/slack/
📝 Edit actions/send-message.ts to implement
🧪 Write tests in __tests__/
📚 See docs/ADDING_INTEGRATIONS.md for details
```

**Creates:**
```
src/integrations/slack/
├── index.ts                    # Integration definition
├── actions/
│   └── send-message.ts         # First action
├── __tests__/
│   └── send-message.test.ts    # Test file
└── README.md                   # Documentation
```

#### `workway integration generate action [name]`
Generate a new action for existing integration.

```bash
cd src/integrations/slack
workway integration generate action list-channels

# Output:
# ✅ Created actions/list-channels.ts
# ✅ Created __tests__/list-channels.test.ts
# ✅ Updated index.ts
# 📝 Edit actions/list-channels.ts to implement
```

#### `workway integration generate trigger [name]`
Generate a new trigger.

```bash
workway integration generate trigger new-message

# Output:
# ✅ Created triggers/new-message.ts
# ✅ Updated index.ts
# 📝 Implement webhook setup in triggers/new-message.ts
```

#### `workway integration test`
Run integration tests.

```bash
workway integration test

# Runs all tests for current integration
# Uses vitest under the hood
```

---

### OAuth Management

#### `workway oauth connect [provider]`
Connect an OAuth account for testing.

```bash
workway oauth connect gmail

# Output:
# 🔐 Opening OAuth flow for gmail...
# ✅ Successfully connected gmail
# Email: developer@example.com
# Scopes: gmail.readonly, gmail.send
```

**Implementation:**
1. Start local OAuth callback server
2. Open browser to WORKWAY OAuth authorization URL
3. User completes OAuth flow
4. Tokens saved to `.workway/oauth-tokens.json`
5. Used for `workway workflow test --live`

#### `workway oauth list`
List connected OAuth accounts.

```bash
workway oauth list

# Output:
# Connected OAuth accounts:
#
# gmail (developer@example.com)
#   Scopes: gmail.readonly, gmail.send
#   Expires: 2025-12-01
#
# slack (workspace.slack.com)
#   Scopes: chat:write, channels:read
#   Expires: Never
```

#### `workway oauth disconnect [provider]`
Disconnect an OAuth account.

```bash
workway oauth disconnect gmail

# Output:
# ✅ Disconnected gmail
```

---

### Developer Profile

#### `workway developer register`
Register as a workflow developer.

```bash
workway developer register

# Interactive signup:
? Email: john@example.com
? Password: ********
? Company/Name: John Doe
? Bio: Full-stack developer building productivity tools
? Website: https://johndoe.dev

✅ Developer account created!
📧 Check your email to verify your account
```

#### `workway developer profile`
View/edit developer profile.

```bash
workway developer profile

# Output:
# Developer Profile
#
# Name: John Doe
# Email: john@example.com
# Bio: Full-stack developer building productivity tools
# Website: https://johndoe.dev
#
# Workflows Published: 3
# Total Installs: 142
# Revenue (This Month): $394.80
#
# Payout Settings:
# Method: Stripe Connect
# Status: Connected
```

#### `workway developer earnings`
View earnings and payouts.

```bash
workway developer earnings

# Output:
# Earnings Dashboard
#
# Month       Installs  Revenue   Payout    Status
# Nov 2025    47        $263.20   $184.24   Pending
# Oct 2025    39        $218.40   $152.88   Paid
# Sep 2025    31        $173.60   $121.52   Paid
#
# Total Earned: $655.20
# Total Paid: $274.40
# Pending: $184.24 (Payout on Nov 15)
```

---

## Configuration

### Global Config (`~/.workway/config.json`)

```json
{
  "apiUrl": "https://marketplace-api.half-dozen.workers.dev",
  "credentials": {
    "token": "jwt_token_here",
    "userId": "usr_abc123",
    "email": "john@example.com"
  },
  "oauth": {
    "callbackPort": 3456
  },
  "editor": "code"
}
```

### Project Config (`workway.config.ts`)

```typescript
import { defineConfig } from '@workway/cli';

export default defineConfig({
  // Workflow metadata (overrides workflow.ts if provided)
  name: 'Gmail to Slack Alerts',
  version: '1.0.0',

  // Development settings
  dev: {
    port: 3000,
    hotReload: true,
    mockMode: true, // Use mocked integrations by default
  },

  // Testing settings
  test: {
    testDataFile: './test-data.json',
    timeout: 30000,
  },

  // Build settings
  build: {
    outDir: './dist',
    minify: false,
  },

  // Publishing settings
  publish: {
    screenshots: ['./screenshots/*.png'],
    demoVideo: './demo.mp4',
  },
});
```

---

## API Client

### `WorkwayAPIClient`

```typescript
import { WorkwayAPIClient } from '@workway/cli/client';

const client = new WorkwayAPIClient({
  apiUrl: 'https://marketplace-api.half-dozen.workers.dev',
  token: 'jwt_token',
});

// Workflows
await client.workflows.publish(workflow);
await client.workflows.get(id);
await client.workflows.list();

// OAuth
await client.oauth.getAuthUrl('gmail');
await client.oauth.exchange(code);

// Developer
await client.developer.getProfile();
await client.developer.getEarnings();
```

---

## OAuth Flow

### Local OAuth Server

When running `workway oauth connect` or `workway workflow test --live`:

1. **Start local server** on `http://localhost:3456`
2. **Open browser** to WORKWAY OAuth URL with callback
3. **User authorizes** the connection
4. **WORKWAY API redirects** back to `http://localhost:3456/callback?code=...`
5. **CLI exchanges code** for tokens
6. **Tokens saved** to `.workway/oauth-tokens.json`

```typescript
// OAuth flow implementation
class LocalOAuthServer {
  private server: http.Server;
  private port: number = 3456;

  async start(): Promise<string> {
    return new Promise((resolve, reject) => {
      this.server = http.createServer((req, res) => {
        const url = new URL(req.url!, `http://localhost:${this.port}`);

        if (url.pathname === '/callback') {
          const code = url.searchParams.get('code');
          const error = url.searchParams.get('error');

          if (error) {
            res.end('OAuth failed. You can close this window.');
            reject(new Error(error));
          } else {
            res.end('Success! You can close this window.');
            resolve(code!);
          }

          this.server.close();
        }
      });

      this.server.listen(this.port);
    });
  }
}
```

---

## Templates

### Workflow Templates

**Basic Workflow:**
```typescript
// templates/workflow/basic.ts
export default defineWorkflow({
  metadata: { /* ... */ },
  integrations: ['integration1', 'integration2'],
  async execute({ trigger, actions }) {
    // Simple sequential execution
  }
});
```

**Scheduled Workflow:**
```typescript
// templates/workflow/scheduled.ts
export default defineWorkflow({
  trigger: {
    type: 'schedule',
    config: {
      cron: '0 9 * * *', // Daily at 9am
    }
  },
  async execute({ actions }) {
    // Scheduled task
  }
});
```

**Webhook Workflow:**
```typescript
// templates/workflow/webhook.ts
export default defineWorkflow({
  trigger: {
    type: 'webhook',
    config: {
      method: 'POST',
    }
  },
  async execute({ trigger, actions }) {
    // Handle webhook payload
  }
});
```

### Integration Templates

**Action Template:**
```typescript
// templates/integration/action.ts
import { z } from 'zod';
import type { ActionDefinition } from '@workway/sdk/integration';

const InputSchema = z.object({
  // Define inputs
});

export const myAction: ActionDefinition = {
  id: 'integration.my-action',
  name: 'My Action',
  description: 'Description here',
  integration: 'integration',
  requiredAuth: ['integration'],
  inputSchema: InputSchema,

  async execute(input, context) {
    const tokens = await context.oauth.getTokens(context.userId, 'integration');

    const response = await fetch('https://api.example.com/endpoint', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${tokens.accessToken}`,
      },
      body: JSON.stringify(input),
    });

    return await response.json();
  },
};
```

---

## Error Handling

### User-Friendly Errors

```typescript
try {
  await client.workflows.publish(workflow);
} catch (error) {
  if (error.code === 'VALIDATION_ERROR') {
    console.error('❌ Validation failed:');
    for (const err of error.errors) {
      console.error(`  - ${err.field}: ${err.message}`);
    }
  } else if (error.code === 'UNAUTHORIZED') {
    console.error('❌ Not authenticated. Run: workway login');
  } else if (error.code === 'PAYMENT_REQUIRED') {
    console.error('❌ Developer account not verified');
    console.error('   Complete Stripe Connect setup at: https://workway.dev/developer/settings');
  } else {
    console.error('❌ Unexpected error:', error.message);
  }
  process.exit(1);
}
```

---

## Testing Strategy

### Unit Tests
```bash
npm test
```

Uses Vitest for:
- Command parsing
- API client methods
- Template generation
- Validation logic

### Integration Tests
```bash
npm test:integration
```

Tests against staging API:
- OAuth flows
- Workflow publishing
- API authentication

### E2E Tests
```bash
npm test:e2e
```

Full workflow from init to publish using test account.

---

## Distribution

### NPM Package

```bash
npm install -g @workway/cli
```

### Standalone Binaries (Future)

```bash
# Download binary for your platform
curl -fsSL https://cli.workway.dev/install.sh | sh

# Creates:
# - macOS: /usr/local/bin/workway
# - Linux: /usr/local/bin/workway
# - Windows: C:\Program Files\workway\workway.exe
```

---

## Implementation Roadmap

### Phase 1: MVP (Week 1-2)
- [ ] Core CLI framework (Commander.js)
- [ ] `workway login/logout/whoami`
- [ ] `workway workflow init`
- [ ] `workway workflow test --mock`
- [ ] Basic templates
- [ ] API client

### Phase 2: Development Tools (Week 3-4)
- [ ] `workway workflow dev` with hot reload
- [ ] `workway workflow test --live`
- [ ] `workway oauth connect/list/disconnect`
- [ ] Local OAuth server
- [ ] Better error messages

### Phase 3: Publishing (Week 5-6)
- [ ] `workway workflow build`
- [ ] `workway workflow publish`
- [ ] Screenshot upload
- [ ] Validation checks
- [ ] `workway developer earnings`

### Phase 4: Integration Tools (Week 7-8)
- [ ] `workway integration init`
- [ ] `workway integration generate`
- [ ] Integration testing helpers
- [ ] Documentation generation

### Phase 5: Polish (Week 9-10)
- [ ] Interactive prompts with Inquirer
- [ ] Beautiful logging with Chalk
- [ ] Progress bars with Ora
- [ ] Auto-updates
- [ ] Telemetry (opt-in)

---

## Dependencies

### Core
- `commander` - CLI framework
- `inquirer` - Interactive prompts
- `chalk` - Terminal colors
- `ora` - Spinners
- `axios` - HTTP client
- `dotenv` - Environment variables

### Development
- `chokidar` - File watching
- `esbuild` - Fast bundling
- `typescript` - Type safety

### OAuth
- `express` - Local OAuth server
- `open` - Open browser

### Testing
- `vitest` - Unit testing
- `@workway/sdk` - SDK package

---

## Next Steps

1. **Create GitHub repository**: `workway/cli`
2. **Set up TypeScript project** with proper configuration
3. **Implement Phase 1 MVP** (login, init, test)
4. **Publish to NPM** as `@workway/cli@0.1.0`
5. **Gather feedback** from early adopters
6. **Iterate** based on developer experience

---

## Example Usage Session

```bash
# Install CLI
npm install -g @workway/cli

# Authenticate
workway login

# Create workflow
workway workflow init email-to-crm
cd email-to-crm

# Connect OAuth for testing
workway oauth connect gmail
workway oauth connect salesforce

# Test locally
workway workflow test --live

# Publish to marketplace
workway workflow publish

# Check earnings
workway developer earnings
```

---

**Status:** Design Complete ✅
**Next:** Begin implementation (Phase 1 MVP)
**Owner:** WORKWAY Team
**Timeline:** 10 weeks to full feature set
