# Design System - Implementation Fixes

Quick reference guide for implementing visual consistency improvements across WORKWAY.

## 1. CREATE ANIMATION CONSTANTS

**File to create:** `/apps/web/src/lib/animations.ts`

```typescript
/**
 * Centralized animation configuration for framer-motion
 * Ensures consistent timing and easing across all pages
 */

export const ANIMATION_TIMING = {
  fast: 0.15,
  standard: 0.3,
  slow: 0.5,
  verySlow: 0.8,
} as const;

export const ANIMATION_PRESETS = {
  // Fade in with upward slide
  fadeInUp: {
    initial: { opacity: 0, y: 20 },
    animate: { opacity: 1, y: 0 },
    exit: { opacity: 0, y: 20 },
  },
  
  // Fade in with scale
  fadeInScale: {
    initial: { opacity: 0, scale: 0.95 },
    animate: { opacity: 1, scale: 1 },
    exit: { opacity: 0, scale: 0.95 },
  },
  
  // Fade only
  fadeOnly: {
    initial: { opacity: 0 },
    animate: { opacity: 1 },
    exit: { opacity: 0 },
  },
  
  // Modal animation (more dramatic)
  modalReveal: {
    initial: { opacity: 0, scale: 0.95, y: 20 },
    animate: { opacity: 1, scale: 1, y: 0 },
    exit: { opacity: 0, scale: 0.95, y: 20 },
  },
} as const;

export const EASING = {
  default: 'easeInOut',
  smooth: [0.25, 0.46, 0.45, 0.94],
  elastic: [0.16, 1, 0.3, 1],
} as const;

export function getStaggerDelay(index: number): number {
  return index * ANIMATION_TIMING.standard * 0.15; // 0.045s per item
}

export const ANIMATION_CONFIGS = {
  card: {
    initial: ANIMATION_PRESETS.fadeInUp.initial,
    whileInView: ANIMATION_PRESETS.fadeInUp.animate,
    viewport: { once: true, margin: '-10%' },
    transition: { duration: ANIMATION_TIMING.standard },
  },
  
  staggeredCard: (index: number) => ({
    initial: ANIMATION_PRESETS.fadeInUp.initial,
    whileInView: ANIMATION_PRESETS.fadeInUp.animate,
    viewport: { once: true, margin: '-10%' },
    transition: { 
      duration: ANIMATION_TIMING.standard,
      delay: getStaggerDelay(index),
    },
  }),
  
  modal: {
    initial: ANIMATION_PRESETS.modalReveal.initial,
    animate: ANIMATION_PRESETS.modalReveal.animate,
    exit: ANIMATION_PRESETS.modalReveal.exit,
    transition: { duration: ANIMATION_TIMING.standard },
  },
  
  backdrop: {
    initial: { opacity: 0 },
    animate: { opacity: 1 },
    exit: { opacity: 0 },
    transition: { duration: ANIMATION_TIMING.fast },
  },
} as const;
```

**Usage:**
```typescript
import { ANIMATION_CONFIGS, ANIMATION_TIMING } from '@/lib/animations'

// Instead of:
<motion.div
  initial={{ opacity: 0, y: 10 }}
  whileInView={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3, delay: index * 0.05 }}
/>

// Use:
<motion.div {...ANIMATION_CONFIGS.staggeredCard(index)} />
```

---

## 2. CREATE ICON CONFIGURATION

**File to create:** `/apps/web/src/lib/icons.ts`

```typescript
/**
 * Centralized icon sizing and mapping
 */

import {
  Check,
  CheckCircle2,
  AlertCircle,
  Clock,
  Eye,
  Zap,
  Sparkles,
  Bot,
  type LucideIcon,
} from 'lucide-react';

// Standard icon sizes
export const ICON_SIZES = {
  xs: { w: 3, h: 3, size: 12 },      // w-3 h-3
  sm: { w: 4, h: 4, size: 16 },      // w-4 h-4
  base: { w: 5, h: 5, size: 20 },    // w-5 h-5
  lg: { w: 6, h: 6, size: 24 },      // w-6 h-6
  xl: { w: 8, h: 8, size: 32 },      // w-8 h-8
  '2xl': { w: 12, h: 12, size: 48 }, // w-12 h-12
} as const;

export const getIconClass = (size: keyof typeof ICON_SIZES): string => {
  const s = ICON_SIZES[size];
  return `w-${s.w} h-${s.h}`;
};

// Icon mapping for common concepts
export const ICON_MAP = {
  // Status indicators
  status: {
    active: CheckCircle2,
    paused: Clock,
    error: AlertCircle,
    draft: Eye,
    success: Check,
  },
  
  // Workflow types
  workflow: {
    integration: Zap,
    aiEnhanced: Sparkles,
    aiNative: Bot,
  },
} as const;

export const getStatusIcon = (status: string): LucideIcon => {
  const statusKey = status.toLowerCase() as keyof typeof ICON_MAP.status;
  return ICON_MAP.status[statusKey] || AlertCircle;
};

export const getWorkflowIcon = (type: string): LucideIcon => {
  const typeKey = type.replace(/-/g, '') as keyof typeof ICON_MAP.workflow;
  return ICON_MAP.workflow[typeKey] || Zap;
};
```

**Usage:**
```typescript
import { ICON_SIZES, getIconClass, getStatusIcon } from '@/lib/icons'

// Instead of:
<Star className="w-6 h-6" />
<Check className="w-4 h-4" />

// Use:
<Star className={getIconClass('lg')} />
<Check className={getIconClass('sm')} />

// Instead of custom status icons:
const icon = status === 'active' ? <CheckCircle2 /> : status === 'paused' ? <Clock /> : ...

// Use:
const StatusIcon = getStatusIcon(status)
<StatusIcon className={getIconClass('base')} />
```

---

## 3. CREATE CARD COMPONENT LIBRARY

**File to create:** `/apps/web/src/components/Card.tsx`

```typescript
import React from 'react';
import { motion, MotionProps } from 'framer-motion';
import { ANIMATION_CONFIGS } from '@/lib/animations';

interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: 'interactive' | 'stat' | 'config';
  animate?: boolean;
  index?: number;
  as?: 'div' | 'a' | 'button';
  href?: string;
}

const cardVariants = {
  interactive: 'border border-neutral-400 hover:border-neutral-600 p-8 transition-colors duration-300',
  stat: 'bg-neutral-800 border border-neutral-700 p-6',
  config: 'border border-neutral-700 hover:border-neutral-400 p-6 transition-colors duration-300',
} as const;

export const Card = React.forwardRef<HTMLDivElement, CardProps>(
  (
    {
      variant = 'interactive',
      animate = true,
      index = 0,
      as: Component = 'div',
      className = '',
      ...props
    },
    ref
  ) => {
    const cardClass = cardVariants[variant];
    const combinedClass = `${cardClass} ${className}`.trim();
    
    if (!animate) {
      return React.createElement(Component, {
        ref,
        className: combinedClass,
        ...props,
      } as any);
    }
    
    return React.createElement(motion[Component as keyof typeof motion], {
      ref,
      className: combinedClass,
      {...ANIMATION_CONFIGS.staggeredCard(index)},
      ...props,
    } as any);
  }
);

Card.displayName = 'Card';

// Card subcomponents for consistent structure
export const CardHeader = ({ className = '', ...props }: React.HTMLAttributes<HTMLDivElement>) => (
  <div className={`border-b border-neutral-700 pb-4 mb-4 ${className}`} {...props} />
);

export const CardBody = ({ className = '', ...props }: React.HTMLAttributes<HTMLDivElement>) => (
  <div className={`space-y-3 ${className}`} {...props} />
);

export const CardFooter = ({ className = '', ...props }: React.HTMLAttributes<HTMLDivElement>) => (
  <div className={`border-t border-neutral-700 pt-4 mt-4 ${className}`} {...props} />
);
```

**Usage:**
```typescript
import { Card, CardHeader, CardBody, CardFooter } from '@/components/Card'

// Interactive card
<Card variant="interactive" index={0}>
  <CardHeader>
    <h3>Card Title</h3>
  </CardHeader>
  <CardBody>
    <p>Card content</p>
  </CardBody>
  <CardFooter>
    <button>Action</button>
  </CardFooter>
</Card>

// Stat card
<Card variant="stat">
  <div className="flex items-start gap-4">
    <div className="p-3 bg-blue-600/10 rounded-lg">
      <DollarSign className="w-6 h-6 text-blue-400" />
    </div>
    <div>
      <div className="text-2xl font-semibold">$1,234</div>
      <div className="text-sm text-neutral-400">Total Earnings</div>
    </div>
  </div>
</Card>
```

---

## 4. CREATE BUTTON COMPONENT LIBRARY

**File to create:** `/apps/web/src/components/Button.tsx`

```typescript
import React from 'react';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger' | 'success' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

const variants = {
  primary: 'bg-neutral-800 border border-neutral-400 hover:bg-neutral-700 text-neutral-100',
  secondary: 'border border-neutral-700 hover:bg-neutral-700 text-neutral-100',
  danger: 'border border-red-400 text-red-400 hover:bg-red-400/10',
  success: 'bg-green-400 hover:bg-green-300 text-neutral-900 font-medium',
  ghost: 'text-neutral-400 hover:text-neutral-100',
} as const;

const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
} as const;

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', className = '', isLoading = false, ...props }, ref) => {
    const variantClass = variants[variant];
    const sizeClass = sizes[size];
    const baseClass = 'font-inter-tight font-medium transition-colors duration-300 flex items-center justify-center gap-2';
    
    const combinedClass = `${baseClass} ${variantClass} ${sizeClass} ${className}`.trim();
    
    return (
      <button
        ref={ref}
        disabled={isLoading || props.disabled}
        className={`${combinedClass} ${isLoading ? 'opacity-50 cursor-not-allowed' : ''}`}
        {...props}
      />
    );
  }
);

Button.displayName = 'Button';
```

**Usage:**
```typescript
import { Button } from '@/components/Button'
import { Loader2 } from 'lucide-react'

<Button variant="primary" size="md">
  Install Workflow
</Button>

<Button variant="danger" size="sm">
  Uninstall
</Button>

<Button variant="success" isLoading={isInstalling}>
  {isInstalling && <Loader2 className="w-4 h-4 animate-spin" />}
  Enable Workflow
</Button>
```

---

## 5. EXTEND DESIGN SYSTEM COLORS

**Update:** `/apps/web/src/styles.css`

```css
@theme {
  /* Add missing button colors */
  --color-button-primary: #64a7ff;      /* blue-400 */
  --color-button-primary-hover: #4a8ae6;
  --color-button-secondary: #e3e3e3;    /* neutral-400 */
  --color-button-secondary-hover: #ababab;
  
  /* Add semantic color variants */
  --color-success-light: #eef9f5;
  --color-success-bg: rgba(116, 209, 132, 0.1);
  --color-error-light: #fff5f6;
  --color-error-bg: rgba(254, 86, 107, 0.1);
  --color-warning-light: #fffae9;
  --color-warning-bg: rgba(239, 196, 44, 0.1);
  --color-info-light: #e3efff;
  --color-info-bg: rgba(100, 167, 255, 0.1);
  
  /* Add overlay colors */
  --color-overlay-dark: rgba(0, 0, 0, 0.6);
  --color-overlay-light: rgba(0, 0, 0, 0.5);
}
```

---

## 6. UPDATE TAILWIND CONFIG

**Update:** `/apps/web/tailwind.config.ts`

```typescript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      fontFamily: {
        'inter-tight': ['Inter Tight', 'sans-serif'],
        inter: ['Inter', 'sans-serif'],
      },
      fontSize: {
        'display-10': 'var(--font-size-display-10)',
        'display-9': 'var(--font-size-display-9)',
        'display-8': 'var(--font-size-display-8)',
        'display-7': 'var(--font-size-display-7)',
        'display-6': 'var(--font-size-display-6)',
        'display-5': 'var(--font-size-display-5)',
        'display-4': 'var(--font-size-display-4)',
        'display-3': 'var(--font-size-display-3)',
        'display-2': 'var(--font-size-display-2)',
        'display-1': 'var(--font-size-display-1)',
      },
      colors: {
        'button-primary': 'var(--color-button-primary)',
        'button-primary-hover': 'var(--color-button-primary-hover)',
        'success-bg': 'var(--color-success-bg)',
        'error-bg': 'var(--color-error-bg)',
        'warning-bg': 'var(--color-warning-bg)',
        'info-bg': 'var(--color-info-bg)',
      },
    },
  },
  plugins: [],
}
```

---

## 7. UPDATE COMPONENT EXPORTS

**Update:** `/apps/web/src/components/index.ts` (create if missing)

```typescript
// Export all reusable components
export { Card, CardHeader, CardBody, CardFooter } from './Card'
export { Button } from './Button'
export { default as Header } from './Header'
export { default as CustomCursor } from './CustomCursor'
```

---

## 8. MIGRATION CHECKLIST

### Phase 1: Foundation (Week 1)
- [ ] Create `/apps/web/src/lib/animations.ts`
- [ ] Create `/apps/web/src/lib/icons.ts`
- [ ] Create `/apps/web/src/components/Card.tsx`
- [ ] Create `/apps/web/src/components/Button.tsx`
- [ ] Update `/apps/web/src/styles.css` with new tokens
- [ ] Update `/apps/web/tailwind.config.ts`

### Phase 2: Critical Pages (Week 2-3)
- [ ] Update `marketplace.tsx` - Use new Card and animation constants
- [ ] Update `integrations.tsx` - Standardize card styling
- [ ] Update `workflows.tsx` - Use new Button and Card components
- [ ] Update `workflow.$workflowId.tsx` - Use animations and icons

### Phase 3: Secondary Pages (Week 4)
- [ ] Update `developer.dashboard.tsx` - Add animations, use Card component
- [ ] Update `settings.oauth.tsx` - Use Card component
- [ ] Update `developer.settings.tsx` - Add animations
- [ ] Update auth pages - Add animations

### Phase 4: Polish (Week 5+)
- [ ] Create Storybook stories for all components
- [ ] Update documentation
- [ ] Add design token comments to code
- [ ] Regular design audits

---

## 9. BEFORE/AFTER EXAMPLES

### Animation Example

**BEFORE:**
```typescript
<motion.div
  className="border border-neutral-400 p-8"
  initial={{ opacity: 0, y: 10 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true }}
  transition={{ duration: 0.3, delay: index * 0.05 }}
>
```

**AFTER:**
```typescript
<Card variant="interactive" index={index}>
```

### Icon Example

**BEFORE:**
```typescript
<Check className="w-4 h-4 text-green-400" />
<CheckCircle2 className="w-3 h-3 text-neutral-100" />
<Star size={16} className="text-yellow-400" />
```

**AFTER:**
```typescript
<Check className={getIconClass('sm')} />
<CheckCircle2 className={getIconClass('xs')} />
<Star className={getIconClass('base')} />
```

### Button Example

**BEFORE:**
```typescript
<button className="px-4 py-2 bg-neutral-800 border border-neutral-400 hover:bg-neutral-700 text-neutral-100 font-inter-tight font-medium transition-colors duration-300">
  Enable Workflow
</button>
```

**AFTER:**
```typescript
<Button variant="primary" size="md">
  Enable Workflow
</Button>
```

---

## VALIDATION CHECKLIST

After implementing changes:

- [ ] All animations use `ANIMATION_TIMING` constants
- [ ] All icons use `getIconClass()` or `ICON_SIZES`
- [ ] All buttons use `<Button />` component
- [ ] All cards use `<Card />` component
- [ ] No inline `transition` props in framer-motion
- [ ] No arbitrary color values in className
- [ ] No mixed unit systems (all px or all rem)
- [ ] Page load animations consistent across all pages
- [ ] Stagger delays consistent (0.045s per item)
- [ ] No hardcoded colors - use design system values
- [ ] Button hover states working across all variants
- [ ] Modal animations smooth and consistent
- [ ] Loading spinners unified (use Lucide Loader2)
- [ ] Error messages follow new pattern

