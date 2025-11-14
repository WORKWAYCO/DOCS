# Developer Pricing Guide

> Complete guide to pricing your workflows on the WORKWAY marketplace

## Table of Contents

- [Overview](#overview)
- [Pricing Model](#pricing-model)
- [Setting Your Upfront Price](#setting-your-upfront-price)
- [Workflow Complexity Classification](#workflow-complexity-classification)
- [Revenue Structure](#revenue-structure)
- [Pricing Psychology](#pricing-psychology)
- [Industry-Specific Pricing Examples](#industry-specific-pricing-examples)
- [Best Practices](#best-practices)
- [FAQs](#faqs)

---

## Overview

WORKWAY uses a unique two-sided marketplace pricing model that benefits both developers and businesses:

- **Developers**: Set your own upfront price and keep 100% (minus payment processing)
- **Businesses**: Pay upfront, get 20 free trial runs, then pay-as-you-go for usage

This model aligns incentives: developers earn from value creation, and businesses pay predictable, usage-based pricing.

---

## Pricing Model

### How It Works

1. **Upfront Payment**
   - Developer sets the price (one-time or subscription)
   - Developer receives 100% of this amount
   - Only Stripe processing fees are deducted (~2.9% + $0.30)
   - No platform revenue share

2. **Trial Period**
   - Every buyer gets 20 free workflow executions
   - Allows businesses to validate value before committing
   - No time limit on trial period

3. **Usage-Based Pricing**
   - After 20 free runs, buyers pay per execution
   - Platform charges usage fees (you don't need to handle this)
   - Pricing based on your workflow complexity classification

### Pricing Tiers

| Complexity | Price per Execution | Use Case |
|------------|---------------------|----------|
| **Light** | 5¢ | Simple integrations, single API calls, basic data syncs |
| **Heavy** | 25¢ | AI processing, multiple integrations, complex logic |

---

## Setting Your Upfront Price

### Recommended Price Ranges

| Workflow Type | One-Time | Monthly Subscription |
|---------------|----------|---------------------|
| **Simple Automation** | $19-49 | $9-29/mo |
| **Moderate Complexity** | $49-99 | $29-59/mo |
| **Complex Integration** | $99-249 | $59-149/mo |
| **Enterprise Solution** | $249+ | $149+/mo |

### Pricing Considerations

**1. Target Market Budget**
   - Dental offices, coffee shops, service businesses typically budget $20-100/mo for automation
   - Price accordingly for your target audience

**2. Value Proposition**
   - Time saved: "Saves 10 hours/week" → justify higher price
   - Revenue impact: "Increases bookings by 20%" → premium pricing
   - Cost savings: "Reduces manual errors by 80%" → mid-tier pricing

**3. Complexity vs. Value**
   - Don't confuse technical complexity with business value
   - Simple workflows that save lots of time can command premium prices
   - Complex workflows with minimal impact should be priced lower

**4. Pricing Model Choice**

   **One-Time Payment**
   - Best for: Standalone tools, one-time setup workflows
   - Pros: Lower barrier to entry, easier sell
   - Cons: No recurring revenue
   - Example: Email list cleaner ($29 one-time)

   **Monthly Subscription**
   - Best for: Ongoing automations, continuous value
   - Pros: Predictable recurring revenue, higher lifetime value
   - Cons: Requires ongoing value demonstration
   - Example: Daily appointment reminders ($39/mo)

---

## Workflow Complexity Classification

Choosing the right complexity tier ensures fair pricing and sets proper expectations.

### Light Workflows (5¢/execution)

**Choose "Light" if your workflow:**

✅ Makes 1-3 API calls per execution
✅ Processes data under 1MB
✅ Completes in less than 5 seconds
✅ Performs simple transformations
✅ Single integration point

**Examples:**

- **Email Notifications**
  - Trigger: New booking
  - Action: Send confirmation email
  - API calls: 1 (email service)
  - Complexity: Light

- **Data Sync**
  - Trigger: New customer in POS
  - Action: Create contact in CRM
  - API calls: 2 (read POS, write CRM)
  - Complexity: Light

- **Simple Webhook**
  - Trigger: Form submission
  - Action: Send to Google Sheets
  - API calls: 1-2
  - Complexity: Light

- **Status Updates**
  - Trigger: Order status change
  - Action: Update tracking number
  - API calls: 2
  - Complexity: Light

### Heavy Workflows (25¢/execution)

**Choose "Heavy" if your workflow:**

✅ Uses AI/LLM processing (ChatGPT, Claude, etc.)
✅ Makes 4+ API calls per execution
✅ Processes large data (1MB+)
✅ Complex conditional logic or transformations
✅ Multiple integration points
✅ Image/video processing
✅ Takes longer than 10 seconds to complete

**Examples:**

- **AI Content Generation**
  - Trigger: New product added
  - Action: Generate description with ChatGPT + create social media posts + schedule
  - API calls: 4+ (AI model, social platforms)
  - Complexity: Heavy

- **Multi-Step Patient Workflow**
  - Trigger: Appointment completed
  - Action: Send survey + analyze sentiment with AI + update CRM + schedule follow-up + send personalized email
  - API calls: 5+ with AI processing
  - Complexity: Heavy

- **Complex Data Processing**
  - Trigger: Daily report needed
  - Action: Pull data from 3 sources + merge + analyze + generate PDF + email
  - API calls: 6+
  - Complexity: Heavy

- **Image Processing Pipeline**
  - Trigger: New product photo uploaded
  - Action: Resize + optimize + remove background + add watermark + distribute to platforms
  - API calls: Multiple with heavy processing
  - Complexity: Heavy

### Gray Areas: How to Decide

| Scenario | Suggested Classification | Reasoning |
|----------|-------------------------|-----------|
| 3 API calls + simple logic | Light | Borderline, lean toward simplicity |
| 3 API calls + AI processing | Heavy | AI processing is resource-intensive |
| 4 API calls, all fast | Light | If total runtime < 5s, can stay light |
| Simple workflow but large file processing | Heavy | File size matters for infrastructure |
| Complex logic but 1 API call | Light | Logic alone doesn't warrant heavy classification |

**When in doubt**: Choose the classification that aligns with your infrastructure costs and processing time.

---

## Revenue Structure

### What You Earn

**From Upfront Payments:**
- 100% of your set price
- Minus Stripe processing fee (~2.9% + $0.30)
- No platform revenue share
- Immediate payout to your Stripe Connect account

**Example:**
- You charge: $49
- Stripe fee: ~$1.72
- You receive: ~$47.28

**From Usage Charges:**
- $0 - You don't receive usage fees
- Platform collects usage charges to cover infrastructure
- Your upfront pricing should reflect your workflow's value

### Pricing Strategy

Since you don't earn from usage, your upfront price should reflect:

1. **Development effort**
2. **Value provided to businesses**
3. **Competitive positioning**
4. **Support and maintenance commitment**

**Don't worry about usage volume** - that's covered by platform usage fees. Focus on delivering value that justifies your upfront price.

---

## Pricing Psychology

### Effective Pricing Strategies

**1. Value-Based Pricing**
Show total cost of ownership:
> "$49/mo + ~$15/mo usage for 200 patients = $64/mo total"
>
> *Saves 10 hours/week ($400 value at $40/hr labor)*
>
> **ROI: 525% monthly**

**2. Competitive Positioning**
- Research similar solutions (Zapier, Make, n8n)
- Price 30-50% lower due to specialization
- Highlight your focused use case as advantage

**3. Anchor Pricing**
Present pricing with context:
> "Manual process cost: $400/mo in labor"
> "Our solution: $64/mo total (saves $336/mo)"

**4. Tiered Pricing (Optional)**
Offer variations for different business sizes:
- **Small Business**: $29/mo (basic features)
- **Professional**: $49/mo (advanced features)
- **Enterprise**: $99/mo (premium support + customization)

### Target Market Considerations

**Service Businesses (Dental, Coffee Shops, Pickleball Facilities)**

- **Budget range**: $20-100/mo for automation tools
- **Decision makers**: Owners/managers (value time savings)
- **Pain points**: Manual processes, missed follow-ups, booking issues
- **Pricing sweet spot**: $29-59/mo for subscriptions, $39-79 one-time

**What they care about:**
1. Time savings (quantify hours saved per week)
2. Revenue protection (prevent lost bookings/appointments)
3. Customer satisfaction (better communication)
4. Simplicity (easy setup and maintenance)

---

## Industry-Specific Pricing Examples

### Dental Offices

**Patient Appointment Reminders**
- Upfront: $49/mo subscription
- Complexity: Light (5¢/execution)
- Average usage: 200 reminders/mo
- Usage cost: $10/mo
- Total cost to customer: ~$59/mo
- Value: Reduces no-shows by 30%, saves 5 hours/week

**Your pricing strategy**: $49/mo captures the value of preventing no-shows (worth $500+/mo for average dental practice)

---

**Patient Follow-Up with AI Sentiment Analysis**
- Upfront: $99/mo subscription
- Complexity: Heavy (25¢/execution)
- Average usage: 100 follow-ups/mo
- Usage cost: $25/mo
- Total cost to customer: ~$124/mo
- Value: Identifies unhappy patients early, improves retention

**Your pricing strategy**: $99/mo reflects AI processing value and retention impact

---

### Coffee Shops

**Loyalty Points Tracker**
- Upfront: $29 one-time
- Complexity: Light (5¢/execution)
- Average usage: 150 transactions/mo
- Usage cost: $7.50/mo
- Total cost to customer: $29 + ~$7.50/mo
- Value: Increases repeat customer rate by 25%

**Your pricing strategy**: Low upfront barrier, simple value proposition

---

**Inventory Reorder Automation with AI Predictions**
- Upfront: $79/mo subscription
- Complexity: Heavy (25¢/execution)
- Average usage: 50 predictions/mo
- Usage cost: $12.50/mo
- Total cost to customer: ~$91.50/mo
- Value: Prevents stockouts, optimizes inventory levels

**Your pricing strategy**: Premium pricing for AI-powered optimization

---

### Pickleball Facilities

**Court Booking Automation**
- Upfront: $39/mo subscription
- Complexity: Light (5¢/execution)
- Average usage: 120 bookings/mo
- Usage cost: $6/mo
- Total cost to customer: ~$45/mo
- Value: Saves 8 hours/week of manual booking management

**Your pricing strategy**: Mid-tier pricing for clear time savings

---

**Tournament Management System**
- Upfront: $149/mo subscription
- Complexity: Heavy (25¢/execution)
- Average usage: 200 operations/mo (bracket updates, notifications, results)
- Usage cost: $50/mo
- Total cost to customer: ~$199/mo
- Value: Enables hosting tournaments, new revenue stream

**Your pricing strategy**: Premium pricing for revenue-generating capability

---

## Best Practices

### Do's ✅

1. **Show Total Cost Transparency**
   - Include typical usage costs in your listing
   - Provide example scenarios: "For 200 appointments/mo: $49 + ~$10 usage"

2. **Highlight Value, Not Features**
   - Wrong: "Connects Gmail to Notion with 5 API calls"
   - Right: "Never miss a client email again - auto-track in Notion"

3. **Offer Tiered Options**
   - Basic, Professional, Enterprise variations
   - Different feature sets, not just limits
   - Clear upgrade path

4. **Provide ROI Calculator**
   - Help customers understand savings
   - "Saves X hours/week at $Y/hour = $Z/month value"

5. **Be Competitive, Not Cheapest**
   - Don't race to bottom on price
   - Differentiate on specialization and support
   - Premium pricing with premium support is valid

6. **Use Social Proof**
   - "Trusted by 50+ dental practices"
   - Testimonials with results: "Reduced no-shows 40%"

### Don'ts ❌

1. **Don't Underprice**
   - Underpricing signals low quality
   - You can't provide great support at $5/mo
   - Minimum viable: $19-29 for most workflows

2. **Don't Hide Complexity Tier**
   - Be upfront about light vs heavy classification
   - Explain why heavy tier is justified
   - Customers appreciate transparency

3. **Don't Over-Promise**
   - Realistic usage estimates only
   - Under-promise, over-deliver
   - Build trust through honesty

4. **Don't Ignore Target Market Budget**
   - $500/mo pricing won't work for coffee shops
   - Match pricing to audience willingness to pay

5. **Don't Forget Ongoing Costs**
   - Price for sustainability
   - Include buffer for support time
   - Account for API costs you incur

---

## FAQs

### How do I get paid?

Upfront payments go directly to your Stripe Connect account. You'll receive funds according to your Stripe Connect payout schedule (typically 2-7 days).

### Do I earn anything from usage charges?

No. Usage charges (5¢ or 25¢ per execution) go to WORKWAY to cover platform infrastructure costs. Your revenue comes entirely from upfront pricing.

### Can I change my pricing after publishing?

Yes, but changes only apply to new customers. Existing customers keep their original pricing. This protects customer trust.

### What if I misclassify complexity tier?

Contact support to reclassify before many customers purchase. After significant sales, reclassification may not be possible as it affects customer costs.

### Should I offer free workflows?

You can set upfront price to $0. Customers still get 20 free trial runs, then pay usage fees. Good for:
- Building audience
- Freemium strategy
- Portfolio building
- Simple utilities

Not sustainable as only monetization strategy.

### How do I handle refunds?

Contact WORKWAY support. Refund policies:
- Within 20 trial executions: Full refund available
- After trial: Partial refund at discretion
- Clear refund policy in your workflow description

### Can I offer discounts or promotions?

Contact WORKWAY to set up promotional pricing for:
- Launch discounts
- Limited-time offers
- Bulk purchases
- Enterprise deals

### What about annual plans?

Coming soon. For now, monthly subscriptions only. Annual options will be available Q1 2026.

---

## Need Help?

- **Pricing Strategy Consultation**: Email developers@workway.com
- **Classification Questions**: Use the Slack channel #pricing-help
- **Technical Implementation**: See [Developer Documentation](./COMPLETE_DEVELOPER_GUIDE.md)

---

**Remember**: Your pricing should reflect the value you deliver to businesses. Don't compete on price alone - compete on specialization, support quality, and clear ROI.
