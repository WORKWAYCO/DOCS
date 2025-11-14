# WORKWAY - Open Source Workflow Automation Platform

<div align="center">
  <h3>Build and deploy workflow automations on Cloudflare's edge network</h3>
  <p>
    <a href="https://github.com/workway/workway/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License"></a>
    <a href="https://www.npmjs.com/package/@workway/sdk"><img src="https://img.shields.io/npm/v/@workway/sdk.svg" alt="npm version"></a>
    <a href="https://docs.workway.dev"><img src="https://img.shields.io/badge/docs-available-brightgreen.svg" alt="Documentation"></a>
    <a href="https://discord.gg/workway"><img src="https://img.shields.io/discord/123456789?color=7289da&label=Discord" alt="Discord"></a>
  </p>
</div>

## What is WORKWAY?

WORKWAY is an open source workflow automation platform that runs on Cloudflare Workers, providing a cost-effective and highly scalable alternative to traditional workflow automation tools. Build integrations between your favorite services, automate repetitive tasks, and create powerful workflows that run at the edge.

## 🏗️ Monorepo Structure

```
WORKWAY-Cloudflare/
├── apps/
│   ├── api/           # Cloudflare Workers API backend
│   └── web/           # TanStack Start frontend (coming soon)
├── packages/
│   └── shared/        # Shared types & utilities (coming soon)
└── turbo.json         # Turborepo configuration
```

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- npm 10+
- Cloudflare account (free tier works!)

### Installation

```bash
# Install all dependencies
npm install

# Set up environment variables
cp apps/api/.dev.vars.example apps/api/.dev.vars
# Edit apps/api/.dev.vars with your keys

# Run the development server
npm run dev
```

This will start all apps in development mode:
- API: http://localhost:8787
- Web (coming soon): http://localhost:3000

### Available Commands

```bash
npm run dev       # Start all apps in development mode
npm run build     # Build all apps
npm run deploy    # Deploy all apps to Cloudflare
npm run test      # Run all tests
npm run lint      # Lint all packages
npm run clean     # Clean all build artifacts
npm run format    # Format code with Prettier
```

## 📦 Applications

### API (`apps/api`)
Cloudflare Workers-based REST API with:
- **68 API endpoints** covering users, integrations, marketplace, reviews, workflows
- **Stripe Connect** integration for payments
- **OAuth 2.0** support (Gmail, Notion, and extensible to other providers)
- **Cloudflare Workflows** engine with Durable Objects for reliable automation execution
- **D1 Database** for relational data storage

[View API Documentation →](./apps/api/ORIGINAL_README.md)

### Web Frontend (`apps/web`) - Coming Soon
TanStack Start-based marketplace frontend with:
- Server-side rendering on Cloudflare Pages
- Type-safe API integration
- Modern UI/UX for browsing and installing integrations

## 💰 Cost Analysis

Running on Cloudflare is **dramatically cheaper** than traditional cloud platforms:

| Service | Free Tier | After Free Tier | Typical Monthly Cost |
|---------|-----------|-----------------|---------------------|
| Workers (API) | 100,000 req/day | $0.50/million requests | **$0-5** |
| D1 Database | 5GB storage, 5M reads/day | $0.75/GB, $0.001/1K reads | **$0-3** |
| Pages (Frontend) | 500 builds/month | $5/500 builds | **$0** |
| Workflows (NEW) | 10M steps/month | $0.02/million steps | **$0-1** |
| **Total** | - | - | **$0-9/month** 🎉 |

**vs. AWS/GCP/Azure**: $50-200/month for equivalent services

## 🛠️ Tech Stack

### Backend
- **Runtime**: Cloudflare Workers (V8 isolates, not containers)
- **Framework**: Hono.js (ultra-fast edge router)
- **Database**: Cloudflare D1 (SQLite at the edge)
- **Storage**: Cloudflare R2 (coming soon for file uploads)
- **Queues**: Cloudflare Workflows with Durable Objects
- **Language**: TypeScript
- **Validation**: Zod schemas
- **Payments**: Stripe Connect

### Frontend (Coming Soon)
- **Framework**: TanStack Start (RSC on the edge)
- **Hosting**: Cloudflare Pages
- **Styling**: TailwindCSS or UnoCSS
- **State**: TanStack Query
- **Forms**: TanStack Form with Zod

### DevOps
- **Monorepo**: Turborepo
- **Package Manager**: npm workspaces
- **CI/CD**: GitHub Actions (coming soon)
- **Deployment**: Wrangler CLI

## 📚 Documentation

### Frontend Fixes & Analysis
- **[Quick Reference](./docs/QUICK_REFERENCE.md)** - Fast lookup guide for recent fixes
- **[Fixes Summary](./docs/FIXES_SUMMARY.md)** - What was completed
- **[Fix Roadmap](./docs/FIX_ROADMAP.md)** - Detailed roadmap with remaining work
- **[All Documentation](./docs/)** - Complete technical documentation

### API Documentation
- [API Documentation](./apps/api/ORIGINAL_README.md)
- [Quick Start Guide](./apps/api/QUICK_START.md)
- [API Testing Guide](./apps/api/API_TESTING_GUIDE.md)
- [Project Status](./apps/api/PROJECT_STATUS.md)

## 🔑 Environment Variables

Each app has its own environment configuration:

### API (`apps/api/.dev.vars`)
```env
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
GMAIL_CLIENT_ID=...
GMAIL_CLIENT_SECRET=...
NOTION_CLIENT_ID=...
NOTION_CLIENT_SECRET=...
```

See [apps/api/.dev.vars.example](./apps/api/.dev.vars.example) for full list.

## 🧪 Testing

```bash
# Run all tests
npm run test

# Run tests for specific app
npm run test --filter=@workway/api
```

## 📦 Database Migrations

```bash
# Create a migration
cd apps/api
npx wrangler d1 migrations create marketplace-db <migration-name>

# Apply migrations to local DB
npx wrangler d1 migrations apply marketplace-db --local

# Apply migrations to production
npx wrangler d1 migrations apply marketplace-db --remote
```

## 🚢 Deployment

### Deploy Everything
```bash
npm run deploy
```

### Deploy Specific App
```bash
npm run deploy --filter=@workway/api
```

### Manual Deployment
```bash
cd apps/api
npx wrangler deploy
```

## 🎯 Roadmap

- [x] Complete REST API with 68 endpoints
- [x] Stripe Connect integration
- [x] OAuth 2.0 (Gmail, Notion)
- [x] Cloudflare Workflows engine
- [x] Monorepo setup with Turborepo
- [ ] TanStack Start frontend
- [ ] Shared type packages
- [ ] CI/CD pipeline
- [ ] Production deployment
- [ ] Custom domain setup
- [ ] OAuth for more providers (Slack, Discord, etc.)
- [ ] R2 storage for file uploads
- [ ] Email notifications (Resend or SendGrid)
- [ ] Analytics dashboard

## 📝 License

MIT

## 🤝 Contributing

This is a private monorepo. For questions or issues, contact the development team.

---

**Built with ❤️ using Cloudflare's edge platform**
