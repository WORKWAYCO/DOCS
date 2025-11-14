# WORKWAY Integration Testing Guide

## Overview

This document describes the comprehensive integration testing suite for WORKWAY that demonstrates data flowing through the entire system from authentication to workflow execution.

## Test Architecture

### Test Files

- **`apps/web/src/tests/integration.test.ts`** - Main integration test suite
- **`apps/web/src/tests/test-utils.ts`** - Test utilities and helpers
- **`apps/web/src/tests/setup.ts`** - Global test configuration
- **`apps/web/vitest.config.ts`** - Vitest configuration

## Running Tests

### Run All Tests
```bash
cd apps/web
npm test
```

### Run Tests in Watch Mode
```bash
npm run test:watch
```

### Run Tests with Coverage
```bash
npm run test:coverage
```

### Run Specific Test Suite
```bash
npm test -- integration.test.ts
```

## Test Coverage

The integration test suite covers the following system components:

### 1. Authentication Flow ✓
- User registration
- User login
- Session validation
- Logout

**Data Flow:**
```
User Input → API Request → Database → Session Token → Client
```

### 2. Integrations Marketplace ✓
- List available integrations
- Get integration details
- Search integrations
- Filter by category

**Data Flow:**
```
Search Query → API → Database → Integration List → UI
```

### 3. OAuth Flow & Installation ✓
- Select integration
- Initiate OAuth
- Handle OAuth callback
- Create installation
- List installations
- Get installation details

**Data Flow:**
```
Integration Selection → OAuth Provider → Authorization Code →
Token Exchange → Installation Record → Active Connection
```

### 4. Workflow Creation & Execution ✓
- List workflow templates
- Create workflow from template
- List user workflows
- Trigger workflow execution
- Get execution details
- View execution logs
- Get workflow analytics

**Data Flow:**
```
Template → Workflow Config → Trigger Event →
Action Execution → Data Transformation → Logs → Analytics
```

### 5. Error Handling ✓
- Invalid authentication
- Nonexistent resources
- Missing required fields
- Rate limiting

### 6. End-to-End Data Flow ✓
Complete system demonstration showing data moving through:
- Integration → Installation → Workflow → Execution

## Test Scenarios

### Scenario 1: New User Onboarding
```typescript
1. Register new user
   Input: { email, password, displayName }
   Output: { userId, token }

2. Login user
   Input: { email, password }
   Output: { token, user }

3. Validate session
   Input: { token }
   Output: { user }
```

### Scenario 2: Installing an Integration
```typescript
1. Browse marketplace
   Input: { search: 'email', category: 'productivity' }
   Output: { integrations: [...] }

2. Select integration
   Input: { integrationId }
   Output: { integration }

3. Initiate OAuth
   Input: { integrationId, provider }
   Output: { authUrl }

4. Complete OAuth
   Input: { code, state }
   Output: { installationId, status: 'active' }
```

### Scenario 3: Creating and Running a Workflow
```typescript
1. Select template
   Input: { templateId }
   Output: { template }

2. Create workflow
   Input: {
     name,
     trigger: { type, integrationId, config },
     actions: [{ type, config }, ...]
   }
   Output: { workflowId, status: 'active' }

3. Trigger execution
   Input: { workflowId, testData }
   Output: { executionId, status }

4. View execution logs
   Input: { executionId }
   Output: {
     logs: [
       { step: 'trigger', data: {...} },
       { step: 'action_0', data: {...} },
       { step: 'completed', data: {...} }
     ]
   }
```

## Data Transformation Examples

### Example 1: Email to Slack Workflow

**Input Data (Trigger):**
```json
{
  "type": "email_received",
  "from": "customer@example.com",
  "subject": "Order Inquiry",
  "body": "I have a question about order #12345",
  "timestamp": "2025-11-09T14:30:00Z"
}
```

**Transformation (Workflow):**
```json
{
  "trigger": {
    "type": "email_received",
    "integrationId": "gmail_inst_123"
  },
  "actions": [
    {
      "type": "extract_data",
      "config": {
        "pattern": "order #(\\d+)",
        "output": "orderId"
      }
    },
    {
      "type": "send_notification",
      "integrationId": "slack_inst_456",
      "config": {
        "channel": "#support",
        "message": "New inquiry for order {{orderId}} from {{from}}"
      }
    }
  ]
}
```

**Output Data (Execution Log):**
```json
{
  "executionId": "exec_789",
  "status": "completed",
  "steps": [
    {
      "step": "trigger",
      "status": "completed",
      "data": {
        "from": "customer@example.com",
        "subject": "Order Inquiry"
      }
    },
    {
      "step": "extract_data",
      "status": "completed",
      "data": {
        "orderId": "12345"
      }
    },
    {
      "step": "send_notification",
      "status": "completed",
      "data": {
        "messageId": "slack_msg_xyz",
        "channel": "#support",
        "sent": true
      }
    }
  ]
}
```

## Test Utilities

### Mock Data Generators
```typescript
import { createMockIntegration, createMockInstallation } from './test-utils';

// Create mock integration
const integration = createMockIntegration({
  name: 'Gmail',
  category: 'productivity'
});

// Create mock installation
const installation = createMockInstallation({
  integrationId: integration.id,
  status: 'active'
});
```

### Async Helpers
```typescript
import { waitFor, retry, measurePerformance } from './test-utils';

// Wait for condition
await waitFor(() => installation.status === 'active', {
  timeout: 5000,
  interval: 100
});

// Retry with backoff
const result = await retry(() => api.getWorkflow(id), {
  retries: 3,
  delay: 1000,
  backoff: 2
});

// Measure performance
const { result, duration } = await measurePerformance('API Call', async () => {
  return await api.listIntegrations();
});
```

### Data Validation
```typescript
import {
  verifyResponseStructure,
  verifyDataTransformation,
  validateExecutionLogs
} from './test-utils';

// Verify response structure
verifyResponseStructure(response, ['id', 'name', 'status']);

// Verify data transformation
verifyDataTransformation(logs, [
  { step: 'trigger', expectedData: { type: 'webhook' } },
  { step: 'action_0', expectedData: { status: 'completed' } }
]);

// Validate execution logs
const isValid = validateExecutionLogs(logs);
```

## Performance Benchmarks

### Expected Response Times

| Operation | Expected Time | Threshold |
|-----------|--------------|-----------|
| User Registration | < 500ms | 1000ms |
| User Login | < 300ms | 500ms |
| List Integrations | < 200ms | 500ms |
| OAuth Initiate | < 400ms | 800ms |
| Create Workflow | < 600ms | 1200ms |
| Trigger Execution | < 1000ms | 2000ms |
| Get Execution Logs | < 300ms | 600ms |

### Load Testing

The test suite includes basic load testing:
- 10 concurrent requests to integration listing
- Verifies rate limiting is enforced
- Ensures system remains stable under load

## Continuous Integration

### GitHub Actions Workflow

```yaml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm install
      - run: npm test
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3
```

## Debugging Failed Tests

### View Detailed Logs
```bash
npm test -- --reporter=verbose
```

### Run Single Test
```bash
npm test -- -t "should register a new user"
```

### Debug Mode
```bash
node --inspect-brk ./node_modules/vitest/vitest.mjs run
```

## Test Data Cleanup

All tests include automatic cleanup:
- Delete created workflows
- Delete installations
- Logout sessions
- Clean test data

Cleanup runs in `afterAll()` hook to ensure no test data pollution.

## Common Issues & Solutions

### Issue 1: Tests Timing Out
**Solution:** Increase timeout in vitest.config.ts
```typescript
testTimeout: 60000 // 60 seconds
```

### Issue 2: API Connection Refused
**Solution:** Ensure API is running on correct port
```bash
# Start API server first
cd apps/api
npm run dev

# Then run tests
cd apps/web
npm test
```

### Issue 3: Authentication Failures
**Solution:** Check test user credentials and session handling
```typescript
// Ensure login is called before protected endpoints
await authAPI.login(testUser);
```

## Next Steps

1. **Add More Test Scenarios:**
   - Multi-step workflows
   - Error recovery
   - Concurrent executions
   - Webhook retries

2. **Performance Testing:**
   - Load testing with K6
   - Stress testing
   - Endurance testing

3. **E2E Testing:**
   - Playwright for UI testing
   - Full user journey testing
   - Cross-browser testing

4. **Monitoring:**
   - Add test result tracking
   - Performance regression detection
   - Alerting on test failures

## Resources

- [Vitest Documentation](https://vitest.dev/)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [Integration Testing Guide](https://martinfowler.com/bliki/IntegrationTest.html)
