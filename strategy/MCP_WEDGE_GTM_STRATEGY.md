# MCP-Wedge Go-To-Market Strategy

> **Zuhandenheit**: The tool recedes; the outcome remains.

---

## Executive Summary

**MCP is the wedge, not the product.**

WORKWAY's go-to-market strategy uses open-source MCP (Model Context Protocol) servers as the trust-building entry point. The intelligence layer—Skills that deliver real construction outcomes—is the premium product.

The strategy is simple:
1. **Give away connectivity** (open-source Procore MCP server)
2. **Earn trust** (users audit the code, credentials never touch our servers)
3. **Sell outcomes** (Skills that draft RFIs, summarize logs, flag compliance)

This document outlines the complete GTM strategy for WORKWAY's construction vertical, from MCP wedge through enterprise deals.

---

## MCP-First Thesis

### The Entry Point is Connectivity, Not Intelligence

Most AI startups lead with "AI-powered" features. We lead with **connectivity**.

Why? Because construction teams don't trust AI vendors with their Procore data. They need to see the code. They need to verify nothing suspicious is happening. They need an escape hatch.

**MCP provides all three:**
- Open-source code they can audit
- OAuth means credentials never touch our servers
- Standard protocol they can switch away from

Once trust is established through connectivity, we earn the right to sell intelligence.

### Open-Source Builds Trust

The Procore MCP server is fully open-source. Anyone can:
- Read every line of code
- Fork and modify it
- Self-host if they prefer
- Verify exactly what data flows where

**This transparency is our competitive advantage.** Competitors hide their integrations. We expose ours.

### Low Friction Activation

**50 free runs. No account required.**

Users can experience WORKWAY value before any commitment:
- No credit card
- No account creation
- No sales call

After 50 runs, they've seen the value. Conversion becomes natural, not forced.

---

## The Two-Layer Model

```
┌─────────────────────────────────────────────────────────────┐
│ INTELLIGENCE LAYER                                          │
│ Skills, Agents, Automations (the upsell)                    │
│ "Draft this RFI" · "Summarize daily logs" · "Flag risk"     │
├─────────────────────────────────────────────────────────────┤
│ AUTOMATION LAYER                                            │
│ MCP Servers (the entry point)                               │
│ Connect Procore to AI with trust boundaries                 │
└─────────────────────────────────────────────────────────────┘
```

### Automation Layer (MCP)

**Purpose**: Connect construction tools to AI with trust boundaries.

| Characteristic | Value |
|----------------|-------|
| **Pricing** | Free / low-cost |
| **Code** | Open-source |
| **Trust Model** | OAuth (credentials never touch WORKWAY) |
| **Data Flow** | Transparent, auditable |
| **Target User** | IT/DevOps evaluating AI tools |

**What it does:**
- Connects Procore to Claude, Cursor, other AI tools
- Provides structured access to projects, RFIs, daily logs, submittals
- Respects Procore's permission model
- Works with any MCP-compatible client

**What it doesn't do:**
- Generate content
- Make decisions
- Store construction data
- Require WORKWAY account

### Intelligence Layer (Skills)

**Purpose**: Deliver construction outcomes, not just data access.

| Characteristic | Value |
|----------------|-------|
| **Pricing** | Per-run (premium) |
| **Code** | Proprietary |
| **Trust Model** | Built on MCP foundation |
| **Data Flow** | Input → Outcome → Output |
| **Target User** | Project managers, superintendents |

**What it does:**
- Draft RFIs from meeting notes
- Summarize daily logs across projects
- Flag compliance issues before they escalate
- Prepare pay application documentation
- Generate submittal packages

**What it doesn't do:**
- Require understanding of AI
- Need technical implementation
- Replace human judgment (augments it)

### Clear Upgrade Path (Three-Tier Funnel)

```
Week 1: IT installs Procore MCP (free)              → DATABASE LAYER
        ↓ "This actually works -- and it's open source"
Week 2: PM tries draft_rfi skill (50 free runs)      → AUTOMATION LAYER
        ↓ "This saved me 2 hours -- and it pulls from Zoom too"
Week 3: Team adopts cross-system daily log digest     → AUTOMATION LAYER
        ↓ "We need this for all projects"
Week 4: PM asks "can we set our own compliance rules?"→ JUDGMENT LAYER
        ↓ "We need governance per project type"
Week 5: Enterprise conversation begins                → FULL STACK
```

Each tier creates demand for the next:
- **MCP wedge** earns trust (free connectivity)
- **Skills Marketplace** delivers outcomes (per-run cross-system automation)
- **Policy Marketplace** captures governance (per-project policy artifacts)
- **Enterprise** bundles everything (per-seat + governance infrastructure)

The MCP wedge earns trust. Skills deliver value. Policy captures governance. Enterprise captures revenue.

---

## Procore as Trust-Builder

### Why Procore First

| Factor | Rationale |
|--------|-----------|
| **Market Size** | 17,000+ customers, $1B+ ARR, 2M+ daily users |
| **API Access** | Comprehensive, well-documented REST API |
| **Pain Points** | RFI backlog, daily log synthesis, compliance tracking |
| **Helix as Catalyst** | Procore Helix proves AI demand. Buyers now understand AI -- they need cross-system reach and policy control that Helix can't offer. |
| **Control Model Gap** | Procore Helix is Procore-controlled end-to-end. Construction teams need user-controlled governance. MCP provides this. |
| **Trust Requirements** | High -- perfect for open-source wedge |

Procore customers are sophisticated enough to evaluate open-source code and paranoid enough to appreciate it. With Helix, they've now *experienced* AI -- making the conversation about differentiation, not category creation.

### Open-Source MCP Establishes Credibility

When a GC's IT team evaluates WORKWAY, they can:

1. **Clone the repo** and read every function
2. **Run locally** without our servers involved
3. **Verify OAuth flow** doesn't expose credentials
4. **Check data handling** for compliance requirements
5. **Modify for custom needs** without vendor dependency

This transparency converts skeptics into advocates.

### OAuth Means Zero Credential Risk

**Traditional integration**: Customer provides API keys → Vendor stores them → Security liability

**WORKWAY integration**: OAuth flow → Tokens stay with customer → No credential exposure

We never see, store, or have access to Procore credentials. This eliminates the primary objection from security teams.

---

## Construction as Primary Vertical

### Why Construction First

**Not generic automation. Construction automation.**

| Reason | Impact |
|--------|--------|
| **Domain complexity** | Generic tools can't handle construction semantics |
| **High pain, high value** | RFI delays cost real money |
| **Relationship-driven** | Trust matters more than features |
| **Underserved by AI** | No serious competitors yet |
| **Expansion potential** | Success here proves the model |

Construction is hard. That's the point. If we can deliver AI outcomes in construction—with its documentation requirements, liability concerns, and trust barriers—we can expand anywhere.

### Domain Expertise as Moat

Generic AI tools don't understand:
- The difference between an RFI and an RFC
- Why daily logs matter for disputes
- How pay applications flow through approval
- What "substantial completion" means contractually

**WORKWAY Skills speak construction.** Tool names use domain language: `draft_rfi`, not `generate_text`. Outputs match industry formats. Workflows respect construction's approval hierarchies.

This domain expertise compounds. Each Skill teaches the next. Each customer interaction deepens understanding. Competitors starting from generic will never catch up.

### Expansion Path

```
Phase 1: Construction (now)
         └─ Procore MCP → Construction Skills
         
Phase 2: Adjacent Verticals (after $10M ARR)
         └─ Engineering → Autodesk MCP
         └─ Real Estate → Yardi MCP
         
Phase 3: Horizontal (after vertical dominance)
         └─ Platform for any industry vertical
```

We expand only after proving the model in construction. No premature generalization.

---

## Conversion Funnel

### Awareness

**Channels:**
- Open-source MCP repository (GitHub stars, forks)
- Developer community (Procore developer forums, construction tech Slack)
- Content marketing (construction AI use cases)
- Conference presence (Procore Groundbreak, AGC conventions)

**Message:**
> "Open-source MCP server for Procore. Connect your construction data to AI tools you trust."

**Metrics:**
- GitHub stars
- MCP installations
- Developer forum mentions

### Activation

**Mechanism:**
- 50 free runs
- No account required
- No credit card

**First Value:**
- User connects Procore via OAuth
- Runs first Skill (e.g., `summarize_daily_log`)
- Sees immediate time savings

**Metrics:**
- OAuth connections completed
- First Skill executed
- Return within 7 days

### Retention

**What keeps users:**
- Skills that deliver measurable outcomes
- Time savings they can quantify
- Outcomes that make them look good

**Proof points:**
- "Draft RFI" saves 2+ hours per RFI
- "Summarize daily logs" synthesizes 20+ logs in minutes
- "Flag compliance" catches issues before they escalate

**Metrics:**
- Weekly active Skill runs
- Skill variety (using 3+ different Skills)
- Team expansion (inviting colleagues)

### Revenue

**Per-run pricing:**

| Skill Type | Price/Run | Example |
|------------|-----------|---------|
| Standard | $0.03 | `list_projects` |
| Advanced | $0.15 | `draft_rfi` |
| Premium | $0.50 | `prepare_pay_app` |

**Enterprise deals:**
- Per-seat licensing for teams
- Volume discounts for high-run customers
- Priority support and SLAs
- Custom Skill development

**Metrics:**
- Monthly recurring revenue
- Average revenue per user
- Enterprise conversion rate

---

## Messaging Framework

### For Construction Teams

**Pain**: RFIs take too long. Daily logs pile up. Compliance issues surface too late.

**Promise**: "RFIs that draft themselves."

**Proof**: "Draft RFI skill reduces RFI creation time from 2 hours to 15 minutes."

**Call to action**: "Connect Procore. Run 50 Skills free."

### For Developers / IT

**Pain**: Evaluating AI tools is risky. Vendors want your credentials. Code is hidden.

**Promise**: "Open-source MCP you can trust."

**Proof**: "Full source code on GitHub. OAuth means credentials never leave your systems."

**Call to action**: "Clone the repo. Run locally. Audit everything."

### For Enterprise / Executives

**Pain**: AI promises don't deliver. Security teams block everything. ROI is unclear.

**Promise**: "Automation infrastructure with trust boundaries."

**Proof**: "Open-source foundation. SOC 2 compliant. Per-run pricing with cost transparency."

**Call to action**: "Pilot with one project. Measure the results."

### Message Examples by Audience

| Audience | Wrong | Right |
|----------|-------|-------|
| PM | "AI-powered workflow automation" | "RFIs that draft themselves while you sleep" |
| IT | "Enterprise integration platform" | "Audit the code. We have nothing to hide." |
| Exec | "Digital transformation solution" | "Cut RFI creation time by 75%. Here's the math." |

---

## What We DON'T Say

### Banned Phrases

| Phrase | Why Not | Instead Say |
|--------|---------|-------------|
| "Workflow automation" | Too generic, Zapier territory | "Construction automation" or specific outcome |
| "AI-powered" | Unless specifically about Skills | Focus on outcomes, not technology |
| "Better than [competitor]" | Different weight class | "Built for construction teams" |
| "Digital transformation" | Meaningless buzzword | Specific outcome with metrics |
| "Enterprise-grade" | Empty marketing speak | Specific capabilities (SOC 2, SLAs) |
| "Seamless integration" | Everyone says this | "Open-source. Audit it yourself." |

### Positioning Discipline

**We are:**
- MCP server provider (open-source, trustworthy)
- Construction Skills developer (domain expert)
- Automation infrastructure (platform, not point solution)

**We are not:**
- Zapier competitor (different market, different user)
- General-purpose AI (construction-specific)
- Procore replacement (complementary, not competitive)

### The Zuhandenheit Test

Before any external communication, apply the test:

> Can you describe the value without mentioning technology?

**Wrong**: "Our MCP server provides OAuth-based connectivity to Procore's REST API, enabling AI tools to access construction data through the Model Context Protocol."

**Right**: "RFIs that draft themselves. Daily logs that summarize themselves. Compliance issues that flag themselves."

The technology should be invisible. The outcome should be obvious.

---

## Competitive Positioning

### Who We're Not Competing With

| Competitor | Why Not |
|------------|---------|
| Zapier | Different user (non-technical), different use case (simple automation) |
| Make | Same as Zapier—visual builders for marketers |
| Procore | Complementary, not competitive—we make Procore more valuable |
| Generic AI tools | They don't understand construction |

### Our Actual Competition

| Alternative | Our Advantage |
|-------------|---------------|
| Manual processes | 10x faster, zero errors |
| Custom development | Instant deployment, no maintenance |
| Consultants | Always available, per-run pricing |
| Doing nothing | Quantifiable time savings |

### Moat Development

1. **Trust Moat**: Open-source MCP establishes credibility competitors can't replicate without similar transparency
2. **Domain Moat**: Construction expertise compounds with each customer
3. **Ecosystem Moat**: Skills marketplace creates network effects
4. **Data Moat**: Usage patterns inform better Skills (with consent)

---

## Implementation Roadmap

### Phase 1: MCP Foundation (Now)

- [x] Procore MCP server (open-source)
- [x] OAuth flow implementation
- [x] Basic Skills (list_projects, get_rfi, get_daily_log)
- [ ] GitHub presence and documentation
- [ ] Developer community seeding

### Phase 2: Cross-System Skills Library (Next 90 Days)

- [ ] `draft_rfi_from_meeting` Skill -- Zoom transcript + Procore context + P6 schedule
- [ ] `daily_log_digest` Skill -- Procore logs + Slack + email in one briefing
- [ ] `change_order_impact` Skill -- Procore changes + Sage budgets + P6 schedules
- [ ] 50 free runs mechanism
- [ ] Usage analytics

### Phase 3: Judgment Layer + Enterprise Motion (90-180 Days)

- [ ] Policy template framework (versioned constraint artifacts)
- [ ] Trust profile configuration (per-project AI governance)
- [ ] Per-seat pricing implementation
- [ ] Team management features
- [ ] Audit trail infrastructure
- [ ] First enterprise pilots

### Phase 4: Marketplace Expansion (180+ Days)

- [ ] Skills Marketplace (Automation artifacts)
- [ ] Policy Marketplace (Judgment artifacts)
- [ ] Partner channel development
- [ ] Adjacent vertical evaluation

---

## Success Metrics

### Leading Indicators

| Metric | Target | Timeframe |
|--------|--------|-----------|
| MCP installations | 500 | 90 days |
| OAuth connections | 100 | 90 days |
| Skill runs (free tier) | 5,000 | 90 days |
| Return users (7-day) | 30% | Ongoing |

### Lagging Indicators

| Metric | Target | Timeframe |
|--------|--------|-----------|
| Paid conversions | 50 | 180 days |
| Monthly recurring revenue | $10K | 180 days |
| Enterprise deals | 3 | 12 months |
| Net promoter score | >50 | Ongoing |

### Qualitative Indicators

- IT teams recommending us internally
- Unprompted testimonials
- Conference speaking invitations
- Procore partnership conversations

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Procore API changes | Active monitoring, quick adaptation, relationship building |
| Security concerns | Open-source transparency, SOC 2, third-party audits |
| Slow enterprise sales | Start with mid-market GCs, build case studies |
| Skills don't deliver value | User research, rapid iteration, outcome focus |
| Procore Helix overlap | Procore Helix validates the market and creates buyer demand for cross-system + governance that Helix can't offer. Single-system Skills (draft_rfi from Procore-only data) are now table-stakes. Cross-system Skills and the Judgment Layer (policy artifacts, trust profiles, audit trails) are the moat. |
| Procore Developer Studio | Window of 12-18 months before Developer Studio ships. Establish cross-system and governance position now. Even if Developer Studio enables cross-system on Procore's platform, the Judgment Layer (user-controlled policy) remains structurally out of Procore's reach. |

---

## Appendix: Key Definitions

**MCP (Model Context Protocol)**: Open standard for connecting AI tools to external data sources. Created by Anthropic, adopted industry-wide.

**Skill**: A WORKWAY-specific AI capability that delivers a construction outcome (e.g., draft_rfi, summarize_daily_log).

**Trust Boundary**: The line between what data users control and what WORKWAY can access. Our architecture ensures credentials and raw data never cross this boundary.

**Zuhandenheit**: Heidegger's concept of "ready-to-hand"—tools that recede into transparent use. The goal is outcomes without awareness of mechanism.

---

## Appendix: Investor FAQ

**Q: Why construction?**
A: High pain (RFI delays cost real money), high trust barriers (perfect for open-source wedge), underserved by AI (no serious competitors), and domain complexity creates moat.

**Q: Why MCP?**
A: Industry-standard protocol (Anthropic-created), open-source builds trust, OAuth eliminates credential risk, and it's the wedge that earns the right to sell Skills.

**Q: What's the competitive moat?**
A: Trust (open-source), domain expertise (construction semantics), and ecosystem (Skills marketplace). Generic AI tools can't replicate the combination.

**Q: What's the business model?**
A: Per-run pricing for Skills, per-seat licensing for enterprise, marketplace commission for third-party Skills.

**Q: What's the expansion path?**
A: Prove the model in construction, then adjacent verticals (engineering, real estate), then horizontal platform.

---

*Last updated: February 2026*
*Document owner: GTM Team*
*Review cadence: Monthly*
