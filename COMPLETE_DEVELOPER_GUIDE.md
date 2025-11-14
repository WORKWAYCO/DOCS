# Complete Developer Guide - Quick Reference

**For developers building on WORKWAY**

---

## Table of Contents

1. [Developer Types](#developer-types)
2. [Quick Start](#quick-start)
3. [Integration Developer Path](#integration-developer-path)
4. [Workflow Developer Path](#workflow-developer-path)
5. [Registry System](#registry-system)
6. [Testing](#testing)
7. [Submission & Publishing](#submission--publishing)
8. [Revenue & Payouts](#revenue--payouts)

---

## Developer Types

### TYPE 1: Integration Developer
**Build:** Service connectors (Gmail, Slack, Stripe)
**Skills:** OAuth, APIs, TypeScript
**Approval:** Required (security review)
**Revenue:** Salary/contractor payment
**Access:** GitHub repo access needed

### TYPE 2: Workflow Developer
**Build:** Automation workflows (compose integrations)
**Skills:** Logic, workflows, no code/low code
**Approval:** Automatic (if using approved integrations)
**Revenue:** 70% of user subscriptions
**Access:** Self-service signup

---

## Quick Start

### Integration Developer

```bash
# 1. Apply & get approved
# Visit: https://workway.dev/developers/integrations/apply

# 2. Clone repo
git clone git@github.com:workway/workway.git
cd workway/apps/api

# 3. Create integration
mkdir -p src/integrations/slack/actions
# (See full guide for details)

# 4. Test locally
npm run dev
curl http://localhost:8787/registry/integrations

# 5. Submit PR
git checkout -b integration/slack
git push origin integration/slack
# Create PR on GitHub

# 6. Wait for review & approval
# Typical timeline: 2-3 business days

# 7. Merged → Auto-deployed!
```

### Workflow Developer

```bash
# 1. Sign up (instant)
# Visit: https://workway.dev/developers/signup

# 2. Option A: Use web builder (easiest)
# Go to: https://workway.dev/developer/builder

# 2. Option B: Use CLI (advanced)
npm install -g @workway/cli
workway login
workway workflow init my-workflow

# 3. Test
workway workflow test --mock
workway workflow test --live

# 4. Publish
workway workflow publish

# 5. Start earning! 💰
```

---

## Integration Developer Path

### Full Process

```
Apply → Onboarding → Build → Test → Submit → Review → Deploy
  ↓         ↓          ↓       ↓       ↓        ↓       ↓
3-5 days   1 day    2-3 days 1 day  instant  2-3 days instant
```

### File Structure

```
apps/api/src/integrations/slack/
├── index.ts                  # Integration definition
├── actions/
│   ├── send-message.ts       # Action: Send message
│   └── list-channels.ts      # Action: List channels
├── triggers/
│   └── new-message.ts        # Trigger: New message (optional)
└── __tests__/
    ├── send-message.test.ts  # Unit tests (required)
    └── list-channels.test.ts
```

### Code Template

```typescript
// src/integrations/slack/actions/send-message.ts
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

    // Get OAuth tokens
    const tokens = await context.oauth.getTokens(context.userId, 'slack');
    if (!tokens) throw new Error('Slack not connected');

    // Call Slack API
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
```

### Testing

```bash
# Unit tests
npm test src/integrations/slack

# Integration test
npm run dev
curl -X POST http://localhost:8787/workflows/test \
  -H "Content-Type: application/json" \
  -d '{"steps":[{"type":"slack.send-message","config":{"channel":"#test","text":"Hello!"}}]}'

# Check Slack for message
```

### Submission

```bash
git checkout -b integration/slack
git add src/integrations/slack
git commit -m "feat: Add Slack integration"
git push origin integration/slack

# Create PR on GitHub
# Fill out PR template
# Wait for CI checks to pass
# Wait for code review
```

---

## Workflow Developer Path

### Full Process

```
Sign Up → Build → Test → Publish → Earn
   ↓        ↓       ↓       ↓        ↓
instant  30 min  10 min  instant  ongoing
```

### Option A: Web Builder (No Code)

**1. Create Workflow**
```
Name: Gmail to Slack Alerts
Category: Productivity
Icon: 📧
Price: $8/month
Trial: 7 days
```

**2. Configure Trigger**
```
Trigger: gmail.new-email
Test data: Load sample
```

**3. Add Steps**
```
Step 1: gmail.fetch-email
  messageId: {{trigger.messageId}}

Step 2: slack.send-message
  channel: #inbox
  text: New email from {{steps.fetch_email.from}}: {{steps.fetch_email.subject}}
```

**4. Test**
```
[Test with Sample Data] ✅
[Test with Live OAuth] ✅
```

**5. Publish**
```
[Publish to Marketplace]
→ Automatic approval ✅
→ Live instantly! 🎉
```

### Option B: CLI (Code)

```bash
# Install CLI
npm install -g @workway/cli
workway login

# Create workflow
workway workflow init gmail-to-slack
cd gmail-to-slack

# Edit workflow.ts
# (See template below)

# Test
workway workflow test --mock
workway workflow test --live

# Publish
workway workflow publish
```

### Code Template

```typescript
// workflow.ts
import { defineWorkflow } from '@workway/sdk';
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
      schema: z.string(),
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

---

## Registry System

### How Registry Works

**Key Concept:** Registry is **code-driven**, not database-driven.

```typescript
// At Worker startup (index.ts)
registerAllIntegrations();

// This function loads all integrations from code
export function registerAllIntegrations() {
  integrationRegistry.registerIntegration(gmailIntegration);
  integrationRegistry.registerIntegration(notionIntegration);
  integrationRegistry.registerIntegration(slackIntegration); // New!
}

// No database updates needed!
// As soon as code is deployed, integration is available ✅
```

### Registry Update Process

```
Developer merges integration PR
  ↓
GitHub Actions triggers deployment
  ↓
Worker builds with new integration code
  ↓
Worker starts → registerAllIntegrations() runs
  ↓
Registry populated with new integration
  ↓
Available immediately via API
```

### Check Registry

```bash
# List all integrations
curl https://marketplace-api.half-dozen.workers.dev/registry/integrations

# Check specific integration
curl https://marketplace-api.half-dozen.workers.dev/registry/integrations/slack

# List all actions
curl https://marketplace-api.half-dozen.workers.dev/registry/actions

# Check specific action
curl https://marketplace-api.half-dozen.workers.dev/registry/actions/slack.send-message

# Get stats
curl https://marketplace-api.half-dozen.workers.dev/registry/stats
```

---

## Testing

### Integration Testing Checklist

- [ ] OAuth flow works (authorize → callback → store tokens)
- [ ] Action executes successfully with valid input
- [ ] Action throws error on invalid input
- [ ] Action handles API errors gracefully
- [ ] Action handles rate limits
- [ ] Action handles expired tokens
- [ ] Unit tests written (>80% coverage)
- [ ] Integration registered in registry
- [ ] Registry API returns integration
- [ ] No secrets in code

### Workflow Testing Checklist

- [ ] Works with sample data
- [ ] Works with live OAuth
- [ ] All required integrations connected
- [ ] Config fields validated
- [ ] Template variables resolved correctly
- [ ] Error handling works
- [ ] Preview listing looks good
- [ ] Screenshots uploaded
- [ ] Pricing configured
- [ ] Test purchase/install flow

### Commands

```bash
# Integration: Run unit tests
npm test src/integrations/slack

# Integration: Test locally
npm run dev
curl http://localhost:8787/registry/integrations

# Workflow: Test with mock data
workway workflow test --data test.json --mock

# Workflow: Test with live OAuth
workway workflow test --live

# Workflow: Preview listing
workway workflow publish --draft
```

---

## Submission & Publishing

### Integration Submission

**1. Create PR**
```bash
git checkout -b integration/slack
git push origin integration/slack
# Create PR on GitHub
```

**2. PR Template (auto-filled)**
```markdown
## Integration Submission: Slack

### Checklist
- [x] OAuth configured
- [x] Actions defined
- [x] Unit tests (100% coverage)
- [x] Local testing complete
- [x] Documentation added
- [x] No secrets committed

### Test Results
✅ Unit tests: 5/5 passed
✅ OAuth flow: Working
✅ Action execution: Working
```

**3. Automated Checks**
- Security scan
- Secret detection
- Unit tests
- Build verification
- Type checking

**4. Manual Review**
- Code quality
- Security audit
- OAuth review
- Error handling
- Documentation

**5. Approval & Deploy**
- PR merged → Auto-deploy
- Registry updates automatically
- Integration available immediately

### Workflow Publishing

**1. Test thoroughly**
```bash
workway workflow test --mock
workway workflow test --live
```

**2. Publish**
```bash
# Via CLI
workway workflow publish

# Or via web builder
[Publish to Marketplace] button
```

**3. Automatic Approval** (if using approved integrations)
- Validation runs
- Tests pass
- Metadata complete
- Instantly live! 🎉

**4. Manual Review** (if using custom code)
- Code review by WORKWAY team
- 1-2 business days
- Feedback or approval

---

## Revenue & Payouts

### Workflow Developer Revenue

**Revenue Split:**
- User pays: $8/month
- You earn: $5.60 (70%)
- Platform: $2.40 (30%)

**Payout Schedule:**
- Monthly on the 15th
- Via Stripe Connect
- Minimum balance: $25

**Track Earnings:**
```bash
# Via dashboard
https://workway.dev/developer/dashboard

# Via CLI
workway earnings

# Output:
# Month      Installs  Revenue   Payout
# Nov 2025   47        $263.20   $184.24
# Oct 2025   39        $218.40   $152.88
# Sep 2025   31        $173.60   $121.52
```

### Integration Developer Compensation

**Payment Model:**
- Salary (if employee)
- Contractor rate (if freelance)
- One-time bounty (for specific integrations)

**Not paid per usage** - integrations are platform infrastructure

---

## Support & Resources

### Documentation

- **Architecture:** `docs/PLUGIN_ARCHITECTURE.md`
- **Adding Integrations:** `docs/ADDING_INTEGRATIONS.md`
- **Submission Process:** `docs/DEVELOPER_SUBMISSION_PROCESS.md`
- **Testing Guide:** `docs/DEVELOPER_TESTING_GUIDE.md`
- **This Quick Reference:** `docs/COMPLETE_DEVELOPER_GUIDE.md`

### API Endpoints

**Base URL:** `https://marketplace-api.half-dozen.workers.dev`

**Registry API:**
- `GET /registry/stats` - Registry statistics
- `GET /registry/integrations` - List integrations
- `GET /registry/integrations/:id` - Integration details
- `GET /registry/actions` - List actions
- `GET /registry/actions/:id` - Action details

**Developer API:**
- `POST /developers/register` - Sign up
- `GET /developers/me` - Profile
- `GET /developers/earnings` - Earnings
- `POST /workflows/publish` - Publish workflow

### Community

- **GitHub:** https://github.com/workway/workway
- **Discord:** https://discord.gg/workway
- **Community:** https://community.workway.dev
- **Email:** developers@workway.dev

### Getting Help

**Integration Developers:**
- Slack: `#integration-developers`
- Email: integrations@workway.dev
- Docs: https://docs.workway.dev/integrations

**Workflow Developers:**
- Dashboard: https://workway.dev/developer/dashboard
- Community: https://community.workway.dev
- Support: support@workway.dev

---

## Next Steps

### For Integration Developers

1. **Apply for access** → https://workway.dev/developers/integrations/apply
2. **Read full guide** → `docs/ADDING_INTEGRATIONS.md`
3. **Clone repo** → `git clone git@github.com:workway/workway.git`
4. **Build integration** → Follow templates
5. **Submit PR** → Wait for review

### For Workflow Developers

1. **Sign up** → https://workway.dev/developers/signup
2. **Try web builder** → https://workway.dev/developer/builder
3. **Or install CLI** → `npm install -g @workway/cli`
4. **Build workflow** → Compose integrations
5. **Publish** → Start earning

---

## FAQ

**Q: How long does integration approval take?**
A: 2-3 business days for code review + security audit.

**Q: Can I build workflows without coding?**
A: Yes! Use the web builder at https://workway.dev/developer/builder

**Q: How do I test OAuth locally?**
A: Add OAuth app credentials to `.dev.vars`, run `npm run dev`, test flow.

**Q: What if my integration is rejected?**
A: You'll get feedback on what to fix. Address comments and resubmit.

**Q: How do workflow payouts work?**
A: Monthly via Stripe Connect. You get 70% of subscription revenue.

**Q: Can I use external APIs in workflows?**
A: Yes, but requires manual review for security.

**Q: How do I update a published workflow?**
A: Make changes, test, run `workway workflow publish` again. Auto-approved if no new integrations.

**Q: What happens if a user uninstalls my workflow?**
A: You stop earning from that user. Focus on retention!

---

**Ready to build?** Pick your path and start creating! 🚀
