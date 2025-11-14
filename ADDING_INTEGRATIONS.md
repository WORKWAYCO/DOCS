# Adding New Integrations to WORKWAY

This guide walks you through adding a new OAuth-based integration to the WORKWAY platform.

## Overview

Adding an integration involves 4 main steps:
1. **Configure OAuth** - Set up OAuth app and credentials
2. **Create Integration Plugin** - Define actions using the SDK
3. **Register Integration** - Add to the registry
4. **Test & Deploy** - Verify everything works

---

## Example: Adding Slack Integration

Let's add Slack integration with a `send-message` action.

---

## Step 1: Configure OAuth

### 1.1 Create Slack OAuth App

1. Go to https://api.slack.com/apps
2. Click **Create New App** → **From scratch**
3. Name: "WORKWAY Marketplace"
4. Workspace: Select your development workspace
5. Click **Create App**

### 1.2 Configure OAuth Scopes

1. Navigate to **OAuth & Permissions**
2. Under **Scopes** → **Bot Token Scopes**, add:
   - `chat:write` - Send messages
   - `channels:read` - List channels
   - `groups:read` - List private channels

### 1.3 Set Redirect URL

Add your callback URL:
```
https://marketplace-api.half-dozen.workers.dev/oauth/slack/callback
```

### 1.4 Get Credentials

1. Navigate to **Basic Information**
2. Copy **Client ID** and **Client Secret**
3. Add to your `.dev.vars`:

```bash
# .dev.vars
SLACK_CLIENT_ID=your_client_id_here
SLACK_CLIENT_SECRET=your_client_secret_here
```

4. Add to Cloudflare Workers secrets (production):

```bash
npx wrangler secret put SLACK_CLIENT_ID
npx wrangler secret put SLACK_CLIENT_SECRET
```

---

## Step 2: Create Integration Plugin

### 2.1 Create Directory Structure

```bash
cd apps/api/src/integrations
mkdir -p slack/actions
```

### 2.2 Create Action: Send Message

Create `apps/api/src/integrations/slack/actions/send-message.ts`:

```typescript
/**
 * Slack: Send Message Action
 *
 * Sends a message to a Slack channel
 */

import { z } from 'zod';
import type { ActionDefinition, ActionContext } from '../../../sdk/integration-sdk';

// ============================================================================
// INPUT SCHEMA
// ============================================================================

const SendMessageInputSchema = z.object({
	/** Channel ID or name (e.g., '#general' or 'C1234567890') */
	channel: z.string().min(1, 'Channel is required'),

	/** Message text */
	text: z.string().min(1, 'Message text is required'),

	/** Optional: Send as thread reply */
	thread_ts: z.string().optional(),

	/** Optional: Username to display */
	username: z.string().optional(),

	/** Optional: Icon emoji */
	icon_emoji: z.string().optional(),
});

type SendMessageInput = z.infer<typeof SendMessageInputSchema>;

// ============================================================================
// OUTPUT TYPE
// ============================================================================

interface SendMessageOutput {
	ok: boolean;
	channel: string;
	ts: string;
	message: {
		text: string;
		username?: string;
		bot_id: string;
		type: string;
		ts: string;
	};
}

// ============================================================================
// ACTION DEFINITION
// ============================================================================

export const sendMessageAction: ActionDefinition<SendMessageInput, SendMessageOutput> = {
	id: 'slack.send-message',
	name: 'Send Message',
	description: 'Send a message to a Slack channel',
	integration: 'slack',
	requiredAuth: ['slack'],
	inputSchema: SendMessageInputSchema,

	async execute(input, context) {
		const validated = SendMessageInputSchema.parse(input);

		// Get Slack OAuth tokens
		const tokens = await context.oauth.getTokens(context.userId, 'slack');
		if (!tokens) {
			throw new Error('Slack not connected for this user');
		}

		// Build request body
		const body: any = {
			channel: validated.channel,
			text: validated.text,
		};

		if (validated.thread_ts) {
			body.thread_ts = validated.thread_ts;
		}

		if (validated.username) {
			body.username = validated.username;
		}

		if (validated.icon_emoji) {
			body.icon_emoji = validated.icon_emoji;
		}

		// Send message via Slack API
		const response = await fetch('https://slack.com/api/chat.postMessage', {
			method: 'POST',
			headers: {
				Authorization: `Bearer ${tokens.accessToken}`,
				'Content-Type': 'application/json',
			},
			body: JSON.stringify(body),
		});

		if (!response.ok) {
			const errorText = await response.text();
			throw new Error(`Slack API error: ${response.status} ${errorText}`);
		}

		const result = await response.json();

		if (!result.ok) {
			throw new Error(`Slack API error: ${result.error}`);
		}

		return result;
	},
};
```

### 2.3 Create Integration Definition

Create `apps/api/src/integrations/slack/index.ts`:

```typescript
/**
 * Slack Integration
 *
 * Provides actions for interacting with Slack:
 * - Send messages
 * - Create channels
 * - Invite users
 * - And more...
 */

import type { IntegrationDefinition } from '../../sdk/integration-sdk';
import { sendMessageAction } from './actions/send-message';

/**
 * Slack Integration Definition
 */
export const slackIntegration: IntegrationDefinition = {
	id: 'slack',
	name: 'Slack',
	description: 'Slack integration for WORKWAY - send messages, manage channels',
	version: '1.0.0',
	author: 'WORKWAY Team',
	icon: '💬',
	docsUrl: 'https://docs.workway.dev/integrations/slack',

	oauth: {
		provider: 'slack',
		scopes: [
			'chat:write',
			'channels:read',
			'groups:read'
		],
	},

	actions: [sendMessageAction],

	triggers: [],

	async initialize(env) {
		// Optional: Verify Slack API credentials are configured
		if (!env.SLACK_CLIENT_ID || !env.SLACK_CLIENT_SECRET) {
			console.warn('Slack integration: OAuth credentials not configured');
		}
	},
};

export default slackIntegration;
```

---

## Step 3: Register Integration

### 3.1 Add OAuth Configuration

Edit `apps/api/src/routes/oauth.ts`:

```typescript
const OAUTH_CONFIGS: Record<string, OAuthConfig> = {
	gmail: { /* existing */ },
	notion: { /* existing */ },

	// Add Slack
	slack: {
		authUrl: 'https://slack.com/oauth/v2/authorize',
		tokenUrl: 'https://slack.com/api/oauth.v2.access',
		scope: 'chat:write channels:read groups:read',
		clientIdEnvKey: 'SLACK_CLIENT_ID',
		clientSecretEnvKey: 'SLACK_CLIENT_SECRET',
	},
};
```

### 3.2 Register in Registry

Edit `apps/api/src/integrations/registry.ts`:

```typescript
import { ActionRegistry } from '../sdk/integration-sdk';
import gmailIntegration from './gmail';
import notionIntegration from './notion';
import slackIntegration from './slack'; // Add this

export const integrationRegistry = new ActionRegistry();

export function registerAllIntegrations(env?: any): void {
	// Register Gmail integration
	integrationRegistry.registerIntegration(gmailIntegration);

	// Register Notion integration
	integrationRegistry.registerIntegration(notionIntegration);

	// Register Slack integration
	integrationRegistry.registerIntegration(slackIntegration); // Add this

	// Initialize integrations
	if (env) {
		gmailIntegration.initialize?.(env);
		notionIntegration.initialize?.(env);
		slackIntegration.initialize?.(env); // Add this
	}

	console.log(`✅ Registered ${integrationRegistry.getAllIntegrations().length} integrations`);
	console.log(`✅ Registered ${integrationRegistry.getAllActions().length} actions`);
}
```

### 3.3 Add Backward Compatibility (Optional)

If migrating from old format, edit `apps/api/src/durable-objects/WorkflowExecutor.ts`:

```typescript
private convertLegacyStepType(legacyType: string): string {
	const legacyMap: Record<string, string> = {
		// Gmail legacy types
		gmail: 'gmail.send-email',
		gmail_fetch: 'gmail.fetch-email',

		// Notion legacy types
		notion: 'notion.create-page',

		// Slack legacy types (add if needed)
		slack_send: 'slack.send-message',
	};

	return legacyMap[legacyType] || legacyType;
}
```

---

## Step 4: Test & Deploy

### 4.1 Test Locally (Optional)

```bash
npm run dev
```

### 4.2 Check Registry

```bash
curl http://localhost:8787/registry/stats
```

Should show:
```json
{
  "integrations": {
    "total": 3,
    "list": ["gmail", "notion", "slack"]
  },
  "actions": {
    "total": 7,
    "byIntegration": {
      "gmail": 3,
      "notion": 3,
      "slack": 1
    }
  }
}
```

### 4.3 Deploy

```bash
npm run deploy
```

### 4.4 Verify Deployment

```bash
# Check registry
curl https://marketplace-api.half-dozen.workers.dev/registry/stats

# Check Slack integration
curl https://marketplace-api.half-dozen.workers.dev/registry/integrations/slack

# Check send-message action
curl https://marketplace-api.half-dozen.workers.dev/registry/actions/slack.send-message
```

---

## Using the Integration

### In Workflows

```typescript
import { defineWorkflow } from '@workway/sdk/workflow';

export default defineWorkflow({
  metadata: {
    name: 'Gmail to Slack Alerts',
    category: 'productivity',
  },
  pricing: { price: 8 },
  integrations: ['gmail', 'slack'],

  async execute({ trigger, actions }) {
    const email = await actions.execute('gmail.fetch-email', {
      messageId: trigger.data.messageId
    });

    await actions.execute('slack.send-message', {
      channel: '#inbox',
      text: `New email from ${email.from}: ${email.subject}`
    });
  }
});
```

### Manual Workflow Definitions

```json
{
  "steps": [
    {
      "id": "send_slack",
      "name": "Send Slack Message",
      "type": "slack.send-message",
      "config": {
        "channel": "#general",
        "text": "Hello from WORKWAY!"
      }
    }
  ]
}
```

---

## OAuth Flow Testing

### 1. Connect OAuth (via UI)

1. Navigate to `/settings/oauth`
2. Click "Connect Slack"
3. Authorize with Slack
4. Verify tokens are stored

### 2. Test OAuth Connection

```bash
# Get user's OAuth connections
curl https://marketplace-api.half-dozen.workers.dev/oauth/connections \
  -H "Cookie: session=YOUR_SESSION_TOKEN"
```

Should show:
```json
{
  "connections": [
    {
      "provider": "slack",
      "email": "user@example.com",
      "createdAt": "2025-11-12T05:00:00.000Z"
    }
  ]
}
```

---

## Common Integration Patterns

### 1. Action with File Upload

```typescript
export const uploadFileAction: ActionDefinition = {
  id: 'slack.upload-file',
  name: 'Upload File',
  description: 'Upload a file to Slack',
  integration: 'slack',
  requiredAuth: ['slack'],
  inputSchema: z.object({
    channels: z.string(),
    file: z.string(), // Base64 encoded
    filename: z.string(),
  }),

  async execute(input, context) {
    const tokens = await context.oauth.getTokens(context.userId, 'slack');

    const formData = new FormData();
    formData.append('channels', input.channels);
    formData.append('file', Buffer.from(input.file, 'base64'));
    formData.append('filename', input.filename);

    const response = await fetch('https://slack.com/api/files.upload', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${tokens.accessToken}`,
      },
      body: formData,
    });

    return await response.json();
  }
};
```

### 2. Action with Pagination

```typescript
export const listChannelsAction: ActionDefinition = {
  id: 'slack.list-channels',
  name: 'List Channels',
  description: 'List all Slack channels',
  integration: 'slack',
  requiredAuth: ['slack'],
  inputSchema: z.object({
    limit: z.number().default(100),
    cursor: z.string().optional(),
  }),

  async execute(input, context) {
    const tokens = await context.oauth.getTokens(context.userId, 'slack');

    const url = new URL('https://slack.com/api/conversations.list');
    url.searchParams.set('limit', input.limit.toString());
    if (input.cursor) {
      url.searchParams.set('cursor', input.cursor);
    }

    const response = await fetch(url, {
      headers: {
        Authorization: `Bearer ${tokens.accessToken}`,
      },
    });

    return await response.json();
  }
};
```

### 3. Action with Webhook Setup (Trigger)

```typescript
export const newMessageTrigger: TriggerDefinition = {
  id: 'slack.new-message',
  name: 'New Message Posted',
  description: 'Triggers when a new message is posted to a channel',
  type: 'webhook',
  integration: 'slack',
  configSchema: z.object({
    channel: z.string(),
  }),
  requiredAuth: ['slack'],

  async setup(config, context) {
    const tokens = await context.oauth.getTokens(context.userId, 'slack');

    // Register webhook with Slack
    const response = await fetch('https://slack.com/api/apps.event.subscriptions.create', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${tokens.accessToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        url: `${context.env.API_URL}/webhooks/slack/${context.userId}`,
        event_types: ['message.channels'],
      }),
    });

    // Store webhook ID for cleanup
    await context.storage.put('slack:webhook_id', response.webhook_id);
  },

  async cleanup(config, context) {
    const webhookId = await context.storage.get('slack:webhook_id');
    const tokens = await context.oauth.getTokens(context.userId, 'slack');

    // Unregister webhook
    await fetch(`https://slack.com/api/apps.event.subscriptions.delete/${webhookId}`, {
      method: 'DELETE',
      headers: {
        Authorization: `Bearer ${tokens.accessToken}`,
      },
    });
  }
};
```

---

## Checklist

Before deploying your integration:

- [ ] OAuth app created and configured
- [ ] Client ID and Secret added to `.dev.vars` and Wrangler secrets
- [ ] Integration directory created (`integrations/{name}/`)
- [ ] Actions defined with Zod schemas
- [ ] Integration definition created (`index.ts`)
- [ ] OAuth config added to `oauth.ts`
- [ ] Integration registered in `registry.ts`
- [ ] Tested locally (registry endpoints)
- [ ] Deployed to production
- [ ] OAuth flow tested (connect → use → disconnect)
- [ ] Action execution tested in workflow

---

## Troubleshooting

### "Integration not found in registry"

Check:
1. Integration is imported in `registry.ts`
2. `registerAllIntegrations()` is called in `index.ts`
3. Integration `id` matches what you're looking up

### "OAuth tokens not found"

Check:
1. User has connected OAuth (`/settings/oauth`)
2. OAuth provider name matches integration `oauth.provider`
3. Tokens are not expired (check `expires_at`)

### "Action schema validation failed"

Check:
1. Input matches Zod schema definition
2. Required fields are provided
3. Types are correct (string vs number, etc.)

---

## Next Steps

- Add more actions to your integration
- Create triggers for event-driven workflows
- Write tests for actions
- Document integration in user docs
- Create example workflows using your integration

---

## Resources

- [Integration SDK Reference](./PLUGIN_ARCHITECTURE.md)
- [Workflow SDK Reference](./PLUGIN_ARCHITECTURE.md#layer-2-workflow-sdk)
- [OAuth Configuration](./OAUTH_CONFIGURATION.md)
- [Action Registry API](../src/routes/integrations-registry.ts)
