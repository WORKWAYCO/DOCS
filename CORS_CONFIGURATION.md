# CORS Configuration Guide

## Overview

This document provides a comprehensive overview of CORS (Cross-Origin Resource Sharing) configuration throughout the WORKWAY project.

## Current Architecture

### Services That Need CORS

#### 1. **Marketplace API** (`apps/api`)
**Status:** ✅ CORS Configured

**Location:** `apps/api/src/index.ts` (lines 35-52)

**Configuration:**
```typescript
app.use('*', cors({
  origin: [
    'http://localhost:3000',
    'http://localhost:5173',
    'https://workway.com',
    'https://*.workway.com',
    'https://workway-web.half-dozen.workers.dev'
  ],
  credentials: true,
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowHeaders: ['Content-Type', 'Authorization'],
  exposeHeaders: ['Content-Length', 'X-Request-Id'],
  maxAge: 600,
}))
```

**Allowed Origins:**
- `http://localhost:3000` - Local development (Next.js default)
- `http://localhost:5173` - Local development (Vite/TanStack Start default)
- `https://workway.com` - Production domain (planned)
- `https://*.workway.com` - Production subdomains wildcard
- `https://workway-web.half-dozen.workers.dev` - Current production deployment

**Why CORS is needed:**
- The frontend makes authenticated API requests from the browser
- Cookies/sessions require `credentials: true`
- Browser enforces same-origin policy, requiring explicit CORS headers

### Services That DON'T Need CORS

#### 1. **Web Frontend** (`apps/web`)
**Status:** ❌ No CORS Needed

**Why:** This is a TanStack Start SSR application deployed as a Cloudflare Worker. It serves HTML/JavaScript to browsers but doesn't receive cross-origin requests itself.

#### 2. **Webhook Endpoints** (`/webhooks/stripe`)
**Status:** ❌ No CORS Needed

**Why:** Webhooks are server-to-server requests from Stripe. They don't originate from a browser, so CORS doesn't apply. These requests use signature verification instead.

#### 3. **OAuth Callback Endpoints** (`/oauth/:provider/callback`)
**Status:** ❌ No CORS Needed

**Why:** OAuth callbacks are HTTP redirects from OAuth providers (Google, Notion) back to the API. These are browser redirects, not AJAX requests, so CORS doesn't apply.

#### 4. **Durable Objects** (`OAuth`, `WorkflowExecutor`)
**Status:** ❌ No CORS Needed

**Why:** Durable Objects are not HTTP endpoints. They're invoked internally by Workers and don't receive external HTTP requests.

## Frontend API Client Configuration

**Location:** `apps/web/src/lib/api-client.ts`

**Key Settings:**
```typescript
const response = await fetch(url, {
  credentials: 'include', // Essential for CORS with cookies
  headers: {
    'Content-Type': 'application/json',
  },
});
```

**API Base URL:**
- Development: `http://localhost:8787`
- Production: `https://marketplace-api.half-dozen.workers.dev`

## Security Considerations

### 1. Credentials Mode
The API uses `credentials: true` which allows cookies/sessions to be sent cross-origin. This requires:
- The `origin` must be explicitly listed (no wildcards like `*`)
- The frontend must send `credentials: 'include'` in fetch requests

### 2. Origin Validation
All origins are explicitly whitelisted. Never use `'*'` with credentials enabled.

### 3. Preflight Caching
`maxAge: 600` caches preflight OPTIONS requests for 10 minutes, reducing network overhead.

## Future Considerations

### Adding New Environments

When adding new deployment environments, update CORS origins in `apps/api/src/index.ts`:

```typescript
// Example: Adding staging environment
origin: [
  'http://localhost:3000',
  'http://localhost:5173',
  'https://workway.com',
  'https://*.workway.com',
  'https://workway-web.half-dozen.workers.dev',
  'https://staging.workway.com',  // Add staging
],
```

### Adding Custom Domain

When setting up custom domain (on roadmap):

1. Update DNS to point to Workers deployment
2. Add custom domain to CORS origins:
   ```typescript
   'https://app.workway.com',  // Custom domain
   ```
3. Update `API_BASE_URL` in `apps/web/src/lib/api-client.ts`
4. Update frontend metadata/manifest.json

### Adding New API Endpoints

All new route modules automatically inherit the global CORS middleware. No additional CORS configuration needed unless you have specific requirements.

**Example: Adding a new route module**
```typescript
// apps/api/src/routes/my-new-route.ts
import { Hono } from 'hono';

const myRoute = new Hono();

// CORS is already handled by global middleware in index.ts
myRoute.get('/endpoint', (c) => {
  return c.json({ data: 'value' });
});

export default myRoute;
```

Then mount it in `apps/api/src/index.ts`:
```typescript
import myRoute from './routes/my-new-route';
app.route('/my-route', myRoute);
```

## Troubleshooting CORS Issues

### Common Error: "No 'Access-Control-Allow-Origin' header"

**Symptoms:**
```
Access to fetch at 'https://marketplace-api.half-dozen.workers.dev/endpoint'
from origin 'https://new-frontend.com' has been blocked by CORS policy
```

**Solution:**
Add the new origin to the CORS configuration in `apps/api/src/index.ts`.

### Common Error: "Credentials flag is true, but Access-Control-Allow-Credentials is not"

**Symptoms:**
Cookies not being sent with requests, authentication failing.

**Solution:**
Ensure both:
1. API has `credentials: true` in CORS config
2. Frontend uses `credentials: 'include'` in fetch requests

### Common Error: "Access-Control-Allow-Methods does not allow method"

**Symptoms:**
POST/PUT/DELETE requests failing with CORS error.

**Solution:**
The method is already in `allowMethods`. Check if:
1. The preflight OPTIONS request is succeeding
2. The frontend is sending the correct `Content-Type` header

## Testing CORS

### Manual Testing with curl

```bash
# Test preflight request
curl -X OPTIONS \
  -H "Origin: https://workway-web.half-dozen.workers.dev" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type" \
  -v https://marketplace-api.half-dozen.workers.dev/auth/login

# Test actual request
curl -X POST \
  -H "Origin: https://workway-web.half-dozen.workers.dev" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}' \
  -v https://marketplace-api.half-dozen.workers.dev/auth/login
```

### Browser Console Testing

```javascript
// Test from browser console (on frontend domain)
fetch('https://marketplace-api.half-dozen.workers.dev/health', {
  credentials: 'include',
})
  .then(r => r.json())
  .then(console.log)
  .catch(console.error);
```

## Summary Table

| Service | CORS Required | Configuration Location | Status |
|---------|--------------|------------------------|--------|
| Marketplace API | ✅ Yes | `apps/api/src/index.ts` | Configured |
| Web Frontend | ❌ No | N/A | N/A |
| Webhook Endpoints | ❌ No | N/A | N/A |
| OAuth Callbacks | ❌ No | N/A | N/A |
| Durable Objects | ❌ No | N/A | N/A |

## Checklist for New Deployments

- [ ] Add new origin to CORS configuration in `apps/api/src/index.ts`
- [ ] Deploy API with updated CORS config: `cd apps/api && npx wrangler deploy`
- [ ] Update `API_BASE_URL` in frontend if needed
- [ ] Test authentication flow (login/signup)
- [ ] Verify cookies are being sent (check Network tab > Request Headers)
- [ ] Test all HTTP methods (GET, POST, PUT, DELETE, PATCH)

## References

- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Hono CORS Middleware](https://hono.dev/middleware/builtin/cors)
- [Cloudflare Workers CORS Guide](https://developers.cloudflare.com/workers/examples/cors-header-proxy/)
