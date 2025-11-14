# Developer Testing Guide

This guide shows how Integration and Workflow developers test their code before submission.

---

## Integration Developer Testing

### 1. Local Development Setup

```bash
# Clone repo
git clone git@github.com:workway/workway.git
cd workway/apps/api

# Install dependencies
npm install

# Set up environment
cp .dev.vars.example .dev.vars

# Add your OAuth credentials
echo "SLACK_CLIENT_ID=xoxb-..." >> .dev.vars
echo "SLACK_CLIENT_SECRET=..." >> .dev.vars
```

### 2. Create Test Integration

```bash
# Create directory
mkdir -p src/integrations/slack/actions

# Create action
cat > src/integrations/slack/actions/send-message.ts << 'EOF'
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

    const response = await fetch('https://slack.com/api/chat.postMessage', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${tokens.accessToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ channel, text }),
    });

    const result = await response.json();
    if (!result.ok) throw new Error(`Slack error: ${result.error}`);
    return result;
  },
};
EOF
```

### 3. Register Integration

```typescript
// Edit src/integrations/registry.ts
import slackIntegration from './slack';

export function registerAllIntegrations(env?: any): void {
  integrationRegistry.registerIntegration(gmailIntegration);
  integrationRegistry.registerIntegration(notionIntegration);
  integrationRegistry.registerIntegration(slackIntegration); // Add
}
```

### 4. Run Local Server

```bash
# Start dev server
npm run dev

# Output:
# ⛅️ wrangler 4.46.0
# ⬣ Your worker has access to the following bindings:
# Starting local server...
# [wrangler:inf] Ready on http://localhost:8787
# ✅ Registered 3 integrations
# ✅ Registered 7 actions
```

### 5. Test Registry API

```bash
# Check integration registered
curl http://localhost:8787/registry/integrations | jq '.integrations[] | select(.id == "slack")'

# Output:
# {
#   "id": "slack",
#   "name": "Slack",
#   "description": "Slack integration for WORKWAY",
#   "version": "1.0.0",
#   "actionsCount": 1,
#   "oauth": {
#     "provider": "slack",
#     "scopesCount": 1
#   }
# }

# Check action registered
curl http://localhost:8787/registry/actions/slack.send-message | jq .

# Output:
# {
#   "id": "slack.send-message",
#   "name": "Send Message",
#   "description": "Send a message to a Slack channel",
#   "integration": "slack",
#   "requiredAuth": ["slack"],
#   "hasExecuteFunction": true,
#   "hasInputSchema": true
# }
```

### 6. Test OAuth Flow

**Option A: Via API**

```bash
# 1. Start OAuth flow
curl "http://localhost:8787/oauth/slack/authorize?redirectUri=http://localhost:8787/oauth/callback"

# You'll be redirected to Slack
# After authorizing, you'll be redirected back with code

# 2. Check tokens in database
npx wrangler d1 execute marketplace-development --local \
  --command "SELECT provider, user_id FROM oauth_credentials WHERE provider = 'slack'"
```

**Option B: Via Test Script**

```javascript
// test-oauth.mjs
async function testSlackOAuth() {
  const userId = 'test-user-123';

  // Simulate OAuth flow
  const oauthId = env.OAUTH.idFromName('oauth-manager');
  const oauthStub = env.OAUTH.get(oauthId);

  // Store test tokens
  await oauthStub.storeTokens(userId, 'slack', {
    accessToken: 'xoxb-test-token',
    expiresAt: Date.now() + 3600000,
    scope: 'chat:write',
  });

  // Verify stored
  const tokens = await oauthStub.getTokens(userId, 'slack');
  console.log('✅ Tokens stored:', tokens);
}
```

### 7. Test Action Execution

**Create Test Workflow:**

```json
{
  "id": "test-slack-send",
  "name": "Test Slack Send",
  "trigger": { "type": "manual" },
  "steps": [
    {
      "id": "send_test",
      "name": "Send Test Message",
      "type": "slack.send-message",
      "config": {
        "channel": "#test",
        "text": "Hello from WORKWAY test!"
      }
    }
  ],
  "config": {}
}
```

**Trigger Workflow:**

```bash
# Via API
curl -X POST http://localhost:8787/workflows/trigger \
  -H "Content-Type: application/json" \
  -d @test-workflow.json

# Or use Wrangler
npx wrangler dev --test
```

**Check Logs:**

```bash
# Watch logs
npx wrangler tail --format pretty

# Should see:
# [Backward Compatibility] Converted slack_send → slack.send-message
# Executing action: slack.send-message from integration: slack
# ✅ Workflow completed successfully
```

### 8. Write Unit Tests

```typescript
// src/integrations/slack/__tests__/send-message.test.ts
import { describe, it, expect, vi } from 'vitest';
import { sendMessageAction } from '../actions/send-message';

describe('slack.send-message', () => {
  it('should send message successfully', async () => {
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
      json: async () => ({ ok: true, ts: '1234567890.123' }),
    });

    const result = await sendMessageAction.execute(
      { channel: '#test', text: 'Hello!' },
      mockContext as any
    );

    expect(result.ok).toBe(true);
  });

  it('should throw on invalid input', async () => {
    await expect(
      sendMessageAction.execute(
        { channel: '', text: '' }, // Invalid
        {} as any
      )
    ).rejects.toThrow();
  });

  it('should throw on Slack error', async () => {
    const mockContext = {
      userId: 'test-user',
      oauth: {
        getTokens: vi.fn().mockResolvedValue({ accessToken: 'test-token' }),
      },
    };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ ok: false, error: 'channel_not_found' }),
    });

    await expect(
      sendMessageAction.execute(
        { channel: '#invalid', text: 'Hi' },
        mockContext as any
      )
    ).rejects.toThrow('Slack error: channel_not_found');
  });
});
```

**Run Tests:**

```bash
npm test src/integrations/slack

# Output:
# PASS  src/integrations/slack/__tests__/send-message.test.ts
#   slack.send-message
#     ✓ should send message successfully (5 ms)
#     ✓ should throw on invalid input (2 ms)
#     ✓ should throw on Slack error (3 ms)
#
# Test Suites: 1 passed, 1 total
# Tests:       3 passed, 3 total
```

### 9. Integration Testing

**Test End-to-End Flow:**

```bash
# 1. Start local server
npm run dev &

# 2. Connect OAuth
curl "http://localhost:8787/oauth/slack/authorize"
# Complete OAuth flow in browser

# 3. Trigger workflow
curl -X POST http://localhost:8787/workflows/test-slack \
  -H "Cookie: session=test-session"

# 4. Check Slack for message
# Message should appear in #test channel

# 5. Check database for workflow run
npx wrangler d1 execute marketplace-development --local \
  --command "SELECT id, status, error FROM workflow_runs ORDER BY started_at DESC LIMIT 1"

# Output:
# {
#   "id": "run_...",
#   "status": "completed",
#   "error": null
# }
```

---

## Workflow Developer Testing

### 1. Web Builder Testing (No Code)

**Step 1: Create Workflow**
1. Go to https://workway.dev/developer/builder
2. Click "New Workflow"
3. Configure:
   - Name: "Gmail to Slack Alerts"
   - Category: Productivity
   - Price: $8/month

**Step 2: Add Trigger**
1. Select: `gmail.new-email`
2. Load sample data: `{ "messageId": "123", "from": "test@example.com" }`

**Step 3: Add Steps**
1. Add step: `gmail.fetch-email`
   - messageId: `{{trigger.messageId}}`

2. Add step: `slack.send-message`
   - channel: `#inbox`
   - text: `New email from {{steps.fetch_email.from}}: {{steps.fetch_email.subject}}`

**Step 4: Test with Sample Data**
```
[Test Run]

✅ Step 1: gmail.fetch-email completed
   Output: { from: "test@example.com", subject: "Test Email" }

✅ Step 2: slack.send-message completed
   Output: { ok: true, ts: "1234567890.123" }

✅ Workflow completed in 1.2s
```

**Step 5: Test with Live OAuth**
```
[Connect OAuth Accounts]
- Gmail: [Connected ✓]
- Slack: [Connected ✓]

[Test with Live Data]

✅ Gmail email fetched
✅ Slack message sent
✅ Check Slack for message

Message in Slack:
"New email from customer@example.com: Invoice Payment Received"
```

### 2. CLI Testing (Code)

**Install CLI:**
```bash
npm install -g @workway/cli
workway login
```

**Create Workflow:**
```bash
workway workflow init gmail-to-slack
cd gmail-to-slack
```

**Edit workflow:**
```typescript
// workflow.ts
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

**Test with Mock Data:**
```bash
# Create test data
cat > test-data.json << EOF
{
  "trigger": {
    "type": "gmail.new-email",
    "data": {
      "messageId": "msg_123"
    }
  }
}
EOF

# Run test
workway workflow test --data test-data.json --mock

# Output:
# 🧪 Testing workflow: Gmail to Slack Alerts
#
# ✅ gmail.fetch-email (mocked)
#    Output: { from: "test@example.com", subject: "Test" }
#
# ✅ slack.send-message (mocked)
#    Output: { ok: true }
#
# ✅ Workflow completed in 0.1s
```

**Test with Live OAuth:**
```bash
# Connect accounts
workway oauth connect gmail
workway oauth connect slack

# Test with live data
workway workflow test --live

# Output:
# 🧪 Testing workflow: Gmail to Slack Alerts
#
# 🔗 Using live OAuth tokens
#
# ✅ gmail.fetch-email
#    Fetched email from customer@example.com
#
# ✅ slack.send-message
#    Sent to #inbox
#    https://workspace.slack.com/archives/C123/p1234567890
#
# ✅ Workflow completed in 1.3s
```

### 3. Preview in Marketplace

**Publish as Draft:**
```bash
workway workflow publish --draft

# Output:
# 📦 Building workflow...
# ✅ Validated
# ✅ Published as draft
#
# Preview URL:
# https://workway.dev/marketplace/gmail-to-slack?preview=YOUR_TOKEN
```

**Share Preview:**
- Send preview URL to team/beta testers
- They can see listing (but can't install yet)
- Gather feedback before public launch

### 4. Load Testing (Optional)

**Test High Volume:**
```bash
# Install load test tool
npm install -g autocannon

# Test workflow endpoint
autocannon -c 100 -d 30 \
  https://workway.dev/workflows/gmail-to-slack/trigger \
  -m POST \
  -H "Content-Type: application/json" \
  -b '{"messageId":"test"}'

# Output:
# Running 30s test @ https://...
# 100 connections
#
# Stat         Avg     Stdev   Max
# Latency      45ms    12ms    120ms
# Req/Sec      2200    150     2500
# Bytes/Sec    1.2MB   80KB    1.5MB
#
# 66k requests in 30s, 36MB read
```

---

## Common Testing Patterns

### 1. Testing OAuth Token Refresh

```typescript
it('should refresh expired token', async () => {
  const mockContext = {
    userId: 'test-user',
    oauth: {
      getTokens: vi.fn()
        .mockResolvedValueOnce(null) // First call: no tokens
        .mockResolvedValueOnce({     // After refresh
          accessToken: 'new-token',
        }),
      refreshTokens: vi.fn().mockResolvedValue({
        accessToken: 'new-token',
      }),
    },
  };

  // Should handle missing tokens
  await expect(
    sendMessageAction.execute(input, mockContext as any)
  ).rejects.toThrow('Slack not connected');
});
```

### 2. Testing Rate Limiting

```typescript
it('should handle rate limits', async () => {
  global.fetch = vi.fn().mockResolvedValue({
    ok: false,
    status: 429,
    json: async () => ({ error: 'rate_limited' }),
  });

  await expect(
    sendMessageAction.execute(input, mockContext as any)
  ).rejects.toThrow('rate_limited');
});
```

### 3. Testing Template Variables

```typescript
it('should resolve template variables', async () => {
  const workflow = {
    steps: [{
      type: 'slack.send-message',
      config: {
        channel: '#test',
        text: 'Email from {{trigger.email.from}}'
      }
    }]
  };

  const context = {
    triggerData: {
      email: { from: 'user@example.com' }
    }
  };

  // After resolution, text should be:
  // "Email from user@example.com"
});
```

---

## Troubleshooting

### "Integration not found in registry"

**Check:**
```bash
# Is integration imported in registry.ts?
grep -r "slackIntegration" src/integrations/registry.ts

# Is server restarted?
npm run dev

# Check registry endpoint
curl http://localhost:8787/registry/integrations | jq '.integrations[].id'
```

### "OAuth tokens not found"

**Check:**
```bash
# Is OAuth flow completed?
npx wrangler d1 execute marketplace-development --local \
  --command "SELECT * FROM oauth_credentials WHERE provider = 'slack'"

# Are tokens expired?
# Check expires_at timestamp

# Re-connect OAuth
curl http://localhost:8787/oauth/slack/authorize
```

### "Action execution failed"

**Check:**
```bash
# View logs
npx wrangler tail --format pretty

# Check for:
# - OAuth token errors
# - API rate limits
# - Invalid input data
# - Network issues
```

### "Tests failing"

**Common issues:**
```typescript
// ❌ Wrong: Not mocking fetch
await sendMessageAction.execute(input, context);

// ✅ Right: Mock fetch first
global.fetch = vi.fn().mockResolvedValue({...});
await sendMessageAction.execute(input, context);

// ❌ Wrong: Missing OAuth tokens
const context = { userId: 'test' };

// ✅ Right: Mock OAuth
const context = {
  userId: 'test',
  oauth: {
    getTokens: vi.fn().mockResolvedValue({
      accessToken: 'test-token'
    })
  }
};
```

---

## Testing Checklist

### Before Submitting Integration

- [ ] Unit tests written (>80% coverage)
- [ ] Integration tests passed
- [ ] OAuth flow tested (authorize → use → refresh)
- [ ] Action execution tested with real API
- [ ] Error cases tested (rate limits, invalid input)
- [ ] Registry loading verified
- [ ] No secrets in code
- [ ] Documentation complete

### Before Publishing Workflow

- [ ] Tested with sample data
- [ ] Tested with live OAuth
- [ ] All integrations connected
- [ ] Config fields tested
- [ ] Error handling verified
- [ ] Preview listing checked
- [ ] Screenshots uploaded
- [ ] Pricing configured

---

## Next Steps

After testing:

**Integration Developers:**
1. Create PR with test results
2. Include test coverage report
3. Document any edge cases
4. Wait for code review

**Workflow Developers:**
1. Publish to marketplace
2. Monitor analytics dashboard
3. Respond to user feedback
4. Iterate and improve

---

**Need Help?**
- Docs: https://docs.workway.dev
- Community: https://community.workway.dev
- Support: support@workway.dev
