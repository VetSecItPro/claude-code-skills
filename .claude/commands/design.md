---
description: Complete frontend design system — auto-detects what to do. No pages? Generates. Pages exist? Audits & improves. One command, premium results.
## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

allowed-tools: Bash(ls *), Bash(cat *), Bash(head *), Bash(tail *), Bash(wc *), Bash(echo *), Bash(find *), Bash(grep *), Bash(jq *), Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Bash(mkdir *), Bash(date *), Read, Write, Edit, Glob, Grep, Task, WebSearch, mcp__playwright__browser_navigate, mcp__playwright__browser_click, mcp__playwright__browser_resize, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_snapshot, mcp__playwright__browser_evaluate, mcp__playwright__browser_close, mcp__playwright__browser_wait_for, mcp__playwright__browser_tabs
---

# /design — Intelligent Frontend Design System

**One command. It figures out the rest.**

Type `/design` and the skill auto-detects what your project needs:
- **No frontend pages?** → Generates premium pages from scratch
- **Pages exist?** → Audits every screen, fixes safe issues, reports the rest
- **Either path** → Governed by strict anti-slop design engineering standards

Every invocation creates a timestamped report in `.design-reports/`. Every phase is checkpointed. Every decision is documented.

**FIRE AND FORGET** — No permission prompts. No "should I proceed?" Just results.

---

## Usage

```bash
# Smart auto-routing (recommended) — figures out what to do
/design

# Force generation even when pages exist
/design --generate

# Force generation with specific template/style/vibe
/design --generate --template saas --components aceternity
/design --generate --vibe ethereal-glass
/design --generate --vibe editorial-luxury

# Pull latest design trends before running
/design --research

# Resume incomplete run
/design --resume
```

---

## Execution Rules (CRITICAL)

- **NO permission requests** — just scan, generate, fix, and improve
- **NO "should I proceed?" questions** — decide and act
- **Auto-detect EVERYTHING** — stack, pages, what to do, how to fix
- **Always create a report** — `.design-reports/[audit|generate]-TIMESTAMP.md`
- **Update report after EVERY phase** — the file is the single source of truth
- **Fix safe issues immediately** — verify build passes after each fix
- **Mark findings FOUND/FIXING/FIXED/DEFERRED/BLOCKED** — never delete entries
- **SITREP at conclusion** — summary + next action

---

## How It Works — The Pipeline

```
/design
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 0: DISCOVERY                                         │
│  Detect stack, scan for pages/routes, determine path        │
│  Output: Stack profile + routing decision + report file     │
└──────────────────────────┬──────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
   Pages/Routes FOUND          No Pages/Routes Found
              │                         │
              ▼                         ▼
┌──────────────────────┐  ┌──────────────────────────┐
│  PHASE 1: ASSESS     │  │  PHASE 1: PLAN           │
│  14-domain audit     │  │  Template + vibe select   │
│  10-dimension score  │  │  Section planning         │
│  Conversion scoring  │  │  Anti-slop pre-check      │
│  Anti-slop check     │  │  Token system setup       │
│  Viewport matrix     │  │  Component library pick   │
└──────────┬───────────┘  └────────────┬─────────────┘
           │                           │
           ▼                           ▼
┌──────────────────────┐  ┌──────────────────────────┐
│  PHASE 2: IMPLEMENT  │  │  PHASE 2: IMPLEMENT      │
│  Auto-fix safe issues│  │  Generate all sections    │
│  Apply improvements  │  │  Apply design standards   │
│  Smart adaptations   │  │  Wire up components       │
└──────────┬───────────┘  └────────────┬─────────────┘
           │                           │
           └────────────┬──────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: VERIFY                                            │
│  Type-check + build + anti-slop compliance + performance    │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: REPORT                                            │
│  Finalize report + scores + executive summary + next steps  │
└─────────────────────────────────────────────────────────────┘
```

---

# THE DESIGN BRAIN

Everything below governs EVERY phase of EVERY path. Audit findings are scored against these standards. Generated code must comply with them. Improvements are driven by them.

---

## 1. Design Dials (Global Aesthetic Controls)

Three dials control output aesthetic. Defaults below. User can override via chat — adapt dynamically.

```
DESIGN_VARIANCE:  8  (1=Perfect Symmetry ──── 10=Artsy Chaos)
MOTION_INTENSITY: 6  (1=Static/No movement ── 10=Cinematic Physics)
VISUAL_DENSITY:   4  (1=Art Gallery/Airy ──── 10=Pilot Cockpit/Packed)
```

### DESIGN_VARIANCE (1-10)
- **1-3:** Flexbox `justify-center`, strict 12-col symmetrical grids, equal paddings
- **4-7:** `margin-top: -2rem` overlapping, varied aspect ratios (4:3 next to 16:9), left-aligned headers over center-aligned data
- **8-10:** Masonry layouts, CSS Grid fractional units (`2fr 1fr 1fr`), massive empty zones (`padding-left: 20vw`)
- **MOBILE OVERRIDE:** Levels 4-10 MUST fall back to single-column (`w-full`, `px-4`, `py-8`) below 768px

### MOTION_INTENSITY (1-10)
- **1-3:** No auto-animations. CSS `:hover` and `:active` only
- **4-7:** `transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1)`, animation-delay cascades, strictly `transform` + `opacity`
- **8-10:** Scroll-triggered reveals, parallax, Framer Motion hooks, scroll-driven animations. NEVER `window.addEventListener('scroll')`

### VISUAL_DENSITY (1-10)
- **1-3 (Art Gallery):** Generous white space, huge section gaps, expensive/clean feel
- **4-7 (Daily App):** Standard web app spacing
- **8-10 (Cockpit):** Tiny paddings, 1px line separators, everything packed. `font-mono` for all numbers. `font-variant-numeric: tabular-nums`

---

## 2. Vibe Archetypes

Pre-defined aesthetic directions. Select via `--vibe` flag or infer from project context. Each archetype defines typography, color, surfaces, and motion.

### Ethereal Glass (SaaS / AI / Tech)
- **Background:** OLED black `#050505`, radial mesh gradients
- **Surfaces:** Vantablack cards, `backdrop-blur-2xl`, frosted borders `border-white/[0.06]`
- **Inner glow:** `shadow-[inset_0_1px_1px_rgba(255,255,255,0.08)]`
- **Typography:** Geometric Grotesk (`Geist`, `Outfit`), tight tracking, extreme weight contrast (200 vs 800)
- **Accent:** Single high-chroma color (Emerald, Electric Blue) at low opacity for glows
- **Motion:** Perpetual micro-animations, spring physics, mesh gradient background shifts

### Editorial Luxury (Lifestyle / Agency / Creative)
- **Background:** Warm cream `#FDFBF7`, muted sage accents
- **Surfaces:** Paper-textured cards, CSS noise overlay at `opacity-[0.03]` via SVG `feTurbulence`
- **Typography:** Variable serif headlines (`Newsreader`, `Instrument Serif`) + sans-serif body (`Satoshi`)
- **Accent:** Deep warm tones (terracotta, burgundy, forest green)
- **Motion:** Subtle, editorial — text reveals, parallax images, no spring physics

### Soft Structuralism (Consumer / Portfolio / Product)
- **Background:** Silver-grey `#F8F9FA` or warm white
- **Surfaces:** Double-bezel cards (see Component Patterns), ultra-diffuse ambient shadows
- **Typography:** Massive bold Grotesk headlines (`Cabinet Grotesk`, `Plus Jakarta Sans`), generous tracking
- **Accent:** Singular muted accent, desaturated
- **Motion:** Fluid layout transitions, magnetic interactions, staggered reveals

---

## 3. Color System — OKLCH Architecture

Use OKLCH instead of HSL/RGB for perceptually uniform color scales. HSL lies about perceptual brightness — OKLCH doesn't.

### Three-Layer Token Architecture

```css
/* Layer 1 — Global Primitives (derived from OKLCH) */
--color-blue-50: oklch(0.978 0.011 240);
--color-blue-500: oklch(0.648 0.147 240);
--color-blue-950: oklch(0.238 0.054 240);

/* Layer 2 — Semantic Mappings */
--color-interactive: var(--color-blue-500);
--color-surface: var(--color-gray-50);
--color-on-surface: var(--color-gray-900);
--color-error: var(--color-red-500);
--color-success: var(--color-green-500);

/* Layer 3 — Component Tokens */
--button-bg: var(--color-interactive);
--button-text: var(--color-on-surface);
--card-bg: var(--color-surface);
--card-border: var(--color-gray-200);
```

**NEVER use raw hex in components.** Always reference semantic tokens.

### Color Rules
- **Max 1 accent color** per project. Saturation < 80%
- **LILA BAN:** "AI Purple/Blue" aesthetic banned. No purple glows, no neon gradients
- **Neutral bases:** Zinc/Slate + singular high-contrast accent (Emerald, Electric Blue, Deep Rose)
- **NO Pure Black (#000000)** — use `#050505`, `#0a0a0a`, `#121212`, or zinc-950
- **CONSISTENCY:** One gray family (warm OR cool). Never mix within same project
- **Tinted shadows:** Tint shadows to match background hue, not generic black
- **Contrast:** Foreground/background pairs MUST meet 4.5:1 (AA) or 7:1 (AAA)

### Muted Badge/Tag Palette
For status indicators, tags, badges — never use saturated backgrounds:

| Color | Background | Text |
|-------|-----------|------|
| Red | `#FDEBEC` | `#9F2F2D` |
| Blue | `#E1F3FE` | `#1F6C9F` |
| Green | `#EDF3EC` | `#346538` |
| Yellow | `#FBF3DB` | `#956400` |
| Purple | `#F3EDFD` | `#6B3FA0` |

### Programmatic Palette Generation
Store only the hue angle (0-360). Derive everything mathematically:
- Complementary: `hue + 180`
- Triadic: `hue + 120, hue + 240`
- Analogous: `hue ± 30`
- Lightness scale (11 shades): `[97.78, 93.56, 88.11, 82.67, 74.22, 64.78, 57.33, 46.89, 39.44, 32, 23.78]`

---

## 4. Dark Mode Engineering

Dark mode is NOT inverted light mode. It requires deliberate engineering.

### Surface Hierarchy (5 Elevation Levels)

| Level | Use | Light Mode | Dark Mode |
|-------|-----|------------|-----------|
| 0 | Base background | `#FFFFFF` | `#121212` |
| 1 | Cards, sheets | `#F8F9FA` | `#1E1E1E` (overlay white 5%) |
| 2 | Elevated cards, nav | `#F1F3F5` | `#252525` (overlay white 7%) |
| 3 | Popovers, dropdowns | `#E9ECEF` | `#2C2C2C` (overlay white 9%) |
| 4 | Modals, dialogs | `#DEE2E6` | `#333333` (overlay white 11%) |

**Key principle:** In dark mode, elevation = lighter surface (Material Design). Shadows are invisible on dark backgrounds — do NOT rely on them.

### Text Opacity System (Dark Mode)
- **Primary text:** 87% opacity (`text-white/[0.87]`)
- **Secondary text:** 60% opacity (`text-white/60`)
- **Disabled/hint text:** 38% opacity (`text-white/[0.38]`)

### Dark Mode Rules
- Desaturate accent colors slightly — vibrant colors glow harshly on dark backgrounds
- Images need `brightness(0.85)` filter or dark overlay/scrim
- Scrollbar styling: match dark surfaces
- Use `color-scheme: dark` for native form control theming
- CSS `light-dark()` function for inline conditional colors (modern CSS)
- Prevent FOUC: inject theme via cookies/inline script BEFORE first paint, not via `useEffect`
- Test contrast separately — colors that pass in light mode may fail in dark

---

## 5. Typography System

### Scale
- **Display:** `text-5xl md:text-7xl tracking-tighter leading-[0.9] font-bold`
- **H1:** `text-4xl md:text-6xl tracking-tighter leading-none`
- **H2:** `text-3xl md:text-4xl tracking-tight`
- **H3:** `text-2xl md:text-3xl`
- **Body:** `text-base text-gray-600 leading-relaxed max-w-[65ch]`
- **Small/Caption:** `text-sm text-gray-500`
- **Label:** `text-xs uppercase tracking-[0.15em] font-medium`

### Font Pairing Tiers
- **Premium Sans:** `Geist`, `Outfit`, `Cabinet Grotesk`, `Satoshi`, `Plus Jakarta Sans`, `Clash Display`
- **Editorial Serif:** `Newsreader`, `Instrument Serif`, `Playfair Display`
- **Monospace:** `Geist Mono`, `JetBrains Mono`, `SF Mono`
- **BANNED:** Inter, Roboto, Arial, Open Sans, Helvetica, system-ui

### Rules
- **SERIF BAN:** Serif BANNED for Dashboard/Software UIs. Only for editorial/creative
- Fluid typography via `clamp()`: `font-size: clamp(1.25rem, 1rem + 1.5vw, 2.5rem)`
- Use `text-wrap: balance` for headings, `text-wrap: pretty` for body to prevent orphans
- `font-variant-numeric: tabular-nums` for all data/numbers/tables
- Only 4 weights active: Regular (400), Medium (500), SemiBold (600), Bold (700)
- For premium feel: use weight extremes (200 + 800) not safe middle (400 + 600)
- Negative tracking for large headers, positive tracking for small caps/labels
- Mobile body text minimum 16px (avoids iOS auto-zoom)
- Line length: mobile 35-60 chars, desktop 60-75 chars

---

## 6. Spacing & Layout System

### 8pt Grid
All spacing uses multiples of 8: `8, 16, 24, 32, 40, 48, 64, 80, 96, 128`
Tailwind: `p-2, p-4, p-6, p-8, p-10, p-12, p-16, p-20, p-24, p-32`

### Section Spacing
- Between major sections: `py-24` to `py-40` — "the layout breathes heavily"
- Within sections: `space-y-6` to `space-y-8`
- Bottom padding slightly larger than top (optical adjustment)
- Card internal: `p-8` or `p-10`
- Container: `max-w-[1400px] mx-auto` or `max-w-7xl`
- Content width for reading: `max-w-4xl` or `max-w-5xl`

### Layout Rules
- **ANTI-CENTER BIAS:** Centered Hero banned when `DESIGN_VARIANCE > 4`. Force Split Screen, Left/Right Aligned, Asymmetric White-space
- **ANTI-3-COLUMN:** Generic "3 equal cards horizontally" banned. Use 2-column Zig-Zag, asymmetric grid, or horizontal scroll
- **ANTI-CARD OVERUSE:** For `VISUAL_DENSITY > 7`, cards banned. Use `border-t`, `divide-y`, negative space
- **Grid over Flex-Math:** NEVER `w-[calc(33%-1rem)]`. ALWAYS CSS Grid
- **Viewport Stability:** NEVER `h-screen`. ALWAYS `min-h-[100dvh]`

---

## 7. Architecture Conventions

- **DEPENDENCY VERIFICATION [MANDATORY]:** Before importing ANY library, check `package.json`. If missing, output install command. Never assume.
- **Framework:** React/Next.js default. Server Components (RSC) default.
  - **RSC SAFETY:** Global state in Client Components only. Wrap providers in `"use client"`.
  - **INTERACTIVITY ISOLATION:** Interactive components extracted as `'use client'` leaf components.
- **Styling:** Tailwind CSS (v3/v4) for 90%.
  - **VERSION LOCK:** Check `package.json`. Don't use v4 syntax in v3 projects.
  - **T4 CONFIG GUARD:** v4 uses `@tailwindcss/postcss`, not `tailwindcss` plugin.
- **ANTI-EMOJI POLICY:** NEVER use emojis in generated code, markup, or text content. Use Phosphor/Radix icons or SVG.
- **Icons:** `@phosphor-icons/react` or `@radix-ui/react-icons`. Consistent `strokeWidth` (1.5 or 2.0).

---

## 8. Interactive State Rules

**ALL interactive elements MUST have 5 states:**

| State | Implementation | Required |
|-------|---------------|----------|
| Default | Resting appearance | Always |
| Hover | `hover:` classes (desktop) | Always |
| Active/Pressed | `active:scale-[0.98]` or `active:-translate-y-px` | Always |
| Focus | `focus-visible:ring-2 focus-visible:ring-offset-2` | Always |
| Disabled | `disabled:opacity-50 disabled:cursor-not-allowed` | When applicable |

**Also mandatory:**
- **Loading:** Skeletal loaders matching layout shape (no generic spinners). Show skeleton after 300ms delay
- **Empty states:** Illustration + helpful message + primary action button
- **Error states:** Inline, human-friendly, with specific next steps. NEVER "Something went wrong"
- **Success feedback:** Brief visual (checkmark animation, color flash, or toast). NEVER silent completion

---

## 9. Form UX Patterns

### Layout Rules
- Label MUST sit above input, never placeholder-only
- Error text below input. Helper text optional
- Standard `gap-2` for input blocks
- Single-column forms (120% fewer errors than multi-column)
- Mobile input height >= 44px

### Validation
- Validate on blur (not keystroke)
- After submit error, auto-focus first invalid field
- Error message states cause + how to fix (not just "Invalid input")
- For multiple errors: summary at top with anchor links to each field
- Use `aria-live` regions or `role="alert"` for screen reader announcements

### Input Optimization
- Semantic `inputmode` on every input: `numeric`, `email`, `tel`, `url`, `search`
- `autocomplete` attributes for autofill
- Password show/hide toggle mandatory
- Confirm before dismissing unsaved changes

### Conversion Stats (Data-Backed)
- 4 fields vs 11 fields = 120% more conversions
- Phone number field = 58% abandon rate increase
- Multi-step with progress > single long form

### Multi-Step Forms
- Progress indicator mandatory
- Allow back navigation
- Auto-save drafts for forms > 5 fields
- Confirm destructive navigation

---

## 10. Toast & Notification System

### Toast Timing Formula
**Duration = 500ms × word count + 3,000ms base. Minimum 6 seconds for accessibility.**

### Toast Rules
- Auto-dismiss in calculated duration
- Must NOT steal focus
- Use `aria-live="polite"` for screen reader
- Provide "Undo" toast for destructive actions
- No "Oops!" — be specific and helpful

### Notification Hierarchy

| Type | Use | Behavior |
|------|-----|----------|
| Inline | Field-level validation, contextual | Persistent until resolved |
| Toast | System-generated, non-critical | Auto-dismiss |
| Banner | System-wide status | Persistent, dismissable |
| Modal | Blocking, critical decisions | Requires action |

---

## 11. Data Visualization Rules

### Chart Selection
- **Trend over time:** Line chart
- **Comparison:** Bar chart (horizontal if >5 categories)
- **Proportion:** Donut chart (NO pie chart for >5 categories)
- **Relationship:** Scatter plot
- **Part-to-whole:** Stacked bar or treemap

### Design Rules
- Grid lines: low-contrast (`gray-200`) — don't compete with data
- Direct labeling for small datasets (reduce eye travel)
- Legends clickable to toggle series visibility
- Time series: clearly label granularity (day/week/month) with switching
- Large datasets (1000+): aggregate/sample with drill-down
- Locale-aware formatting for numbers, dates, currencies
- Skeleton shimmer placeholder while loading

### Accessibility
- All chart colors: 3:1 contrast against adjacent colors
- Use patterns/textures in addition to color (colorblind-safe)
- Provide table alternative for screen readers
- Interactive elements: 44px+ tap area
- Text summary via `aria-label` describing key insight
- Entrance animations respect `prefers-reduced-motion`
- Provide CSV/image export for data-heavy products

---

## 12. UX Writing & Microcopy

### Quality Standards
Every piece of UI text must be: **Purposeful → Concise → Conversational → Clear**

### Word Count Benchmarks
- 8 words = 100% comprehension
- 14 words = 90% comprehension
- 25+ words = significant drop

### Character Limits by Element
- Button labels: 2-4 words
- Page titles: 3-6 words
- Notifications: 10-15 words
- Error messages: 15-25 words
- Tooltips: max 2 sentences

### Error Message Taxonomy

| Type | Pattern | Placement |
|------|---------|-----------|
| Validation | Inline beneath field | Persistent |
| System | Banner or modal | Persistent until resolved |
| Blocking | Full-screen with recovery | Blocks interaction |
| Permission | In-context explanation | Dismissable |

### Tone Adaptation by User State

| State | Voice Adjustment |
|-------|-----------------|
| Frustrated | Calm, direct, no humor, show path forward |
| Confused | Simple words, shorter sentences, examples |
| Confident | Concise, skip basics, show advanced options |
| Cautious | Reassuring, explain consequences, offer undo |
| Successful | Brief celebration, suggest next action |

### Content Anti-Patterns
- **BANNED verbs:** "Elevate", "Seamless", "Unleash", "Next-Gen", "Delve", "Leverage", "Synergy"
- **BANNED error text:** "Oops!", "Something went wrong", "Please try again later"
- **BANNED placeholders:** "Enter text here", "Type something"
- **BANNED button text:** "Submit", "OK", "Click Here", "Send", "Go"
- Use sentence case, not Title Case On Every Header
- No exclamation marks in success messages
- No passive voice in UI copy
- Front-load key information for scannability

---

## 13. Conversion Psychology

### Behavioral Models (Apply During Generation & Optimization)

| Model | Application |
|-------|------------|
| **Anchoring Effect** | Show original price crossed out next to discounted price |
| **Decoy Effect** | Add a 3rd pricing tier that makes the target tier look best |
| **Loss Aversion** | Frame as "Don't miss out" not "Get this" |
| **Social Proof** | Place testimonials/logos near CTAs, not isolated sections |
| **Goal-Gradient Effect** | Show progress bars — people accelerate near completion |
| **Peak-End Rule** | Make the last interaction memorable (success celebration) |
| **Zeigarnik Effect** | Incomplete progress indicators drive completion |
| **Zero-Price Effect** | "Free" is disproportionately powerful — use in CTAs |
| **IKEA Effect** | Let users customize/personalize early — increases perceived value |
| **Endowment Effect** | Free trials create ownership feeling before purchase |

### BJ Fogg Behavior Model
**Behavior = Motivation × Ability × Prompt**
- Reduce friction (Ability): fewer form fields, simpler flows
- Increase motivation: social proof, scarcity, authority
- Time prompts correctly: CTAs when motivation peaks (after value demonstration)

### EAST Framework
- **Easy:** Default to the desired action, reduce steps
- **Attractive:** Visual hierarchy draws eye to CTA, contrasting colors
- **Social:** Show what others do ("10,000 teams use this")
- **Timely:** Prompt at the moment of highest receptivity

### Pricing Page Psychology
- 3-4 tiers maximum. Highlight recommended tier
- Good-Better-Best naming (not Tier 1/2/3)
- Charm pricing: $29 not $30
- Annual pricing shown as monthly equivalent
- Feature matrix: checkmarks, not text descriptions
- Most popular badge on target tier

### Headline Formulas
1. **[Outcome] in [Timeframe] without [Pain Point]**
2. **Get [Benefit] that [Authority] uses**
3. **The [Adjective] way to [Outcome]**
4. **[Do Thing] like [Aspirational Group]**

### CTA Patterns
| Avoid | Use | Why |
|-------|-----|-----|
| Submit | Start Free Trial | Action + benefit |
| Click Here | Get Your Free Guide | Specific outcome |
| Learn More | See How It Works | Clear expectation |
| Sign Up | Join 10,000+ Teams | Social proof + action |

---

## 14. Forbidden Patterns — AI Tells (Comprehensive)

These make output look AI-generated. STRICTLY avoid unless explicitly requested.

### Visual & CSS
- NO neon/outer glows — use inner borders or tinted shadows
- NO pure black `#000000` — use off-black
- NO oversaturated accents — desaturate to blend with neutrals
- NO excessive gradient text on large headers
- NO custom mouse cursors — kills perf/a11y
- NO same shadow on every element — vary by elevation

### Typography
- NO Inter/Roboto/Arial/Open Sans — use premium fonts
- NO oversized H1s — control with weight + color, not just scale
- NO serif on dashboards
- NO Title Case On Every Header — use sentence case

### Layout
- NO 3-column equal card rows — use asymmetric grids
- NO centered hero (when DESIGN_VARIANCE > 4)
- NO generic cards in dashboards (VISUAL_DENSITY > 7)
- NO 3-card carousel testimonials with dot navigation
- NO accordion FAQ sections — try searchable help or side-by-side list
- NO footer link farm (4+ columns of links)
- NO sun/moon dark mode toggle icon — use a more creative approach
- NO pill-shaped "New"/"Beta" badges — try square badges or plain text

### Content (The "Jane Doe" Effect)
- NO generic names ("John Doe", "Sarah Chan", "Jane Smith")
- NO generic avatars (SVG eggs, Lucide user icons) — use photo placeholders or styled initials
- NO round numbers (`99.99%`, `50%`, `1234567`) — use organic data (`47.2%`, `+1 (312) 847-1928`)
- NO startup slop names ("Acme", "Nexus", "SmartFlow", "Flowspace")
- NO Lorem ipsum — write contextual placeholder copy
- NO Unsplash links — use `picsum.photos/seed/{descriptive}/800/600` or SVG
- NO default shadcn/ui — MUST customize radii, colors, shadows
- NO emojis in code, markup, or content
- NO exclamation marks in success messages
- NO "Oops!" error messages
- NO avatar circles exclusively — try squircles or rounded squares

---

## 15. Creative Techniques

### Component Patterns

**Double-Bezel (Nested Physical Hardware):**
- Outer Shell: `bg-black/5 ring-1 ring-black/5 p-1.5 rounded-[2rem]`
- Inner Core: own background + `shadow-[inset_0_1px_1px_rgba(255,255,255,0.15)] rounded-[calc(2rem-0.375rem)]`
- Simulates machined hardware — premium tactile feel

**Liquid Glass Refraction:**
- Beyond `backdrop-blur`: add `border-white/10` + `shadow-[inset_0_1px_0_rgba(255,255,255,0.1)]`

**Button-in-Button Trailing Icon:**
- Arrow in circular wrapper (`w-8 h-8 rounded-full bg-black/5`) flush with padding
- On hover: icon translates diagonally + slight scale

**Eyebrow Tags:**
- `rounded-full px-3 py-1 text-[10px] uppercase tracking-[0.2em] font-medium`

**Noise Texture Overlay:**
```css
.noise::after {
  content: '';
  position: fixed; inset: 0; z-index: 50;
  pointer-events: none; opacity: 0.03;
  background: url("data:image/svg+xml,...feTurbulence...");
}
```

### Motion Patterns

**Magnetic Micro-physics (MOTION > 5):** Buttons pull toward cursor. Use Framer Motion `useMotionValue`/`useTransform`, NEVER React `useState`

**Perpetual Micro-Interactions (MOTION > 5):** Infinite micro-animations (Pulse, Typewriter, Float, Shimmer). Spring: `stiffness: 100, damping: 20`. MUST be `React.memo` + isolated Client Component

**Staggered Orchestration:** `staggerChildren: 0.05` (50-80ms, max 5-8 elements). Parent + children MUST share same Client Component tree

**Motion Benchmarks:**
- Entrance: 150-300ms, `ease-out`
- Exit: 100-200ms, `ease-in` (60-70% of enter duration)
- State change: 200-300ms, `ease-in-out`
- Spring: stiffness 200-400, damping 20-30
- Custom bezier: `cubic-bezier(0.32, 0.72, 0, 1)`

### Creative Arsenal

**Navigation:** Dock Magnification | Magnetic Button | Gooey Menu | Dynamic Island | Radial Menu | Speed Dial | Mega Menu | Floating pill navbar (`mt-6 mx-auto w-max rounded-full`)
**Layout:** Bento Grid | Masonry | Chroma Grid | Split Screen Scroll | Curtain Reveal
**Cards:** Parallax Tilt | Spotlight Border | Glassmorphism | Holographic Foil | Morphing Modal
**Scroll:** Sticky Stack | Horizontal Hijack | Locomotive Sequence | Zoom Parallax | Progress Path | Liquid Swipe
**Gallery:** Dome | Coverflow Carousel | Drag-to-Pan | Accordion Slider | Hover Image Trail | Glitch Effect
**Typography:** Kinetic Marquee | Text Mask Reveal | Scramble Effect | Circular Path | Gradient Stroke
**Micro:** Particle Explosion Button | Skeleton Shimmer | Directional Hover | Ripple Click | Mesh Gradient Background

---

## 16. Modern CSS Features (2025-2026)

Use these where supported. Check browser compatibility before shipping.

### Container Queries
Replace viewport media queries for reusable components:
```css
.parent { container-type: inline-size; }
@container (min-width: 400px) { .child { /* adapt */ } }
```
Tailwind v4: native `@container` support, no plugin needed.

### Scroll-Driven Animations
Replace 45KB JavaScript scroll libraries entirely:
```css
.element {
  animation: fade-in linear;
  animation-timeline: view();  /* animate based on visibility */
  animation-range: entry 0% entry 100%;
}
```

### View Transitions API
Smooth page transitions for SPAs:
```css
::view-transition-old(root) { animation: slide-out 200ms ease-in; }
::view-transition-new(root) { animation: slide-in 200ms ease-out; }
```

### `@starting-style` (Entry Animations)
Animate elements from initial state on DOM insertion — no JavaScript needed:
```css
dialog[open] {
  opacity: 1; transform: scale(1);
  @starting-style { opacity: 0; transform: scale(0.95); }
  transition: opacity 200ms, transform 200ms;
}
```

### `interpolate-size: allow-keywords`
Animate to/from `auto` height — no more `max-height` hacks:
```css
:root { interpolate-size: allow-keywords; }
.panel { height: 0; transition: height 300ms; }
.panel.open { height: auto; }
```

---

## 17. Motion-Engine Bento Paradigm

For SaaS dashboards and feature sections — "Bento 2.0":

- **Palette:** Background `#f9fafb`, cards pure white + 1px `border-slate-200/50`
- **Surfaces:** `rounded-[2.5rem]`, diffusion shadow `shadow-[0_20px_40px_-15px_rgba(0,0,0,0.05)]`
- **Typography:** `Geist`/`Satoshi`/`Cabinet Grotesk`, `tracking-tight` headers
- **Labels:** Outside and below cards (gallery-style)
- **Spring Physics:** `stiffness: 100, damping: 20` (no linear easing)
- **Perpetual Motion:** Every card has infinite "alive" loop — memoized + isolated Client Component

**5-Card Archetypes:**
1. **Intelligent List** — auto-sorting `layoutId` swaps
2. **Command Input** — typewriter + blinking cursor + processing shimmer
3. **Live Status** — breathing indicators + overshoot spring badge
4. **Wide Data Stream** — infinite horizontal carousel
5. **Contextual UI** — staggered highlight + float-in toolbar

---

## 18. Performance Guardrails

- Animate ONLY `transform` and `opacity` — never `top`, `left`, `width`, `height`
- Grain/noise filters ONLY on fixed `pointer-events-none` elements
- `backdrop-blur` only on fixed/sticky elements, never scrolling containers
- Z-index for systemic layers only: `0 / 10 / 20 / 40 / 100 / 1000`
- NEVER `window.addEventListener('scroll')` — use IntersectionObserver, `animation-timeline: scroll()`, or Framer Motion `whileInView`
- **LCP < 2.5s** | **INP < 200ms** | **CLS < 0.1**
- Hero images: `priority` + `sizes` prop
- Below-fold: `loading="lazy"` + skeleton placeholders
- WebP/AVIF with fallbacks. Declare `width`/`height` or `aspect-ratio` on all images
- `touch-action: manipulation` to eliminate 300ms tap delay
- `-webkit-tap-highlight-color: transparent` for native feel
- `select-none` on buttons, nav items, labels (not content text)

---

## 19. Native CSS Replacements (Zero-JS Patterns)

### Anchor Positioning + Popover API (Replace JS Tooltip Libraries)
```css
/* Anchor element */
.trigger { anchor-name: --my-anchor; }

/* Pure CSS positioned tooltip */
.tooltip {
  position: fixed;
  position-anchor: --my-anchor;
  top: anchor(bottom);
  left: anchor(center);
  margin-top: 8px;
  position-try-fallbacks: flip-block, flip-inline;
}
```
```html
<!-- Combined with Popover API — zero JS show/hide -->
<button popovertarget="tip" style="anchor-name: --btn">Hover</button>
<div id="tip" popover="hint" style="position-anchor: --btn">Tooltip text</div>
```

**Decision tree:**
| Need | Use |
|------|-----|
| Tooltip/popover | Anchor Positioning + `popover="hint"` (zero JS) |
| Dropdown menu | Anchor Positioning + `popover="auto"` (auto light-dismiss) |
| Complex multi-step | Radix/headless UI (still needed) |

### CSS `@property` (Animatable Custom Properties)
```css
@property --gradient-angle {
  syntax: "<angle>";
  initial-value: 0deg;
  inherits: false;
}
/* Now animatable — impossible without registration */
.conic-loader {
  background: conic-gradient(from var(--gradient-angle), #6366f1, transparent);
  animation: spin 2s linear infinite;
}
@keyframes spin { to { --gradient-angle: 360deg; } }

/* Animatable color lightness */
@property --accent-l {
  syntax: "<number>";
  initial-value: 0.65;
  inherits: true;
}
.button:hover { --accent-l: 0.55; transition: --accent-l 200ms; }
.button { background: oklch(var(--accent-l) 0.15 250); }
```

**Rule:** Register any custom property you want to transition. Unregistered CSS variables are strings — browsers cannot interpolate them.

### Relative Color Syntax (Replace JS Color Libraries)
```css
/* Hover — darken by reducing lightness */
.button:hover { background: oklch(from var(--bg) calc(l - 0.1) c h); }

/* Disabled — zero chroma for grayscale */
.button:disabled { background: oklch(from var(--bg) l 0 h); }

/* Complementary color — rotate hue 180deg */
.complement { color: oklch(from var(--accent) l c calc(h + 180)); }

/* Semi-transparent version */
.overlay { background: oklch(from var(--bg) l c h / 0.5); }

/* Tinted shadow matching element color */
.card { box-shadow: 0 8px 24px oklch(from var(--bg) calc(l - 0.3) c h / 0.2); }
```

**Rule:** Use relative color syntax for hover/active/disabled/focus variants instead of maintaining separate tokens per state.

### `content-visibility: auto` (Rendering Performance)
```css
.section {
  content-visibility: auto;
  contain-intrinsic-size: auto 500px;
}
```
- Use on pages with 5+ sections below fold, long lists, dashboard tabs
- **NEVER** on hero/above-fold content
- Measured: **7x rendering speedup** on initial load for content-heavy pages

### CSS `if()` Function (Chrome 137+)
```css
/* Touch vs pointer sizing */
button { width: if(media(any-pointer: fine): 30px; else: 44px); }

/* Status-driven styling from data attributes */
.card { border-color: if(
  style(--status: pending): oklch(0.6 0.15 260);
  style(--status: complete): oklch(0.6 0.15 145);
  else: oklch(0.7 0 0)
); }
```

---

## 20. Animation Decision Tree

| Need | Use | Bundle Cost |
|------|-----|-------------|
| Hover/focus/active states | CSS transitions | 0 KB |
| Scroll-linked animation | `animation-timeline: scroll()` CSS | 0 KB |
| Entry animation on mount | `@starting-style` CSS | 0 KB |
| Simple entrance/exit | WAAPI (`el.animate()`) | 0 KB |
| Height to `auto` | `interpolate-size: allow-keywords` | 0 KB |
| Tooltips/popovers | CSS Anchor + Popover API | 0 KB |
| Stagger, layout, gestures | Framer Motion | ~32 KB |
| Complex timeline sequencing | GSAP | ~25 KB |
| Page transitions | View Transitions API | 0 KB |
| 3D/Canvas | ThreeJS (isolated) | ~150 KB |

**WAAPI for zero-dependency animations:**
```typescript
function animateEntrance(el: HTMLElement) {
  el.animate(
    [
      { opacity: 0, transform: 'translateY(12px)' },
      { opacity: 1, transform: 'translateY(0)' },
    ],
    { duration: 300, easing: 'cubic-bezier(0.32, 0.72, 0, 1)', fill: 'forwards' }
  );
}

function springScale(el: HTMLElement) {
  el.animate(
    [
      { transform: 'scale(0.95)' },
      { transform: 'scale(1.02)', offset: 0.6 },
      { transform: 'scale(1)' },
    ],
    { duration: 400, easing: 'ease-out', fill: 'forwards' }
  );
}
```

**Rule:** Default to zero-JS CSS techniques. Only pull in Framer Motion when you need layout animations, gestures, or complex orchestration. NEVER mix GSAP/ThreeJS with Framer Motion in the same component tree.

---

## 21. Responsive Images — Art Direction

### Decision Tree

| Image Type | Format | Quality | Attributes |
|-----------|--------|---------|------------|
| Hero/LCP | AVIF+WebP+JPEG | 80-85 | `fetchpriority="high"`, NO `loading="lazy"` |
| Content photos | AVIF+WebP | 75-80 | `loading="lazy" decoding="async"` |
| Thumbnails | WebP | 60-70 | Small srcset (200w, 400w) |
| Icons/logos | SVG | n/a | Inline if < 2KB, external if > 2KB |
| Decorative | CSS gradient/SVG | n/a | Never raster for decorative |

### Full Art Direction Pattern
```html
<picture>
  <source type="image/avif"
    srcset="hero-400.avif 400w, hero-800.avif 800w, hero-1600.avif 1600w"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 800px" />
  <source type="image/webp"
    srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1600.webp 1600w"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 800px" />
  <img src="hero-800.jpg"
    srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1600.jpg 1600w"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 800px"
    alt="Descriptive alt" width="1600" height="900"
    loading="lazy" decoding="async" fetchpriority="low" />
</picture>
```

**`sizes` formula:** Match CSS layout. `w-full md:w-1/2 lg:w-1/3` → `sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"`

**Dark mode images:**
```html
<source media="(prefers-color-scheme: dark)" srcset="hero-dark.avif" type="image/avif" />
```

---

## 22. Skeleton & Shimmer CSS

### Synchronized Shimmer (All Skeletons Animate Together)
```css
.skeleton {
  background: linear-gradient(90deg,
    oklch(0.92 0 0) 0%, oklch(0.96 0 0) 40%, oklch(0.92 0 0) 80%);
  background-size: 200% 100%;
  background-attachment: fixed; /* KEY: syncs all skeletons */
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: 4px;
}
@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Dark mode */
.dark .skeleton {
  background: linear-gradient(90deg,
    oklch(0.25 0 0) 0%, oklch(0.30 0 0) 40%, oklch(0.25 0 0) 80%);
  background-size: 200% 100%;
  background-attachment: fixed;
  animation: shimmer 1.5s ease-in-out infinite;
}
```

### Dimension-Matched Skeleton Components
```css
.skeleton-text   { height: 1em; width: 80%; }
.skeleton-title  { height: 1.5em; width: 60%; }
.skeleton-avatar { width: 40px; height: 40px; border-radius: 50%; }
.skeleton-card   { height: 200px; width: 100%; }
.skeleton-button { height: 40px; width: 120px; border-radius: 8px; }
```

**Rules:**
- Shapes MUST match dimensions of actual content
- Show skeleton after **300ms delay** (avoid flash on fast loads)
- `background-attachment: fixed` so all shimmer moves in unison
- Each `<Suspense>` fallback = dimension-matched skeleton

---

## 23. i18n / RTL / Logical Properties

### CSS Logical Properties (Use EVERYWHERE)

| Physical (BANNED) | Logical (USE) | Tailwind |
|-------------------|---------------|----------|
| `margin-left` | `margin-inline-start` | `ms-4` |
| `margin-right` | `margin-inline-end` | `me-4` |
| `padding-left` | `padding-inline-start` | `ps-4` |
| `padding-right` | `padding-inline-end` | `pe-4` |
| `text-align: left` | `text-align: start` | `text-start` |
| `text-align: right` | `text-align: end` | `text-end` |
| `border-left` | `border-inline-start` | `border-s` |
| `left: 0` | `inset-inline-start: 0` | `start-0` |
| `right: 0` | `inset-inline-end: 0` | `end-0` |
| `float: left` | `float: inline-start` | — |

### RTL Rules
- Set `dir="rtl"` on `<html>` — flexbox and grid auto-reverse
- Mirror directional icons (arrows, chevrons): `[dir="rtl"] .icon { transform: scaleX(-1); }`
- Do NOT mirror: media controls, clocks, checkmarks, phone icons
- Use `<bdi>` for user-generated content with unknown direction
- Numbers always LTR even in RTL: `direction: ltr; unicode-bidi: isolate;`
- Text expansion room: German +30%, CJK needs more height — no fixed-width text containers

---

## 24. SEO / Structured Data / OG Tags

### Every Generated Page Must Have:
- `<title>` — 50-60 chars, keyword + brand
- `meta description` — 150-155 chars, includes CTA
- OG image — 1200x630px, < 1MB, JPEG/PNG
- `canonical` URL
- `lang` attribute on `<html>`
- Structured data matching page type

### Next.js Metadata Pattern
```tsx
export const metadata: Metadata = {
  title: 'Product Name — Solve X Problem',
  description: 'Y benefit in Z timeframe. Join N users.',
  openGraph: {
    title: 'Product Name — Solve X Problem',
    description: 'Y benefit in Z timeframe',
    type: 'website',
    images: [{ url: '/og-image.png', width: 1200, height: 630, alt: 'Product' }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Product Name — Solve X Problem',
    description: 'Y benefit in Z timeframe',
    images: ['/og-image.png'],
  },
};
```

### JSON-LD Structured Data
```tsx
<script type="application/ld+json" dangerouslySetInnerHTML={{ __html: JSON.stringify({
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "Product",
  "applicationCategory": "BusinessApplication",
  "offers": { "@type": "Offer", "price": "29", "priceCurrency": "USD" },
  "aggregateRating": { "@type": "AggregateRating", "ratingValue": "4.8", "reviewCount": "1247" }
})}} />
```

| Page Type | Schema Type |
|-----------|-------------|
| SaaS/App | `SoftwareApplication` |
| Product | `Product` |
| Blog/Article | `Article` |
| FAQ section | `FAQPage` |
| Pricing page | `Offer` / `PriceSpecification` |
| Person/About | `Person` or `Organization` |

---

## 25. Fluid Spacing (Beyond Typography)

### Fluid Scale — Matches 8pt Grid at Both Ends
```css
:root {
  --space-xs:      clamp(0.25rem, 0.2rem + 0.25vw, 0.5rem);   /* 4→8 */
  --space-sm:      clamp(0.5rem, 0.4rem + 0.5vw, 1rem);        /* 8→16 */
  --space-md:      clamp(1rem, 0.8rem + 1vw, 2rem);             /* 16→32 */
  --space-lg:      clamp(1.5rem, 1rem + 2.5vw, 4rem);           /* 24→64 */
  --space-xl:      clamp(2rem, 1rem + 5vw, 8rem);               /* 32→128 */
  --space-section: clamp(4rem, 2rem + 10vw, 10rem);             /* 64→160 */
  --radius-card:   clamp(0.75rem, 0.5rem + 1vw, 1.5rem);       /* 12→24 */
  --container-pad: clamp(1rem, 0.5rem + 2.5vw, 3rem);          /* 16→48 */
}
```

**Container query units for component-scoped fluid sizing:**
```css
.parent { container-type: inline-size; }
.child { padding: clamp(0.5rem, 2cqw, 2rem); }
```

**Formula:** `clamp(min, preferred, max)` where preferred = `min + (max - min) × vw-factor`. Use Utopia (utopia.fyi) to generate scales.

---

## 26. Component API Patterns

### Decision Tree

| Need | Pattern |
|------|---------|
| Multi-part UI (tabs, select, dialog) | Compound components |
| Customizable regions | Slot pattern (named children) |
| Different HTML elements | Polymorphic `asChild` prop |
| Behavior without markup | Headless/hook pattern |
| Simple variants | Props + `cva()` |

**Rule:** Max 7 props per component. More than 7 → switch to compound components or slots.

### Compound Components
```tsx
<Select>
  <Select.Trigger>Choose option</Select.Trigger>
  <Select.Content>
    <Select.Item value="a">Option A</Select.Item>
    <Select.Item value="b">Option B</Select.Item>
  </Select.Content>
</Select>
```

### CVA (Class Variance Authority) for Tailwind Variants
```tsx
import { cva } from 'class-variance-authority';

const button = cva('inline-flex items-center justify-center rounded-lg font-medium transition-colors', {
  variants: {
    intent: {
      primary: 'bg-zinc-900 text-white hover:bg-zinc-800',
      secondary: 'bg-zinc-100 text-zinc-900 hover:bg-zinc-200',
      danger: 'bg-red-600 text-white hover:bg-red-700',
    },
    size: {
      sm: 'h-8 px-3 text-sm',
      md: 'h-10 px-4 text-sm',
      lg: 'h-12 px-6 text-base',
    },
  },
  defaultVariants: { intent: 'primary', size: 'md' },
});
```

---

## 27. Optimistic UI & Slow Network Patterns

### React 19 useOptimistic
```tsx
const [optimisticItems, setOptimistic] = useOptimistic(
  items,
  (current, newItem) => [...current, { ...newItem, pending: true }]
);

async function handleAdd(formData) {
  const item = { id: crypto.randomUUID(), text: formData.get('text') };
  setOptimistic(item);   // Instant UI update
  await addItem(item);   // Server sync in background
}

// Render with pending indicator
{optimisticItems.map(item => (
  <li style={{ opacity: item.pending ? 0.6 : 1 }}>{item.text}</li>
))}
```

### Slow Network Rules
- Show skeleton after **300ms delay** (avoid flash on fast loads)
- Optimistic UI for ALL write operations (add, edit, delete)
- Stale-while-revalidate: show cached data, refresh silently
- Progressive image: LQIP (low-quality placeholder) → full resolution
- Offline indicator: subtle top banner, NOT blocking modal
- Queue failed mutations: retry with exponential backoff
- Never show spinner > 5 seconds — switch to progress bar with cancel

---

## 28. WCAG 2.2 New Criteria

### Focus Not Obscured (2.4.11 AA)
```css
/* Sticky headers must not cover focused elements */
html { scroll-padding-top: 80px; /* header height + 16px buffer */ }
```

### Dragging Movements (2.5.7 AA)
- Every drag-to-reorder MUST have button alternative (move up/down)
- Every slider MUST accept direct text input
- No functionality exclusively behind drag gestures

### Target Size (2.5.8 AA)
- Minimum **24x24px** for ALL interactive targets (desktop)
- **44x44px** remains recommendation for touch
- Inline text links exempt if paragraph text

### Consistent Help (3.2.6 A)
- Help/support link must appear in same relative location across all pages

---

## 29. PWA Design Patterns

### Manifest Essentials
```json
{
  "name": "App Name",
  "short_name": "App",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#121212",
  "theme_color": "#121212",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icon-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

### PWA Design Rules
- `display: standalone` = no browser chrome. App MUST provide own back button
- Custom install prompt: intercept `beforeinstallprompt`, show after user engagement (not first visit)
- Update notification: detect SW update, show "New version available" banner with refresh
- Offline page: cache branded fallback, not browser default
- `theme_color` MUST match app header/status bar color
- Maskable icon: safe zone = center 80% circle — keep logo within
- Splash screen: `background_color` + icon from manifest — must match app theme

---

## 30. Server Component Streaming Patterns

### Donut Pattern (Server Shell, Client Leaves)
```tsx
// layout.tsx — Server Component (zero JS)
export default function Layout({ children }) {
  return (
    <div className="grid grid-cols-[240px_1fr]">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

### Independent Suspense Boundaries
```tsx
// page.tsx — each data dep gets own boundary
export default function Dashboard() {
  return (
    <div className="grid grid-cols-2 gap-6">
      <Suspense fallback={<MetricsSkeleton />}>
        <Metrics />
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <Chart />
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <RecentActivity />
      </Suspense>
    </div>
  );
}
```

### RSC Streaming Rules
- Push `"use client"` to leaf components only — never on layouts or pages
- Each `<Suspense>` fallback = dimension-matched skeleton
- Sibling server components fetch data in parallel automatically
- Never nest Suspense more granularly than the user-visible loading sequence
- PPR (Partial Prerendering): static shell from edge, dynamic parts stream in

---

## 31. Tailwind v4 Specific Syntax

### CSS-First Configuration
```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.65 0.15 250);
  --color-surface: oklch(0.98 0.005 250);
  --font-display: "Cabinet Grotesk", sans-serif;
  --radius-card: 1.25rem;
  --ease-spring: cubic-bezier(0.32, 0.72, 0, 1);
}
/* Tokens auto-expose as utilities: bg-brand, text-brand, font-display, rounded-card */
```

### v3 → v4 Migration

| v3 | v4 |
|----|-----|
| `tailwind.config.js` | `@theme {}` in CSS |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `flex-shrink-0` | `shrink-0` |
| `flex-grow` | `grow` |
| `content: ['./src/**']` | Auto-detected (zero config) |
| `@apply` | Discouraged — use CSS directly |
| Container query plugin | Native `@container` |
| `ring-` utilities | Unchanged but ring is now `outline` under the hood |

---

# PHASE 0: DISCOVERY

The entry point. Determines everything that happens next.

---

## 0.1 Detect Project Stack

```bash
# Package manager
if [ -f "bun.lockb" ]; then PM="bun"
elif [ -f "pnpm-lock.yaml" ]; then PM="pnpm"
elif [ -f "yarn.lock" ]; then PM="yarn"
else PM="npm"; fi
```

### Detection Matrix

| Category | Check | Result |
|----------|-------|--------|
| Framework | `next.config.*` / `vite.config.*` / `astro.config.*` / etc. | Next.js / Vite / Astro / etc. |
| CSS | `tailwind.config.*` / `*.module.css` / styled-components | Tailwind v3/v4 / Modules / etc. |
| Components | `components/ui/` / `@mui/*` / `@chakra-ui/*` | shadcn / MUI / Chakra / custom |
| Animation | `framer-motion` / `gsap` / CSS only | Framer Motion / GSAP / CSS |
| Theme | `dark:` classes / `prefers-color-scheme` | Dark support yes/no |
| Images | `next/image` / raw `<img>` | Optimized / raw |
| Icons | Lucide / Heroicons / Phosphor | Detected library |
| Breakpoints | Custom screens config or defaults | sm/md/lg/xl/2xl |

## 0.2 Scan for Pages/Routes

Framework-specific route discovery. Count pages and components.

## 0.3 Route Decision

```
IF --generate flag → GENERATION PATH
ELSE IF page count == 0 → GENERATION PATH
ELSE → AUDIT PATH
```

## 0.4 Create Report File

Every invocation creates a report:
```bash
mkdir -p .design-reports
TIMESTAMP=$(date +"%Y%m%d-%H%M%S")
REPORT_FILE=".design-reports/${PATH_TYPE}-${TIMESTAMP}.md"
```

Ensure `.design-reports/` in `.gitignore`. Write header immediately.

## 0.5 Resume Protocol

If recent (<1hr) incomplete report exists: read it, check Status, resume from next step.

## 0.6 Trend Research (Optional — `--research` flag)

Parallel web searches for latest design trends, component libraries, animation patterns, performance standards. Cross-reference against Design Brain standards. Cache for session.

---

# AUDIT PATH

Triggered when pages/routes exist.

---

## Phase 1A: ASSESS

### 1A.1 Enumerate Routes & Components
Discover all pages, categorize all components (Layout, Modal, Form, Data, Feedback, Navigation).

### 1A.2 Map Responsive Coverage
Classify each component: Full (3+bp) / Partial (1-2bp) / None.

### 1A.3 Build Route Viewport Matrix
Score every route at Mobile / Tablet / Desktop. **Tablet gap detection:** if `md:` usage < 30% of `sm:`, flag missing tablet optimization.

### 1A.4 Run 14-Domain Design Scan (5 Parallel Agents)

Each finding gets a DES-XXX ID. After each domain: add to findings table, update progress log, write to disk.

**Agent 1 — Layout & Stability (Domains 1-2):**
Breakpoint coverage, horizontal overflow, hardcoded widths, grid/flex overflow, tablet gap, missing mobile layouts, CLS, skeleton states, font loading, progressive rendering

**Agent 2 — Touch & Navigation (Domains 3-4):**
Touch targets <44px, spacing <8px, hover-only interactions, inputmode, scroll hijacking, mobile nav (CRITICAL), modal adaptation, focus trap, safe areas, scroll restoration

**Agent 3 — Typography & Media (Domains 5-6):**
Font size <16px mobile, fluid typography, line length, heading hierarchy, contrast, Image sizes/priority, raw `<img>`, chart responsiveness/accessibility. **Anti-slop check:** Inter font, serif on dashboards

**Agent 4 — Accessibility & Consistency (Domains 7-8):**
Zoom blocked (CRITICAL), reduced-motion, reflow at 320px, focus-visible, skip-to-content, spacing/radius/shadow/color consistency, missing states, z-index chaos, dark mode quality. **Anti-slop check:** AI Tells, token violations

**Agent 5 — Visual Polish (Domains 9-14):**
White space/hierarchy, 5-state audit on every interactive element, emotional design/microcopy (generic labels, cold errors, no onboarding), cognitive load (Hick's/Miller's law, progressive disclosure), motion quality (timing, choreography, reduced-motion), platform conventions (px vs rem, safe areas, PWA manifest, tap highlight). **Anti-slop check:** "Jane Doe" effect, filler words, banned patterns

**Additional Checks (All Agents):**
- i18n: physical properties used instead of logical (`ml-4` → `ms-4`)
- SEO: missing metadata, OG tags, structured data
- Images: missing art direction (`<picture>`), wrong `fetchpriority`
- Skeletons: dimension mismatch with actual content
- WCAG 2.2: focus obscured by sticky headers, drag-only interactions, target size < 24px
- PWA: incomplete manifest, `theme_color` mismatch
- Performance: missing `content-visibility: auto` on long pages
- RSC: `"use client"` too high in tree, missing Suspense boundaries
- Fluid: fixed spacing where `clamp()` should be used
- Component API: props count > 7 (needs compound pattern)

### 1A.5 10-Dimension Design Scoring

Score the project across 10 dimensions (0-10 each):

| Dimension | What It Measures |
|-----------|-----------------|
| 1. Visual Hierarchy | Eye flow, focal points, CTA prominence |
| 2. Typography | Font choice, scale, fluid sizing, readability |
| 3. Color System | Palette quality, contrast, dark mode, token usage |
| 4. Spacing & Rhythm | 8pt grid adherence, section breathing, consistency |
| 5. Visual Consistency | Patterns unified across pages, design system coherence |
| 6. Responsiveness | Viewport matrix coverage, tablet optimization, touch targets |
| 7. Motion & Interaction | State coverage, timing, choreography, performance |
| 8. Content Quality | Microcopy, error messages, empty states, tone |
| 9. Performance | LCP, CLS, image optimization, lazy loading |
| 10. Anti-Slop Compliance | AI Tell violations, forbidden pattern count |

**Overall = average of 10 dimensions, mapped to grade:**
- 90-100: A+ (Award-Worthy)
- 80-89: A (Premium Production)
- 70-79: B (Professional)
- 60-69: C (Adequate)
- 50-59: D (Needs Work)
- Below 50: F (Significant Issues)

### 1A.6 Conversion Scoring (Landing/Marketing Pages)

If landing pages exist, also score Hero (0-100), CTA (0-100), Social Proof (0-100), Performance (0-100).

### 1A.7 Smart Adaptation Analysis

Per-route adaptation plans (Desktop → Mobile → Tablet per element) and component-level recommendations.

### 1A.8 Generate Prioritized Task List

Critical → High → Medium → Low. Write to report.

---

## Phase 2A: IMPLEMENT (Audit Path)

Auto-fix safe issues. No pause. Severity order.

### Safe Auto-Fixes

| Category | Action |
|----------|--------|
| Missing responsive grid | Add `grid-cols-1` mobile fallback |
| Hardcoded widths | `w-[500px]` → `w-full max-w-[500px]` |
| Missing Image props | Add `sizes`, `priority`, `aspect-ratio` |
| Touch targets too small | Increase padding |
| Missing `aria-hidden` on decorative SVGs | Add attribute |
| Missing focus-visible | Add ring classes |
| Missing empty/loading/error states | Add basic components |
| Missing active/pressed states | Add `active:scale-[0.98]` |
| Linear easing | Replace with `ease-out` |
| Wrong transition timing | Fix to 150-300ms |
| Tap highlight visible | Add CSS override |
| UI text selectable | Add `select-none` to UI elements |
| Cramped padding | `p-2` → `p-4` on containers |
| Generic button labels | "Submit" → specific action verb |
| `h-screen` usage | Replace with `min-h-[100dvh]` |
| Inter font detected | Flag for manual replacement |

### Deferred (Needs Design Decision)
Modal → bottom sheet, sidebar → mobile nav, table → card stack, design system unification, onboarding flow, dark mode elevation rework

### Fix Cycle
1. Update status → FIXING → apply fix → verify build → FIXED or BLOCKED → write checkpoint

---

# GENERATION PATH

Triggered when no pages exist, or `--generate` flag set.

---

## Phase 1B: PLAN

### 1B.1 Template Selection

| Signal | Template |
|--------|----------|
| SaaS, subscriptions, pricing | **saas** |
| Physical/digital product | **product** |
| Personal/agency showcase | **portfolio** |
| Pre-launch, waitlist | **waitlist** |
| Mobile/web app | **app** |
| Unclear | **saas** (default) |

### 1B.2 Vibe Selection

Via `--vibe` flag or infer from project:
- **ethereal-glass** → Dark, glassy, tech (see Vibe Archetypes)
- **editorial-luxury** → Warm, serif, editorial
- **soft-structuralism** → Clean, structural, consumer
- **modern** (default if no vibe) → Glassmorphism, bento, bold
- **minimal** → White space, muted, simple
- **enterprise** → Professional, conservative, trust

### 1B.3 Component Library

| Library | When |
|---------|------|
| **shadcn** (default) | Professional — MUST customize beyond defaults |
| **aceternity** | Wow-factor (3D cards, Spotlight, Parallax) |
| **magic-ui** | Playful/consumer (Scratch cards, Morphing buttons) |
| **mix** | shadcn base + Aceternity hero + Magic UI CTAs |

### 1B.4 Section Planning

Based on template, plan sections. Apply Design Brain: anti-slop pre-check on hero layout, color palette, font pairing, content strategy, motion plan.

### 1B.5 Token System Setup

Generate CSS custom properties following Three-Layer Token Architecture. Set up semantic color mappings. Configure dark mode surface hierarchy.

---

## Phase 2B: IMPLEMENT (Generation Path)

### 2B.1 Install Dependencies
Check what's needed, install what's missing.

### 2B.2 Generate Token System
CSS custom properties: primitives → semantic → component tokens.

### 2B.3 Generate Sections Sequentially
For each section: generate component → apply Design Brain standards → wire into page.

All generated code MUST comply with: typography rules, color tokens, layout rules (variance dial), motion rules (intensity dial), 5 interactive states, no AI Tells, conversion psychology, form UX patterns, mobile-first responsive.

### 2B.4 Built-In Conversion Optimization
Headlines from formula library, specific CTA verbs, trust indicators, social proof near CTAs, behavioral models applied (anchoring, social proof, goal-gradient).

### 2B.5 Performance & Responsive
Hero: `priority` + `sizes`. Below-fold: lazy. Skeletons for async. `min-h-[100dvh]`. Every component responsive: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`. Tablet intermediate states.

---

# PHASE 3: VERIFY (Both Paths)

```bash
$PM run typecheck --if-present
$PM run lint --if-present
$PM run build
```

If fails → auto-fix (max 3 attempts per issue).

### Anti-Slop Compliance Scan

Grep all modified/generated files for:
- Inter/Roboto/Arial font usage
- Purple/violet AI colors
- Emojis in code
- Generic names/data
- `h-screen` usage
- Generic button text ("Submit", "Click Here")
- Raw hex colors (should be tokens)
- `#000000` pure black
- Physical CSS properties (`ml-`/`mr-`/`pl-`/`pr-` → `ms-`/`me-`/`ps-`/`pe-`)
- Missing SEO metadata / OG tags on pages
- Missing `content-visibility: auto` on pages with 5+ below-fold sections
- `"use client"` too high in component tree (should be leaf only)

If violations → auto-fix and re-verify.

---

# PHASE 4: REPORT (Both Paths)

### Audit Report — Executive Summary

```markdown
## Executive Summary

| Metric | Count |
|--------|-------|
| Routes Scanned | [X] |
| Components Analyzed | [X] |
| Issues Found | [X] |
| Auto-Fixed | [X] |
| Deferred | [X] |
| Blocked | [X] |

### Design Score: [X]/100 (Grade: [A-F])

| Dimension | Score |
|-----------|-------|
| Visual Hierarchy | [X]/10 |
| Typography | [X]/10 |
| Color System | [X]/10 |
| Spacing & Rhythm | [X]/10 |
| Visual Consistency | [X]/10 |
| Responsiveness | [X]/10 |
| Motion & Interaction | [X]/10 |
| Content Quality | [X]/10 |
| Performance | [X]/10 |
| Anti-Slop Compliance | [X]/10 |

### Responsive Coverage: [X]% → [Y]% (after fixes)
```

### Status Report Display

**Audit — Clean:**
```
═══════════════════════════════════════════════════════════════
DESIGN AUDIT — [project]
═══════════════════════════════════════════════════════════════
Stack: [Framework] + [CSS] + [Components]
Score: [X]/100 (Grade [A+])

RESULTS
├─ Routes:              [X] scanned
├─ Components:          [X] analyzed
├─ Issues:              0
├─ Anti-Slop:           COMPLIANT
└─ Responsive Coverage: 100%

Result: IMMACULATE
Report: .design-reports/audit-[timestamp].md
Next: /gh-ship
═══════════════════════════════════════════════════════════════
```

**Audit — Issues Found:**
```
═══════════════════════════════════════════════════════════════
DESIGN AUDIT — [project]
═══════════════════════════════════════════════════════════════
Score: [X]/100 → [Y]/100 (Grade [B] → [A])

RESULTS
├─ Issues Found:        [X]
├─ Auto-Fixed:          [X]
├─ Deferred:            [X]
├─ Anti-Slop:           [X] violations corrected
└─ Responsive:          [X]% → [Y]%

Fixed: [list]
Deferred: [list]

Result: [X] FIXED | [Y] DEFERRED — [duration]
Report: .design-reports/audit-[timestamp].md
Next: Address deferred, then /design again
═══════════════════════════════════════════════════════════════
```

**Generation Complete:**
```
═══════════════════════════════════════════════════════════════
DESIGN GENERATE — [project]
═══════════════════════════════════════════════════════════════
Template: [type] | Vibe: [archetype] | Components: [lib]
Dials: V=[X] M=[X] D=[X]

GENERATED
├─ Pages:          [X] created
├─ Components:     [X] created
├─ Anti-Slop:      COMPLIANT
├─ Responsive:     Mobile-first (320px → 1920px)
└─ Build:          PASS

Files: [list]

Result: Premium [template] generated — [duration]
Report: .design-reports/generate-[timestamp].md
Next: /design (audit) → /gh-ship
═══════════════════════════════════════════════════════════════
```

---

## Context Management

Follows **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.
- Sub-agents: < 500 tokens each, full findings in report
- State checkpoint: `.design-reports/state-YYYYMMDD-HHMMSS.json`
- Max 2 parallel agents. Lean orchestrator messages.

---

## Smart Adaptation Reference

| Desktop | Mobile | Tablet |
|---------|--------|--------|
| Multi-column grid | Single column | 2 columns |
| Sidebar nav | Bottom tabs / hamburger | Collapsible sidebar |
| Data table 8+ cols | Card stack (3 key fields) | Scroll + frozen col |
| Hover tooltips | Long-press / inline | Long-press |
| Large modal | Bottom sheet | Centered (max 600px) |
| Complex dashboard | Scrollable cards | 2-col grid |
| Wide hero + overlay | Stacked: image/text | Split 60/40 |
| Horizontal wizard | Step counter | Vertical progress |

---

## Pipeline Position

```
/design → /a11y → /perf → /sec-ship → /redteam → /gh-ship
```

---

## Pre-Flight Checklist (All Output)

- [ ] Mobile layout collapse guaranteed?
- [ ] `min-h-[100dvh]` not `h-screen`?
- [ ] All 5 interactive states present?
- [ ] Empty, loading, error states provided?
- [ ] Color tokens used (no raw hex)?
- [ ] Dark mode surface hierarchy correct?
- [ ] No AI Tells (fonts, colors, content, patterns)?
- [ ] Design dials reflected?
- [ ] Performance guardrails met?
- [ ] Toast timing formula applied?
- [ ] Form UX patterns followed?
- [ ] Conversion psychology applied (if marketing page)?
- [ ] `useEffect` animations have cleanup?
- [ ] Perpetual animations isolated?
- [ ] CSS logical properties used (not physical `ml-`/`mr-`)?
- [ ] SEO metadata + OG tags on every page?
- [ ] Structured data (JSON-LD) for page type?
- [ ] `content-visibility: auto` on long pages?
- [ ] Skeleton dimensions match actual content?
- [ ] Responsive images with `<picture>` + AVIF/WebP?
- [ ] `scroll-padding-top` accounts for sticky headers?
- [ ] Drag interactions have button alternatives (WCAG 2.5.7)?
- [ ] `"use client"` only on leaf components?
- [ ] Fluid spacing with `clamp()` where appropriate?

---

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

1. Close all Playwright browser instances (`browser_close`)
2. Keep screenshots in `.design-reports/screenshots/`
3. Verify no orphaned Chromium processes
4. Release dev server port
5. Enforce `.design-reports/` in `.gitignore`
6. Document installed deps in report

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
<!-- Incorporates design engineering concepts from: -->
<!-- - taste-skill by Leonxlnx (MIT) -->
<!-- - design-with-claude by imsaif -->
<!-- - skill.color-expert by meodai -->
<!-- - ui-ux-pro-max-skill by nextlevelbuilder -->
<!-- - elite-frontend-ux by majidmanzarpour -->
<!-- - marketingskills by coreyhaines31 -->
