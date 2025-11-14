# WORKWAY Workflow Templates

Pre-built templates for common use cases. Copy, customize, and publish!

---

## 📦 Integration Workflows (70% of Marketplace)

### Template 1: Stripe → Notion Invoice Tracker

```typescript
import { defineWorkflow, webhook } from '@workway/sdk'

export default defineWorkflow({
  name: 'Stripe to Notion Invoice Tracker',
  description: 'Automatically create Notion invoice pages when Stripe payments succeed',
  type: 'integration',
  category: 'finance',
  icon: '💰',

  pricing: {
    model: 'subscription',
    price: 5,
    executions: 'unlimited'
  },

  integrations: [
    { service: 'stripe', scopes: ['read_payments'] },
    { service: 'notion', scopes: ['write_pages'] }
  ],

  inputs: {
    notionDatabaseId: {
      type: 'notion_database_picker',
      label: 'Notion Invoices Database',
      required: true
    },
    includeRefunds: {
      type: 'boolean',
      label: 'Track refunds separately?',
      default: false
    }
  },

  trigger: webhook({
    service: 'stripe',
    event: 'payment_intent.succeeded'
  }),

  async execute({ trigger, inputs, integrations }) {
    const payment = trigger.data.object

    await integrations.notion.pages.create({
      parent: { database_id: inputs.notionDatabaseId },
      properties: {
        'Invoice ID': { title: [{ text: { content: payment.id } }] },
        'Amount': { number: payment.amount / 100 },
        'Currency': { select: { name: payment.currency.toUpperCase() } },
        'Customer Email': { email: payment.receipt_email },
        'Date': { date: { start: new Date().toISOString() } },
        'Status': { select: { name: 'Paid' } }
      }
    })

    return { success: true, invoiceId: payment.id }
  }
})
```

**Earnings Potential:** $3.50/mo per user | **Cost:** $0.001/execution | **Margin:** 98%

---

### Template 2: Google Forms → Airtable Submissions

```typescript
import { defineWorkflow, webhook } from '@workway/sdk'

export default defineWorkflow({
  name: 'Google Forms to Airtable',
  description: 'Auto-sync form responses to Airtable in real-time',
  type: 'integration',
  category: 'productivity',
  icon: '📝',

  pricing: {
    model: 'subscription',
    price: 5,
    executions: 'unlimited'
  },

  integrations: [
    { service: 'google_forms', scopes: ['read_responses'] },
    { service: 'airtable', scopes: ['write_records'] }
  ],

  inputs: {
    formId: {
      type: 'google_form_picker',
      label: 'Google Form',
      required: true
    },
    airtableBaseId: {
      type: 'airtable_base_picker',
      label: 'Airtable Base',
      required: true
    },
    airtableTableId: {
      type: 'airtable_table_picker',
      label: 'Airtable Table',
      required: true
    }
  },

  trigger: webhook({
    service: 'google_forms',
    event: 'form_response'
  }),

  async execute({ trigger, inputs, integrations }) {
    const response = trigger.data

    // Map form fields to Airtable fields
    const fields: Record<string, any> = {}

    for (const [question, answer] of Object.entries(response.answers)) {
      fields[question] = answer
    }

    fields['Submitted At'] = new Date().toISOString()
    fields['Response ID'] = response.id

    await integrations.airtable.records.create({
      baseId: inputs.airtableBaseId,
      tableId: inputs.airtableTableId,
      fields
    })

    return { success: true, recordId: response.id }
  }
})
```

**Earnings Potential:** $3.50/mo per user | **Cost:** $0.001/execution | **Margin:** 98%

---

### Template 3: GitHub Issues → Slack Notifications

```typescript
import { defineWorkflow, webhook } from '@workway/sdk'

export default defineWorkflow({
  name: 'GitHub Issues to Slack',
  description: 'Send Slack notifications when GitHub issues are created or updated',
  type: 'integration',
  category: 'development',
  icon: '🔔',

  pricing: {
    model: 'subscription',
    price: 5,
    executions: 'unlimited'
  },

  integrations: [
    { service: 'github', scopes: ['read_issues'] },
    { service: 'slack', scopes: ['send_messages'] }
  ],

  inputs: {
    slackChannel: {
      type: 'slack_channel_picker',
      label: 'Slack Channel',
      required: true
    },
    notifyOnEvents: {
      type: 'select',
      label: 'Notify on',
      options: ['created', 'updated', 'closed', 'all'],
      default: 'all'
    }
  },

  trigger: webhook({
    service: 'github',
    event: 'issues'
  }),

  async execute({ trigger, inputs, integrations }) {
    const issue = trigger.data.issue
    const action = trigger.data.action

    // Filter events
    if (inputs.notifyOnEvents !== 'all' && action !== inputs.notifyOnEvents) {
      return { success: true, skipped: true }
    }

    const emoji = action === 'opened' ? '🆕' : action === 'closed' ? '✅' : '📝'

    await integrations.slack.chat.postMessage({
      channel: inputs.slackChannel,
      text: `${emoji} Issue ${action}: ${issue.title}`,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*${emoji} Issue ${action}*\n*${issue.title}*\n${issue.body?.substring(0, 200) || 'No description'}${issue.body?.length > 200 ? '...' : ''}`
          }
        },
        {
          type: 'context',
          elements: [
            { type: 'mrkdwn', text: `Created by: ${issue.user.login}` },
            { type: 'mrkdwn', text: `Labels: ${issue.labels.map((l: any) => l.name).join(', ') || 'None'}` }
          ]
        },
        {
          type: 'actions',
          elements: [
            {
              type: 'button',
              text: { type: 'plain_text', text: 'View Issue' },
              url: issue.html_url
            }
          ]
        }
      ]
    })

    return { success: true, issueNumber: issue.number }
  }
})
```

**Earnings Potential:** $3.50/mo per user | **Cost:** $0.002/execution | **Margin:** 97%

---

### Template 4: Airtable → Google Calendar Sync

```typescript
import { defineWorkflow, schedule } from '@workway/sdk'

export default defineWorkflow({
  name: 'Airtable to Google Calendar Sync',
  description: 'Keep your Airtable events synced with Google Calendar automatically',
  type: 'integration',
  category: 'productivity',
  icon: '📅',

  pricing: {
    model: 'subscription',
    price: 5,
    executions: 'unlimited'
  },

  integrations: [
    { service: 'airtable', scopes: ['read_records'] },
    { service: 'google_calendar', scopes: ['write_events'] }
  ],

  inputs: {
    airtableBaseId: {
      type: 'airtable_base_picker',
      label: 'Airtable Base',
      required: true
    },
    airtableTableId: {
      type: 'airtable_table_picker',
      label: 'Airtable Table',
      required: true
    },
    calendarId: {
      type: 'google_calendar_picker',
      label: 'Google Calendar',
      required: true
    },
    dateField: {
      type: 'text',
      label: 'Date field name in Airtable',
      default: 'Date',
      required: true
    },
    titleField: {
      type: 'text',
      label: 'Title field name in Airtable',
      default: 'Title',
      required: true
    }
  },

  trigger: schedule('*/15 * * * *'), // Every 15 minutes

  async execute({ inputs, integrations, state }) {
    const lastSync = await state.get('lastSync') || new Date(Date.now() - 24 * 60 * 60 * 1000)

    const records = await integrations.airtable.records.list({
      baseId: inputs.airtableBaseId,
      tableId: inputs.airtableTableId,
      filterByFormula: `IS_AFTER({Modified}, '${lastSync.toISOString()}')`
    })

    const synced = []

    for (const record of records) {
      const eventDate = record.fields[inputs.dateField]
      const eventTitle = record.fields[inputs.titleField]

      if (!eventDate || !eventTitle) continue

      const existingEventId = await state.get(`record_${record.id}`)

      if (existingEventId) {
        // Update existing event
        await integrations.google_calendar.events.update({
          calendarId: inputs.calendarId,
          eventId: existingEventId,
          summary: eventTitle,
          start: { dateTime: new Date(eventDate).toISOString() },
          end: { dateTime: new Date(new Date(eventDate).getTime() + 60 * 60 * 1000).toISOString() }
        })
      } else {
        // Create new event
        const event = await integrations.google_calendar.events.create({
          calendarId: inputs.calendarId,
          summary: eventTitle,
          start: { dateTime: new Date(eventDate).toISOString() },
          end: { dateTime: new Date(new Date(eventDate).getTime() + 60 * 60 * 1000).toISOString() }
        })

        await state.set(`record_${record.id}`, event.id)
      }

      synced.push(record.id)
    }

    await state.set('lastSync', new Date())

    return { success: true, synced: synced.length, total: records.length }
  }
})
```

**Earnings Potential:** $3.50/mo per user | **Cost:** $0.001/execution | **Margin:** 98%

---

## 🤖 AI-Enhanced Workflows (20% of Marketplace)

### Template 5: Smart Support Ticket Router

```typescript
import { defineWorkflow, webhook, ai } from '@workway/sdk'

export default defineWorkflow({
  name: 'Smart Support Ticket Router',
  description: 'AI categorizes support tickets and routes to the right team',
  type: 'ai-enhanced',
  category: 'customer-support',
  icon: '🎫',

  pricing: {
    model: 'subscription',
    price: 15,
    executions: 200
  },

  integrations: [
    { service: 'zendesk', scopes: ['read_tickets', 'update_tickets'] },
    { service: 'slack', scopes: ['send_messages'] }
  ],

  inputs: {
    teams: {
      type: 'array',
      label: 'Support Teams',
      items: {
        name: 'string',
        slackChannel: 'string'
      },
      default: [
        { name: 'Billing', slackChannel: '#billing' },
        { name: 'Technical', slackChannel: '#tech-support' },
        { name: 'Sales', slackChannel: '#sales' }
      ]
    }
  },

  trigger: webhook({
    service: 'zendesk',
    event: 'ticket.created'
  }),

  async execute({ trigger, inputs, integrations }) {
    const ticket = trigger.data.ticket

    // AI categorization
    const category = await ai.completion({
      model: 'claude-haiku', // Cheapest model for simple classification
      systemPrompt: `You are a support ticket classifier. Categories: ${inputs.teams.map(t => t.name).join(', ')}. Respond with just the category name.`,
      prompt: `Ticket: ${ticket.subject}\n\n${ticket.description}`,
      maxTokens: 20,
      cache: false
    })

    const team = inputs.teams.find(t => t.name.toLowerCase() === category.toLowerCase()) || inputs.teams[0]

    // Update Zendesk
    await integrations.zendesk.tickets.update(ticket.id, {
      tags: [team.name.toLowerCase()],
      custom_fields: [{ id: 'category', value: team.name }]
    })

    // Notify team in Slack
    await integrations.slack.chat.postMessage({
      channel: team.slackChannel,
      text: `New ${team.name} ticket from ${ticket.requester.name}`,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*${ticket.subject}*\n${ticket.description.substring(0, 200)}...`
          }
        },
        {
          type: 'actions',
          elements: [
            { type: 'button', text: { type: 'plain_text', text: 'View Ticket' }, url: ticket.url }
          ]
        }
      ]
    })

    return { success: true, category: team.name, ticketId: ticket.id }
  }
})
```

**Earnings Potential:** $8.40/mo per user | **Cost:** $0.008/execution | **Margin:** 85%

---

### Template 6: Email Sentiment Analyzer

```typescript
import { defineWorkflow, webhook, ai } from '@workway/sdk'

export default defineWorkflow({
  name: 'Email Sentiment Analyzer',
  description: 'Analyze customer email sentiment and flag negative messages for priority response',
  type: 'ai-enhanced',
  category: 'customer-support',
  icon: '😊',

  pricing: {
    model: 'subscription',
    price: 12,
    executions: 300
  },

  integrations: [
    { service: 'gmail', scopes: ['read_emails'] },
    { service: 'slack', scopes: ['send_messages'] }
  ],

  inputs: {
    slackChannel: {
      type: 'slack_channel_picker',
      label: 'Alert Channel',
      required: true
    },
    emailLabel: {
      type: 'text',
      label: 'Gmail label to monitor',
      default: 'customer-support',
      required: true
    }
  },

  trigger: webhook({
    service: 'gmail',
    event: 'message.received'
  }),

  async execute({ trigger, inputs, integrations }) {
    const email = trigger.data

    // Check if email has the label
    if (!email.labels?.includes(inputs.emailLabel)) {
      return { success: true, skipped: true }
    }

    // AI sentiment analysis
    const sentiment = await ai.completion({
      model: 'claude-haiku',
      systemPrompt: 'Analyze email sentiment. Respond with ONLY: positive, negative, or neutral',
      prompt: `Subject: ${email.subject}\n\nBody: ${email.body}`,
      maxTokens: 10,
      cache: false
    })

    // If negative, alert team
    if (sentiment.toLowerCase().includes('negative')) {
      await integrations.slack.chat.postMessage({
        channel: inputs.slackChannel,
        text: `⚠️ Negative customer email detected`,
        blocks: [
          {
            type: 'section',
            text: {
              type: 'mrkdwn',
              text: `*⚠️ Negative Customer Email*\n*From:* ${email.from}\n*Subject:* ${email.subject}\n*Sentiment:* Negative`
            }
          },
          {
            type: 'actions',
            elements: [
              { type: 'button', text: { type: 'plain_text', text: 'View Email' }, url: email.url }
            ]
          }
        ]
      })
    }

    return {
      success: true,
      sentiment,
      alerted: sentiment.toLowerCase().includes('negative')
    }
  }
})
```

**Earnings Potential:** $6.30/mo per user | **Cost:** $0.005/execution | **Margin:** 88%

---

## 🧠 AI-Native Workflows (10% of Marketplace)

### Template 7: Daily AI Newsletter Generator

```typescript
import { defineWorkflow, schedule, ai, http } from '@workway/sdk'

export default defineWorkflow({
  name: 'Daily AI Newsletter',
  description: 'Generate personalized newsletters with AI based on your interests',
  type: 'ai-native',
  category: 'content-generation',
  icon: '📰',

  pricing: {
    model: 'subscription',
    price: 25,
    executions: 30
  },

  inputs: {
    topics: {
      type: 'array',
      label: 'Topics to cover',
      itemType: 'text',
      required: true,
      default: ['AI', 'startups', 'technology']
    },
    email: {
      type: 'email',
      label: 'Your email',
      required: true
    },
    tone: {
      type: 'select',
      label: 'Writing tone',
      options: ['professional', 'casual', 'technical'],
      default: 'professional'
    }
  },

  trigger: schedule('0 8 * * *'), // Daily at 8am

  async execute({ inputs, cache }) {
    // Step 1: Research (with caching)
    const cacheKey = `research_${inputs.topics.join('_')}_${new Date().toDateString()}`
    let research = await cache.get(cacheKey)

    if (!research) {
      research = await ai.completion({
        model: 'claude-sonnet-4',
        systemPrompt: 'You are a research assistant. Find the latest and most interesting developments.',
        prompt: `Research the latest developments in: ${inputs.topics.join(', ')}. Focus on today's news and significant updates.`,
        maxTokens: 2000,
        routing: 'cost-optimized'
      })

      await cache.set(cacheKey, research, { ttl: 3600 }) // Cache for 1 hour
    }

    // Step 2: Generate newsletter
    const newsletter = await ai.completion({
      model: 'gpt-4o',
      systemPrompt: `You are a newsletter writer. Tone: ${inputs.tone}. Format as HTML email.`,
      prompt: `Write a compelling newsletter based on this research:\n\n${research}\n\nInclude:\n- Engaging subject line\n- Brief intro\n- 3-5 key highlights\n- Links where relevant`,
      maxTokens: 1500,
      routing: 'quality-optimized'
    })

    // Step 3: Send email
    await http.post('https://api.sendgrid.com/v3/mail/send', {
      headers: {
        'Authorization': `Bearer ${process.env.SENDGRID_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: {
        personalizations: [{ to: [{ email: inputs.email }] }],
        from: { email: 'newsletter@workway.com', name: 'WORKWAY Newsletter' },
        subject: `Your Daily ${inputs.topics[0]} Update`,
        content: [{ type: 'text/html', value: newsletter }]
      }
    })

    return {
      success: true,
      preview: newsletter.substring(0, 200) + '...'
    }
  }
})
```

**Earnings Potential:** $14.70/mo per user | **Cost:** $0.15/execution (with caching) | **Margin:** 82%

---

### Template 8: AI Meeting Notes Summarizer

```typescript
import { defineWorkflow, webhook, ai } from '@workway/sdk'

export default defineWorkflow({
  name: 'AI Meeting Notes Summarizer',
  description: 'Automatically summarize meeting recordings and send notes to Notion',
  type: 'ai-native',
  category: 'productivity',
  icon: '🎥',

  pricing: {
    model: 'subscription',
    price: 20,
    executions: 50
  },

  integrations: [
    { service: 'google_meet', scopes: ['read_recordings'] },
    { service: 'notion', scopes: ['write_pages'] }
  ],

  inputs: {
    notionDatabaseId: {
      type: 'notion_database_picker',
      label: 'Meeting Notes Database',
      required: true
    },
    summaryLength: {
      type: 'select',
      label: 'Summary length',
      options: ['brief', 'detailed'],
      default: 'detailed'
    }
  },

  trigger: webhook({
    service: 'google_meet',
    event: 'recording_ready'
  }),

  async execute({ trigger, inputs, integrations }) {
    const recording = trigger.data

    // Get transcript
    const transcript = await integrations.google_meet.getTranscript(recording.id)

    // AI summarization
    const summary = await ai.completion({
      model: 'claude-sonnet-4',
      systemPrompt: `Summarize meeting transcript. Format: ${inputs.summaryLength}. Include key points, decisions made, and next steps.`,
      prompt: transcript,
      maxTokens: inputs.summaryLength === 'brief' ? 500 : 1500,
      routing: 'quality-optimized'
    })

    // Extract action items
    const actionItems = await ai.completion({
      model: 'claude-haiku',
      systemPrompt: 'Extract action items from meeting summary. Return as JSON array: [{"task": "...", "assignee": "...", "deadline": "..."}]',
      prompt: summary,
      maxTokens: 300,
      routing: 'cost-optimized'
    })

    // Save to Notion
    await integrations.notion.pages.create({
      parent: { database_id: inputs.notionDatabaseId },
      properties: {
        'Title': { title: [{ text: { content: recording.title } }] },
        'Date': { date: { start: recording.date } },
        'Summary': { rich_text: [{ text: { content: summary } }] },
        'Action Items': { rich_text: [{ text: { content: actionItems } }] },
        'Recording Link': { url: recording.url }
      }
    })

    return {
      success: true,
      summary,
      actionItems: JSON.parse(actionItems)
    }
  }
})
```

**Earnings Potential:** $11.20/mo per user | **Cost:** $0.12/execution | **Margin:** 80%

---

## 🎯 Template Selection Guide

### Choose Integration Workflow When:
- ✅ Connecting two services (API → API)
- ✅ Simple data transformation
- ✅ No AI needed
- ✅ Prefer high margins (95%+)
- ✅ Want low price point ($5-10/mo)

### Choose AI-Enhanced Workflow When:
- 🤖 Need 1-2 AI calls for categorization/analysis
- ✅ Most logic is still integration-based
- ✅ AI adds clear value
- ✅ Good margins (80-90%)
- ✅ Medium price point ($12-20/mo)

### Choose AI-Native Workflow When:
- 🧠 Heavy AI processing required
- 🧠 Multiple AI steps
- 🧠 Content generation is core value
- ⚠️ Need cost optimization
- 💰 Higher price point ($20-50/mo)

---

## 💡 Tips for Success

1. **Start with Integration Workflows**
   - Easiest to build
   - Highest margins
   - Most user demand

2. **Use AI Strategically**
   - Only where it adds real value
   - Cache when possible
   - Use cheapest model for task

3. **Price for Value, Not Cost**
   - Integration saving 2 hours = worth $5-10/mo
   - AI analysis = worth $15-20/mo
   - Content generation = worth $25-50/mo

4. **Optimize Execution**
   - Cache expensive operations
   - Batch API calls
   - Use cheapest AI models

5. **Test Thoroughly**
   - Test with real data
   - Check cost estimates
   - Verify margins

---

Ready to publish? Copy a template, customize it, and `npx workway publish`!
