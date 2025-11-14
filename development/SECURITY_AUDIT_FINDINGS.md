# Security Audit Findings - WORKWAY Open Source Preparation

## Audit Date: November 2025

## Summary
This document lists all sensitive data and secrets found in the codebase that must be addressed before open sourcing.

## Critical Findings

### 1. Cloudflare Account & Resource IDs
**Location:** `apps/api/wrangler.jsonc`
- **Account ID:** `5c3e9cf4d55ce171b844fad0931607f9`
- **D1 Database IDs:**
  - Development: `d2093e22-3df6-4746-b055-578102bc7946`
  - Production: `8d65abf7-297f-4c72-b2d0-0deb22beea6a`
- **KV Namespace IDs:**
  - SESSIONS: `4d8b3acab531437a8287a8cbb072d13c`
  - CACHE: `505518afdc54485688aaa3e8562d5e38`

**Action Required:**
- Create `wrangler.jsonc.template` with placeholder values
- Add actual `wrangler.jsonc` to `.gitignore`
- Document how to obtain these IDs for self-hosting

### 2. Hardcoded URLs
**Location:** `apps/api/src/index.ts` (lines 62-73)
```typescript
const ALLOWED_ORIGINS = [
  'http://localhost:3000',
  'http://localhost:5173',
  'https://workway.co',
  'https://www.workway.co',
  'https://workway-web.half-dozen.workers.dev',
];
```

**Action Required:**
- Move to environment variable: `ALLOWED_ORIGINS`
- Update code to parse comma-separated list

### 3. Environment Variables (Properly Externalized)
**Location:** `apps/api/src/types/env.ts`

These are already properly externalized and not hardcoded:
- ✅ `STRIPE_SECRET_KEY`
- ✅ `STRIPE_WEBHOOK_SECRET`
- ✅ `JWT_SECRET`
- ✅ `GOOGLE_CLIENT_ID`
- ✅ `GOOGLE_CLIENT_SECRET`
- ✅ `NOTION_CLIENT_ID`
- ✅ `NOTION_CLIENT_SECRET`

## Files Checked
- ✅ `apps/api/wrangler.jsonc` - Found account/database IDs
- ✅ `apps/web/wrangler.jsonc` - Needs review
- ✅ `apps/api/src/index.ts` - Found hardcoded URLs
- ✅ `apps/api/src/types/env.ts` - Properly externalized
- ✅ `.gitignore` - Properly excludes `.env*` and `.dev.vars`
- ✅ Integration files - No hardcoded secrets found

## Recommendations

### Before Open Sourcing

1. **Create Template Files:**
   ```bash
   # Create wrangler.jsonc.template
   cp apps/api/wrangler.jsonc apps/api/wrangler.jsonc.template
   # Replace actual IDs with placeholders
   ```

2. **Update .gitignore:**
   ```
   # Add to .gitignore
   apps/api/wrangler.jsonc
   apps/web/wrangler.jsonc
   ```

3. **Environment Variables:**
   - Create `.env.example` with all required variables (no values)
   - Document how to obtain OAuth credentials for each provider

4. **Update Code:**
   ```typescript
   // apps/api/src/index.ts
   const ALLOWED_ORIGINS = (env.ALLOWED_ORIGINS || 'http://localhost:3000')
     .split(',')
     .map(origin => origin.trim())
     .filter(Boolean);
   ```

5. **Add Pre-commit Hook:**
   Use `gitleaks` or similar tool to scan for secrets before commits

### Safe to Open Source
These components have no hardcoded secrets:
- ✅ SDK (`apps/api/src/sdk/`)
- ✅ Integration implementations (use env vars)
- ✅ Workflow engine (abstract from DB)
- ✅ CLI (configurable API URL)

### Keep Private
These should remain in the proprietary repository:
- ❌ Actual `wrangler.jsonc` files with real IDs
- ❌ `.env` files with real values
- ❌ Any deployment scripts with production URLs
- ❌ Stripe webhook endpoints with customer data

## Next Steps

1. Create template files with placeholders
2. Update code to use environment variables for URLs
3. Document self-hosting setup process
4. Run final scan with secret detection tool
5. Review all migration files for any customer data

## Verification Checklist
- [ ] All Cloudflare IDs replaced with placeholders
- [ ] All URLs moved to environment variables
- [ ] Template files created for configuration
- [ ] Documentation written for obtaining credentials
- [ ] Pre-commit hooks set up for secret scanning
- [ ] Final manual review of all files