# WORKWAY Marketplace - Testing & Debugging Checklist

## Deployment Info
- **Local Dev**: http://localhost:3000/
- **Production**: https://04fef178.workway.pages.dev
- **Main URL**: https://workway.pages.dev

## Testing Checklist

### ✅ 1. Page Load & Performance
- [ ] Page loads without errors
- [ ] All assets load (fonts, CSS, JS)
- [ ] No console errors
- [ ] Page loads in < 3 seconds
- [ ] Google Fonts (Inter) load correctly
- [ ] Lucide icons display properly

### ✅ 2. Animations
- [ ] **Hero Section**
  - [ ] WORKWAY letters animate in individually (kinetic typography)
  - [ ] Workflow icon rotates continuously
  - [ ] Gradient mesh background animates
  - [ ] Text fades in with stagger delay
- [ ] **Scroll Animations**
  - [ ] Background parallax works (moves slower than scroll)
  - [ ] Integration cards fade/slide in on scroll
  - [ ] Feature cards pop up with stagger
  - [ ] Animations reverse when scrolling up
- [ ] **Hover Effects**
  - [ ] Buttons scale on hover (1.05x) and tap (0.98x)
  - [ ] Integration cards lift and glow orange
  - [ ] Feature card icons rotate on hover
  - [ ] Links have smooth transitions

### ✅ 3. Custom Cursor (Desktop Only)
- [ ] Custom cursor appears on desktop
- [ ] Cursor follows mouse smoothly
- [ ] Cursor scales up on hover over links/buttons
- [ ] Outer ring has smooth lag effect
- [ ] Standard cursor on mobile/tablet
- [ ] No cursor issues with devtools

### ✅ 4. Responsive Design
- [ ] **Desktop (>1024px)**
  - [ ] Bento grid shows asymmetric layout
  - [ ] All 6 feature cards visible
  - [ ] Header shows full navigation
  - [ ] Custom cursor works
- [ ] **Tablet (768px - 1024px)**
  - [ ] Layout adapts properly
  - [ ] Mobile menu accessible
  - [ ] Cards stack appropriately
- [ ] **Mobile (<768px)**
  - [ ] Single column layout
  - [ ] Hamburger menu works
  - [ ] Touch interactions smooth
  - [ ] No horizontal scroll

### ✅ 5. Navigation & Links
- [ ] Header logo links to home
- [ ] All navigation links work
- [ ] Mobile menu opens/closes
- [ ] Mobile menu backdrop click closes menu
- [ ] CTA buttons navigate correctly
- [ ] Footer links work
- [ ] "View all integrations" link works

### ✅ 6. Accessibility
- [ ] **Keyboard Navigation**
  - [ ] Tab key navigates through interactive elements
  - [ ] Focus states visible (orange outline)
  - [ ] Enter/Space activates buttons
  - [ ] Escape closes mobile menu
- [ ] **Screen Reader**
  - [ ] ARIA labels on all buttons
  - [ ] Semantic HTML (nav, header, footer, sections)
  - [ ] Alt text on icons (aria-hidden on decorative)
  - [ ] Heading hierarchy correct (h1 → h2 → h3)
- [ ] **Color Contrast**
  - [ ] Text meets WCAG AA (orange on dark: 4.5:1+)
  - [ ] Links distinguishable
  - [ ] Focus indicators visible

### ✅ 7. Typography
- [ ] Inter font loads and displays
- [ ] Headings use tight letter-spacing (-0.03em)
- [ ] Font weights correct (400, 600, 700, 900)
- [ ] Line heights appropriate
- [ ] Text hierarchy clear

### ✅ 8. Bento Grid Layout
- [ ] Large cards (2x2): Lightning Fast, Analytics
- [ ] Medium cards (2x1): Enterprise Security, Developer Friendly
- [ ] Small cards (1x1): Affordable Pricing, Automated Workflows
- [ ] Grid adapts on mobile (single column)
- [ ] Cards have equal heights in rows

### ✅ 9. Known Issues to Check

#### Potential Issues:
1. **GSAP Parallax on iOS Safari**
   - Issue: iOS Safari may have scrolling performance issues
   - Test: Scroll on iPhone/iPad
   - Fix: Add `will-change: transform` if needed

2. **Custom Cursor Conflicts**
   - Issue: Devtools may interfere with cursor
   - Test: Open/close devtools
   - Fix: Cursor should hide/show correctly

3. **Framer Motion in Production**
   - Issue: Animations may not work in production build
   - Test: Check deployed site animations
   - Fix: Ensure SSR compatibility

4. **Font Loading Flash**
   - Issue: FOUT (Flash of Unstyled Text)
   - Test: Hard refresh page
   - Fix: font-display: swap already configured

5. **Mobile Menu Scroll Lock**
   - Issue: Body should not scroll when menu open
   - Test: Open menu, try to scroll
   - Fix: Add `overflow: hidden` if needed

6. **ScrollTrigger on Page Navigation**
   - Issue: Animations may not trigger on back/forward
   - Test: Navigate away and back
   - Fix: ScrollTrigger.refresh() if needed

## Manual Testing Steps

### Step 1: Desktop Testing (Chrome/Firefox/Safari)
```
1. Open http://localhost:3000
2. Watch hero animation (letters should animate in)
3. Move mouse (custom cursor should appear)
4. Hover over buttons (cursor grows, buttons scale)
5. Scroll down slowly (parallax background)
6. Scroll to features (cards pop up)
7. Tab through elements (focus states visible)
8. Resize window (responsive breakpoints)
```

### Step 2: Mobile Testing (iOS/Android)
```
1. Open on mobile device
2. Tap hamburger menu
3. Navigate through menu
4. Scroll and check animations
5. Test touch interactions
6. Check performance (should be smooth)
```

### Step 3: Production Testing
```
1. Visit https://04fef178.workway.pages.dev
2. Repeat all desktop tests
3. Check Network tab (all assets load)
4. Check Console (no errors)
5. Run Lighthouse audit
```

## Performance Targets

### Lighthouse Scores (Desktop)
- Performance: 95+
- Accessibility: 100
- Best Practices: 95+
- SEO: 90+

### Core Web Vitals
- LCP (Largest Contentful Paint): < 2.0s
- INP (Interaction to Next Paint): < 150ms
- CLS (Cumulative Layout Shift): < 0.05

## Debugging Commands

### Check for console errors:
```bash
# Open browser devtools
# Console tab should show no errors
```

### Test build locally:
```bash
npm run build
npm run preview
```

### Check bundle size:
```bash
npm run build
# Look for warnings about chunk sizes
```

### Clear cache and rebuild:
```bash
rm -rf node_modules/.vite dist
npm run build
```

## Common Fixes

### Fix 1: Animations not working
```
Issue: GSAP animations don't trigger
Solution: Check ScrollTrigger is registered
File: src/routes/index.tsx line 8
```

### Fix 2: Custom cursor not showing
```
Issue: Cursor CSS not loaded
Solution: Check CustomCursor component mounted
File: src/routes/__root.tsx line 59
```

### Fix 3: Fonts not loading
```
Issue: FOUT or system fonts showing
Solution: Check Google Fonts link in head
File: src/routes/__root.tsx line 28-40
```

### Fix 4: Mobile menu not working
```
Issue: Menu doesn't open
Solution: Check Header state management
File: src/components/Header.tsx
```

### Fix 5: Responsive layout broken
```
Issue: Grid doesn't adapt
Solution: Check Tailwind breakpoints
File: src/routes/index.tsx (Bento grid classes)
```

## Test Results

### ✅ Passing Tests:
- [ ] All critical features working
- [ ] No console errors
- [ ] Animations smooth
- [ ] Responsive on all devices
- [ ] Accessible via keyboard
- [ ] Performance targets met

### ❌ Failing Tests:
- [ ] List any failing tests here

### 🔧 Bugs Found:
- [ ] List bugs and fixes here

## Next Steps After Testing

1. Fix all critical bugs
2. Optimize any performance issues
3. Add analytics tracking
4. Set up error monitoring (Sentry)
5. Configure custom domain
6. Set up CI/CD pipeline
