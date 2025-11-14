# WORKWAY Production Deployment Guide

## Overview

This guide walks you through deploying WORKWAY to production on Cloudflare, including security hardening, monitoring, scaling, and cost optimization.

## Prerequisites

- Cloudflare account (Pro recommended for production)
- Domain name configured with Cloudflare DNS
- GitHub account for CI/CD
- Stripe account (if using payments)

## Step 1: Initial Setup

### 1.1 Clone and Configure

```bash
# Clone the repository
git clone https://github.com/workway/workway.git
cd workway

# Install dependencies
npm install

# Run setup script
chmod +x scripts/setup.sh
./scripts/setup.sh
```

### 1.2 Production Configuration

Create `apps/api/wrangler.production.toml`:

```toml
name = "workway-api-production"
main = "src/index.ts"
compatibility_date = "2025-01-01"
compatibility_flags = ["nodejs_compat"]

account_id = "YOUR_ACCOUNT_ID"

# Custom domain
routes = [
  { pattern = "api.yourdomain.com/*", zone_name = "yourdomain.com" }
]

# Production D1 Database
[[d1_databases]]
binding = "DB"
database_name = "workway-production"
database_id = "YOUR_PRODUCTION_DB_ID"

# Production KV Namespaces
[[kv_namespaces]]
binding = "SESSIONS"
id = "YOUR_PRODUCTION_SESSIONS_ID"

[[kv_namespaces]]
binding = "CACHE"
id = "YOUR_PRODUCTION_CACHE_ID"

# Rate limiting
[[unsafe.bindings]]
name = "RATE_LIMITER"
type = "ratelimit"
namespace_id = "YOUR_RATE_LIMITER_ID"
simple = { limit = 100, period = 60 }

# Workers AI
[ai]
binding = "AI"

# Vectorize for embeddings
[[vectorize]]
binding = "VECTORDB"
index_name = "workway-embeddings-prod"

# Durable Objects
[durable_objects]
bindings = [
  { name = "OAUTH", class_name = "OAuth" },
  { name = "WORKFLOW_EXECUTOR", class_name = "WorkflowExecutor" }
]

# Analytics Engine
[[analytics_engine_datasets]]
binding = "ANALYTICS"
dataset = "workway_analytics"
```

## Step 2: Security Hardening

### 2.1 Environment Variables

Set production secrets securely:

```bash
# Never commit secrets to git!
wrangler secret put JWT_SECRET --env production
wrangler secret put STRIPE_SECRET_KEY --env production
wrangler secret put STRIPE_WEBHOOK_SECRET --env production
wrangler secret put GOOGLE_CLIENT_SECRET --env production
wrangler secret put NOTION_CLIENT_SECRET --env production

# Generate strong JWT secret
openssl rand -base64 64
```

### 2.2 CORS Configuration

Update `apps/api/src/index.ts`:

```typescript
const ALLOWED_ORIGINS = process.env.NODE_ENV === 'production'
  ? [
      'https://yourdomain.com',
      'https://www.yourdomain.com',
      'https://app.yourdomain.com'
    ]
  : ['http://localhost:3000'];
```

### 2.3 Rate Limiting

Implement rate limiting middleware:

```typescript
// apps/api/src/middleware/rateLimit.ts
export async function rateLimit(c: Context, next: Next) {
  const ip = c.req.header('CF-Connecting-IP');
  const key = `rate_limit:${ip}`;

  const { success } = await c.env.RATE_LIMITER.limit({ key });

  if (!success) {
    return c.json({ error: 'Rate limit exceeded' }, 429);
  }

  await next();
}
```

### 2.4 Input Validation

Add Zod validation to all endpoints:

```typescript
// apps/api/src/routes/workflows.ts
import { z } from 'zod';

const createWorkflowSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500),
  steps: z.array(z.object({
    type: z.string(),
    config: z.record(z.any())
  })).min(1).max(50)
});

app.post('/workflows', async (c) => {
  const body = await c.req.json();

  try {
    const validated = createWorkflowSchema.parse(body);
    // Process validated input
  } catch (error) {
    return c.json({ error: 'Invalid input' }, 400);
  }
});
```

## Step 3: Database Setup

### 3.1 Create Production Database

```bash
# Create production D1 database
wrangler d1 create workway-production

# Note the database_id and update wrangler.production.toml
```

### 3.2 Run Migrations

```bash
# Apply all migrations
wrangler d1 migrations apply workway-production --env production

# Verify tables
wrangler d1 execute workway-production --command "SELECT name FROM sqlite_master WHERE type='table'" --env production
```

### 3.3 Database Backups

Set up automated backups:

```bash
# Create backup script
cat > scripts/backup-db.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
wrangler d1 execute workway-production --command ".dump" > backups/db_$DATE.sql
# Upload to R2 or external storage
wrangler r2 object put workway-backups/db_$DATE.sql --file backups/db_$DATE.sql
EOF

chmod +x scripts/backup-db.sh

# Add to crontab for daily backups
0 3 * * * /path/to/scripts/backup-db.sh
```

## Step 4: Monitoring & Observability

### 4.1 Analytics Engine Setup

```typescript
// apps/api/src/lib/analytics.ts
export async function trackEvent(
  c: Context,
  event: string,
  properties: Record<string, any>
) {
  const analytics = c.env.ANALYTICS;

  analytics.writeDataPoint({
    blobs: [event, c.get('userId')],
    doubles: [1],
    indexes: [Date.now()],
    ...properties
  });
}

// Usage
await trackEvent(c, 'workflow_executed', {
  workflowId: workflow.id,
  duration: executionTime,
  success: true
});
```

### 4.2 Error Tracking

Integrate with Sentry:

```typescript
// apps/api/src/lib/sentry.ts
import * as Sentry from '@sentry/cloudflare';

export function initSentry(env: Env) {
  Sentry.init({
    dsn: env.SENTRY_DSN,
    environment: env.ENVIRONMENT || 'production',
    tracesSampleRate: 0.1,
  });
}

// Wrap handlers
export function withSentry(handler: Handler) {
  return async (c: Context) => {
    try {
      return await handler(c);
    } catch (error) {
      Sentry.captureException(error);
      throw error;
    }
  };
}
```

### 4.3 Real-time Monitoring

```bash
# Watch real-time logs
wrangler tail --env production

# With filtering
wrangler tail --env production --filter "status=500"

# Save logs
wrangler tail --env production > logs/production.log
```

### 4.4 Uptime Monitoring

Use Cloudflare Analytics or external service:

```javascript
// Health check endpoint
app.get('/health', async (c) => {
  // Check critical services
  const checks = {
    database: await checkDatabase(c.env.DB),
    ai: await checkAI(c.env.AI),
    cache: await checkKV(c.env.CACHE),
  };

  const healthy = Object.values(checks).every(v => v);

  return c.json({
    status: healthy ? 'healthy' : 'degraded',
    checks,
    timestamp: Date.now()
  }, healthy ? 200 : 503);
});
```

## Step 5: Performance Optimization

### 5.1 Caching Strategy

```typescript
// Implement smart caching
const cache = {
  // Cache AI responses
  async getOrSet(key: string, fn: () => Promise<any>, ttl = 3600) {
    const cached = await c.env.CACHE.get(key, 'json');
    if (cached) return cached;

    const fresh = await fn();
    await c.env.CACHE.put(key, JSON.stringify(fresh), {
      expirationTtl: ttl
    });

    return fresh;
  }
};

// Usage with AI
const summary = await cache.getOrSet(
  `ai:summary:${contentHash}`,
  () => ai.generateText({ prompt: content }),
  7200 // 2 hours
);
```

### 5.2 Edge Caching

```typescript
// Add cache headers
app.get('/api/workflows', async (c) => {
  c.header('Cache-Control', 'public, max-age=60, s-maxage=300');
  c.header('CDN-Cache-Control', 'max-age=300');

  // Cloudflare will cache at edge
  return c.json(workflows);
});
```

### 5.3 Database Query Optimization

```sql
-- Add indexes for common queries
CREATE INDEX idx_workflow_runs_user_created
ON workflow_runs(user_id, created_at DESC);

CREATE INDEX idx_workflow_runs_status
ON workflow_runs(status)
WHERE status IN ('running', 'failed');

-- Use prepared statements
const stmt = c.env.DB.prepare(
  'SELECT * FROM workflows WHERE user_id = ?1 LIMIT ?2'
).bind(userId, limit);
```

## Step 6: Scaling Configuration

### 6.1 Cloudflare Workers Limits

Current limits and optimization:

```javascript
const workerLimits = {
  cpuTime: '30 seconds',       // Use Durable Objects for longer
  memory: '128 MB',            // Stream large data
  scriptSize: '10 MB',         // Code split if needed
  subrequests: 50,             // Batch API calls

  // Optimization strategies
  longRunning: 'Use Durable Objects',
  largeData: 'Use R2 or stream',
  manyRequests: 'Batch and cache'
};
```

### 6.2 Auto-scaling Configuration

Workers auto-scale by default, but optimize for it:

```typescript
// Use connection pooling for external services
const connectionPool = new Map();

async function getConnection(service: string) {
  if (!connectionPool.has(service)) {
    connectionPool.set(service, await createConnection(service));
  }
  return connectionPool.get(service);
}

// Clean up after request
addEventListener('fetch', event => {
  event.waitUntil(
    handleRequest(event.request).finally(() => {
      // Cleanup connections
      connectionPool.clear();
    })
  );
});
```

## Step 7: Cost Optimization

### 7.1 Monitor Usage

```javascript
// Track costs per operation
const costTracking = {
  workers: 0.0000005,      // $0.50 per million requests
  d1_reads: 0.001 / 1000,  // $0.001 per 1K rows read
  d1_writes: 0.001 / 100,  // $0.001 per 100 rows written
  ai_tokens: 0.01 / 1000000, // $0.01 per 1M tokens
  kv_reads: 0.5 / 1000000, // $0.50 per million reads

  calculateMonthlyCost(usage) {
    return {
      workers: usage.requests * this.workers,
      database: (usage.d1_reads * this.d1_reads) +
                (usage.d1_writes * this.d1_writes),
      ai: usage.ai_tokens * this.ai_tokens,
      storage: usage.kv_reads * this.kv_reads,
      total: // sum all
    };
  }
};
```

### 7.2 Optimization Tips

```typescript
// 1. Batch database operations
const batchInsert = async (records: any[]) => {
  const stmt = c.env.DB.prepare(
    'INSERT INTO records VALUES (?1, ?2, ?3)'
  );
  const batch = records.map(r => stmt.bind(r.id, r.name, r.data));
  await c.env.DB.batch(batch);
};

// 2. Use appropriate AI models
const modelSelection = {
  simple: AIModels.LLAMA_2_7B,     // Cheapest
  balanced: AIModels.LLAMA_3_8B,   // Good quality/cost
  complex: AIModels.MISTRAL_7B,    // Best quality
};

// 3. Implement request coalescing
const pending = new Map();

async function coalesceRequests(key: string, fn: Function) {
  if (!pending.has(key)) {
    pending.set(key, fn());
  }
  return pending.get(key);
}
```

## Step 8: Deployment

### 8.1 GitHub Actions CI/CD

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Deploy to Cloudflare
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          environment: production
          workingDirectory: apps/api
```

### 8.2 Blue-Green Deployment

```bash
# Deploy to staging first
wrangler deploy --env staging

# Test staging
curl https://staging-api.yourdomain.com/health

# If healthy, promote to production
wrangler deploy --env production

# Rollback if needed
wrangler rollback --env production
```

## Step 9: Custom Domain Setup

### 9.1 Configure DNS

In Cloudflare Dashboard:
1. Go to DNS settings
2. Add CNAME record:
   - Name: `api`
   - Target: `workway-api-production.username.workers.dev`
   - Proxy: Enabled (orange cloud)

### 9.2 SSL/TLS Configuration

1. Set SSL/TLS mode to "Full (strict)"
2. Enable "Always Use HTTPS"
3. Set minimum TLS version to 1.2
4. Enable HSTS:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

## Step 10: Launch Checklist

### Pre-launch (1 week before)
- [ ] Security audit complete
- [ ] Load testing performed
- [ ] Backup strategy tested
- [ ] Monitoring alerts configured
- [ ] Documentation updated
- [ ] Support process defined

### Launch Day
- [ ] Deploy to production
- [ ] Verify health checks
- [ ] Monitor error rates
- [ ] Check performance metrics
- [ ] Test critical user flows
- [ ] Enable auto-scaling

### Post-launch (24 hours)
- [ ] Review error logs
- [ ] Analyze performance data
- [ ] Check cost projections
- [ ] Gather user feedback
- [ ] Plan optimizations

## Production Best Practices

### DO ✅
- Use environment variables for secrets
- Implement comprehensive logging
- Set up automated backups
- Monitor costs daily
- Use appropriate caching
- Test disaster recovery

### DON'T ❌
- Hardcode credentials
- Skip input validation
- Ignore rate limiting
- Neglect monitoring
- Over-provision resources
- Deploy without testing

## Troubleshooting

### Common Issues and Solutions

```javascript
const troubleshooting = {
  'High latency': {
    check: ['Cache hit rates', 'Database queries', 'AI model selection'],
    solution: 'Optimize caching and use faster models'
  },

  'Rate limits': {
    check: ['Request patterns', 'Bot traffic', 'API abuse'],
    solution: 'Implement better rate limiting and bot protection'
  },

  'High costs': {
    check: ['AI token usage', 'Database operations', 'Cache misses'],
    solution: 'Optimize model selection and implement caching'
  },

  'Errors spike': {
    check: ['Recent deployments', 'External service status', 'Rate limits'],
    solution: 'Roll back deployment or implement circuit breakers'
  }
};
```

## Support & Resources

- **Documentation**: [docs.workway.dev](https://docs.workway.dev)
- **Status Page**: [status.workway.dev](https://status.workway.dev)
- **Discord**: [discord.gg/workway](https://discord.gg/workway)
- **Emergency**: security@workway.com (security issues only)

## Cost Projections

### Expected Monthly Costs

| Usage Tier | Requests | AI Operations | Database | Total |
|------------|----------|--------------|----------|--------|
| Startup | 100K | 10K | 1M ops | ~$10 |
| Growth | 1M | 100K | 10M ops | ~$75 |
| Scale | 10M | 1M | 100M ops | ~$500 |
| Enterprise | 100M | 10M | 1B ops | ~$3,000 |

Remember: These are infrastructure costs only. Your margin comes from the value you provide on top!