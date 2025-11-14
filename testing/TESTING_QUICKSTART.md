# WORKWAY Testing - Quick Start Guide

## 🚀 Run Tests in 3 Steps

### Step 1: Navigate to Web App
```bash
cd apps/web
```

### Step 2: Install Dependencies (if needed)
```bash
npm install
```

### Step 3: Run Tests
```bash
npm test
```

That's it! You should see tests running and data flowing through the system.

## 📊 Expected Output

```
╔════════════════════════════════════════╗
║  WORKWAY Integration Test Suite       ║
║  Testing data flow through system     ║
╚════════════════════════════════════════╝

✓ Authentication Flow (3)
  ✓ should register a new user
  ✓ should login with registered user
  ✓ should get current user session

✓ Integrations Marketplace (4)
  ✓ should list available integrations
  ✓ should get integration details
  ✓ should search integrations
  ✓ should filter by category

✓ OAuth Flow & Installation (4)
  ✓ should select an integration for installation
  ✓ should initiate OAuth flow
  ✓ should simulate OAuth callback and create installation
  ✓ should list user installations

✓ Workflow Creation & Execution (6)
  ✓ should list available workflow templates
  ✓ should create a new workflow from template
  ✓ should list user workflows
  ✓ should trigger workflow execution
  ✓ should get execution details and logs
  ✓ should get workflow analytics

✓ Error Handling & Edge Cases (4)
✓ Data Flow End-to-End (1)

Test Files  1 passed (1)
     Tests  25 passed (25)
  Start at  14:30:00
  Duration  5.42s
```

## 🎯 What Gets Tested

### 1. User Registration & Login
```
Input:  { email, password }
        ↓
Output: { userId, token }
```

### 2. Browse Integrations
```
Input:  { search: 'email', category: 'productivity' }
        ↓
Output: { integrations: [...] }
```

### 3. Install Integration (OAuth)
```
Input:  { integrationId, provider }
        ↓
OAuth:  Authorization flow
        ↓
Output: { installationId, status: 'active' }
```

### 4. Create Workflow
```
Input:  { name, trigger, actions }
        ↓
Output: { workflowId, status: 'active' }
```

### 5. Execute Workflow & Verify Data
```
Input:  { workflowId, testData }
        ↓
Execution: Data transformation
        ↓
Output: { logs: [...], analytics: {...} }
```

## 🔍 Common Commands

### Run specific test
```bash
npm test -- -t "should register a new user"
```

### Watch mode (auto-rerun on changes)
```bash
npm run test:watch
```

### Generate coverage report
```bash
npm run test:coverage
```

### Verbose output
```bash
npm test -- --reporter=verbose
```

## 🐛 Troubleshooting

### Tests timing out?
Increase timeout in `vitest.config.ts`:
```typescript
testTimeout: 60000 // 60 seconds
```

### API connection issues?
Set the correct API URL:
```bash
export TEST_API_URL=http://localhost:8787
```

### Need to debug?
Add `console.log()` in tests or use:
```bash
npm test -- --inspect-brk
```

## 📁 Test Files Location

```
apps/web/src/tests/
├── integration.test.ts    # Main test suite
├── test-utils.ts          # Helper functions
└── setup.ts               # Global configuration
```

## 📖 More Information

- **Full Documentation:** See `INTEGRATION_TESTING.md`
- **Implementation Summary:** See `TEST_IMPLEMENTATION_SUMMARY.md`
- **Vitest Docs:** https://vitest.dev/

## ✅ Success Criteria

All tests passing means:
- ✅ User authentication works
- ✅ Integration marketplace loads
- ✅ OAuth flow completes
- ✅ Installations are created
- ✅ Workflows can be configured
- ✅ Workflows execute correctly
- ✅ Data flows through system
- ✅ Logs capture transformations
- ✅ Analytics are generated
- ✅ Error handling works

## 🎉 Next Steps

1. Run the tests: `npm test`
2. Check the output
3. Review logs showing data flow
4. Explore test code in `src/tests/`
5. Add your own tests!

---

**Need help?** Check the full documentation in `INTEGRATION_TESTING.md`
