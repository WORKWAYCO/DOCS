# WORKWAY Architecture Visualizations

## 1. System Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         USERS (BUSINESSES)                              │
└────────────────────────────────────┬─────────────────────────────────────┘
                                     │
                ┌────────────────────┴────────────────────┐
                │                                         │
         ┌──────▼──────┐                          ┌──────▼──────┐
         │  Web UI     │                          │ Integrations│
         │ (Browse)    │                          │   (Deploy)  │
         └──────┬──────┘                          └──────┬──────┘
                │                                       │
                └────────────────────────┬──────────────┘
                                         │
        ┌────────────────────────────────▼──────────────────────────┐
        │                  CLOUDFLARE WORKERS                       │
        │              (API Layer - Hono.js)                        │
        │                                                            │
        │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────────┐        │
        │  │Auth    │ │Workflow│ │Marketplace Integrations        │
        │  │Routes  │ │Routes  │ │Routes   │ │Routes     │        │
        │  └────────┘ └────────┘ └────────┘ └────────────┘        │
        │                                                            │
        │  ┌────────────────────────────────────────────┐          │
        │  │        OAuth Manager (Durable Object)      │          │
        │  │  - Token storage & refresh                │          │
        │  │  - Provider coordination                  │          │
        │  └────────────────────────────────────────────┘          │
        │                                                            │
        └────────────────┬──────────────────────────────────────────┘
                         │
        ┌────────────────┼──────────────────────────────────────┐
        │                │                                      │
    ┌───▼────┐    ┌──────▼──────┐    ┌──────────┐    ┌────────▼────┐
    │  D1    │    │  KV Cache   │    │Durable   │    │  R2 Storage │
    │ SQLite │    │             │    │Objects   │    │             │
    │ Store  │    │ - Sessions  │    │(Workflow │    │ - Logos     │
    │        │    │ - Rate      │    │Executor) │    │ - Banners   │
    │- Users │    │   Limiting  │    │          │    │ - Videos    │
    │- Integ │    │             │    │- Execute │    │             │
    │- Runs  │    │             │    │- State   │    │             │
    └────────┘    └─────────────┘    └──────────┘    └─────────────┘
        │                                   │
        │                    ┌──────────────┘
        │                    │
        │                ┌───▼────────────────┐
        │                │  Integration Plugins
        │                │  - Gmail
        │                │  - Notion
        │                │  - Slack
        └────────────────┤  (ActionRegistry)
                         └────────────────────┘
```

## 2. Workflow Execution Flow

```
TRIGGER EVENT
    │
    ├─ Webhook (incoming)
    ├─ Schedule (cron)
    ├─ Manual (button click)
    └─ Event (database)
         │
         ▼
    TriggerService.triggerWorkflow()
         │
         ├─ Get installation config
         ├─ Create workflow_run record
         └─ Spawn WorkflowExecutor Durable Object
              │
              ▼
    WorkflowExecutor.initialize()
         │
         ├─ Load workflow definition
         ├─ Initialize state
         └─ Begin step execution
              │
              ▼
    FOR EACH WORKFLOW STEP
         │
         ├─ Get action from ActionRegistry
         │  (e.g., "gmail.send-email")
         │
         ├─ Get OAuth credentials
         │  (from KV, decrypt if needed)
         │
         ├─ Execute action with ActionContext
         │  │
         │  ├─ Call integration
         │  ├─ Handle errors (IntegrationError)
         │  └─ Wrap result in ActionResult<T>
         │
         ├─ Store step result in database
         │
         ├─ If error:
         │  ├─ Check if retryable
         │  ├─ Refresh OAuth if AUTH_EXPIRED
         │  └─ Retry with backoff
         │
         └─ Continue to next step
              │
              ▼
    WORKFLOW COMPLETE
         │
         ├─ Update workflow_run status
         ├─ Record execution metrics
         ├─ Charge usage fees (if applicable)
         └─ Notify user (if configured)
```

## 3. Plugin Architecture (Narrow Waist)

```
M INTEGRATIONS                   N CONSUMERS
┌──────────────┐                ┌──────────────────┐
│   Gmail      │                │  WorkflowExecutor│
├──────────────┤                ├──────────────────┤
│   Notion     │                │  Marketplace UI  │
├──────────────┤                ├──────────────────┤
│   Slack      │────────┬───────┤  Custom Flows    │
├──────────────┤        │       ├──────────────────┤
│   Stripe     │        │       │  Analytics Dash  │
├──────────────┤        │       ├──────────────────┤
│   ...        │    ┌───▼────┐  │  Developer CLI   │
└──────────────┘    │NARROW  │  └──────────────────┘
                    │WAIST   │
                    │        │
                    │LAYERS:│
                    │1. ActionResult<T>
                    │2. IntegrationError
                    │3. TriggerService
                    └────────┘

BEFORE (M × N complexity):
Gmail → Transform → Notion
Gmail → Transform → Slack
Gmail → Transform → Stripe
Notion → Transform → Slack
Notion → Transform → Stripe
Slack → Transform → Stripe
(6 custom transformations)

AFTER (M + N complexity):
Gmail ──┐
        │
Notion──┼──→ ActionResult<T> ──→ Workflow Engine
        │
Slack ──┘

(No custom transforms needed!)
```

## 4. Data Flow: From Integration to Marketplace

```
DEVELOPER
    │
    ├─ Build Integration (locally using CLI)
    │  ├─ Define actions
    │  ├─ Implement execute()
    │  └─ Return ActionResult<T>
    │
    └─ Publish to Marketplace
         │
         ▼
    POST /developers/integrations
         │
         ├─ Validate schema
         ├─ Store in marketplace_integrations table
         │  └─ status = "pending_review"
         │
         └─ Admin review
              │
              ├─ Check code quality
              ├─ Verify OAuth scopes
              └─ Approve or reject
                   │
                   ▼
              IF APPROVED:
                   │
                   ├─ status = "published"
                   ├─ Register in ActionRegistry
                   └─ Make available to users
                        │
                        ▼
                   USER INSTALLS
                        │
                        ├─ POST /installations
                        ├─ Create user_installations record
                        ├─ Trigger OAuth if needed
                        │  (via OAuthManager Durable Object)
                        │
                        └─ Installation ready!
                             │
                             ▼
                        USER RUNS WORKFLOW
                             │
                             ├─ Trigger event
                             ├─ WorkflowExecutor loads actions
                             │  from ActionRegistry
                             │
                             └─ Execute with user's OAuth tokens
```

## 5. Revenue Flow

```
UPFRONT PURCHASE
    │
    ├─ Developer sets price ($19-$249)
    ├─ User pays via Stripe
    │  │
    │  └─ Stripe fee: 2.9% + $0.30
    │
    └─ Developer gets: 100% - Stripe fee
       (WORKWAY takes $0 cut)


USAGE-BASED BILLING (After 20 free executions)
    │
    ├─ Execute workflow
    │  └─ Log execution in workflow_runs
    │
    ├─ TriggerService measures complexity
    │  ├─ Light: $0.05/execution
    │  └─ Heavy: $0.25/execution
    │
    ├─ Monthly billing:
    │  └─ Sum all executions × rate
    │
    ├─ Platform invoice user
    │  └─ Via Stripe
    │
    ├─ WORKWAY takes platform cut (~30%)
    │
    └─ Developer gets remainder (~70%)
       └─ Via Stripe Connect payout


YEAR 1 REVENUE (Example Developer)
    │
    ├─ Upfront: 50 customers × $39 = $1,950
    │  └─ Dev net: $1,900 (after Stripe fees)
    │
    └─ Usage: 50 customers × 1,000 executions/mo × $0.05
              × 12 months × 70% share
       = 50 × 1,000 × 12 × 0.05 × 0.70 = $21,000
       
       TOTAL: $1,900 + $21,000 = $22,900 (Year 1)
```

## 6. Technology Stack Visualization

```
PRESENTATION LAYER
┌──────────────────────────────────────────────┐
│  React 19                                    │
│  TanStack Router (Type-safe routing)         │
│  TailwindCSS (Styling)                       │
│  Framer Motion (Animations)                  │
└────────────────┬─────────────────────────────┘
                 │
DEPLOYMENT LAYER
┌────────────────▼─────────────────────────────┐
│  Cloudflare Pages (Edge hosting)             │
│  Vite (Build tool)                           │
│  @cloudflare/vite-plugin (Integration)       │
└────────────────┬─────────────────────────────┘
                 │
API LAYER
┌────────────────▼─────────────────────────────┐
│  Hono.js (Router)                            │
│  Zod (Validation)                            │
│  Stripe (Payments)                           │
│  CORS (Cross-origin)                         │
└────────────────┬─────────────────────────────┘
                 │
DEPLOYMENT LAYER
┌────────────────▼─────────────────────────────┐
│  Cloudflare Workers (Serverless execution)   │
│  Durable Objects (Stateful)                  │
│  KV Namespace (Cache/sessions)               │
└────────────────┬─────────────────────────────┘
                 │
DATA LAYER
┌────────────────▼─────────────────────────────┐
│  D1 (SQLite at edge)                         │
│  R2 (Object storage)                         │
│  KV (Secrets/tokens)                         │
└──────────────────────────────────────────────┘
```

## 7. Security & Credentials Flow

```
USER AUTHENTICATES
    │
    ├─ POST /auth/login
    └─ Create session in KV + session_logs
         │
         ├─ KV: session_id → encrypted token
         └─ D1: session_logs → audit trail
              │
              ▼
         AUTHENTICATED REQUEST
              │
              ├─ Include session_id cookie
              ├─ Middleware verifies in KV
              └─ Set context.userId


USER CONNECTS OAUTH (Gmail, Notion, etc)
    │
    ├─ Redirect to OAuth provider
    │  (Google, Notion, Slack)
    │
    └─ Provider redirects back to:
         POST /oauth/callback
         │
         ├─ Get authorization code
         ├─ Exchange for tokens (with OAuth.getTokens())
         │
         └─ OAuthManager Durable Object:
              │
              ├─ Store tokens in KV (encrypted)
              │  └─ Key: "oauth:{userId}:{provider}"
              │
              └─ Store metadata in D1
                 └─ oauth_credentials table


WORKFLOW EXECUTION
    │
    ├─ Action needs OAuth token
    ├─ Request: context.oauth.getTokens(userId, 'gmail')
    │
    └─ OAuthManager checks:
         │
         ├─ Token in KV?
         ├─ Token expired?
         │  └─ Auto-refresh via refresh_token
         │
         └─ Return fresh token to action
              │
              └─ Action uses token for API call
```

## 8. Integration Plugin Registration

```
APP STARTUP
    │
    └─ registerAllIntegrations() called
         │
         ├─ Load gmailIntegration
         │  ├─ Define 3 actions
         │  │  ├─ fetch-email
         │  │  ├─ send-email
         │  │  └─ add-label
         │  │
         │  └─ Register in ActionRegistry
         │
         ├─ Load notionIntegration
         │  ├─ Define 3 actions
         │  │  ├─ create-page
         │  │  ├─ update-page
         │  │  └─ query-database
         │  │
         │  └─ Register in ActionRegistry
         │
         └─ Load slackIntegration
            ├─ Define 2 actions
            │  ├─ send-message
            │  └─ list-channels
            │
            └─ Register in ActionRegistry
                 │
                 ▼
            ActionRegistry contents:
            ├─ 8 actions registered
            ├─ Indexed by "service.action"
            └─ Ready for WorkflowExecutor


WORKFLOW EXECUTION
    │
    ├─ WorkflowExecutor gets step:
    │  └─ { type: "gmail.send-email", config: {...} }
    │
    ├─ Look up action:
    │  └─ integrationRegistry.getAction("gmail.send-email")
    │
    ├─ Get context with OAuth:
    │  └─ context.oauth.getTokens(userId, 'gmail')
    │
    └─ Execute:
       └─ action.execute(input, context)
            │
            └─ Returns ActionResult<T>
```

## 9. Database Schema (Simplified)

```
CORE TABLES

┌─────────────────────┐
│      users          │
├─────────────────────┤
│ id (usr_...)        │ ◄──┐
│ email (unique)      │    │
│ password_hash       │    │
│ role (USER/DEV)     │    │
│ display_name        │    │
│ email_verified      │    │
└─────────────────────┘    │
                           │
                      ┌────┴──────────────┬───────────────┐
                      │                   │               │
        ┌─────────────▼──────────┐  ┌─────▼───────────┐  │
        │ developer_profiles     │  │ user_installs   │  │
        ├────────────────────────┤  ├─────────────────┤  │
        │ id (dev_...)           │  │ id (ins_...)    │  │
        │ user_id ────────┐      │  │ user_id ────────┴──┘
        │ company_name    │      │  │ integration_id
        │ stripe_acct_id  │      │  │ configuration
        │ verification    │      │  │ status
        └────────────────┼──────┘  └─────────────────┘
                         │
                    ┌────┴─────────────────────────┐
                    │                              │
        ┌───────────▼──────────┐    ┌─────────────▼──────┐
        │marketplace_integr.   │    │  integration_rev.  │
        ├──────────────────────┤    ├────────────────────┤
        │id (int_...)          │    │ id (rev_...)       │
        │developer_id (dev_)───┤    │ integration_id────┐│
        │name                  │    │ user_id ──────────┼┘
        │description           │    │ rating (1-5)      │
        │icon_url              │    │ review_text       │
        │pricing_model         │    │ verified_purchase │
        │workflow_definition   │    └────────────────────┘
        │required_oauth        │
        │install_count         │
        │active_install_count  │
        │success_rate          │
        │average_rating        │
        └──────────────────────┘
                    │
        ┌───────────┴──────────────┐
        │                          │
    ┌───▼────────────────┐  ┌──────▼──────────────┐
    │  workflow_runs     │  │  workflow_steps     │
    ├────────────────────┤  ├─────────────────────┤
    │ id (run_...)       │  │ id                  │
    │ installation_id ───┼──┤ run_id              │
    │ user_id            │  │ step_index          │
    │ status             │  │ status              │
    │ trigger_type       │  │ result (JSON)       │
    │ trigger_data       │  │ error               │
    │ result (JSON)      │  │ started_at          │
    │ error_details      │  │ completed_at        │
    │ started_at         │  └─────────────────────┘
    │ completed_at       │
    │ execution_count    │
    └────────────────────┘
```

## 10. Cost Comparison: WORKWAY vs Competitors

```
EXECUTION COST COMPARISON (Per 1,000,000 executions)

AWS Lambda:
  Compute:       $20.00
  Data transfer: $15.00
  Duration:      $5.00
  ────────────────────
  Total:         $40.00

GCP Cloud Functions:
  Compute:       $25.00
  Data transfer: $10.00
  ────────────────────
  Total:         $35.00

Cloudflare Workers (WORKWAY):
  Compute:       $0.50
  KV/Storage:    $1.50 (estimated)
  Data transfer: $0 (included)
  ────────────────────
  Total:         $2.00

SAVINGS: 95% cheaper than AWS, 94% cheaper than GCP

───────────────────────────────────────────────────────

WORKFLOW EXECUTION COST
(Stripe→Notion sync, 1,000 executions/month)

AWS Alternative ($40/M compute):
  + Database: $10/mo
  + Storage: $5/mo
  + Monitoring: $5/mo
  ────────────────────
  = $60/mo

WORKWAY:
  + Execution: ~$0.20 (Cloudflare)
  + Stripe fee: ~$0.29 (on $39 upfront)
  + Platform: ~$15/mo (usage-based)
  ────────────────────
  = ~$15.50/mo for platform

Developer can charge $39 upfront + $10/mo
→ 250% profit margin vs AWS competitor
```

---

These visualizations show how WORKWAY achieves its competitive advantage through architecture, cost structure, and developer-friendly design.

