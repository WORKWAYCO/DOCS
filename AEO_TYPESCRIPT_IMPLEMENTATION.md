# AEO & Structured Data - TypeScript Implementation Guide
## For Next.js/Cloudflare Workers Application

---

## Table of Contents
1. [React Component Schema Implementation](#react-component-schema-implementation)
2. [Server-Side Schema Generation](#server-side-schema-generation)
3. [Dynamic Schema Utilities](#dynamic-schema-utilities)
4. [Next.js Metadata Integration](#nextjs-metadata-integration)
5. [Testing & Validation](#testing--validation)

---

## React Component Schema Implementation

### 1. Reusable Schema Component

```typescript
// components/StructuredData.tsx

import { ReactNode } from 'react';

interface StructuredDataProps {
  data: Record<string, any>;
  children?: ReactNode;
}

export const StructuredData = ({ data, children }: StructuredDataProps) => {
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(data),
        }}
      />
      {children}
    </>
  );
};
```

### 2. Organization Schema Component

```typescript
// components/OrganizationSchema.tsx

import { StructuredData } from './StructuredData';

interface OrganizationSchemaProps {
  name: string;
  url: string;
  logo: string;
  description: string;
  socialLinks?: string[];
  foundingDate?: string;
  contactEmail?: string;
  contactPhone?: string;
}

export const OrganizationSchema = ({
  name,
  url,
  logo,
  description,
  socialLinks = [],
  foundingDate,
  contactEmail,
  contactPhone,
}: OrganizationSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name,
    url,
    logo,
    description,
    sameAs: socialLinks,
    ...(foundingDate && { foundingDate }),
    ...(contactEmail || contactPhone) && {
      contactPoint: {
        '@type': 'ContactPoint',
        ...(contactEmail && { email: contactEmail }),
        ...(contactPhone && { telephone: contactPhone }),
        contactType: 'Customer Service',
      },
    },
  };

  return <StructuredData data={data} />;
};
```

### 3. Product Schema Component

```typescript
// components/ProductSchema.tsx

import { StructuredData } from './StructuredData';

interface ProductSchemaProps {
  name: string;
  description: string;
  image: string;
  url: string;
  category: string;
  price: number | string;
  priceCurrency?: string;
  ratingValue?: number;
  ratingCount?: number;
  reviewCount?: number;
  creatorName?: string;
  creatorUrl?: string;
  datePublished?: string;
  dateModified?: string;
}

export const ProductSchema = ({
  name,
  description,
  image,
  url,
  category,
  price,
  priceCurrency = 'USD',
  ratingValue,
  ratingCount,
  reviewCount,
  creatorName,
  creatorUrl,
  datePublished,
  dateModified,
}: ProductSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name,
    description,
    image,
    url,
    category,
    offers: {
      '@type': 'Offer',
      price: price.toString(),
      priceCurrency,
      availability: 'https://schema.org/InStock',
    },
    ...(ratingValue && ratingCount) && {
      aggregateRating: {
        '@type': 'AggregateRating',
        ratingValue,
        ratingCount,
        ...(reviewCount && { reviewCount }),
      },
    },
    ...(creatorName || creatorUrl) && {
      creator: {
        '@type': 'Person',
        ...(creatorName && { name: creatorName }),
        ...(creatorUrl && { url: creatorUrl }),
      },
    },
    ...(datePublished && { datePublished }),
    ...(dateModified && { dateModified }),
  };

  return <StructuredData data={data} />;
};
```

### 4. SoftwareApplication Schema Component

```typescript
// components/SoftwareApplicationSchema.tsx

import { StructuredData } from './StructuredData';

interface SoftwareApplicationSchemaProps {
  name: string;
  description: string;
  url: string;
  applicationCategory: 'Productivity' | 'Business' | 'Utility' | string;
  applicationSubCategory?: string;
  operatingSystem: 'Web' | 'Android' | 'iOS' | 'Windows' | string;
  ratingValue?: number;
  ratingCount?: number;
  price?: number | string;
  priceCurrency?: string;
  softwareVersion: string;
  datePublished: string;
  dateModified: string;
  downloadUrl?: string;
  installUrl?: string;
  screenshot?: string;
}

export const SoftwareApplicationSchema = ({
  name,
  description,
  url,
  applicationCategory,
  applicationSubCategory,
  operatingSystem,
  ratingValue,
  ratingCount,
  price = 0,
  priceCurrency = 'USD',
  softwareVersion,
  datePublished,
  dateModified,
  downloadUrl,
  installUrl,
  screenshot,
}: SoftwareApplicationSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'SoftwareApplication',
    name,
    description,
    url,
    applicationCategory,
    ...(applicationSubCategory && { applicationSubCategory }),
    operatingSystem,
    offers: {
      '@type': 'Offer',
      price: price.toString(),
      priceCurrency,
      availability: 'https://schema.org/InStock',
    },
    ...(ratingValue && ratingCount) && {
      aggregateRating: {
        '@type': 'AggregateRating',
        ratingValue,
        ratingCount,
      },
    },
    softwareVersion,
    datePublished,
    dateModified,
    ...(downloadUrl && { downloadUrl }),
    ...(installUrl && { installUrl }),
    ...(screenshot && { screenshot }),
  };

  return <StructuredData data={data} />;
};
```

### 5. FAQPage Schema Component

```typescript
// components/FAQPageSchema.tsx

import { StructuredData } from './StructuredData';

export interface FAQItem {
  question: string;
  answer: string;
}

interface FAQPageSchemaProps {
  faqs: FAQItem[];
}

export const FAQPageSchema = ({ faqs }: FAQPageSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map((faq) => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  };

  return <StructuredData data={data} />;
};
```

### 6. HowTo Schema Component

```typescript
// components/HowToSchema.tsx

import { StructuredData } from './StructuredData';

export interface HowToStep {
  position: number;
  name: string;
  description: string;
  image?: string;
  url?: string;
}

interface HowToSchemaProps {
  name: string;
  description: string;
  steps: HowToStep[];
  estimatedTime?: string; // ISO 8601 format: PT5M
  estimatedCost?: number;
  estimatedCostCurrency?: string;
  image?: string;
  author?: {
    name: string;
    url?: string;
  };
  datePublished?: string;
  dateModified?: string;
}

export const HowToSchema = ({
  name,
  description,
  steps,
  estimatedTime,
  estimatedCost,
  estimatedCostCurrency = 'USD',
  image,
  author,
  datePublished,
  dateModified,
}: HowToSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'HowTo',
    name,
    description,
    ...(image && { image }),
    ...(author) && {
      author: {
        '@type': 'Organization',
        ...author,
      },
    },
    ...(estimatedTime && { totalTime: estimatedTime }),
    ...(estimatedCost) && {
      estimatedCost: {
        '@type': 'MonetaryAmount',
        currency: estimatedCostCurrency,
        price: estimatedCost.toString(),
      },
    },
    ...(datePublished && { datePublished }),
    ...(dateModified && { dateModified }),
    step: steps.map((step) => ({
      '@type': 'HowToStep',
      position: step.position,
      name: step.name,
      description: step.description,
      ...(step.image && { image: step.image }),
      ...(step.url && { url: step.url }),
    })),
  };

  return <StructuredData data={data} />;
};
```

### 7. BreadcrumbList Schema Component

```typescript
// components/BreadcrumbSchema.tsx

import { StructuredData } from './StructuredData';

export interface BreadcrumbItem {
  position: number;
  name: string;
  item: string;
}

interface BreadcrumbSchemaProps {
  items: BreadcrumbItem[];
}

export const BreadcrumbSchema = ({ items }: BreadcrumbSchemaProps) => {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item) => ({
      '@type': 'ListItem',
      position: item.position,
      name: item.name,
      item: item.item,
    })),
  };

  return <StructuredData data={data} />;
};
```

---

## Server-Side Schema Generation

### 1. Schema Builder Utility

```typescript
// lib/schema-builder.ts

export interface SchemaContext {
  siteUrl: string;
  organizationName: string;
  organizationLogo: string;
}

export class SchemaBuilder {
  private context: SchemaContext;

  constructor(context: SchemaContext) {
    this.context = context;
  }

  /**
   * Build Organization schema
   */
  buildOrganization(data: {
    name: string;
    description: string;
    foundingDate?: string;
    email?: string;
    phone?: string;
    socialLinks?: string[];
  }) {
    return {
      '@context': 'https://schema.org',
      '@type': 'Organization',
      name: data.name,
      url: this.context.siteUrl,
      logo: this.context.organizationLogo,
      description: data.description,
      ...(data.foundingDate && { foundingDate: data.foundingDate }),
      ...(data.email || data.phone) && {
        contactPoint: {
          '@type': 'ContactPoint',
          ...(data.email && { email: data.email }),
          ...(data.phone && { telephone: data.phone }),
          contactType: 'Customer Service',
        },
      },
      ...(data.socialLinks && { sameAs: data.socialLinks }),
    };
  }

  /**
   * Build WebSite schema with SearchAction
   */
  buildWebSite(data: {
    name: string;
    searchUrl: string;
    description?: string;
  }) {
    return {
      '@context': 'https://schema.org',
      '@type': 'WebSite',
      name: data.name,
      url: this.context.siteUrl,
      ...(data.description && { description: data.description }),
      potentialAction: {
        '@type': 'SearchAction',
        target: {
          '@type': 'EntryPoint',
          urlTemplate: data.searchUrl,
        },
        'query-input': 'required name=search_term_string',
      },
    };
  }

  /**
   * Build Product schema
   */
  buildProduct(data: {
    name: string;
    description: string;
    image: string;
    url: string;
    category: string;
    price?: number;
    priceCurrency?: string;
    rating?: { value: number; count: number; reviews?: number };
    creator?: { name: string; url?: string };
    datePublished?: string;
    dateModified?: string;
  }) {
    return {
      '@context': 'https://schema.org',
      '@type': 'Product',
      name: data.name,
      description: data.description,
      image: data.image,
      url: data.url,
      category: data.category,
      offers: {
        '@type': 'Offer',
        price: (data.price ?? 0).toString(),
        priceCurrency: data.priceCurrency || 'USD',
        availability: 'https://schema.org/InStock',
      },
      ...(data.rating) && {
        aggregateRating: {
          '@type': 'AggregateRating',
          ratingValue: data.rating.value,
          ratingCount: data.rating.count,
          ...(data.rating.reviews && { reviewCount: data.rating.reviews }),
        },
      },
      ...(data.creator) && {
        creator: {
          '@type': 'Person',
          name: data.creator.name,
          ...(data.creator.url && { url: data.creator.url }),
        },
      },
      ...(data.datePublished && { datePublished: data.datePublished }),
      ...(data.dateModified && { dateModified: data.dateModified }),
    };
  }

  /**
   * Build FAQPage schema
   */
  buildFAQPage(faqs: Array<{ question: string; answer: string }>) {
    return {
      '@context': 'https://schema.org',
      '@type': 'FAQPage',
      mainEntity: faqs.map((faq) => ({
        '@type': 'Question',
        name: faq.question,
        acceptedAnswer: {
          '@type': 'Answer',
          text: faq.answer,
        },
      })),
    };
  }

  /**
   * Build HowTo schema
   */
  buildHowTo(data: {
    name: string;
    description: string;
    steps: Array<{
      position: number;
      name: string;
      description: string;
      image?: string;
    }>;
    estimatedTime?: string;
    estimatedCost?: number;
    author?: { name: string; url?: string };
    datePublished?: string;
    dateModified?: string;
  }) {
    return {
      '@context': 'https://schema.org',
      '@type': 'HowTo',
      name: data.name,
      description: data.description,
      ...(data.estimatedTime && { totalTime: data.estimatedTime }),
      ...(data.estimatedCost) && {
        estimatedCost: {
          '@type': 'MonetaryAmount',
          currency: 'USD',
          price: data.estimatedCost.toString(),
        },
      },
      ...(data.author) && {
        author: {
          '@type': 'Organization',
          name: data.author.name,
          ...(data.author.url && { url: data.author.url }),
        },
      },
      ...(data.datePublished && { datePublished: data.datePublished }),
      ...(data.dateModified && { dateModified: data.dateModified }),
      step: data.steps.map((step) => ({
        '@type': 'HowToStep',
        position: step.position,
        name: step.name,
        description: step.description,
        ...(step.image && { image: step.image }),
      })),
    };
  }

  /**
   * Build BreadcrumbList schema
   */
  buildBreadcrumbs(items: Array<{ name: string; url: string }>) {
    return {
      '@context': 'https://schema.org',
      '@type': 'BreadcrumbList',
      itemListElement: items.map((item, index) => ({
        '@type': 'ListItem',
        position: index + 1,
        name: item.name,
        item: item.url,
      })),
    };
  }

  /**
   * Combine multiple schemas using @graph
   */
  buildGraphSchema(...schemas: any[]) {
    return {
      '@context': 'https://schema.org',
      '@graph': schemas,
    };
  }
}
```

### 2. Usage Example

```typescript
// lib/schema-examples.ts

import { SchemaBuilder, SchemaContext } from './schema-builder';

const context: SchemaContext = {
  siteUrl: 'https://workflowkit.io',
  organizationName: 'WorkflowKit',
  organizationLogo: 'https://workflowkit.io/logo.png',
};

const builder = new SchemaBuilder(context);

// Generate organization schema
export const organizationSchema = builder.buildOrganization({
  name: 'WorkflowKit',
  description: 'AI-powered workflow automation platform',
  foundingDate: '2023-01-15',
  email: 'support@workflowkit.io',
  phone: '+1-555-0123',
  socialLinks: [
    'https://twitter.com/workflowkit',
    'https://linkedin.com/company/workflowkit',
  ],
});

// Generate product schema
export const productSchema = builder.buildProduct({
  name: 'Email Auto-Reply Workflow',
  description: 'Automatically reply to emails with AI-powered responses',
  image: 'https://workflowkit.io/templates/email-auto-reply.jpg',
  url: 'https://workflowkit.io/workflows/email-auto-reply',
  category: 'Email Automation',
  price: 0,
  rating: { value: 4.8, count: 287, reviews: 125 },
  creator: { name: 'WorkflowKit Team' },
  datePublished: '2024-06-15',
  dateModified: new Date().toISOString().split('T')[0],
});

// Generate FAQ schema
export const faqSchema = builder.buildFAQPage([
  {
    question: 'How do I create a workflow?',
    answer:
      'Click "New Workflow", select your trigger type, define conditions, choose actions, and deploy.',
  },
  {
    question: 'Can I use regex patterns?',
    answer: 'Yes, WorkflowKit supports standard regex for advanced email matching.',
  },
]);

// Generate HowTo schema
export const howToSchema = builder.buildHowTo({
  name: 'How to Set Up an Email Auto-Reply',
  description: 'Step-by-step guide to creating an automated email workflow',
  steps: [
    {
      position: 1,
      name: 'Log in to WorkflowKit',
      description: 'Navigate to workflowkit.io and sign in',
    },
    {
      position: 2,
      name: 'Create New Workflow',
      description: 'Click "New Workflow" and select "Email"',
    },
    {
      position: 3,
      name: 'Define Conditions',
      description: 'Set trigger conditions like sender, keywords, etc.',
    },
  ],
  estimatedTime: 'PT5M',
  estimatedCost: 0,
  author: { name: 'WorkflowKit Support' },
  datePublished: '2024-06-01',
});
```

---

## Dynamic Schema Utilities

### 1. Rating Schema Generator

```typescript
// lib/rating-schema.ts

export interface Rating {
  value: number; // 1-5
  count: number;
  reviews?: number;
}

export const validateRating = (rating: Rating): boolean => {
  if (rating.value < 1 || rating.value > 5) return false;
  if (rating.count < 1) return false;
  if (rating.reviews && rating.reviews > rating.count) return false;
  return true;
};

export const buildAggregateRating = (rating: Rating) => {
  if (!validateRating(rating)) {
    throw new Error('Invalid rating data');
  }

  return {
    '@type': 'AggregateRating',
    ratingValue: rating.value,
    ratingCount: rating.count,
    bestRating: 5,
    worstRating: 1,
    ...(rating.reviews && { reviewCount: rating.reviews }),
  };
};
```

### 2. Price Schema Generator

```typescript
// lib/price-schema.ts

export interface PriceInfo {
  amount: number | string;
  currency?: string;
  validUntil?: string; // ISO 8601 format
  description?: string;
}

export const buildOfferSchema = ({
  amount,
  currency = 'USD',
  validUntil,
  description,
}: PriceInfo) => {
  return {
    '@type': 'Offer',
    price: amount.toString(),
    priceCurrency: currency,
    availability: 'https://schema.org/InStock',
    ...(validUntil && { priceValidUntil: validUntil }),
    ...(description && { description }),
  };
};
```

### 3. Author Schema Generator

```typescript
// lib/author-schema.ts

export interface AuthorInfo {
  name: string;
  url?: string;
  email?: string;
  image?: string;
  role?: string;
}

export const buildAuthorSchema = ({
  name,
  url,
  email,
  image,
  role,
}: AuthorInfo) => {
  return {
    '@type': 'Person',
    name,
    ...(url && { url }),
    ...(email && { email }),
    ...(image && { image }),
    ...(role && { jobTitle: role }),
  };
};

export const buildOrganizationAuthorSchema = ({
  name,
  url,
  email,
  image,
}: AuthorInfo) => {
  return {
    '@type': 'Organization',
    name,
    ...(url && { url }),
    ...(email && { email }),
    ...(image && { logo: image }),
  };
};
```

---

## Next.js Metadata Integration

### 1. Homepage Metadata

```typescript
// app/page.tsx

import type { Metadata } from 'next';
import { OrganizationSchema } from '@/components/OrganizationSchema';
import { SoftwareApplicationSchema } from '@/components/SoftwareApplicationSchema';
import { SchemaBuilder } from '@/lib/schema-builder';

export const metadata: Metadata = {
  title: 'WorkflowKit - No-Code Email Workflow Automation',
  description:
    'Build powerful AI-powered email workflows without writing code. Automate your inbox with triggers, conditions, and actions.',
  openGraph: {
    title: 'WorkflowKit - Email Workflow Automation',
    description:
      'Automate your email workflows with AI-powered triggers and responses.',
    url: 'https://workflowkit.io',
    type: 'website',
    images: [
      {
        url: 'https://workflowkit.io/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'WorkflowKit',
      },
    ],
  },
};

export default function Home() {
  return (
    <>
      <OrganizationSchema
        name="WorkflowKit"
        url="https://workflowkit.io"
        logo="https://workflowkit.io/logo-white.png"
        description="AI-powered workflow automation platform for email, Slack, and productivity tools"
        socialLinks={[
          'https://twitter.com/workflowkit',
          'https://linkedin.com/company/workflowkit',
          'https://github.com/workflowkit',
        ]}
        foundingDate="2023-01-15"
        contactEmail="support@workflowkit.io"
        contactPhone="+1-555-0123"
      />
      <SoftwareApplicationSchema
        name="WorkflowKit"
        description="No-code platform to automate email workflows, Slack messages, and productivity tools using AI-powered triggers"
        url="https://workflowkit.io"
        applicationCategory="Productivity"
        applicationSubCategory="Business Automation"
        operatingSystem="Web"
        ratingValue={4.7}
        ratingCount={1240}
        softwareVersion="2.5.0"
        datePublished="2023-01-15"
        dateModified={new Date().toISOString().split('T')[0]}
        downloadUrl="https://workflowkit.io/download"
        installUrl="https://app.workflowkit.io/signup"
      />
      {/* Rest of homepage content */}
    </>
  );
}
```

### 2. Product Page Metadata

```typescript
// app/workflows/[workflowId]/page.tsx

import type { Metadata } from 'next';
import { ProductSchema } from '@/components/ProductSchema';
import { BreadcrumbSchema } from '@/components/BreadcrumbSchema';
import { getWorkflow } from '@/lib/api';

interface Props {
  params: { workflowId: string };
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const workflow = await getWorkflow(params.workflowId);

  return {
    title: `${workflow.name} - WorkflowKit`,
    description: workflow.description,
    openGraph: {
      title: `${workflow.name} - WorkflowKit`,
      description: workflow.description,
      url: `https://workflowkit.io/workflows/${workflow.id}`,
      type: 'website',
      images: [
        {
          url: workflow.image,
          width: 1200,
          height: 630,
          alt: workflow.name,
        },
      ],
    },
  };
}

export default async function WorkflowPage({ params }: Props) {
  const workflow = await getWorkflow(params.workflowId);

  return (
    <>
      <ProductSchema
        name={workflow.name}
        description={workflow.description}
        image={workflow.image}
        url={`https://workflowkit.io/workflows/${workflow.id}`}
        category={workflow.category}
        price={workflow.price || 0}
        ratingValue={workflow.rating}
        ratingCount={workflow.ratingCount}
        reviewCount={workflow.reviewCount}
        creatorName={workflow.creatorName}
        creatorUrl={workflow.creatorUrl}
        datePublished={workflow.createdAt}
        dateModified={workflow.updatedAt}
      />
      <BreadcrumbSchema
        items={[
          { position: 1, name: 'Home', item: 'https://workflowkit.io' },
          {
            position: 2,
            name: 'Workflows',
            item: 'https://workflowkit.io/workflows',
          },
          {
            position: 3,
            name: workflow.category,
            item: `https://workflowkit.io/workflows?category=${workflow.category}`,
          },
          {
            position: 4,
            name: workflow.name,
            item: `https://workflowkit.io/workflows/${workflow.id}`,
          },
        ]}
      />
      {/* Rest of product page content */}
    </>
  );
}
```

### 3. Help Page Metadata

```typescript
// app/help/[slug]/page.tsx

import type { Metadata } from 'next';
import { FAQPageSchema } from '@/components/FAQPageSchema';
import { HowToSchema } from '@/components/HowToSchema';
import { getHelpPage } from '@/lib/api';

interface Props {
  params: { slug: string };
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const page = await getHelpPage(params.slug);

  return {
    title: `${page.title} - Help | WorkflowKit`,
    description: page.description,
  };
}

export default async function HelpPage({ params }: Props) {
  const page = await getHelpPage(params.slug);

  return (
    <>
      {page.type === 'faq' && (
        <FAQPageSchema faqs={page.content.faqs} />
      )}

      {page.type === 'howto' && (
        <HowToSchema
          name={page.title}
          description={page.description}
          steps={page.content.steps}
          estimatedTime={page.content.estimatedTime}
          author={{ name: 'WorkflowKit Support' }}
          datePublished={page.createdAt}
          dateModified={page.updatedAt}
        />
      )}

      {/* Rest of help page content */}
    </>
  );
}
```

---

## Testing & Validation

### 1. Schema Validation Test

```typescript
// lib/schema-validator.ts

export async function validateSchema(schema: any): Promise<{
  valid: boolean;
  errors: string[];
}> {
  const errors: string[] = [];

  // Check required @context
  if (!schema['@context']) {
    errors.push('Missing @context property');
  }

  // Check required @type
  if (!schema['@type']) {
    errors.push('Missing @type property');
  }

  // Type-specific validation
  switch (schema['@type']) {
    case 'Product':
      if (!schema.name) errors.push('Product: missing name');
      if (!schema.description) errors.push('Product: missing description');
      if (!schema.image) errors.push('Product: missing image');
      if (!schema.offers) errors.push('Product: missing offers');
      break;

    case 'Organization':
      if (!schema.name) errors.push('Organization: missing name');
      if (!schema.url) errors.push('Organization: missing url');
      break;

    case 'FAQPage':
      if (!Array.isArray(schema.mainEntity)) {
        errors.push('FAQPage: mainEntity must be an array');
      }
      schema.mainEntity?.forEach((faq, index) => {
        if (!faq.name) errors.push(`FAQ ${index}: missing question name`);
        if (!faq.acceptedAnswer?.text) {
          errors.push(`FAQ ${index}: missing answer text`);
        }
      });
      break;

    case 'HowTo':
      if (!schema.name) errors.push('HowTo: missing name');
      if (!Array.isArray(schema.step)) {
        errors.push('HowTo: step must be an array');
      }
      break;
  }

  return {
    valid: errors.length === 0,
    errors,
  };
}
```

### 2. Jest Test Example

```typescript
// __tests__/schema-builder.test.ts

import { SchemaBuilder, SchemaContext } from '@/lib/schema-builder';
import { validateSchema } from '@/lib/schema-validator';

describe('SchemaBuilder', () => {
  let builder: SchemaBuilder;
  const context: SchemaContext = {
    siteUrl: 'https://example.com',
    organizationName: 'Test Org',
    organizationLogo: 'https://example.com/logo.png',
  };

  beforeEach(() => {
    builder = new SchemaBuilder(context);
  });

  describe('Organization Schema', () => {
    it('should build valid organization schema', async () => {
      const schema = builder.buildOrganization({
        name: 'Test Company',
        description: 'A test company',
      });

      const validation = await validateSchema(schema);
      expect(validation.valid).toBe(true);
    });

    it('should include all provided fields', () => {
      const schema = builder.buildOrganization({
        name: 'Test Company',
        description: 'A test company',
        foundingDate: '2023-01-15',
        email: 'test@example.com',
        phone: '+1-555-0123',
      });

      expect(schema.name).toBe('Test Company');
      expect(schema.foundingDate).toBe('2023-01-15');
      expect(schema.contactPoint.email).toBe('test@example.com');
    });
  });

  describe('Product Schema', () => {
    it('should build valid product schema', async () => {
      const schema = builder.buildProduct({
        name: 'Test Product',
        description: 'A test product',
        image: 'https://example.com/product.jpg',
        url: 'https://example.com/products/test',
        category: 'Test Category',
        price: 9.99,
      });

      const validation = await validateSchema(schema);
      expect(validation.valid).toBe(true);
    });

    it('should include rating when provided', () => {
      const schema = builder.buildProduct({
        name: 'Test Product',
        description: 'A test product',
        image: 'https://example.com/product.jpg',
        url: 'https://example.com/products/test',
        category: 'Test Category',
        rating: { value: 4.5, count: 100 },
      });

      expect(schema.aggregateRating.ratingValue).toBe(4.5);
      expect(schema.aggregateRating.ratingCount).toBe(100);
    });
  });

  describe('FAQ Schema', () => {
    it('should build valid FAQ schema', async () => {
      const schema = builder.buildFAQPage([
        {
          question: 'What is this?',
          answer: 'This is a test FAQ.',
        },
      ]);

      const validation = await validateSchema(schema);
      expect(validation.valid).toBe(true);
    });

    it('should create FAQ items from input', () => {
      const faqs = [
        { question: 'Q1?', answer: 'A1' },
        { question: 'Q2?', answer: 'A2' },
      ];

      const schema = builder.buildFAQPage(faqs);
      expect(schema.mainEntity).toHaveLength(2);
      expect(schema.mainEntity[0].name).toBe('Q1?');
    });
  });
});
```

### 3. Manual Testing Script

```typescript
// scripts/validate-schemas.ts

import { SchemaBuilder, SchemaContext } from '@/lib/schema-builder';
import { validateSchema } from '@/lib/schema-validator';

async function validateAllSchemas() {
  const context: SchemaContext = {
    siteUrl: 'https://workflowkit.io',
    organizationName: 'WorkflowKit',
    organizationLogo: 'https://workflowkit.io/logo.png',
  };

  const builder = new SchemaBuilder(context);

  const schemas = {
    organization: builder.buildOrganization({
      name: 'WorkflowKit',
      description: 'Email workflow automation',
    }),
    website: builder.buildWebSite({
      name: 'WorkflowKit',
      searchUrl: 'https://workflowkit.io/search?q={search_term_string}',
    }),
    product: builder.buildProduct({
      name: 'Email Auto-Reply',
      description: 'Automated email responses',
      image: 'https://workflowkit.io/templates/email-auto-reply.jpg',
      url: 'https://workflowkit.io/workflows/email-auto-reply',
      category: 'Email Automation',
    }),
  };

  for (const [name, schema] of Object.entries(schemas)) {
    const validation = await validateSchema(schema);
    console.log(`\n${name}:`);
    console.log(`Valid: ${validation.valid}`);
    if (!validation.valid) {
      console.log(`Errors: ${validation.errors.join(', ')}`);
    }
    console.log(JSON.stringify(schema, null, 2));
  }
}

validateAllSchemas().catch(console.error);
```

---

## Summary

This TypeScript implementation provides:

1. **Reusable React Components** for common schema types
2. **Server-Side Schema Builders** for dynamic content
3. **Utility Functions** for ratings, pricing, and authors
4. **Next.js Metadata Integration** for SEO
5. **Comprehensive Testing** with Jest and validation scripts

Key principles:
- Always validate schemas before deployment
- Keep schema data in sync with visible page content
- Use TypeScript for type safety
- Test all schema types thoroughly
- Monitor Search Console for errors
- Update schemas when content changes

Reference files:
- /Users/micahjohnson/Documents/Github/WORKWAY/Cloudflare/docs/AEO_STRUCTURED_DATA_GUIDE_2024.md
- /Users/micahjohnson/Documents/Github/WORKWAY/Cloudflare/docs/AEO_IMPLEMENTATION_CHECKLIST.md
