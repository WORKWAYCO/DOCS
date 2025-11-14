# Workflow Configuration Schema Specification

## Overview
This document defines how developers can create intelligent, guided configuration flows for their workflow integrations on the WORKWAY marketplace platform.

## Core Concept
Instead of hardcoding configuration logic, developers define a **Configuration Schema** that describes:
1. What services they need to connect
2. What data sources they need to select (databases, folders, channels, etc.)
3. How to score/rank those sources intelligently
4. What field mappings are required

---

## 1. Integration Metadata (Database Schema)

```sql
-- marketplace_integrations table
CREATE TABLE marketplace_integrations (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  config_schema JSON,  -- NEW: Configuration schema definition
  ...
);
```

---

## 2. Configuration Schema Format

### Example: Gmail → Notion
```json
{
  "version": "1.0",
  "steps": [
    {
      "id": "oauth_check",
      "type": "oauth_verification",
      "providers": ["gmail", "notion"],
      "failureMessage": "Please connect Gmail and Notion in OAuth settings"
    },
    {
      "id": "notion_database_selection",
      "type": "resource_selection",
      "config": {
        "provider": "notion",
        "resourceType": "database",
        "fetchEndpoint": "https://api.notion.com/v1/search",
        "fetchMethod": "POST",
        "fetchBody": {
          "filter": { "property": "object", "value": "database" },
          "sort": { "direction": "descending", "timestamp": "last_edited_time" }
        },
        "scoringRules": {
          "requiredProperties": [
            { "type": "title", "weight": 1.0, "purpose": "Email subject" },
            { "type": "email", "weight": 0.8, "purpose": "Sender email" },
            { "type": "date", "weight": 0.8, "purpose": "Email date" },
            { "type": "files", "weight": 0.6, "purpose": "Attachments" },
            { "type": "rich_text", "weight": 0.5, "purpose": "Email body" },
            { "type": "url", "weight": 0.4, "purpose": "Thread link" }
          ],
          "nameKeywords": {
            "strong": ["email", "inbox", "message", "mail", "contact", "communication", "interaction"],
            "moderate": ["user", "customer", "client", "people", "crm", "lead", "sign-up", "signup"]
          },
          "strongBoost": 0.3,
          "moderateBoost": 0.15,
          "tiers": {
            "excellent": 0.7,
            "good": 0.5,
            "fair": 0.3
          }
        },
        "fieldMappings": [
          {
            "id": "subject",
            "label": "Email Subject",
            "notionProperty": "title",
            "required": true
          },
          {
            "id": "from",
            "label": "Sender Email",
            "notionProperty": "email",
            "required": true
          },
          {
            "id": "date",
            "label": "Received Date",
            "notionProperty": "date",
            "required": false
          },
          {
            "id": "body",
            "label": "Email Body",
            "notionProperty": "page_content",
            "required": true
          }
        ]
      }
    },
    {
      "id": "gmail_filter",
      "type": "custom_input",
      "config": {
        "label": "Gmail Filter",
        "description": "Which emails should be synced?",
        "inputType": "select",
        "options": [
          { "value": "unread", "label": "Unread emails only" },
          { "value": "all", "label": "All emails" },
          { "value": "starred", "label": "Starred emails only" }
        ],
        "default": "unread"
      }
    }
  ]
}
```

### Example: Slack → Notion
```json
{
  "version": "1.0",
  "steps": [
    {
      "id": "oauth_check",
      "type": "oauth_verification",
      "providers": ["slack", "notion"]
    },
    {
      "id": "slack_channel_selection",
      "type": "resource_selection",
      "config": {
        "provider": "slack",
        "resourceType": "channel",
        "fetchEndpoint": "https://slack.com/api/conversations.list",
        "fetchMethod": "GET",
        "scoringRules": {
          "nameKeywords": {
            "strong": ["announcements", "general", "team", "updates"],
            "moderate": ["random", "social", "discussion"]
          },
          "strongBoost": 0.3,
          "moderateBoost": 0.15,
          "tiers": {
            "excellent": 0.7,
            "good": 0.5,
            "fair": 0.3
          }
        }
      }
    },
    {
      "id": "notion_database_selection",
      "type": "resource_selection",
      "config": {
        "provider": "notion",
        "resourceType": "database",
        "fetchEndpoint": "https://api.notion.com/v1/search",
        "fetchMethod": "POST",
        "scoringRules": {
          "requiredProperties": [
            { "type": "title", "weight": 1.0 },
            { "type": "rich_text", "weight": 0.8 },
            { "type": "date", "weight": 0.6 }
          ],
          "nameKeywords": {
            "strong": ["messages", "slack", "chat", "conversation"],
            "moderate": ["team", "communication", "updates"]
          }
        }
      }
    }
  ]
}
```

---

## 3. Generic Configuration Engine

### Backend Implementation

```typescript
// Generic configuration handler
installations.post('/configure', async (c) => {
  const { integrationId, step, selectionData } = await c.req.json();

  // Get integration with config schema
  const integration = await getIntegration(integrationId);
  const configSchema = JSON.parse(integration.config_schema);

  // Get the current step definition
  const stepDef = configSchema.steps.find(s => s.id === step);

  switch (stepDef.type) {
    case 'oauth_verification':
      return handleOAuthVerification(stepDef, session);

    case 'resource_selection':
      return handleResourceSelection(stepDef, session, selectionData);

    case 'custom_input':
      return handleCustomInput(stepDef, selectionData);
  }
});

// Generic resource selection handler
async function handleResourceSelection(stepDef, session, selectionData) {
  const { provider, resourceType, fetchEndpoint, scoringRules } = stepDef.config;

  // Get OAuth token for the provider
  const tokens = await getOAuthTokens(session.userId, provider);

  // Fetch resources from the provider
  const resources = await fetchResources(fetchEndpoint, tokens, stepDef.config);

  // Score resources based on schema
  const scored = resources.map(resource => ({
    ...resource,
    score: calculateScore(resource, scoringRules),
    compatibility: getCompatibilityTier(score, scoringRules.tiers)
  }));

  // Sort by score
  scored.sort((a, b) => b.score - a.score);

  return { resources: scored, suggested: scored[0]?.id };
}

// Generic scoring function
function calculateScore(resource, rules) {
  let score = 0;

  // Score based on properties
  if (rules.requiredProperties) {
    const maxWeight = rules.requiredProperties.reduce((sum, p) => sum + p.weight, 0);
    const actualWeight = rules.requiredProperties
      .filter(p => hasProperty(resource, p.type))
      .reduce((sum, p) => sum + p.weight, 0);
    score = actualWeight / maxWeight;
  }

  // Boost based on name keywords
  if (rules.nameKeywords && resource.name) {
    const nameLower = resource.name.toLowerCase();
    if (rules.nameKeywords.strong.some(kw => nameLower.includes(kw))) {
      score += rules.strongBoost;
    } else if (rules.nameKeywords.moderate.some(kw => nameLower.includes(kw))) {
      score += rules.moderateBoost;
    }
  }

  return Math.min(score, 1.0);
}
```

---

## 4. Developer Workflow Builder UI

### Configuration Step Builder
Developers would use a visual builder in the Developer Dashboard:

```
┌─────────────────────────────────────────────────────────────┐
│  Workflow Configuration Builder                             │
│                                                              │
│  Integration: "Slack to Notion Message Sync"               │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Step 1: OAuth Verification                           │  │
│  │ ✓ Slack                                              │  │
│  │ ✓ Notion                                             │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Step 2: Select Slack Channel                         │  │
│  │                                                        │  │
│  │ Provider: Slack                                       │  │
│  │ Resource Type: Channel                                │  │
│  │                                                        │  │
│  │ Scoring Keywords:                                     │  │
│  │   Strong: announcements, general, team               │  │
│  │   Moderate: random, social                           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Step 3: Select Notion Database                       │  │
│  │                                                        │  │
│  │ Required Properties:                                  │  │
│  │   ☑ Title (weight: 1.0)                              │  │
│  │   ☑ Rich Text (weight: 0.8)                          │  │
│  │   ☑ Date (weight: 0.6)                               │  │
│  │                                                        │  │
│  │ Scoring Keywords:                                     │  │
│  │   Strong: messages, slack, chat                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  [+ Add Configuration Step]                                 │
│                                                              │
│  [Save Configuration]                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Benefits

### For Developers:
✅ **No code required** - Define configuration via schema
✅ **Reusable patterns** - Same scoring logic works across workflows
✅ **Smart recommendations** - Platform handles intelligent ranking
✅ **Consistent UX** - Users get same experience across all workflows

### For Users:
✅ **Guided setup** - Intelligent recommendations for every workflow
✅ **Search & filter** - Consistent UI across all integrations
✅ **Quick configuration** - Best options surfaced automatically
✅ **Confidence** - Clear compatibility indicators

---

## 6. Migration Path

### Phase 1: Extract Gmail→Notion to Schema (Current Sprint)
- Convert hardcoded logic to first schema
- Test with existing workflow

### Phase 2: Generalize Backend Engine (Next Sprint)
- Build generic resource selection handler
- Implement scoring engine from schema
- Add schema validation

### Phase 3: Developer UI Builder (Future)
- Visual configuration builder
- Schema editor in Developer Dashboard
- Template library for common patterns

### Phase 4: Advanced Features
- Conditional steps (if/else logic)
- Multi-step field mapping
- Custom validation rules
- AI-assisted schema generation

---

## 7. Example Use Cases

| Integration | Step 1 | Step 2 | Step 3 |
|-------------|--------|--------|--------|
| Gmail → Notion | Verify OAuth | Select Notion DB | Configure filter |
| Slack → Notion | Verify OAuth | Select Slack channel | Select Notion DB |
| GitHub → Discord | Verify OAuth | Select GitHub repo | Select Discord channel |
| Airtable → Notion | Verify OAuth | Select Airtable base | Select Notion DB |
| Linear → Slack | Verify OAuth | Select Linear project | Select Slack channel |

All powered by the same generic engine! 🚀
