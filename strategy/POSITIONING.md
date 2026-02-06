# WORKWAY Competitive Positioning

> **Zuhandenheit**: The tool recedes; the outcome remains.

## What We Are

**The Automation Layer for Construction.**

WORKWAY connects construction tools to AI through open-source MCP servers, then delivers outcomes through premium Skills. We're construction-first, not vertical-agnostic.

**The entry point is connectivity, not intelligence.** Open-source Procore MCP server establishes trust. Construction Skills (draft_rfi, summarize_daily_log, flag_compliance) are the premium upsell.

**Business Model Priority**: MCP wedge → Skills adoption → Enterprise deals
- **MCP Layer**: Free/low-cost, open-source, trust-building
- **Skills Layer**: Per-run pricing, premium outcomes
- **Enterprise**: Per-seat + per-run, priority support, custom Skills

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

### Intelligence Layer (Skills)

**Purpose**: Deliver construction outcomes, not just data access.

| Characteristic | Value |
|----------------|-------|
| **Pricing** | Per-run (premium) |
| **Code** | Proprietary |
| **Trust Model** | Built on MCP foundation |
| **Data Flow** | Input → Outcome → Output |
| **Target User** | Project managers, superintendents |

### Clear Upgrade Path

```
Week 1: IT installs Procore MCP (free)
        ↓ "This actually works"
Week 2: PM tries draft_rfi skill (50 free runs)
        ↓ "This saved me 2 hours"
Week 3: Team adopts summarize_daily_log skill
        ↓ "We need this for all projects"
Week 4: Enterprise conversation begins
```

## What We're Not

- **Not a Zapier competitor**: We're construction infrastructure, not drag-and-drop automation
- **Not a general AI platform**: We speak construction (RFIs, daily logs, submittals), not generic workflows
- **Not Procore replacement**: We're complementary—making Procore more valuable through AI

## Core Differentiators

### 1. MCP-First: Open-Source Connectivity Builds Trust

**Competitors**: Hide their integrations behind black boxes
**WORKWAY**: Full source code on GitHub. Audit everything.

- Users can read every line of code
- OAuth means credentials never touch our servers
- Standard protocol with no vendor lock-in
- Fork, modify, self-host if needed

**Why this matters**: Construction teams don't trust AI vendors with their Procore data. They need to see the code. Open-source transparency converts skeptics into advocates.

### 2. Construction Domain Expertise

**Generic AI tools**: Don't understand the difference between an RFI and an RFC
**WORKWAY Skills**: Speak construction natively

- Tool names use domain language (`draft_rfi`, not `generate_text`)
- Outputs match industry formats
- Workflows respect construction's approval hierarchies
- Domain expertise compounds with each customer

**Why this matters**: Generic tools require translation. WORKWAY Skills deliver outcomes directly.

### 3. TypeScript-Native

**Traditional workflow tools**: Visual builders with limited code support
**WORKWAY**: Full TypeScript workflows with complete type safety

```typescript
// This is a WORKWAY workflow - real TypeScript code
export default async function(context: WorkflowContext) {
  const dailyLogs = await context.integrations.procore.getDailyLogs(context.trigger.projectId);
  const summary = await context.ai.summarize(dailyLogs);
  await context.integrations.procore.createReport({
    project: context.config.projectId,
    content: summary,
    type: 'weekly_summary'
  });
}
```

**Why this matters**:
- Full IDE support (autocomplete, type checking, refactoring)
- Unit testable with standard TypeScript testing tools
- Complex construction logic without platform limitations

### 4. Cloudflare Workers Edge Deployment

**Competitors**: Centralized US/EU datacenters
**WORKWAY**: Runs on Cloudflare's edge network (300+ cities globally)

- Sub-100ms latency worldwide
- Auto-scales to any load
- No cold starts
- Built-in DDoS protection

**Why this matters**: Your Skills run **where your projects are**, not in a distant datacenter.

### 5. Git-Native, Version-Controlled

**Traditional platforms**: Workflows live in proprietary UIs
**WORKWAY**: Workflows are TypeScript files in your Git repo

- Commit workflow changes like any other code
- Review changes in pull requests
- Deploy via CI/CD pipelines
- Rollback to any previous version instantly

**Why this matters**: Workflows are critical infrastructure. Treat them like code, not like spreadsheets.

## Who We're For

### Primary: Construction Teams

**Project Managers and Superintendents** at general contractors and specialty trades
- Using Procore for project management
- Drowning in RFI creation, daily log synthesis, compliance tracking
- Need outcomes, not technology
- Value time savings they can quantify

**What they want**: "RFIs that draft themselves."

### Secondary: Construction Software Developers

**Developers building construction tech** or integrating AI into construction workflows
- Already on Cloudflare (Pages, Workers, R2, etc.)
- Need type-safe, version-controlled automations
- Want to build on open-source infrastructure

**What they want**: "Open-source MCP I can trust and extend."

### Tertiary: Other Vertical Enterprises (Future)

**Enterprise teams in adjacent verticals** (engineering, real estate)
- After construction proves the model
- Similar trust requirements, high documentation needs
- Expansion candidates after $10M ARR

### Not For

- **Non-technical teams wanting simple automation** → Use Zapier
- **Generic workflow needs** → Not our focus
- **Teams without Procore (for now)** → We're Procore-first, expanding later

## Why Construction First

### Market Size & Pain

| Factor | Impact |
|--------|--------|
| **Global construction market** | $13T annually |
| **Procore ecosystem** | 17,000+ customers, $1B+ ARR |
| **RFI backlog cost** | Delays cost real money |
| **Daily log burden** | Hours per day of documentation |
| **Compliance risk** | Issues surface too late |
| **AI readiness** | 78% of contractors testing AI, only 10.6% deployed |

### Strategic Rationale

| Reason | Why |
|--------|-----|
| **Domain complexity** | Generic tools can't handle construction semantics |
| **High pain, high value** | RFI delays cost real money |
| **Relationship-driven** | Trust matters more than features |
| **AI-aware but vendor-locked** | Procore Helix proves demand; buyers need alternatives with control |
| **Expansion potential** | Success here proves the model |

### Domain Expertise as Moat

Generic AI tools don't understand:
- The difference between an RFI and an RFC
- Why daily logs matter for disputes
- How pay applications flow through approval
- What "substantial completion" means contractually

**This domain expertise compounds.** Each Skill teaches the next. Each customer interaction deepens understanding. Competitors starting from generic will never catch up.

### Expansion Path

```
Phase 1: Construction (now)
         └─ Procore MCP → Construction Skills
         
Phase 2: Adjacent Verticals (after $10M ARR)
         └─ Engineering → Autodesk MCP
         └─ Real Estate → Yardi MCP
         
Phase 3: Horizontal Platform (after vertical dominance)
         └─ Platform for any industry vertical
```

We expand only after proving the model in construction. No premature generalization.

## Messaging Framework

### For Construction Teams

**Problem**: RFIs take too long. Daily logs pile up. Compliance issues surface too late.

**Solution**: "RFIs that draft themselves."

**Proof**: "Draft RFI skill reduces RFI creation time from 2 hours to 15 minutes."

**Call to action**: "Connect Procore. Run 50 Skills free."

### For Developers / IT

**Problem**: Evaluating AI tools is risky. Vendors want your credentials. Code is hidden.

**Solution**: "Open-source MCP you can trust."

**Proof**: "Full source code on GitHub. OAuth means credentials never leave your systems."

**Call to action**: "Clone the repo. Run locally. Audit everything."

### For Enterprise / Executives

**Problem**: AI promises don't deliver. Security teams block everything. ROI is unclear.

**Solution**: "Automation infrastructure with trust boundaries."

**Proof**: "Open-source foundation. SOC 2 compliant. Per-run pricing with cost transparency."

**Call to action**: "Pilot with one project. Measure the results."

### Message Examples by Audience

| Audience | Wrong | Right |
|----------|-------|-------|
| PM | "AI-powered workflow automation" | "RFIs that draft themselves while you sleep" |
| IT | "Enterprise integration platform" | "Audit the code. We have nothing to hide." |
| Exec | "Digital transformation solution" | "Cut RFI creation time by 75%. Here's the math." |

## Competitive Landscape

### Procore Helix: Market Catalyst

Procore launched **Procore Helix** -- an AI intelligence layer with Procore Assist (conversational AI), AI Agents (RFI drafting, submittal review), Agent Builder (no-code custom agents), and Developer Studio (coming soon).

**This is good news.** Procore going AI-native creates the reference point that makes WORKWAY's differentiation legible:

- **Before Helix**: "Construction needs AI." (Category creation -- expensive, slow.)
- **After Helix**: "You've seen what Procore's AI does. Here's what it can't do." (Differentiation -- faster, buyer already believes.)

Procore spent the money to educate the market. WORKWAY benefits.

### Why Not Just Use Procore Helix?

Procore Helix collapses all three control models (Database, Automation, Judgment) into **Procore-controlled**. WORKWAY preserves the separation:

| Control Question | Procore Helix | WORKWAY |
|-----------------|---------------|---------|
| **Who decides what AI sees?** (Database) | Procore -- only Procore data | You -- any system via MCP |
| **Who decides what AI does?** (Automation) | Procore -- their agent templates | Your AI model -- Claude, GPT, Gemini, local |
| **Who sets the rules?** (Judgment) | Procore -- their defaults | Your team -- versioned policy artifacts |
| **Cross-system?** | No -- Procore only | Yes -- 21+ integrations |
| **Open source?** | No | Yes -- audit every line |

**The structural argument**: Procore needs uniform control to scale platform economics. WORKWAY preserves the control separation that construction's trust and liability requirements demand.

**The outcome argument**: Your RFIs pull context from meetings, schedules, AND Procore. Your compliance rules govern the AI, not Procore's defaults. Your daily briefing synthesizes everything -- Procore, Slack, email, schedules -- in one summary.

### The Judgment Layer Opportunity

Every firm using Procore Helix will eventually ask: "Can I customize the rules?" Procore's answer: "Some, within Agent Builder." WORKWAY's answer: "Fully. Versioned. Per-project. On your terms."

This creates the pathway for **policy as a product**:
- **Policy templates** -- Compliance rule sets per project type, jurisdiction, client
- **Trust profiles** -- Per-project AI governance (what AI can see, do, and auto-approve)
- **Approval workflows** -- Human-in-the-loop gates that cross system boundaries
- **Audit trails** -- Every AI decision traced for litigation support

The Judgment Layer is what turns the enterprise conversation from "tool purchase" into "governance infrastructure."

### Who We're Not Competing With

| Competitor | Why Not |
|------------|---------|
| **Zapier/Make** | Different user (non-technical), different use case (simple automation) |
| **Procore Helix** | Complementary at the Database layer (MCP connects to Procore). Differentiated at Automation (cross-system) and Judgment (user-controlled policy). |
| **Generic AI tools** | They don't understand construction |

### Our Actual Competition

| Alternative | Our Advantage |
|-------------|---------------|
| Manual processes | 10x faster, zero errors |
| Procore Helix alone | Cross-system reach + policy control |
| Custom development | Instant deployment, no maintenance |
| Doing nothing | Quantifiable time savings |

**Strategic positioning**: We're not competing with Procore's AI. We're extending it across systems and giving firms control of the governance layer.

## What We DON'T Say

| Phrase | Why Not | Instead Say |
|--------|---------|-------------|
| ❌ "Workflow automation" | Too generic, Zapier territory | "Construction automation" or specific outcome |
| ❌ "AI-powered" | Unless specifically about Skills | Focus on outcomes, not technology |
| ❌ "Better than Zapier" | Different market | "Built for construction teams" |
| ❌ "Digital transformation" | Meaningless buzzword | Specific outcome with metrics |
| ❌ "Enterprise-grade" | Empty marketing speak | Specific capabilities (SOC 2, SLAs) |
| ❌ "Seamless integration" | Everyone says this | "Open-source. Audit it yourself." |

### The Zuhandenheit Test

Before any external communication, apply the test:

> Can you describe the value without mentioning technology?

**Wrong**: "Our MCP server provides OAuth-based connectivity to Procore's REST API, enabling AI tools to access construction data through the Model Context Protocol."

**Right**: "RFIs that draft themselves. Daily logs that summarize themselves. Compliance issues that flag themselves."

The technology should be invisible. The outcome should be obvious.

## Internal Positioning (Team Alignment)

**We are building**: The Automation Layer for Construction

**We are not building**: A generic workflow tool for non-technical users

**Success looks like**: Construction teams saying "I can't remember how we did RFIs before WORKWAY"

**Failure looks like**: Marketing teams saying "WORKWAY seems like Zapier for developers"

## Moat Development

| Moat Type | How We Build It |
|-----------|-----------------|
| **Trust Moat** | Open-source MCP establishes credibility competitors can't replicate without similar transparency |
| **Domain Moat** | Construction expertise compounds with each customer |
| **Ecosystem Moat** | Skills + Policy marketplace creates network effects |
| **Control Moat** | Judgment Layer governance that Procore structurally can't offer |
| **Data Moat** | Usage patterns inform better Skills (with consent) |

## Honest Realities

**Where Procore Helix helps us:**
- Market education -- buyers understand AI in construction (category creation done for us)
- Reference point -- "you've seen Helix, here's what's beyond it" is a faster conversation
- Validation -- Procore investing in AI proves the market is real
- Urgency -- customers who've tasted single-system AI will feel the gaps faster

**Where Procore Helix challenges us:**
- Distribution advantage (2M+ daily users, embedded in the tool they already use)
- No-code Agent Builder lowers the bar further than our TypeScript-native approach
- Developer Studio, when it ships, could enable cross-system orchestration on Procore's platform
- The control model argument needs to be sold through outcomes, not architecture

**Other honest limitations:**
1. **Construction-first means narrow focus**: We don't do generic automation. If you're not in construction (yet), we're not for you.
2. **Early stage**: We're building the category. Expect iteration, rapid changes, and partnership with early adopters.
3. **Skills require trust**: The premium value requires users to trust our AI with construction context. MCP builds that trust, but it takes time.

## Long-Term Vision

**Phase 1 (Now)**: Procore MCP server + construction Skills (Automation Layer)
**Phase 2 (12 months)**: Skills + Policy marketplace (Automation + Judgment Layers)
**Phase 3 (18 months)**: Enterprise governance infrastructure (full Three-Tier stack)
**Phase 4 (24 months)**: Adjacent verticals (engineering, real estate)
**Phase 5 (36 months)**: Horizontal platform for any industry vertical

**End state**: Every industry vertical has domain-specific MCP servers (free), premium Skills (paid), and governance infrastructure (enterprise) running on WORKWAY.

## Zuhandenheit in Practice

Our positioning isn't "look at our features." It's "look at what disappears from your work."

**Before WORKWAY**:
- 2 hours per RFI, manually drafting from meeting notes
- Daily logs pile up, never synthesized into insights
- Compliance issues surface during disputes, not before
- Pay applications require days of documentation gathering

**After WORKWAY**:
- RFIs draft themselves from meeting transcripts
- Daily logs synthesize into weekly summaries automatically
- Compliance flags surface before they become problems
- Pay application documentation prepares itself

**The platform recedes. The outcomes remain.**

---

**Questions for positioning refinement**:
1. Is "Automation Layer for Construction" specific enough? (Answer: Yes—construction-first, not construction-only)
2. Who's our real competitor? (Answer: Manual processes and status quo, not Zapier)
3. What's our moat? (Answer: Trust (open-source) + Domain (construction) + Ecosystem (Skills marketplace))

---

*Last updated: February 2026*
*Document owner: Strategy Team*
*Review cadence: Monthly*
