# GitHub Repository Setup for WORKWAY Open Source

## Repository Structure

Based on our open source strategy (keep tools open, marketplace proprietary), we'll create these repositories:

### 1. `workway/workway` (Main Repository)
**Status**: PUBLIC ✅
**License**: Apache 2.0
**Description**: The official WORKWAY SDK, CLI, and workflow development tools

```
workway/
├── packages/
│   ├── sdk/              # TypeScript SDK (PUBLIC)
│   ├── cli/              # CLI tool (PUBLIC)
│   └── types/            # Shared types (PUBLIC)
├── examples/             # Example workflows (PUBLIC)
├── docs/                 # Documentation (PUBLIC)
├── .github/
│   └── workflows/        # GitHub Actions
├── LICENSE               # Apache 2.0
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── README.md
└── package.json
```

### 2. `workway/examples` (Workflow Examples)
**Status**: PUBLIC ✅
**License**: Apache 2.0
**Description**: Production-ready workflow examples and templates

```
examples/
├── gmail-to-notion/
├── slack-notifications/
├── stripe-webhooks/
├── ai-email-assistant/
├── invoice-processor/
├── customer-success/
├── content-distribution/
└── README.md
```

### 3. `workway/marketplace` (PRIVATE - Not for Open Source)
**Status**: PRIVATE 🔒
**License**: Proprietary
**Description**: The marketplace platform, API, and web application

```
marketplace/
├── apps/
│   ├── api/              # API server (PRIVATE)
│   └── web/              # Web app (PRIVATE)
├── infrastructure/       # Deployment configs (PRIVATE)
└── LICENSE-PROPRIETARY
```

## Files to Create for Each Repository

### Main Repository (`workway/workway`)

#### README.md
```markdown
# WORKWAY

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![npm version](https://img.shields.io/npm/v/@workway/sdk)](https://www.npmjs.com/package/@workway/sdk)
[![CI](https://github.com/workway/workway/actions/workflows/ci.yml/badge.svg)](https://github.com/workway/workway/actions/workflows/ci.yml)

Build and monetize TypeScript workflows on the edge. WORKWAY is the first marketplace for TypeScript workflow automation.

## 🚀 Quick Start

```bash
# Install the CLI
npm install -g @workway/cli

# Create a new workflow
workway init my-workflow

# Test locally
workway dev

# Deploy to marketplace
workway deploy
```

## 📦 Packages

- **[@workway/sdk](packages/sdk)** - TypeScript SDK for building workflows
- **[@workway/cli](packages/cli)** - Command-line tool for development
- **[@workway/types](packages/types)** - Shared TypeScript types

## 🔧 Features

- ✅ TypeScript-first with full type safety
- ✅ 50+ AI models via Cloudflare Workers AI
- ✅ Built-in OAuth for Gmail, Slack, Notion, etc.
- ✅ Edge deployment with <50ms cold starts
- ✅ Local development and testing
- ✅ Monetize through the marketplace

## 📖 Documentation

- [Getting Started](docs/getting-started.md)
- [API Reference](docs/api-reference.md)
- [Building Workflows](docs/workflows.md)
- [Deploying to Production](docs/deployment.md)

## 💰 Business Model

Developers keep 100% of upfront fees. Platform charges 5¢ (light) or 25¢ (heavy) per execution.

## 🤝 Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

Apache 2.0 - see [LICENSE](LICENSE)
```

### GitHub Actions Workflow

#### `.github/workflows/ci.yml`
```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build packages
      run: npm run build

    - name: Lint code
      run: npm run lint

  publish:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-node@v4
      with:
        node-version: '18.x'
        registry-url: 'https://registry.npmjs.org'

    - run: npm ci
    - run: npm run build

    - name: Publish to npm
      run: npm publish --access public
      env:
        NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}
```

### Examples Repository Structure

#### `examples/README.md`
```markdown
# WORKWAY Workflow Examples

Production-ready workflow templates you can fork, customize, and deploy.

## 📚 Available Examples

### Email & Communication
- [Gmail to Notion](./gmail-to-notion) - Sync emails to Notion database
- [Email AI Assistant](./ai-email-assistant) - AI-powered email responses

### Business Automation
- [Invoice Processor](./invoice-processor) - Extract data from invoices
- [Customer Success](./customer-success) - Monitor customer health

### Developer Tools
- [Code Review Bot](./code-review-bot) - AI code reviews for PRs
- [GitHub to Linear](./github-to-linear) - Sync issues between platforms

## 🚀 Using These Examples

1. **Clone the example**:
```bash
git clone https://github.com/workway/examples
cd examples/gmail-to-notion
```

2. **Install dependencies**:
```bash
npm install
```

3. **Configure integrations**:
```bash
cp .env.example .env
# Add your API keys
```

4. **Test locally**:
```bash
npm run dev
```

5. **Deploy to marketplace**:
```bash
workway deploy
```

## 💰 Monetization

Each example includes suggested pricing:
- **Simple workflows**: $29-49/month
- **Complex workflows**: $99-199/month
- **Enterprise workflows**: $299+/month

## 📄 License

All examples are Apache 2.0 licensed. Free to use and modify!
```

## Repository Creation Commands

```bash
# 1. Create main repository
gh repo create workway/workway --public \
  --description "Build and monetize TypeScript workflows on the edge" \
  --license apache-2.0 \
  --add-readme

# 2. Create examples repository
gh repo create workway/examples --public \
  --description "Production-ready WORKWAY workflow examples" \
  --license apache-2.0 \
  --add-readme

# 3. Create private marketplace repository
gh repo create workway/marketplace --private \
  --description "WORKWAY marketplace platform (proprietary)" \
  --add-readme
```

## Files to Exclude from Open Source

### `.gitignore` for public repos
```gitignore
# Environment
.env
.env.*
!.env.example

# Private keys
*.pem
*.key
*.cert

# Proprietary
/apps/api/
/apps/web/
/infrastructure/

# Secrets
.secrets/
credentials.json
```

## NPM Package Configuration

### `packages/sdk/package.json`
```json
{
  "name": "@workway/sdk",
  "version": "1.0.0",
  "description": "WORKWAY SDK for building TypeScript workflows",
  "license": "Apache-2.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "repository": {
    "type": "git",
    "url": "https://github.com/workway/workway.git",
    "directory": "packages/sdk"
  },
  "keywords": [
    "workway",
    "workflows",
    "automation",
    "typescript",
    "cloudflare",
    "edge"
  ],
  "publishConfig": {
    "access": "public"
  }
}
```

## Security Considerations

### Secrets to Never Commit
- ❌ Cloudflare Account IDs
- ❌ API tokens and keys
- ❌ Database IDs
- ❌ OAuth client secrets
- ❌ Stripe keys
- ❌ JWT secrets

### Environment Template (`.env.example`)
```bash
# Cloudflare
CLOUDFLARE_ACCOUNT_ID=your-account-id
CLOUDFLARE_API_TOKEN=your-api-token

# Database
DATABASE_URL=your-database-url

# OAuth (get from provider dashboards)
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# Never commit actual values!
```

## Open Source Launch Checklist

### Pre-Launch
- [ ] Remove all hardcoded credentials
- [ ] Review code for sensitive data
- [ ] Update all documentation
- [ ] Prepare demo workflows
- [ ] Test installation process
- [ ] Set up Discord/community

### Launch Day
- [ ] Create GitHub repositories
- [ ] Push initial code
- [ ] Publish NPM packages
- [ ] Tweet announcement
- [ ] Post on Hacker News
- [ ] Update website with GitHub links

### Post-Launch
- [ ] Monitor issues and PRs
- [ ] Engage with community
- [ ] Regular releases
- [ ] Update documentation

## Community Links

After creating repos, update these links:
- GitHub: https://github.com/workway
- NPM: https://www.npmjs.com/org/workway
- Discord: https://discord.gg/workway
- Twitter: https://twitter.com/workway
- Website: https://workway.co

## Summary

The open source strategy:
1. **Open**: SDK, CLI, examples, documentation
2. **Closed**: Marketplace platform, billing, analytics
3. **Revenue**: Through marketplace transactions
4. **Community**: Build ecosystem of developers

This allows developers to build and test locally while monetizing through our marketplace.