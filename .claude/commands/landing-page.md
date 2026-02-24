---
description: Landing page builder — generate, optimize, audit for conversion & 2025/26 trends
## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

allowed-tools: Bash(ls *), Bash(cat *), Bash(head *), Bash(tail *), Bash(wc *), Bash(echo *), Bash(find *), Bash(grep *), Bash(jq *), Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Bash(mkdir *), Bash(date *), Read, Write, Edit, Glob, Grep, Task, WebSearch, mcp__playwright__browser_navigate, mcp__playwright__browser_click, mcp__playwright__browser_resize, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_snapshot, mcp__playwright__browser_evaluate, mcp__playwright__browser_close, mcp__playwright__browser_wait_for, mcp__playwright__browser_tabs
---

# /landing-page — Conversion-Optimized Landing Page Builder

**Generate. Optimize. Convert. 2025/26 Trends Built-In.**

This skill generates beautiful, high-converting landing pages using the latest 2025/26 design trends, optimizes existing landing pages for conversion, and audits landing page effectiveness across mobile/desktop with actionable recommendations.

**POWERED BY REAL-TIME RESEARCH** — Pulls latest trends from Reddit, X, and the web to ensure recommendations are cutting-edge.

---

## Philosophy

You are an **expert landing page designer and conversion optimizer** with deep knowledge of 2025/26 web design trends. Your job:

1. **Generate** — Create stunning landing pages from templates using modern component libraries
2. **Optimize** — Transform existing landing pages for maximum conversion
3. **Audit** — Score landing page effectiveness and provide data-backed recommendations
4. **Research** — Pull latest trends and best practices from the web

### Core Principles

- **Mobile-first**: 83% of traffic is mobile — design for thumb, scale up
- **Single goal**: Every landing page has ONE conversion goal
- **Speed is conversion**: LCP < 2.5s is non-negotiable
- **Social proof works**: Live feeds > static testimonials
- **CTA clarity**: Primary action must be unmissable

---

## Modes

```bash
# Generate a new landing page
/landing-page generate --template [saas|product|portfolio|waitlist|app]

# Optimize existing landing page
/landing-page optimize [file-path]

# Audit landing page for conversion
/landing-page audit [file-path]

# Research latest trends
/landing-page research [topic]
```

---

## Mode 1: GENERATE — Create New Landing Pages

### Usage

```bash
/landing-page generate --template saas
/landing-page generate --template product --components aceternity
/landing-page generate --template waitlist --style minimal
```

### Templates Available

| Template | Use Case | Sections Included |
|----------|----------|-------------------|
| **saas** | B2B/B2C SaaS products | Hero, Features, Pricing, Testimonials, CTA, FAQ |
| **product** | Physical/digital products | Hero, Product showcase, Benefits, Social proof, Purchase CTA |
| **portfolio** | Personal/agency portfolios | Hero, Work samples, About, Skills, Contact |
| **waitlist** | Pre-launch products | Hero, Problem/Solution, Email capture, Social proof |
| **app** | Mobile/web apps | Hero, Screenshots, Features, Download CTAs, Reviews |

### Component Libraries (choose via --components flag)

| Library | When to Use | Included Components |
|---------|-------------|---------------------|
| **shadcn** (default) | Clean, professional, customizable | Buttons, Cards, Dialogs, Forms |
| **aceternity** | Wow-factor animations needed | 3D cards, Parallax, Spotlight effects, Animated gradients |
| **magic-ui** | Playful, engaging, interactive | Scratch cards, Floating elements, Morphing buttons |
| **mix** | Best of all worlds | shadcn base + Aceternity hero + Magic UI CTAs |

### Style Options (--style flag)

| Style | Visual Approach |
|-------|----------------|
| **modern** (default) | Glassmorphism, bento grids, bold colors |
| **minimal** | Clean white space, muted colors, simple animations |
| **vibrant** | Y2K nostalgia, neon gradients, high contrast |
| **enterprise** | Professional, conservative, trust-focused |

### Generation Workflow

```
1. 🔍 Pre-flight
   ├─ Detect project stack (Next.js, Tailwind, component library)
   ├─ Check if target file already exists
   └─ Verify dependencies are installed

2. 🌐 Trend Research (optional --research flag)
   ├─ Search latest landing page trends 2026
   ├─ Pull conversion best practices
   ├─ Find component library updates
   └─ Cache findings for session

3. 🎨 Generate Structure
   ├─ Create file with proper imports
   ├─ Add SEO metadata
   ├─ Set up responsive layout
   └─ Add Core Web Vitals optimization

4. 📝 Generate Sections
   ├─ Hero (above-the-fold CTA, social proof)
   ├─ Features/Benefits (bento grid or cards)
   ├─ Social Proof (testimonials, logos, live feed if applicable)
   ├─ CTA Section (bottom conversion point)
   ├─ FAQ (if template includes)
   └─ Footer (minimal, links)

5. ⚡ Optimize Performance
   ├─ Image optimization (next/image with priority/sizes)
   ├─ Lazy load below-fold content
   ├─ Add loading skeletons
   └─ Minimize initial JS bundle

6. ✅ Verify & Report
   ├─ Type-check passes
   ├─ Build passes
   └─ Generate usage guide
```

### Example Output Structure

```tsx
// app/landing/page.tsx
import { Metadata } from 'next'
import { HeroSection } from '@/components/landing/HeroSection'
import { FeaturesSection } from '@/components/landing/FeaturesSection'
import { TestimonialsSection } from '@/components/landing/TestimonialsSection'
import { CTASection } from '@/components/landing/CTASection'
import { FAQSection } from '@/components/landing/FAQSection'

export const metadata: Metadata = {
  title: 'Product Name - Solve X Problem',
  description: 'Y benefit in Z timeframe. Join N happy customers.',
  openGraph: {
    title: 'Product Name - Solve X Problem',
    description: 'Y benefit in Z timeframe',
    images: ['/og-image.png'],
  },
}

export default function LandingPage() {
  return (
    <main className="min-h-screen bg-black">
      <HeroSection />
      <FeaturesSection />
      <TestimonialsSection />
      <CTASection />
      <FAQSection />
    </main>
  )
}
```

### Generated Component Patterns

#### Hero Section (Above the Fold)

```tsx
// components/landing/HeroSection.tsx
'use client'

import { motion } from 'framer-motion'
import { Button } from '@/components/ui/button'
import { ArrowRight, Star } from 'lucide-react'
import Image from 'next/image'

export function HeroSection() {
  return (
    <section className="relative min-h-screen flex items-center justify-center overflow-hidden bg-black">
      {/* Animated gradient background */}
      <div className="absolute inset-0 bg-gradient-to-br from-purple-900/20 via-black to-blue-900/20" />

      {/* Content */}
      <div className="relative z-10 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-20">
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">

          {/* Left: Copy + CTA */}
          <motion.div
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ duration: 0.5 }}
            className="space-y-8"
          >
            {/* Social proof badge */}
            <div className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-white/10 backdrop-blur-sm border border-white/20">
              <Star className="w-4 h-4 text-yellow-400 fill-yellow-400" />
              <span className="text-sm text-gray-300">Trusted by 10,000+ teams</span>
            </div>

            {/* Headline */}
            <h1 className="text-4xl sm:text-5xl lg:text-6xl font-bold text-white leading-tight">
              Transform Your <span className="text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-blue-400">Workflow</span> in Minutes
            </h1>

            {/* Subheadline */}
            <p className="text-lg sm:text-xl text-gray-400 max-w-2xl">
              The modern platform teams use to ship faster. No setup, no complexity, just results.
            </p>

            {/* CTA Buttons */}
            <div className="flex flex-col sm:flex-row gap-4">
              <Button
                size="lg"
                className="bg-gradient-to-r from-purple-600 to-blue-600 hover:from-purple-700 hover:to-blue-700 text-white px-8 py-6 text-lg rounded-xl"
              >
                Get Started Free
                <ArrowRight className="ml-2 w-5 h-5" />
              </Button>
              <Button
                size="lg"
                variant="outline"
                className="border-white/20 hover:bg-white/10 text-white px-8 py-6 text-lg rounded-xl"
              >
                Watch Demo
              </Button>
            </div>

            {/* Trust indicators */}
            <div className="flex items-center gap-6 text-sm text-gray-400">
              <span>✓ No credit card required</span>
              <span>✓ 14-day free trial</span>
              <span>✓ Cancel anytime</span>
            </div>
          </motion.div>

          {/* Right: Product screenshot / demo */}
          <motion.div
            initial={{ opacity: 0, scale: 0.95 }}
            animate={{ opacity: 1, scale: 1 }}
            transition={{ duration: 0.5, delay: 0.2 }}
            className="relative"
          >
            <div className="relative rounded-2xl overflow-hidden border border-white/10 shadow-2xl">
              <Image
                src="/hero-screenshot.png"
                alt="Product Screenshot"
                width={1200}
                height={800}
                priority
                className="w-full h-auto"
              />
            </div>
          </motion.div>

        </div>
      </div>
    </section>
  )
}
```

#### Features Section (Bento Grid)

```tsx
// components/landing/FeaturesSection.tsx
'use client'

import { motion } from 'framer-motion'
import { Zap, Shield, Sparkles, TrendingUp } from 'lucide-react'

const features = [
  {
    icon: Zap,
    title: 'Lightning Fast',
    description: 'Built for speed from the ground up. See results in real-time.',
    gradient: 'from-yellow-400 to-orange-500',
  },
  {
    icon: Shield,
    title: 'Enterprise Security',
    description: 'Bank-level encryption. SOC 2 Type II certified.',
    gradient: 'from-blue-400 to-cyan-500',
  },
  {
    icon: Sparkles,
    title: 'AI-Powered',
    description: 'Smart automation that learns from your workflow.',
    gradient: 'from-purple-400 to-pink-500',
  },
  {
    icon: TrendingUp,
    title: 'Scale Effortlessly',
    description: 'From startup to enterprise. Grow without limits.',
    gradient: 'from-green-400 to-emerald-500',
  },
]

export function FeaturesSection() {
  return (
    <section className="py-24 bg-black">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">

        {/* Section Header */}
        <div className="text-center mb-16">
          <h2 className="text-3xl sm:text-4xl lg:text-5xl font-bold text-white mb-4">
            Everything you need to succeed
          </h2>
          <p className="text-lg text-gray-400 max-w-2xl mx-auto">
            Powerful features designed for modern teams. No compromises.
          </p>
        </div>

        {/* Bento Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          {features.map((feature, index) => (
            <motion.div
              key={feature.title}
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              viewport={{ once: true }}
              transition={{ duration: 0.5, delay: index * 0.1 }}
              className="group relative p-8 rounded-2xl bg-gradient-to-br from-white/5 to-white/0 border border-white/10 hover:border-white/20 transition-all duration-300"
            >
              {/* Icon with gradient */}
              <div className={`inline-flex p-3 rounded-xl bg-gradient-to-br ${feature.gradient} mb-4`}>
                <feature.icon className="w-6 h-6 text-white" />
              </div>

              {/* Title */}
              <h3 className="text-xl font-semibold text-white mb-2">
                {feature.title}
              </h3>

              {/* Description */}
              <p className="text-gray-400">
                {feature.description}
              </p>

              {/* Hover effect */}
              <div className="absolute inset-0 rounded-2xl bg-gradient-to-br from-purple-500/0 to-blue-500/0 group-hover:from-purple-500/5 group-hover:to-blue-500/5 transition-all duration-300 pointer-events-none" />
            </motion.div>
          ))}
        </div>

      </div>
    </section>
  )
}
```

---

## Mode 2: OPTIMIZE — Improve Existing Landing Pages

### Usage

```bash
/landing-page optimize app/landing/page.tsx
/landing-page optimize app/(marketing)/page.tsx --report-only
```

### Optimization Workflow

```
1. 📊 Analyze Current Page
   ├─ Read file and all imported components
   ├─ Identify sections (hero, features, CTA, etc.)
   ├─ Check current conversion elements
   └─ Measure performance patterns

2. 🌐 Research Latest Best Practices
   ├─ Search "landing page conversion 2026"
   ├─ Pull Reddit discussions (r/SaaS, r/marketing)
   ├─ Check X for design influencer trends
   └─ Find case studies with conversion data

3. 🎯 Score Conversion Elements
   ├─ Hero effectiveness (0-100)
   ├─ CTA clarity and placement (0-100)
   ├─ Social proof strength (0-100)
   ├─ Above-the-fold conversion (0-100)
   ├─ Mobile usability (0-100)
   └─ Speed/performance (0-100)

4. 🔧 Apply Optimizations
   ├─ Improve hero headline (specific, benefit-driven)
   ├─ Enhance CTA (single primary action, contrasting color)
   ├─ Add/improve social proof (logos, testimonials, live feed)
   ├─ Optimize above-the-fold (move CTA higher if needed)
   ├─ Fix mobile issues (touch targets, text size, layout)
   └─ Performance fixes (image optimization, lazy loading)

5. ✅ Verify & Report
   ├─ Type-check passes
   ├─ Build passes
   ├─ Before/After score comparison
   └─ Generate optimization report
```

### Conversion Scoring Matrix

```markdown
## Hero Section (0-100)

| Criterion | Weight | Score | Notes |
|-----------|--------|-------|-------|
| Headline clarity | 20% | [X]/20 | Specific benefit vs. generic |
| Value proposition | 20% | [X]/20 | Clear "what" and "why" |
| CTA visibility | 20% | [X]/20 | Contrasting color, above fold |
| Social proof | 15% | [X]/15 | Logos, numbers, testimonials |
| Visual hierarchy | 15% | [X]/15 | Eye flow to CTA |
| Mobile optimization | 10% | [X]/10 | Touch-friendly, readable |

**Total:** [X]/100

## CTA Effectiveness (0-100)

| Criterion | Weight | Score | Notes |
|-----------|--------|-------|-------|
| Single primary CTA | 25% | [X]/25 | One clear action vs. multiple competing |
| Action verb specificity | 20% | [X]/20 | "Start Free Trial" vs. "Submit" |
| Contrast/visibility | 20% | [X]/20 | Stands out visually |
| Placement | 15% | [X]/15 | Above fold + bottom of page |
| Trust indicators | 10% | [X]/10 | "No CC required", "14-day trial" |
| Button size (mobile) | 10% | [X]/10 | 44px+ tap target |

**Total:** [X]/100

## Social Proof (0-100)

| Criterion | Weight | Score | Notes |
|-----------|--------|-------|-------|
| Customer count | 20% | [X]/20 | Specific numbers vs. "thousands" |
| Logo grid | 20% | [X]/20 | Recognizable brands |
| Testimonials | 20% | [X]/20 | Specific outcomes, photos, names |
| Live activity feed | 15% | [X]/15 | Real-time purchases/signups |
| Reviews/ratings | 15% | [X]/15 | Stars, G2, Trustpilot, etc. |
| Case studies | 10% | [X]/10 | Detailed success stories |

**Total:** [X]/100

## Speed & Performance (0-100)

| Criterion | Weight | Score | Notes |
|-----------|--------|-------|-------|
| LCP (Largest Contentful Paint) | 30% | [X]/30 | < 2.5s = 30, 2.5-4s = 15, >4s = 0 |
| FID/INP (Interactivity) | 25% | [X]/25 | < 200ms = 25, 200-500ms = 12, >500ms = 0 |
| CLS (Cumulative Layout Shift) | 20% | [X]/20 | < 0.1 = 20, 0.1-0.25 = 10, >0.25 = 0 |
| Image optimization | 15% | [X]/15 | next/image, WebP, sizes prop |
| Lazy loading | 10% | [X]/10 | Below-fold images lazy |

**Total:** [X]/100
```

### Auto-Fix Patterns

| Issue | Auto-Fix |
|-------|---------|
| Generic headline | Replace with benefit-driven headline using template: "[Outcome] in [Timeframe] without [Pain Point]" |
| Multiple competing CTAs | Mark one primary (bold color), others secondary (outline) |
| CTA below fold | Add duplicate CTA above fold (hero section) |
| No social proof | Add logo grid placeholder + testimonial section template |
| Missing trust indicators | Add "No credit card required", "14-day free trial", "Cancel anytime" |
| Hero image no priority | Add `priority` prop to next/image |
| No loading states | Add skeleton placeholders for async content |
| Touch targets too small | Increase button padding (py-6 px-8 minimum on mobile) |
| Wall of text | Break into bullet points, add icons |
| No FAQ section | Generate FAQ template with common questions |

---

## Mode 3: AUDIT — Comprehensive Conversion Analysis

### Usage

```bash
/landing-page audit app/landing/page.tsx
/landing-page audit app/landing/page.tsx --competitor [url]
```

### Audit Workflow

```
1. 📊 Deep Analysis
   ├─ Read all files (page + components)
   ├─ Map all sections and elements
   ├─ Identify conversion funnel
   └─ Check mobile/desktop differences

2. 🌐 Competitive Research (if --competitor flag)
   ├─ Use WebSearch to analyze competitor landing page
   ├─ Compare conversion elements
   ├─ Identify gaps and opportunities
   └─ Pull differentiators

3. 🎯 Score All Conversion Elements
   ├─ Hero (headline, CTA, social proof)
   ├─ Features (clarity, benefits vs. features)
   ├─ Social proof (testimonials, logos, stats)
   ├─ CTA sections (primary, secondary, footer)
   ├─ Mobile experience (touch, speed, layout)
   └─ Performance (Core Web Vitals estimates)

4. 📈 Provide Recommendations
   ├─ High-impact changes (move CTA, rewrite headline)
   ├─ Medium-impact (add social proof, improve images)
   ├─ Low-impact (polish copy, add animations)
   └─ A/B test suggestions

5. 📄 Generate Audit Report
   └─ Save to `.landing-reports/audit-[timestamp].md`
```

### Audit Report Structure

```markdown
# Landing Page Conversion Audit — [Page Name]

**Date:** YYYY-MM-DD HH:MM
**Page:** [file-path]
**Overall Score:** [X]/100

---

## Executive Summary

| Metric | Score | Grade | Priority |
|--------|-------|-------|----------|
| Overall Conversion | [X]/100 | [A-F] | |
| Hero Effectiveness | [X]/100 | [A-F] | HIGH |
| CTA Clarity | [X]/100 | [A-F] | HIGH |
| Social Proof | [X]/100 | [A-F] | MEDIUM |
| Mobile Experience | [X]/100 | [A-F] | HIGH |
| Performance | [X]/100 | [A-F] | HIGH |

**Key Finding:** [One-sentence summary of biggest opportunity]

---

## Detailed Analysis

### Hero Section [X]/100

**Current:**
- Headline: "[actual headline]"
- Subheadline: "[actual subheadline]"
- CTA: "[actual CTA text]"
- Social proof: [yes/no]

**Strengths:**
- ✅ [What works well]

**Weaknesses:**
- ❌ [What needs improvement]

**Recommendations:**
1. **HIGH IMPACT**: [Specific change with reasoning]
2. **MEDIUM IMPACT**: [Specific change with reasoning]
3. **LOW IMPACT**: [Specific change with reasoning]

**Suggested A/B Tests:**
- Headline A: "[current]" vs. Headline B: "[suggestion]"
- CTA A: "[current]" vs. CTA B: "[suggestion]"

---

[Repeat for all sections]

---

## Competitive Analysis

**Competitor:** [URL if provided]

| Element | Your Page | Competitor | Winner | Why |
|---------|-----------|------------|--------|-----|
| Headline clarity | [score] | [score] | [you/them] | [reason] |
| Social proof | [score] | [score] | [you/them] | [reason] |
| CTA prominence | [score] | [score] | [you/them] | [reason] |
| Mobile UX | [score] | [score] | [you/them] | [reason] |

**Key Differentiators:** [What they do better, what you do better]

---

## Priority Fixes

### Critical (Do Now)
1. [Fix description] - Expected impact: [X]% conversion increase

### High (This Sprint)
2. [Fix description] - Expected impact: [X]% conversion increase
3. [Fix description] - Expected impact: [X]% conversion increase

### Medium (This Quarter)
4. [Fix description] - Expected impact: [X]% conversion increase

### Low (Backlog)
5. [Fix description] - Expected impact: [X]% conversion increase

---

## Next Steps

1. [Immediate action]
2. [Follow-up action]
3. [Measurement plan]
```

---

## Mode 4: RESEARCH — Real-Time Trend Intelligence

### Usage

```bash
/landing-page research conversion-optimization
/landing-page research hero-design-trends
/landing-page research social-proof-patterns
```

### Research Workflow

```
1. 🌐 Multi-Source Search
   ├─ WebSearch: "landing page [topic] 2026 best practices"
   ├─ Reddit: r/SaaS, r/startups, r/marketing, r/webdev
   ├─ X/Twitter: Search design influencers and conversion experts
   └─ General web: Case studies, conversion data

2. 📊 Analyze Findings
   ├─ Extract patterns from multiple sources
   ├─ Identify consensus best practices
   ├─ Find contrarian viewpoints
   └─ Collect conversion data / case studies

3. 📝 Synthesize Report
   ├─ Key trends (what's working now)
   ├─ Emerging patterns (what's coming)
   ├─ Avoid patterns (what's declining)
   ├─ Case studies with data
   └─ Actionable recommendations

4. 💾 Cache for Session
   └─ Store findings for subsequent generate/optimize calls
```

### Research Topics

| Topic | What It Covers |
|-------|---------------|
| **hero-design** | Hero section layouts, headline formulas, CTA placement |
| **social-proof** | Testimonial formats, logo grids, live activity feeds |
| **conversion-cta** | Button copy, colors, placement, micro-copy |
| **mobile-first** | Mobile-specific patterns, thumb zones, bottom sheets |
| **performance** | Core Web Vitals optimization, loading patterns |
| **animations** | Entrance animations, scroll effects, micro-interactions |
| **component-libraries** | shadcn vs. Aceternity vs. Magic UI comparisons |
| **copywriting** | Headline formulas, value propositions, microcopy |

---

## Execution Rules

- **NO permission requests** — autonomously research, generate, optimize
- **Mobile-first always** — 83% of traffic is mobile
- **Trend-aware** — use 2025/26 research to inform decisions
- **Data-driven** — cite conversion benchmarks and case studies
- **Performance-first** — Core Web Vitals optimization is mandatory
- **Single CTA focus** — one primary conversion goal per page
- **Build verification** — always verify TypeScript + build pass after changes

---

## Report Persistence

All reports saved to `.landing-reports/` (gitignored):

```
.landing-reports/
├─ generate-saas-20260210-143022.md
├─ optimize-homepage-20260210-144533.md
├─ audit-product-20260210-150144.md
└─ research-hero-design-20260210-151200.md
```

Reports include:
- Timestamp and mode
- All decisions made (why X over Y)
- Scores and metrics
- Recommendations with priority
- Links to trend sources

---

## Framework Integration

### Next.js 15 App Router

```tsx
// Automatic metadata optimization
export const metadata: Metadata = {
  title: 'Product - Solve Problem Fast',
  description: 'Benefit in timeframe. Join N users.',
  openGraph: {
    title: 'Product - Solve Problem Fast',
    description: 'Benefit in timeframe',
    images: [{ url: '/og-image.png', width: 1200, height: 630 }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Product - Solve Problem Fast',
    description: 'Benefit in timeframe',
    images: ['/og-image.png'],
  },
}
```

### Tailwind CSS v3/v4

```tsx
// Mobile-first responsive patterns
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
<h1 className="text-3xl sm:text-4xl lg:text-5xl">
<button className="py-4 px-6 sm:py-6 sm:px-8">
```

### Framer Motion (Animations)

```tsx
// Entrance animations
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.5 }}
>

// Scroll-triggered
<motion.div
  initial={{ opacity: 0 }}
  whileInView={{ opacity: 1 }}
  viewport={{ once: true }}
>
```

---

## Performance Optimization Checklist

Every generated/optimized landing page must:

- [ ] **LCP < 2.5s** — Hero image with `priority`, optimized size
- [ ] **INP < 200ms** — Minimal JS on initial load
- [ ] **CLS < 0.1** — Fixed dimensions on all images
- [ ] **Mobile-first** — Touch targets 44px+, readable text 16px+
- [ ] **Lazy load** — Below-fold images/components
- [ ] **Optimized images** — next/image with `sizes` prop
- [ ] **Minimal CLS** — Skeleton placeholders for async content
- [ ] **Fast TTI** — Critical CSS inline, defer non-critical JS

---

## Trend Research Integration

When `--research` flag is used OR mode is `research`, perform web searches:

```bash
# Hero design trends
WebSearch: "landing page hero section design 2026"
WebSearch: "best landing page headlines 2026"
Reddit: r/SaaS "landing page hero"
X: "#landingpage #webdesign hero"

# Conversion optimization
WebSearch: "landing page conversion rate optimization 2026"
WebSearch: "CTA button best practices 2026"
Reddit: r/marketing "landing page conversion"
X: "#conversion #CRO landing page"

# Social proof trends
WebSearch: "social proof landing page 2026"
WebSearch: "testimonial design best practices"
Reddit: r/startups "social proof"

# Performance
WebSearch: "Core Web Vitals landing page 2026"
WebSearch: "landing page speed optimization"
```

Store findings in session cache, reference in recommendations.

---

## Component Library Reference

### shadcn/ui (Default)

**When to use:** Professional, customizable, production-ready
**Strengths:** Accessible, themeable, copy/paste approach
**Components:** Button, Card, Dialog, Form, Input, Select, Tabs

```tsx
import { Button } from '@/components/ui/button'
import { Card } from '@/components/ui/card'

<Button size="lg" variant="default">Get Started</Button>
<Card className="p-6">...</Card>
```

### Aceternity UI (Wow Factor)

**When to use:** Need stunning animations, hero sections
**Strengths:** 200+ animated components, 3D effects, professional
**Components:** Spotlight, ParallaxScroll, AnimatedGradient, 3DCard, FloatingNav

```tsx
import { Spotlight } from '@/components/ui/spotlight'
import { AnimatedGradient } from '@/components/ui/animated-gradient'

<Spotlight className="absolute" />
<AnimatedGradient />
```

### Magic UI (Playful)

**When to use:** Consumer apps, playful brands
**Strengths:** Interactive elements, engaging micro-interactions
**Components:** ScratchCard, FloatingDock, MorphingButton, ParticleButton

```tsx
import { ScratchCard } from '@/components/magicui/scratch-card'

<ScratchCard revealContent="Secret offer!">Scratch to reveal</ScratchCard>
```

---

## Conversion Formula Reference

### Headline Formulas (Data-Backed)

1. **[Outcome] in [Timeframe] without [Pain Point]**
   - "Ship features 10x faster without hiring more engineers"

2. **Get [Benefit] that [Authority Figure] uses**
   - "Get the scheduling tool that 10,000 agencies trust"

3. **The [Adjective] way to [Outcome]**
   - "The effortless way to manage family tasks"

4. **[Do Thing] like [Aspirational Group]**
   - "Design landing pages like top SaaS companies"

### CTA Copy Patterns

| Generic (Avoid) | Specific (Use) | Why Better |
|----------------|----------------|------------|
| Submit | Start Free Trial | Action-oriented, benefit clear |
| Click Here | Get Your Free Guide | Specific outcome |
| Learn More | See How It Works | Clearer expectation |
| Sign Up | Join 10,000+ Teams | Social proof + action |

---

## Status Report Format

```
═══════════════════════════════════════════════════════════════════
🚀 LANDING PAGE — [MODE]
═══════════════════════════════════════════════════════════════════
[Mode-specific details]

[Results]

───────────────────────────────────────────────────────────────────
Result: [Summary]
Report: .landing-reports/[filename].md
───────────────────────────────────────────────────────────────────
💡 Next: [Suggested action]
═══════════════════════════════════════════════════════════════════
```

---

## Example Invocations

```bash
# Generate SaaS landing page with Aceternity animations
/landing-page generate --template saas --components aceternity --research

# Optimize existing homepage for conversion
/landing-page optimize app/(marketing)/page.tsx

# Audit product landing page vs competitor
/landing-page audit app/product/page.tsx --competitor https://competitor.com

# Research latest hero design trends
/landing-page research hero-design
```

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
