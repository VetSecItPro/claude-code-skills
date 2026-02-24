---
description: Design audit — scan, analyze, improve UI/UX across all device classes
## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

allowed-tools: Bash(ls *), Bash(cat *), Bash(head *), Bash(tail *), Bash(wc *), Bash(echo *), Bash(find *), Bash(grep *), Bash(jq *), Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Bash(mkdir *), Bash(date *), Read, Write, Edit, Glob, Grep, Task, WebSearch, mcp__playwright__browser_navigate, mcp__playwright__browser_click, mcp__playwright__browser_resize, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_snapshot, mcp__playwright__browser_evaluate, mcp__playwright__browser_close, mcp__playwright__browser_wait_for, mcp__playwright__browser_tabs
---

# /design — UI/UX Audit, Smart Adaptation & Visual Polish

**Scan every screen. Improve every viewport. Make it immaculate.**

This skill scans a project's entire codebase, evaluates every route and component across all device classes (mobile/tablet/desktop), identifies design deficiencies, makes smart adaptation recommendations (what to hide, collapse, rearrange, resize per viewport), auto-fixes safe issues, and produces a living report with prioritized improvement tasks.

**FIRE AND FORGET** — Execute autonomously. Scan everything. Fix safe issues. Report the rest.

**TREND-AWARE (Optional)** — Add `--research` flag to pull latest 2025/26 design trends from Reddit, X, and the web before auditing.

---

## Usage

```bash
# Standard design audit
/design

# Audit with latest trend research
/design --research

# Resume incomplete audit
/design --resume
```

---

## Execution Rules (CRITICAL)

- **NO permission requests** — just scan and improve
- **NO "should I proceed?" questions** — audit everything
- **Auto-detect project stack** — framework, CSS system, component library, breakpoints, theme
- **Think like an expert Apple designer** — every pixel matters, every interaction should feel polished
- **Smart adaptation, not just auditing** — recommend what to HIDE, COLLAPSE, REARRANGE, RESIZE per viewport
- **Auto-fix safe issues** — then verify build still passes
- **Mark findings FOUND/FIXING/FIXED/DEFERRED/BLOCKED** — never delete
- **Write persistent report** — `.design-reports/`
- **UPDATE REPORT AFTER EVERY PHASE** — the markdown file is the single source of truth
- **SITREP at conclusion** — summary + next action

---

## Report Persistence (CRITICAL — Survives Compaction/Restart)

The markdown report file is the **living document**. It must be self-contained and updated continuously so that if the session is compacted, restarted, or resumed, the report captures all progress.

### Rules

1. **Write the report immediately at Phase 0** — even before any scanning, the file must exist with the header, status, and placeholder sections
2. **Update the report at the END of every phase** — after reconnaissance, after each domain scan, after each fix
3. **Every finding gets a permanent entry** — row in the findings table + detail section. NEVER delete entries — only update status (FOUND → FIXED, etc.)
4. **Status field tracks progress** — the report header `Status:` field shows exactly where the skill stopped:
   - `🔴 IN PROGRESS — Phase 0: Pre-flight`
   - `🔴 IN PROGRESS — Phase 1: Reconnaissance`
   - `🔴 IN PROGRESS — Domain 3: Typography`
   - `🟡 IN PROGRESS — Fixing DES-003`
   - `🟢 COMPLETE`
5. **If session restarts**, read the existing report file first. Check the `Status:` field and `## Progress Log` to determine where to resume. Do NOT re-run completed phases.
6. **Progress Log section** — append a timestamped line after each phase/domain completes

### Resume Protocol

If the skill is invoked and a recent report file exists (< 1 hour old) in `.design-reports/`:

1. Read the most recent report file
2. Check `Status:` — if `🟢 COMPLETE`, start a fresh run
3. If NOT complete, check `## Progress Log` to find the last completed step
4. Resume from the next step (skip completed phases/domains)
5. Continue updating the SAME report file

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Sub-agents scanning design domains return < 500 tokens each (full findings written to .design-reports/)
- State file `.design-reports/state-YYYYMMDD-HHMMSS.json` tracks which domains are complete
- Resume from checkpoint if context resets — skip completed domain scans
- Max 2 parallel agents (pair independent domains only)
- Orchestrator messages stay lean: "Domain 3/14 Typography: 12 findings (2 critical)"

---

## Design Philosophy

You are an **expert UI/UX designer and front-end architect** hired to make this application look and feel world-class on every device. Your job:

1. **Discover** — Map every route, component, modal, form, table, and navigation element
2. **Evaluate** — Check each element against design best practices at every viewport
3. **Decide** — Make smart recommendations about what should change per device class
4. **Improve** — Auto-fix safe issues, create tasks for everything else
5. **Verify** — Ensure fixes don't break the build
6. **Report** — Deliver a professional design audit with prioritized improvement tasks

### Smart Adaptation Principles

Not everything on desktop belongs on mobile. Not everything on mobile needs the desktop layout. **Think critically about each element:**

| Desktop Pattern | Mobile Adaptation | Reasoning |
|----------------|-------------------|-----------|
| Multi-column grid | Single column stack | Screen width insufficient for columns |
| Sidebar navigation | Bottom tab bar or hamburger | Thumb-friendly, saves vertical space |
| Data table with 8+ columns | Card stack or priority columns | Tables don't fit, cards are scannable |
| Hover tooltips | Long-press or inline text | No hover on touch devices |
| Large modal dialog | Full-screen sheet or bottom sheet | Modals feel cramped on small screens |
| Inline form labels | Stacked labels above inputs | Horizontal space is precious |
| Breadcrumb trail | Back button + page title | Breadcrumbs overflow on small screens |
| Tab bar with 8 tabs | Scrollable tabs or dropdown | Too many tabs overflow |
| Fixed sidebar + scrollable content | Full-width content + collapsible nav | Need the full width |
| Desktop dropdown menu | Bottom sheet action menu | Easier to reach with thumb |
| Floating action buttons (multiple) | Single FAB or bottom bar | Reduce visual clutter |
| Complex dashboard with widgets | Scrollable widget cards, collapsible sections | Progressive disclosure |
| Wide hero section with text overlay | Stacked: image above, text below | Text overlay illegible on small images |
| Multi-step wizard (horizontal steps) | Vertical progress or step counter | Horizontal steps overflow |

### What "Immaculate UX" Means

- **Zero horizontal overflow** at any viewport — nothing scrolls sideways
- **Zero truncated text** that hides meaning — either show it or hide the whole element
- **Zero unreachable touch targets** — every interactive element is tappable (44x44px minimum)
- **Zero orphaned screens** — every page adapts gracefully, no desktop-only layouts on mobile
- **Consistent spacing** — the spacing system is uniform across the app
- **Fluid typography** — text scales smoothly, headings don't break to awkward line lengths
- **Purposeful animation** — animations enhance understanding, never distract
- **Fast perceived performance** — loading states, skeletons, progressive rendering

---

## Phase 0: Pre-Flight

### 0.1 Detect Project Stack

Read the project's configuration to determine what we're working with:

```bash
# Package manager
if [ -f "pnpm-lock.yaml" ]; then PM="pnpm"
elif [ -f "yarn.lock" ]; then PM="yarn"
elif [ -f "bun.lockb" ]; then PM="bun"
else PM="npm"; fi

PROJECT_NAME=$(jq -r '.name // empty' package.json 2>/dev/null || basename "$PWD")
```

#### 0.1.1 Meta Framework Detection

| Check | Framework |
|-------|-----------|
| `next.config.*` exists | **Next.js** — routes in `app/` or `pages/` |
| `remix.config.*` or `@remix-run` in deps | **Remix** — routes in `app/routes/` |
| `vite.config.*` + react in deps | **Vite + React** — check for react-router |
| `astro.config.*` | **Astro** — routes in `src/pages/` |
| `nuxt.config.*` | **Nuxt** — routes in `pages/` |
| `svelte.config.*` | **SvelteKit** — routes in `src/routes/` |
| `angular.json` | **Angular** — routes in `src/app/` |
| None of the above | **Static/Custom** — look for HTML files |

```bash
# Detect framework
if ls next.config.* 2>/dev/null | head -1 > /dev/null; then FRAMEWORK="nextjs"
elif grep -q "@remix-run" package.json 2>/dev/null; then FRAMEWORK="remix"
elif ls vite.config.* 2>/dev/null | head -1 > /dev/null; then FRAMEWORK="vite"
elif ls astro.config.* 2>/dev/null | head -1 > /dev/null; then FRAMEWORK="astro"
elif ls nuxt.config.* 2>/dev/null | head -1 > /dev/null; then FRAMEWORK="nuxt"
elif ls svelte.config.* 2>/dev/null | head -1 > /dev/null; then FRAMEWORK="sveltekit"
elif [ -f "angular.json" ]; then FRAMEWORK="angular"
else FRAMEWORK="static"; fi
```

#### 0.1.2 CSS System Detection

| Check | CSS System |
|-------|------------|
| `tailwind.config.*` or `@tailwind` in CSS | **Tailwind CSS** — check version (v3 vs v4) |
| `*.module.css` files exist | **CSS Modules** |
| `styled-components` in deps | **Styled Components** |
| `@emotion` in deps | **Emotion** |
| `.scss` or `.sass` files | **Sass/SCSS** |
| `*.css` files only | **Vanilla CSS** |

**Detect Tailwind version:**
- v4: `@import "tailwindcss"` in CSS, or `@tailwindcss/*` in deps
- v3: `@tailwind base; @tailwind components; @tailwind utilities` in CSS

```bash
# Detect CSS system
CSS_SYSTEM="unknown"
if ls tailwind.config.* 2>/dev/null | head -1 > /dev/null; then
  CSS_SYSTEM="tailwind"
  # Check version
  if grep -rq '@import "tailwindcss"' --include="*.css" . 2>/dev/null || \
     grep -q "@tailwindcss" package.json 2>/dev/null; then
    TAILWIND_VERSION="v4"
  else
    TAILWIND_VERSION="v3"
  fi
fi
# Also check for CSS Modules, Styled Components, etc.
```

#### 0.1.3 Breakpoint System Detection

**For Tailwind:** Read `tailwind.config.*` for custom `screens` config. If no custom screens, use defaults:
- v3/v4 defaults: sm(640), md(768), lg(1024), xl(1280), 2xl(1536)

**For non-Tailwind:** Grep all CSS/SCSS files for `@media` queries, extract breakpoint values, find the most common ones.

```bash
# For Tailwind: read config
# For others: extract media query breakpoints
grep -rohE '@media[^{]+' --include="*.css" --include="*.scss" --include="*.module.css" . 2>/dev/null | \
  grep -oE '[0-9]+px' | sort | uniq -c | sort -rn | head -10
```

**Store detected breakpoints for the viewport matrix.**

#### 0.1.4 Component Library Detection

| Check | Library |
|-------|---------|
| `components/ui/` with Radix patterns | **shadcn/ui** |
| `@radix-ui/*` in deps | **Radix UI** |
| `@mui/*` in deps | **Material UI** |
| `@chakra-ui/*` in deps | **Chakra UI** |
| `@mantine/*` in deps | **Mantine** |
| `@headlessui/*` in deps | **Headless UI** |
| `antd` in deps | **Ant Design** |
| None detected | **Custom components** |

#### 0.1.5 Animation Library Detection

| Check | Library |
|-------|---------|
| `framer-motion` in deps | **Framer Motion** |
| `gsap` in deps | **GSAP** |
| `@react-spring/*` in deps | **React Spring** |
| `animejs` in deps | **Anime.js** |
| CSS `@keyframes` only | **CSS Animations** |
| None | **No animations** |

#### 0.1.6 Additional Detection

- **Theme mode:** Check for dark mode support (Tailwind `dark:` classes, CSS `prefers-color-scheme`, theme toggle component)
- **Image component:** `next/image`, custom `<Image>`, or raw `<img>`
- **Device handling:** Look for responsive hooks (`useMediaQuery`, `useDevice`, `useBreakpoint`), device context providers, or CSS-only responsive approach
- **Icon library:** Lucide, Heroicons, Font Awesome, Material Icons, SVG sprites

#### 0.1.7 Write Stack Profile to Report

Store ALL detected info in the report header — this guides every subsequent scan.

### 0.2 Create Report Directory & File

```bash
mkdir -p .design-reports
TIMESTAMP=$(date +"%Y%m%d-%H%M%S")
REPORT_FILE=".design-reports/design-${TIMESTAMP}.md"
```

Ensure `.design-reports/` is in `.gitignore`. Add if missing.

### 0.3 Check for Resumable Report

Same resume protocol as other skills — check for recent (<1hr old) incomplete report.

### 0.4 Initialize New Report

```markdown
# Design Audit Report — [project-name]

**Date:** YYYY-MM-DD HH:MM
**Status:** 🔴 IN PROGRESS — Phase 0: Pre-flight

---

## Stack Profile

| Property | Detected |
|----------|----------|
| Framework | [Next.js 15 / Remix / Vite / etc.] |
| CSS System | [Tailwind v4 / CSS Modules / etc.] |
| Component Library | [shadcn/ui / MUI / custom / etc.] |
| Animation Library | [Framer Motion / CSS / none] |
| Breakpoints | [sm:640 md:768 lg:1024 xl:1280 2xl:1536] |
| Theme Mode | [Dark only / Light+Dark / Light only] |
| Image Handling | [next/image / raw img / custom] |
| Device Handling | [DeviceContext / useMediaQuery / CSS-only] |
| Icon Library | [Lucide / Heroicons / etc.] |
| Routes Discovered | [X] |
| Components Scanned | [X] |

---

## Progress Log

| Time | Phase | Action | Result |
|------|-------|--------|--------|
| [HH:MM] | Phase 0 | Pre-flight | Stack detected |

---

## Route Viewport Matrix

_To be populated during reconnaissance_

---

## Findings

| # | Domain | Issue | Route/Component | Viewport | Severity | Status |
|---|--------|-------|-----------------|----------|----------|--------|

---

## Smart Adaptation Recommendations

_Recommendations for what to hide/collapse/rearrange per viewport_

---

## Improvement Tasks

_Prioritized task list generated from findings_

---

## Fix Log

| # | Finding | File | Action | Build | Status |
|---|---------|------|--------|-------|--------|

---

## Verification

| Check | Status |
|-------|--------|
| Build | ⏳ Pending |
| All auto-fixes verified | ⏳ Pending |
| No regressions | ⏳ Pending |
```

**IMPORTANT:** Write this file to disk IMMEDIATELY. Every subsequent phase updates this same file.

---

## Phase 0.5: Trend Research (Optional — Triggered with --research flag)

**When to use:** Include `--research` flag to pull latest 2025/26 design trends before running audit.

### Research Sources

Run parallel web searches to gather latest design intelligence:

```bash
# Visual & aesthetic trends
WebSearch: "web design trends 2026 UI UX modern"
WebSearch: "glassmorphism bento grid design 2026"
Reddit: r/web_design "2026 trends"
Reddit: r/UI_Design "modern design patterns"

# Component libraries & frameworks
WebSearch: "React component libraries 2026 shadcn Aceternity"
WebSearch: "Next.js 15 best practices design patterns"
X: "#webdesign #UI2026"

# Animation & motion design
WebSearch: "Framer Motion GSAP 2026 trends"
WebSearch: "micro-interactions motion design 2026"
Reddit: r/webdev "animation trends"

# Performance & Core Web Vitals
WebSearch: "Core Web Vitals 2026 optimization"
WebSearch: "landing page performance best practices"

# Accessibility & responsive
WebSearch: "responsive design mobile first 2026"
WebSearch: "WCAG accessibility trends"
```

### Synthesis

After gathering research:
1. Extract **consensus patterns** (what multiple sources agree on)
2. Identify **emerging trends** (what's new in 2026)
3. Note **deprecated patterns** (what to avoid)
4. Cache findings for the session

### Research Report Section

Add to the audit report:

```markdown
## 2025/26 Design Trends Applied

**Researched:** YYYY-MM-DD HH:MM

### Visual Trends
- Glassmorphism for depth (translucent surfaces, blurred backgrounds)
- Bento grids for content organization
- Bold saturated colors (Y2K nostalgia, dopamine design)
- Soft UI for tactile 3D elements

### Component Libraries
- shadcn/ui: Industry standard (copy/paste, full control)
- Aceternity UI: 200+ animated components for wow-factor
- Magic UI: Interactive 3D elements for engagement

### Animation Best Practices
- Motion (Framer Motion): 2.5x faster than GSAP, best for React
- Entrance: 150-300ms ease-out
- Exit: 100-200ms ease-in
- Stagger: 30-60ms between siblings

### Performance Standards
- LCP < 2.5s (Largest Contentful Paint)
- INP < 200ms (Interaction to Next Paint, replaced FID in 2024)
- CLS < 0.1 (Cumulative Layout Shift)
- Mobile-first: 83% of traffic is mobile

### Key Insights
- [Trend-specific insight relevant to this project]
- [Recommendation based on latest research]

**Sources:**
- [Link to source 1]
- [Link to source 2]
```

### Update Status

After research completes:
- Append to Progress Log: `Phase 0.5 | Trend Research | [X] sources analyzed`
- Update Status: `🔴 IN PROGRESS — Phase 1: Reconnaissance`

---

## Phase 1: Reconnaissance

Map the entire design surface from source code.

### 1.1 Enumerate All Routes/Pages

**Framework-specific route discovery:**

| Framework | Discovery Method |
|-----------|-----------------|
| Next.js (App Router) | `find app -name "page.tsx" -o -name "page.jsx" -o -name "page.ts"` |
| Next.js (Pages Router) | `find pages -name "*.tsx" -o -name "*.jsx"` (exclude `_app`, `_document`) |
| Remix | `find app/routes -name "*.tsx" -o -name "*.jsx"` |
| Vite + React Router | Read router config file for route definitions |
| Astro | `find src/pages -name "*.astro" -o -name "*.md"` |
| Nuxt | `find pages -name "*.vue"` |
| SvelteKit | `find src/routes -name "+page.svelte"` |
| Static | `find . -name "*.html" -maxdepth 3` |

For each route:
- Parse the file path to determine the URL
- Identify dynamic segments (`[id]`, `$id`, `:id`)
- Note route groups and layouts
- Flag whether it's a public or auth-protected route

### 1.2 Enumerate Components

Scan for all UI components:

```bash
# Find component directories
find . -type d -name "components" -not -path "*/node_modules/*" 2>/dev/null

# Find all component files
find . -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" -o -name "*.svelte" | \
  grep -v node_modules | grep -v ".test." | grep -v ".spec."
```

Categorize each component:
- **Layout components:** Headers, sidebars, footers, navigation, page wrappers
- **Modal/Dialog components:** Modals, sheets, drawers, popovers, dropdowns
- **Form components:** Inputs, selects, textareas, checkboxes, radio, toggles, date pickers
- **Data display:** Tables, cards, lists, grids, charts
- **Feedback:** Toasts, alerts, loading states, empty states, error states
- **Navigation:** Tabs, breadcrumbs, pagination, menus, nav bars

### 1.3 Map Current Responsive Approach

For each component, determine how it currently handles responsiveness:

**For Tailwind projects:**
```bash
# Count responsive prefixes per file
grep -c "sm:\|md:\|lg:\|xl:\|2xl:" [file]
```

**For CSS Module projects:**
```bash
# Count @media queries per module
grep -c "@media" [file.module.css]
```

**For Styled Components:**
```bash
# Count media query usage
grep -c "@media\|breakpoint" [file]
```

Classify each component's responsive coverage:
- **Full:** Has responsive handling at 3+ breakpoints
- **Partial:** Has 1-2 breakpoints (usually just `md:` for mobile/desktop split)
- **None:** No responsive handling at all — same on all viewports

### 1.4 Build Route Viewport Matrix

Create a matrix of every route vs. the project's breakpoints:

```markdown
## Route Viewport Matrix

| Route | Mobile (<768) | Tablet (768-1023) | Desktop (1024+) | Coverage |
|-------|---------------|-------------------|-----------------|----------|
| / (home) | ✅ Full | ⚠️ Partial | ✅ Full | 83% |
| /dashboard | ❌ None | ❌ None | ✅ Full | 33% |
| /tasks | ✅ Full | ✅ Full | ✅ Full | 100% |
| /settings | ⚠️ Partial | ❌ None | ✅ Full | 50% |
```

**How to score:**
- Read the page file and all components it imports
- Check each for responsive classes/queries at the mobile, tablet, and desktop breakpoints
- ✅ Full = layout changes exist for this viewport
- ⚠️ Partial = some responsive handling but incomplete (e.g., only hides one element)
- ❌ None = no responsive handling, desktop layout shown at all sizes

**CRITICAL — Tablet scoring (768–1023px):**
Most projects have a "tablet gap" — components use `sm:` (640px) and `lg:` (1024px) but skip `md:` (768px) entirely. This causes the desktop layout to render at tablet widths, often with cramped grids, overflowing flex rows, and misaligned elements. To score tablet:
- Count `md:` breakpoint usage per route — if `md:` usage is <30% of `sm:` usage, the route likely has no tablet optimization
- Check grids: any `md:grid-cols-3+` is suspect (columns may be <250px)
- Check flex-rows: `sm:flex-row` with 3+ children likely overflows at 768px
- Check show/hide: `sm:hidden` elements disappear at 640px, but small tablets (640–767px) may still need mobile patterns
- Use Playwright to screenshot at 768px and 1024px — visual verify that layouts have proper intermediate states

### 1.5 Update Report

Write the Route Viewport Matrix to the report file. Append to Progress Log. Update Status.

---

## Phase 2: Design Domain Scans (5 Parallel Agents, 14 Domains)

Run all 14 domains across 5 parallel agents. Each agent scans the codebase through its domains, using the detected stack profile to run framework-appropriate checks.

**AFTER EACH DOMAIN completes:**
1. Add all findings to `## Findings` table (new rows with DES-XXX IDs)
2. Add smart adaptation recommendations to `## Smart Adaptation Recommendations`
3. Append to `## Progress Log`
4. Update `**Status:**`
5. **Write the file to disk**

### Agent Overview

| Agent | Domains | Focus |
|-------|---------|-------|
| Agent 1 | 1-2 | Layout & Breakpoints + Layout Stability & Perceived Performance |
| Agent 2 | 3-4 | Touch & Interaction + Navigation, Modals & Scroll Behavior |
| Agent 3 | 5-6 | Typography & Readability + Images, Media & Data Visualization |
| Agent 4 | 7-8 | Accessibility/Responsive + Design Consistency & Dark Mode Quality |
| Agent 5 | 9-14 | Visual Hierarchy, Micro-interactions, Emotional Design, Cognitive Load, Motion Design, Platform Conventions |

**Sub-agent prompt rule:** Each agent prompt MUST include: "Return ONLY a structured summary under 500 tokens. Full findings go to disk."

---

### Agent 1: Layout & Breakpoint Coverage + Layout Stability

#### Domain 1: Breakpoint Coverage

**Objective:** Verify every route and component adapts properly at every breakpoint.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 1.1 | Horizontal overflow | Find elements with `overflow-x: auto/scroll/hidden` missing on containers with wide children. Grep for fixed-width elements that exceed mobile viewport (>375px hardcoded) | HIGH |
| 1.2 | Missing mobile layout | Routes/components with NO responsive classes below `md:` breakpoint | HIGH |
| 1.3 | Orphaned responsive classes | `hidden md:block` paired with conflicting `flex` on same element. Or `md:hidden` on element that's already `hidden` by default | LOW |
| 1.4 | Hardcoded widths | `w-[500px]`, `width: 500px`, or similar hardcoded values that exceed mobile viewport without responsive alternatives | MEDIUM |
| 1.5 | Grid column overflow | `grid-cols-3` or higher without `grid-cols-1` for mobile | MEDIUM |
| 1.6 | Flex nowrap overflow | `flex-nowrap` or `flex-row` without `flex-col` for mobile (when children would overflow) | MEDIUM |
| 1.7 | Container query candidates | Components in variable-width containers (sidebars, grid cells) using only viewport media queries — recommend `@container` (Tailwind v4+ or modern CSS only) | LOW |
| 1.8 | Non-standard breakpoints | JS using hardcoded pixel values instead of project's breakpoint constants | LOW |
| 1.9 | Desktop-only patterns | Components that only render meaningful content at desktop width (complex dashboards, wide tables) with no mobile alternative | HIGH |
| 1.10 | **Missing tablet breakpoint (`md:`)** | Components that jump from `sm:` (640px) directly to `lg:` (1024px) or desktop layout, with no `md:` (768px) intermediate step. Tablets (768–1023px) get desktop layout crammed into smaller space. Count `sm:` vs `md:` vs `lg:` usage — if `md:` is <20% of `sm:` usage, flag as missing tablet optimization | HIGH |
| 1.11 | **Grid 3+ cols at `md:` (768px)** | `md:grid-cols-3` or higher at 768px — columns are only ~200px each after gaps. Grids with 3+ columns should use `lg:` (1024px+) not `md:`. Check: each column should be ≥250px minimum | HIGH |
| 1.12 | **Flex-row overflow at `sm:` (640px)** | `sm:flex-row` with 3+ children whose total width exceeds 640px. Common with proof bars, nav items, and stat rows. Should use `md:flex-row` or grid with `sm:grid-cols-2` intermediate | MEDIUM |
| 1.13 | **Sticky/fixed elements hidden too early** | Sticky CTAs, mobile navs, or floating elements using `sm:hidden` when they should persist to `md:hidden`. Small tablets (640–767px) still need mobile-style navigation and conversion prompts | MEDIUM |
| 1.14 | **No intermediate sizing steps** | Fixed-width elements (panels, sidebars) with only `sm:w-[X]` and no `md:w-[Y]`. At 768px the proportions are wrong. Should have `sm:` → `md:` → `lg:` progression | LOW |

**Tailwind-specific checks:**
```bash
# Find hardcoded widths that may overflow mobile
grep -rn "w-\[.*px\]" --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find grid columns without mobile fallback
grep -rn "grid-cols-[3-9]\|grid-cols-1[0-2]" --include="*.tsx" --include="*.jsx" | grep -v "grid-cols-1 \|grid-cols-2 " | grep -v node_modules
```

**Tablet breakpoint audit (CRITICAL — run for every project):**
```bash
# Count breakpoint prefix usage — md: should be ≥30% of sm: usage
echo "sm: count:" && grep -rn "sm:" --include="*.tsx" --include="*.jsx" | grep -v node_modules | wc -l
echo "md: count:" && grep -rn "md:" --include="*.tsx" --include="*.jsx" | grep -v node_modules | wc -l
echo "lg: count:" && grep -rn "lg:" --include="*.tsx" --include="*.jsx" | grep -v node_modules | wc -l
# Find 3+ col grids at md: (should be lg:)
grep -rn "md:grid-cols-[3-9]" --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find sm:flex-row with no md: intermediate
grep -rn "sm:flex-row" --include="*.tsx" --include="*.jsx" | grep -v "md:" | grep -v node_modules
# Find sm:hidden elements (should some be md:hidden?)
grep -rn "sm:hidden" --include="*.tsx" --include="*.jsx" | grep -v node_modules
```

**CSS/SCSS checks:**
```bash
# Find fixed widths in CSS
grep -rn "width:\s*[4-9][0-9][0-9]px\|width:\s*[1-9][0-9][0-9][0-9]px" --include="*.css" --include="*.scss" | grep -v node_modules
```

#### Domain 2: Layout Stability

**Objective:** Prevent layout shift (CLS) across breakpoints and during loading.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 2.1 | Images without dimensions | `<img>` or `<Image>` without `width`/`height` or `fill` prop — causes CLS | HIGH |
| 2.2 | Layout-triggering animations | Animations on `width`, `height`, `top`, `left`, `margin`, `padding` instead of `transform`/`opacity` | MEDIUM |
| 2.3 | Missing skeleton/loading states | Pages with async data but no loading placeholder — content jumps when data arrives | MEDIUM |
| 2.4 | Dynamic content injection | Elements that appear/disappear and push other content around without reserved space | MEDIUM |
| 2.5 | `will-change` overuse | More than 5 elements with `will-change` (memory overhead on mobile) | LOW |
| 2.6 | Font loading CLS | No `font-display: swap` or font preloading — text reflows when font loads | MEDIUM |
| 2.7 | No skeleton screens | Loading states use spinners instead of skeleton placeholders that match content layout — skeletons feel 30% faster perceptually | MEDIUM |
| 2.8 | No optimistic UI | State-changing actions (create, update, delete) wait for server response before updating UI — feels sluggish. Check for `await` before state update | LOW |
| 2.9 | Progressive rendering missing | Pages that wait for ALL data before rendering anything, instead of showing content progressively as it arrives | MEDIUM |
| 2.10 | Skeleton-content mismatch | Skeleton placeholder shapes don't match the actual content layout — feels disorienting when content loads | LOW |

---

### Agent 2: Touch & Interaction + Navigation & Modals

#### Domain 3: Touch & Interaction

**Objective:** Every interactive element must be comfortably tappable on touch devices.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 3.1 | Touch target size | Interactive elements (buttons, links, inputs, checkboxes, toggles) smaller than 44x44px at mobile. Check for `h-6`, `w-6`, `p-1` (24px), icon-only buttons without padding | HIGH |
| 3.2 | Touch target spacing | Interactive elements too close together (< 8px gap) — accidental taps | MEDIUM |
| 3.3 | Hover-only interactions | Features that only work on `:hover` (tooltips, dropdown triggers, hover cards) with no touch alternative | HIGH |
| 3.4 | Missing `inputmode` | Number inputs without `inputmode="numeric"`, email inputs without `inputmode="email"`, phone without `inputmode="tel"` | LOW |
| 3.5 | Click vs tap confusion | `onMouseDown`/`onMouseEnter` without corresponding touch events | MEDIUM |
| 3.6 | Scroll hijacking | Custom scroll behaviors that interfere with native touch scrolling | HIGH |
| 3.7 | Missing active/pressed states | Buttons without `:active` or tap feedback — feels unresponsive on mobile | LOW |

**Tailwind-specific touch target check:**
```bash
# Find small interactive elements (likely below 44px)
grep -rn 'className="[^"]*\b\(h-[1-8]\|w-[1-8]\|p-[01]\|text-xs\)[^"]*"' --include="*.tsx" --include="*.jsx" | \
  grep -i "button\|onClick\|href\|<a \|<Link " | grep -v node_modules
```

**Smart adaptation recommendations:**
- Hover tooltips → long-press on mobile, or always-visible help text
- Small icon buttons → add padding on mobile (`md:p-1 p-2` or similar)
- Close-together actions → stack vertically on mobile instead of horizontally

#### Domain 4: Navigation & Modals

**Objective:** Navigation and modals must adapt intelligently to each viewport.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 4.1 | No mobile navigation | Desktop sidebar/top nav but no mobile alternative (hamburger, bottom tabs, drawer) | CRITICAL |
| 4.2 | Modal not adapted for mobile | Modals using centered overlay on mobile instead of bottom sheet or full-screen | HIGH |
| 4.3 | Dropdown positioning | Dropdowns that overflow viewport edges on mobile (not using portal or boundary detection) | MEDIUM |
| 4.4 | Missing focus trap | Modals/sheets without focus trap (Tab key leaves the modal) | HIGH |
| 4.5 | Missing close mechanism | Modals without Escape key handler or swipe-to-dismiss on mobile | MEDIUM |
| 4.6 | Breadcrumb overflow | Breadcrumb trails that overflow on mobile without truncation or back button alternative | LOW |
| 4.7 | Tab bar overflow | Tab components with too many tabs for mobile without scroll or dropdown | MEDIUM |
| 4.8 | Safe area handling | Mobile PWA/native without `env(safe-area-inset-*)` for notch/home indicator | MEDIUM |
| 4.9 | Bottom sheet missing | Forms, action menus, or confirmations that should be bottom sheets on mobile but are centered modals | MEDIUM |
| 4.10 | Navigation depth | More than 3 levels of nested navigation without a clear back/breadcrumb pattern | LOW |
| 4.11 | Sticky header too tall | Sticky/fixed headers taller than 60px on mobile — eats precious vertical space | MEDIUM |
| 4.12 | No scroll-to-top | Long pages (>3 viewport heights of content) without a scroll-to-top affordance | LOW |
| 4.13 | Scroll position lost | No scroll restoration on back navigation — user loses their place | MEDIUM |
| 4.14 | Infinite scroll without escape | Infinite scroll with no way to reach the footer or "end" — user feels trapped | LOW |
| 4.15 | Pull-to-refresh missing | Mobile app/PWA lists without pull-to-refresh gesture (expected native behavior) | LOW |

**Smart adaptation recommendations:**
- Desktop sidebar → Hamburger menu or bottom tab bar on mobile
- Centered modal → Full-screen or bottom sheet on mobile
- Horizontal tabs (6+) → Scrollable tabs or dropdown selector on mobile
- Breadcrumbs → Back button + current page title on mobile
- Multi-level dropdown → Drill-down navigation on mobile
- Fixed header + fixed footer → Consider combining into one bar on mobile (save vertical space)

---

### Agent 3: Typography & Readability + Images & Media

#### Domain 5: Typography & Readability

**Objective:** Text must be readable, well-proportioned, and fluid across all viewports.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 5.1 | Text too small on mobile | Body text below 16px effective size at mobile viewport. Check for `text-xs` (12px), `text-sm` (14px) used for body content (not labels/captions) | HIGH |
| 5.2 | No fluid typography | Headings with fixed sizes at all viewports — should scale with `clamp()` or responsive classes (`text-xl md:text-3xl`) | MEDIUM |
| 5.3 | Line length too long | Text containers wider than 75 characters (~650px) without `max-w-prose` or similar constraint | MEDIUM |
| 5.4 | Line height mismatch | Large text with default line-height (too tight) or small text with excessive line-height | LOW |
| 5.5 | Heading hierarchy broken | `h3` before `h1`, or skipped heading levels on a page | LOW |
| 5.6 | Text truncation hiding meaning | `truncate` or `line-clamp-1` on important content without a way to see the full text (tooltip, expand) | MEDIUM |
| 5.7 | Font weight too thin | `font-light` (300) or `font-thin` (100) used for body text — hard to read on mobile screens | LOW |
| 5.8 | Contrast issues | Text color too close to background color. Check for gray text on gray backgrounds. Calculate contrast ratio if possible. | HIGH |

**Tailwind fluid typography check:**
```bash
# Find headings without responsive sizing
grep -rn '<h[1-3]\|text-[2-4]xl\|text-[5-9]xl' --include="*.tsx" --include="*.jsx" | \
  grep -v "md:\|lg:\|sm:\|clamp" | grep -v node_modules
```

**Smart adaptation recommendations:**
- Fixed heading sizes → `text-2xl md:text-4xl lg:text-5xl`
- Long text blocks → `max-w-prose` (65ch) or `max-w-2xl`
- Tiny labels on mobile → bump to `text-sm` minimum
- Recommend `clamp()` for truly fluid scaling: `font-size: clamp(1.25rem, 1rem + 1.5vw, 2.5rem)`

#### Domain 6: Images & Media

**Objective:** Images must be optimized, responsive, and not cause layout shift.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 6.1 | Missing `sizes` prop | `<Image>` with `fill` but no `sizes` — browser assumes 100vw, downloads oversized images | HIGH |
| 6.2 | Missing `priority` on LCP | Hero/above-fold images without `priority` — delays Largest Contentful Paint | HIGH |
| 6.3 | Raw `<img>` instead of framework image | `<img>` tags that should use `next/image` or equivalent for optimization | MEDIUM |
| 6.4 | Missing aspect ratio | Images without explicit aspect ratio — causes CLS on load | MEDIUM |
| 6.5 | Wildcard remote patterns | Image config allows `*` for remote image sources (security concern) | LOW |
| 6.6 | SVG accessibility | Decorative SVGs without `aria-hidden="true"`, or meaningful SVGs without `aria-label` | LOW |
| 6.7 | Video/media responsiveness | Videos with fixed dimensions that overflow on mobile | MEDIUM |
| 6.8 | Background images | CSS background images without responsive alternatives for mobile (large image on small screen) | LOW |
| 6.9 | Charts not responsive | Chart/graph components with fixed dimensions that don't adapt to viewport. Check for hardcoded width/height on canvas, svg, or chart containers | MEDIUM |
| 6.10 | Chart colors not accessible | Chart palettes using red/green together (color-blind unfriendly). Check for distinct shapes/patterns in addition to color | MEDIUM |
| 6.11 | Chart labels unreadable | Data labels with small fixed font sizes that become illegible on mobile. Chart legends that overflow container | LOW |
| 6.12 | No chart mobile alternative | Complex charts (multi-axis, scatter plots) shown identically on mobile without simplified mobile version or summary card | LOW |

**Next.js specific:**
```bash
# Find <Image> with fill but no sizes
grep -rn '<Image' --include="*.tsx" --include="*.jsx" | grep "fill" | grep -v "sizes" | grep -v node_modules
# Find <img> that should be <Image>
grep -rn '<img ' --include="*.tsx" --include="*.jsx" | grep -v node_modules | grep -v "svg"
```

---

### Agent 4: Accessibility/Responsive + Design Consistency

#### Domain 7: Accessibility/Responsive Overlap

**Objective:** Responsive design must be accessible — zoom, reflow, reduced motion, focus management.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 7.1 | Zoom blocked | `<meta name="viewport">` with `maximum-scale=1` or `user-scalable=no` | CRITICAL |
| 7.2 | No reduced-motion support | Animations without `prefers-reduced-motion` media query or hook | MEDIUM |
| 7.3 | Content reflow at 320px | Elements with `min-width` > 320px that prevent reflow (WCAG 1.4.10) | HIGH |
| 7.4 | Focus visible missing | Interactive elements without visible focus indicator (`:focus-visible` outline) | HIGH |
| 7.5 | Skip-to-content missing | No skip navigation link for keyboard users | MEDIUM |
| 7.6 | Focus lost on viewport change | Elements that get removed when layout shifts at a breakpoint while focused | LOW |
| 7.7 | Reduced-motion not consolidated | Multiple scattered `prefers-reduced-motion` blocks instead of a single global override | LOW |
| 7.8 | ARIA on responsive elements | Elements hidden with CSS (`hidden md:block`) that are still in the accessibility tree without `aria-hidden` | MEDIUM |

```bash
# Check viewport meta
grep -rn "viewport" --include="*.tsx" --include="*.jsx" --include="*.html" | grep -i "maximum-scale=1\|user-scalable=no"
# Check for reduced motion
grep -rn "prefers-reduced-motion\|useReducedMotion" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
```

#### Domain 8: Design Consistency

**Objective:** The design system must be applied consistently across the entire app.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 8.1 | Inconsistent spacing | Mixed spacing scales — some components use `p-4 gap-4`, others use `p-3 gap-6`, `p-5 gap-2` without clear reasoning | MEDIUM |
| 8.2 | Inconsistent border radius | Mixed radius values — some cards `rounded-lg`, others `rounded-xl`, others `rounded-2xl` | LOW |
| 8.3 | Inconsistent shadow usage | Random shadow values across components instead of a defined shadow scale | LOW |
| 8.4 | Color system violations | Hardcoded hex/rgb colors instead of theme/design system tokens | MEDIUM |
| 8.5 | Inconsistent button patterns | Different button styles/sizes for the same action type across different pages | MEDIUM |
| 8.6 | Missing empty states | Pages/lists that show nothing when data is empty — should show helpful empty state | HIGH |
| 8.7 | Missing error states | Components that can fail but show no error feedback | HIGH |
| 8.8 | Missing loading states | Async operations without visual loading feedback (spinner, skeleton, progress) | HIGH |
| 8.9 | Inconsistent card patterns | Cards with different padding, header styles, or content structure across features | MEDIUM |
| 8.10 | Z-index chaos | Scattered z-index values without a defined scale — risk of stacking bugs | LOW |
| 8.11 | Pure white on pure black | `#ffffff` text directly on `#000000` background — causes halation (glowing edges), hard on eyes. Use off-white (`#f5f5f5`, `#e5e5e5`) or off-black (`#0a0a0a`, `#111`) | MEDIUM |
| 8.12 | Dark mode elevation wrong | Dark mode using shadows for elevation instead of lighter surface colors — shadows are invisible on dark backgrounds. Elevation should = lighter surface (Material Design principle) | MEDIUM |
| 8.13 | Oversaturated colors in dark mode | Vibrant/saturated colors on dark backgrounds glow harshly — should be slightly desaturated in dark mode. Check for `*-500` or `*-600` shades on dark surfaces without toning | LOW |
| 8.14 | Bright images on dark backgrounds | User-uploaded or hero images without dark overlay/scrim — bright images on dark UI feel like flashlights. Check for images without `brightness` filter or overlay | LOW |
| 8.15 | No system theme detection | App doesn't respect `prefers-color-scheme` OS preference (if app supports both modes). Check for `dark:` classes without a system-preference listener | LOW |

**Tailwind consistency check:**
```bash
# Find non-standard colors (hardcoded hex/rgb)
grep -rn '#[0-9a-fA-F]\{3,6\}\|rgb(' --include="*.tsx" --include="*.jsx" | grep "className\|style" | grep -v node_modules | grep -v ".css"
# Find z-index values
grep -rn 'z-\[.*\]\|z-index' --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules | sort
```

---

### Agent 5: Visual Polish & Experience Quality

#### Domain 9: White Space & Visual Hierarchy

**Objective:** The eye should flow naturally. The most important element should be the most prominent. Content should breathe.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 9.1 | Cramped layouts | Pages/cards with less than 16px padding on mobile or less than 24px on desktop — content feels suffocated | HIGH |
| 9.2 | No section separation | Long pages without clear visual breaks between content sections — wall of content. Check for missing spacing (`mb-8`, `mt-12`, `space-y-8`) between major sections | MEDIUM |
| 9.3 | Content-to-chrome ratio | Excessive UI decoration (borders, headers, badges, icons) relative to actual content. Pages where >40% of visible area is chrome/UI, not user content | MEDIUM |
| 9.4 | Competing focal points | Multiple elements on the same screen using the same visual weight (same size, same color, same boldness) — nothing stands out | HIGH |
| 9.5 | CTA not obvious | Primary action button not visually distinct from secondary actions. Check: is there exactly ONE clearly prominent action per screen/section? | HIGH |
| 9.6 | Affordance gap | Interactive elements that don't LOOK interactive (plain text that's actually a link, icons without button wrapper) or non-interactive elements that LOOK clickable (cards without `cursor-pointer` that aren't links) | MEDIUM |
| 9.7 | Visual weight hierarchy | Heading sizes/weights don't create clear hierarchy — `h1` and `h2` look too similar, or body text competes with headings | MEDIUM |
| 9.8 | Wall of text | Text blocks >200 words without visual breaks (subheadings, lists, images, callouts). Check for long `<p>` blocks or large `prose` containers without structure | LOW |
| 9.9 | Inconsistent section spacing | Different pages use wildly different section spacing — some use `space-y-4`, others `space-y-12`. Should follow a consistent rhythm | MEDIUM |

```bash
# Find pages with potentially cramped padding
grep -rn 'className="[^"]*\bp-[12]\b' --include="*.tsx" --include="*.jsx" | grep -v node_modules | grep -v "p-1[0-9]\|p-12"
# Find potential wall-of-text (large prose blocks)
grep -rn "prose\|max-w-prose" --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find links that look like plain text (no underline, no color change)
grep -rn '<a\|<Link' --include="*.tsx" --include="*.jsx" | grep -v "className\|button\|btn\|nav" | grep -v node_modules
```

**Smart adaptation recommendations:**
- Cramped mobile cards → increase padding from `p-3` to `p-4` on mobile
- Wall of text → break into sections with subheadings, use bullet lists
- Competing focal points → establish one primary CTA per viewport section
- Section spacing → define a rhythm (e.g., `space-y-6` within sections, `space-y-12` between sections)

---

#### Domain 10: Micro-interactions & State Feedback

**Objective:** Every user action must have immediate, clear visual feedback. Every interactive element needs all 5 states.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 10.1 | Missing hover state | Buttons/links/cards without `:hover` styling — feels dead on desktop | MEDIUM |
| 10.2 | Missing active/pressed state | Buttons without `:active` or `active:` styling — no tap feedback on mobile | HIGH |
| 10.3 | Missing disabled state | Interactive elements that can be disabled but have no `disabled:` styling — looks identical to enabled | HIGH |
| 10.4 | Missing focus state | Interactive elements without `focus:` or `focus-visible:` ring — invisible to keyboard users | HIGH |
| 10.5 | No form submission feedback | Forms that submit without any visual indicator (button stays the same, no spinner, no "Saving...") | HIGH |
| 10.6 | No delete/destructive confirmation | Delete/remove actions without confirmation step or undo option | MEDIUM |
| 10.7 | No success feedback | Actions that complete silently — no toast, no checkmark, no state change visible to user | MEDIUM |
| 10.8 | Transition timing wrong | Transitions <100ms (feels instant/jarring) or >500ms (feels sluggish). Sweet spot is 150-300ms | LOW |
| 10.9 | Linear easing | Transitions using `linear` or `ease` instead of `ease-out` (for exits) or `ease-in-out` (for state changes) — feels robotic | LOW |
| 10.10 | No toggle feedback | Toggle/switch components without animation between states — flips instantly instead of sliding | LOW |

```bash
# Find buttons without hover state
grep -rn '<button\|<Button\|EnhancedButton' --include="*.tsx" --include="*.jsx" | \
  grep -v "hover:\|onMouseEnter\|whileHover" | grep -v node_modules | head -20
# Find interactive elements without disabled styling
grep -rn 'disabled' --include="*.tsx" --include="*.jsx" | \
  grep -v "disabled:\|opacity\|cursor-not-allowed\|pointer-events-none" | grep -v node_modules
# Find transitions with wrong timing
grep -rn "duration-\[.*\]\|transition-duration\|duration-75\b\|duration-1000" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
```

**State coverage audit pattern:**
For every interactive component, verify it has ALL 5 states:
1. **Default** — resting appearance
2. **Hover** — `:hover` / `hover:` (desktop)
3. **Active/Pressed** — `:active` / `active:` (universal)
4. **Focus** — `:focus-visible` / `focus-visible:` (keyboard)
5. **Disabled** — `disabled:` / `[disabled]` (when applicable)

Flag components missing 2+ states as HIGH severity.

---

#### Domain 11: Emotional Design & Microcopy

**Objective:** The app should feel human, helpful, and alive — not like a cold spreadsheet. Words matter as much as pixels.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 11.1 | Generic empty states | Empty lists/pages showing just "No items" or nothing at all — should show illustration, helpful message, and action button | HIGH |
| 11.2 | Cold error messages | Error states showing technical codes (`Error 500`, `Unexpected error`) instead of human-friendly messages with next steps | HIGH |
| 11.3 | No success celebration | Important accomplishments (task complete, goal achieved, streak maintained) without any celebration moment (confetti, checkmark animation, congratulatory message) | MEDIUM |
| 11.4 | Generic button labels | Buttons saying "Submit", "OK", "Click here" instead of specific action verbs ("Save Changes", "Create Task", "Send Invite") | MEDIUM |
| 11.5 | Unhelpful placeholder text | Input placeholders saying "Enter text here" or "Type something" instead of meaningful examples ("e.g., Buy groceries", "Search tasks...") | LOW |
| 11.6 | No onboarding guidance | First-time experience with no guidance, tooltips, or progressive disclosure — user dropped into empty app cold | HIGH |
| 11.7 | Inconsistent tone/voice | Some pages formal ("Please input your credentials"), others casual ("What's up? Let's go!") — should be consistent | MEDIUM |
| 11.8 | Tooltip quality | Tooltips too vague ("Info"), too long (>2 sentences), or missing entirely on icon-only buttons | LOW |
| 11.9 | No undo option | Destructive actions (delete, archive, remove) without undo/restore capability — user anxiety | MEDIUM |
| 11.10 | Confirmation dialog quality | Confirmation dialogs with generic "Are you sure?" instead of specific consequence ("Delete 'Weekly Groceries'? This cannot be undone.") | MEDIUM |

```bash
# Find generic empty states
grep -rn "No items\|No data\|Nothing here\|No results\|Empty\|No .* found" --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find generic button labels
grep -rn '>Submit<\|>OK<\|>Click here<\|>Send<\|>Go<' --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find generic error messages
grep -rn "Something went wrong\|Unexpected error\|Error occurred\|Please try again" --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find generic placeholders
grep -rn 'placeholder="[^"]*\(Enter\|Type\|Input\)[^"]*"' --include="*.tsx" --include="*.jsx" | grep -v node_modules
# Find "Are you sure" patterns
grep -rn "Are you sure\|Do you want to" --include="*.tsx" --include="*.jsx" | grep -v node_modules
```

**Smart adaptation recommendations:**
- Generic empty state → illustration + specific message + primary action ("No tasks yet. Create your first task to get started.")
- Cold error → human message + specific guidance ("Couldn't save your changes. Check your connection and try again.")
- Generic "Submit" → specific verb matching the action ("Create Task", "Update Profile", "Send Message")
- "Are you sure?" → specific consequence ("Remove 'John' from your family? They'll lose access to shared tasks.")

---

#### Domain 12: Cognitive Load & Information Architecture

**Objective:** Reduce mental effort. Show the right amount of information at the right time. Follow proven cognitive science principles.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 12.1 | Too many choices (Hick's law) | Screens with >7 equally-weighted action buttons or >10 navigation items visible simultaneously — decision paralysis | HIGH |
| 12.2 | Information overload (Miller's law) | Lists/groups with >9 items without sub-grouping or pagination — exceeds working memory | MEDIUM |
| 12.3 | No progressive disclosure | Complex forms showing ALL fields at once instead of multi-step or accordion — overwhelming | HIGH |
| 12.4 | No Fitts's law compliance | Primary action buttons that are small or far from where the user's cursor/thumb naturally rests. On mobile, primary actions should be in thumb zone (bottom half of screen) | MEDIUM |
| 12.5 | Poor proximity grouping | Related controls separated by unrelated elements — form fields for the same concept scattered across the page | MEDIUM |
| 12.6 | No visual grouping | Groups of related items without clear visual boundary (card, section, heading, divider) — everything blends together | MEDIUM |
| 12.7 | Multi-step flow without progress | Multi-step wizards/flows without step indicator — user doesn't know where they are or how much is left | HIGH |
| 12.8 | Exit points unhandled | Multi-step flows without save/draft capability — user loses all progress if they leave | MEDIUM |
| 12.9 | Redundant information | Same data shown in multiple places on the same screen — wasted space and confusion about which is canonical | LOW |
| 12.10 | Too many notification types | Toasts + banners + badges + inline alerts all fighting for attention simultaneously | LOW |

```bash
# Find forms with many fields (potential for progressive disclosure)
grep -rn '<input\|<Input\|<select\|<Select\|<textarea\|<Textarea' --include="*.tsx" --include="*.jsx" | grep -v node_modules | \
  awk -F: '{print $1}' | sort | uniq -c | sort -rn | head -10
# Find navigation with many items
grep -rn 'navItems\|menuItems\|links\|tabs' --include="*.tsx" --include="*.jsx" | grep "\[" | grep -v node_modules
# Find pages without step indicators
grep -rn "step\|Step\|wizard\|Wizard\|onboard\|Onboard" --include="*.tsx" --include="*.jsx" | grep -v "progress\|indicator\|stepper" | grep -v node_modules
```

**Smart adaptation recommendations on mobile especially:**
- Forms with >5 fields → multi-step wizard with progress bar
- Navigation with >5 items → bottom tab bar (5 max) + "More" menu
- Dashboard with >6 widgets → prioritize top 3, collapse rest into "Show more"
- Settings page → grouped sections with accordion/collapsible pattern
- Long lists → virtual scrolling or "Load more" with clear count ("Showing 10 of 47")

---

#### Domain 13: Motion Design Quality

**Objective:** Animation should serve a purpose — guide attention, show spatial relationships, provide feedback. Never gratuitous, never janky.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 13.1 | Gratuitous animation | Elements that animate purely for decoration with no functional purpose — distracts rather than guides | MEDIUM |
| 13.2 | No spatial relationship | Page transitions or element additions with no directional cue — things appear from nowhere instead of sliding from where they originated | LOW |
| 13.3 | Unchoreo­graphed sequences | Multiple elements animating simultaneously with different timing/easing — feels chaotic. Elements should animate in staggered sequence | LOW |
| 13.4 | Animation blocks interaction | Animations that prevent user interaction while playing (long entrance animations, loading animations that block the UI) | HIGH |
| 13.5 | No shared element transitions | Navigating from list → detail page without any visual continuity — the thing you tapped should "become" the detail page | LOW |
| 13.6 | Jarring appear/disappear | Elements popping in/out without any transition (opacity, scale, slide) — feels broken | MEDIUM |
| 13.7 | Infinite animations without purpose | `animate-spin`, `animate-pulse`, `animate-bounce` running continuously when there's nothing loading — anxious/distracting | MEDIUM |
| 13.8 | Animation on scroll without throttle | Scroll-linked animations using `onScroll` event listener without `requestAnimationFrame` or IntersectionObserver — causes jank | HIGH |
| 13.9 | Exit animations missing | Elements that animate IN but just disappear on removal — asymmetric feels unpolished | LOW |
| 13.10 | Mobile animation too heavy | Complex animations (blur, multi-property, spring physics) running on mobile without simplification — causes frame drops | MEDIUM |

```bash
# Find infinite animations
grep -rn "animate-spin\|animate-pulse\|animate-bounce\|animation:.*infinite\|repeat: Infinity" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
# Find scroll event listeners (should use IntersectionObserver)
grep -rn "onScroll\|addEventListener.*scroll\|scroll.*event" --include="*.tsx" --include="*.jsx" | grep -v "IntersectionObserver\|requestAnimationFrame\|throttle\|debounce" | grep -v node_modules
# Find Framer Motion without exit animations
grep -rn "animate=" --include="*.tsx" --include="*.jsx" | grep -v "exit=\|AnimatePresence" | grep -v node_modules | head -10
# Find elements without transitions
grep -rn "hidden\|opacity-0\|scale-0" --include="*.tsx" --include="*.jsx" | grep -v "transition\|duration\|animate\|motion" | grep -v node_modules
```

**Quality benchmarks:**
- Entrance: 150-300ms, `ease-out`
- Exit: 100-200ms, `ease-in`
- State change: 200-300ms, `ease-in-out`
- Spring: stiffness 200-400, damping 20-30 (Framer Motion)
- Stagger: 30-60ms between sibling elements

---

#### Domain 14: Platform Conventions & Native Feel

**Objective:** The app should feel native to each platform. Web apps can feel just as polished as native when they respect platform conventions.

**Checks:**

| # | Check | How | Severity |
|---|-------|-----|----------|
| 14.1 | Not respecting system font size | App ignores user's OS font size preference — all text in fixed `px` instead of `rem` | HIGH |
| 14.2 | iOS safe area violations | Content rendered behind notch, dynamic island, or home indicator. Check for `env(safe-area-inset-*)` usage | HIGH |
| 14.3 | No iOS swipe-back support | SPA navigation that overrides browser swipe-back gesture without providing alternative | MEDIUM |
| 14.4 | Android back button broken | PWA/SPA that doesn't handle Android hardware back button — traps the user | MEDIUM |
| 14.5 | Status bar color mismatch | PWA `theme-color` doesn't match the app's header/background color — visible seam | LOW |
| 14.6 | Missing PWA manifest fields | Incomplete `manifest.json` — missing `icons`, `start_url`, `display`, `theme_color`, `background_color`, or `screenshots` | MEDIUM |
| 14.7 | Splash screen mismatch | PWA splash screen colors/background don't match the app's first painted screen — flash of wrong color | LOW |
| 14.8 | No native-like transitions | Page transitions feel web-like (full page reload flash) instead of native-like (smooth slide or fade) | MEDIUM |
| 14.9 | Missing haptic feedback | Mobile app (Capacitor/native wrapper) without haptic feedback on important interactions (toggle, delete, success) | LOW |
| 14.10 | Text selection where inappropriate | Users can select/highlight UI text (button labels, nav items) — native apps don't allow this. Check for missing `select-none` / `user-select: none` on UI elements | LOW |
| 14.11 | Tap highlight visible | Default browser tap highlight (blue/gray rectangle) showing on buttons/links on mobile — looks non-native. Check for `-webkit-tap-highlight-color: transparent` | LOW |
| 14.12 | RTL not considered | No `dir="auto"` or `dir` attribute on text containers — breaks for RTL languages (Arabic, Hebrew). Check if internationalization is a concern | LOW |
| 14.13 | No text expansion room | UI elements with fixed widths containing text — breaks when translated (German is 30% longer, CJK needs more height) | LOW |

```bash
# Check for px-based font sizes (should be rem)
grep -rn "font-size:.*px\|fontSize:.*px\|fontSize: [0-9]" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules | grep -v "tailwind\|config"
# Check for safe area usage
grep -rn "safe-area\|env(safe-area" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
# Check for select-none on UI elements
grep -rn "select-none\|user-select" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
# Check for tap highlight
grep -rn "tap-highlight\|-webkit-tap-highlight" --include="*.tsx" --include="*.jsx" --include="*.css" | grep -v node_modules
# Check manifest completeness
cat public/manifest.json 2>/dev/null | jq 'keys'
```

**Smart adaptation recommendations:**
- Fixed `px` fonts → `rem` everywhere (respects user's OS font size)
- Missing safe areas → add `pb-[env(safe-area-inset-bottom)]` to bottom nav, `pt-[env(safe-area-inset-top)]` to headers
- No page transitions → add `PageTransition` wrapper with slide/fade animation
- Tap highlight → add `-webkit-tap-highlight-color: transparent` to global CSS
- UI text selectable → add `select-none` to buttons, nav items, labels (not content text)

---

## Phase 3: Smart Adaptation Analysis

After all 14 domains are scanned, synthesize findings into **smart adaptation recommendations**. This is the creative phase — don't just list problems, recommend elegant solutions.

### 3.1 Per-Route Adaptation Plan

For each route that scored below 100% in the Viewport Matrix, create a specific adaptation plan:

```markdown
### /dashboard — Mobile Adaptation Plan

**Current state:** Desktop-only multi-column dashboard. 4-column grid of widgets, sidebar stats, data table.

**Recommended adaptations:**
| Element | Desktop (current) | Mobile (recommended) | Tablet (recommended) |
|---------|-------------------|----------------------|----------------------|
| Widget grid | 4 columns | Single column, scrollable | 2 columns |
| Stats sidebar | Right sidebar, always visible | Collapsible section above content | Left sidebar, collapsible |
| Data table (8 cols) | Full table | Card stack (show 3 key fields) | Horizontal scroll with frozen col 1 |
| Action buttons (row) | Inline text buttons | Bottom action bar with icons | Inline icon buttons |
| Filters | Horizontal filter bar | Bottom sheet filter panel | Horizontal, scrollable |
| Page header | Title + breadcrumb + actions | Title + back button, actions in menu | Full breadcrumb + actions |
```

### 3.2 Component-Level Recommendations

For components that appear across multiple routes, recommend once:

```markdown
### Modal → Bottom Sheet Pattern

**Affected components:** TaskModal, GoalModal, BudgetModal, SettingsModal

**Current:** All use centered overlay at all viewports
**Recommendation:**
- Mobile: Bottom sheet (slides up from bottom, swipe to dismiss, full-width, rounded top corners)
- Tablet: Centered overlay (max-width: 600px)
- Desktop: Centered overlay (max-width: 700px)

**Implementation:**
- Wrap modal content in responsive container
- For Tailwind: `fixed inset-x-0 bottom-0 md:inset-auto md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2`
- Add swipe-to-dismiss gesture handler on mobile (if Framer Motion available)
```

### 3.3 Generate Improvement Tasks

Convert all findings and recommendations into a prioritized task list:

```markdown
## Improvement Tasks

### Critical (fix before release)
- [ ] **DES-001**: Add mobile navigation — /dashboard has no mobile nav
- [ ] **DES-004**: Fix horizontal overflow on /settings at 375px
- [ ] **DES-012**: Viewport meta blocks zoom — remove `maximum-scale=1`

### High (fix within sprint)
- [ ] **DES-002**: Convert TaskModal to bottom sheet on mobile
- [ ] **DES-003**: Add responsive grid to dashboard — `grid-cols-1 md:grid-cols-2 lg:grid-cols-4`
- [ ] **DES-008**: Touch targets below 44px — add padding to icon buttons in table rows
- [ ] **DES-015**: Missing loading states on /tasks, /goals, /budget

### Medium (improve within cycle)
- [ ] **DES-005**: Add fluid typography to headings — use responsive classes
- [ ] **DES-007**: Data table → card stack on mobile for /tasks
- [ ] **DES-011**: Consistent card border radius — standardize to rounded-xl

### Low (backlog)
- [ ] **DES-009**: Container query migration for dashboard widgets
- [ ] **DES-010**: Consolidate reduced-motion overrides into single global block
- [ ] **DES-014**: Add empty states to /goals, /meals when list is empty
```

### 3.4 Update Report

Write all adaptation recommendations and improvement tasks to the report. Update Progress Log and Status.

---

## Phase 4: Auto-Fix Confirmed Issues

Immediately after analysis completes, start fixing safe issues. No pause, no user prompt.

### Finding Statuses

| Status | Meaning |
|--------|---------|
| `FOUND` | Issue identified, not yet addressed |
| `🔧 FIXING` | Currently being worked on |
| `✅ FIXED` | Fix applied, build verified |
| `⏸️ DEFERRED` | Too complex for auto-fix, needs manual design decision |
| `🚫 BLOCKED` | Fix broke the build, reverted |

### What Can Be Auto-Fixed (Safe)

| Category | Auto-Fix Action |
|----------|----------------|
| Missing responsive grid | Add `grid-cols-1` mobile fallback before existing `md:grid-cols-*` |
| Hardcoded widths | Replace `w-[500px]` with `w-full max-w-[500px]` |
| Missing `sizes` on Image | Add appropriate `sizes` prop based on layout context |
| Missing `priority` on hero | Add `priority` to above-fold images |
| Touch target too small | Add padding classes to increase tap area (e.g., `p-2` → `p-3`) |
| Missing `aria-hidden` on decorative SVGs | Add `aria-hidden="true"` |
| Missing focus-visible | Add `focus-visible:ring-2 focus-visible:ring-offset-2` |
| Orphaned responsive classes | Remove conflicting class combinations |
| Missing `overflow-x-auto` | Add to table containers that may overflow |
| Heading without responsive sizing | Add responsive text size classes |
| Missing empty states | Add basic empty state component with message and icon |
| Missing loading states | Add basic loading skeleton or spinner |
| Missing active/pressed states | Add `active:scale-95` or `active:opacity-80` to buttons |
| Linear easing | Replace `ease-linear` / `linear` with `ease-out` on transitions |
| Wrong transition timing | Replace `duration-75` with `duration-150`, `duration-1000` with `duration-300` |
| Tap highlight visible | Add `-webkit-tap-highlight-color: transparent` to global CSS |
| UI text selectable | Add `select-none` to buttons, nav items, labels |
| Cramped mobile padding | Bump `p-2` → `p-4` on page-level containers at mobile |
| Missing section spacing | Add `space-y-8` or `space-y-12` between major page sections |
| Jarring appear/disappear | Add `transition-opacity duration-200` to elements that toggle visibility |
| Infinite animation without loading | Remove `animate-pulse`/`animate-spin` from elements not in loading state |
| Generic button labels | Replace "Submit" → specific verb matching the action |

### What Gets Deferred (Needs Design Decision)

| Category | Why Deferred |
|----------|-------------|
| Modal → bottom sheet | Requires structural component change, new animation |
| Sidebar → mobile nav | Architectural change, needs navigation strategy decision |
| Table → card stack | Requires new mobile component, field selection |
| Complex layout rearrangement | Multiple interdependent changes |
| Design system inconsistency | Need to pick canonical pattern, then update everywhere |
| Container query migration | New CSS approach, needs testing |
| Empty state redesign | Needs illustrations, custom copy, action buttons per context |
| Onboarding flow | Requires product decision on first-time experience |
| Multi-step form → wizard | Structural change, needs step definition and progress component |
| Shared element transitions | Requires animation library support, component restructuring |
| Haptic feedback integration | Needs Capacitor/native bridge, per-action tuning |
| Dark mode elevation rework | Systematic change across all card/surface components |

### Fix → Verify → Update Report Cycle

For EACH safe finding, in severity order:

```
1. Update finding row Status → 🔧 FIXING
2. Update report header Status → 🟡 FIXING — DES-XXX [description]
3. Write report to disk
4. Apply fix to source code
5. Verify build: $PM run build (or type-check if faster)
6.   If build fails → revert fix, update row → 🚫 BLOCKED, move to next
7.   If build passes → update row → ✅ FIXED
8. Add entry to ## Fix Log table
9. Append to ## Progress Log
10. Write report to disk — checkpoint
11. Move to next finding
```

---

## Phase 5: Finalize Report

The report has been built incrementally. Now finalize:

1. **Add Executive Summary:**

```markdown
## Executive Summary

| Metric | Count |
|--------|-------|
| Routes Scanned | [X] |
| Components Analyzed | [X] |
| Viewports Checked | [X] (320, 375, 768, 1024, 1280, 1920) |
| Issues Found | [X] |
| ✅ Auto-Fixed | [X] |
| ⏸️ Deferred (needs manual) | [X] |
| 🚫 Blocked | [X] |

### Responsive Coverage Score

| Viewport | Routes Passing | Score |
|----------|----------------|-------|
| Mobile (< 768px) | [X]/[Y] | [Z]% |
| Tablet (768-1023px) | [X]/[Y] | [Z]% |
| Desktop (1024px+) | [X]/[Y] | [Z]% |
| **Overall** | | **[Z]%** |
```

2. **Update Verification section** with actual build results
3. **Set final status:** `🟢 COMPLETE`
4. **Add duration**
5. **Write to disk**

---

## Phase 6: Status Report (Display on Screen)

### All Clean:

```
═══════════════════════════════════════════════════════════════════
🎨 DESIGN AUDIT — [project-name]
═══════════════════════════════════════════════════════════════════
Stack: [Framework] + [CSS System] + [Component Library]

📊 AUDIT RESULTS
├─ Routes Scanned:        [X]
├─ Components Analyzed:   [X]
├─ Issues Found:          0
├─ Responsive Coverage:   100%
└─ Design Consistency:    ✅ Uniform

───────────────────────────────────────────────────────────────────
Result: ✅ IMMACULATE — Every screen looks great on every device
Report: .design-reports/design-[timestamp].md
───────────────────────────────────────────────────────────────────
💡 Next: /gh-ship — design is polished, ship it
═══════════════════════════════════════════════════════════════════
```

### Issues Found & Addressed:

```
═══════════════════════════════════════════════════════════════════
🎨 DESIGN AUDIT — [project-name]
═══════════════════════════════════════════════════════════════════
Stack: [Framework] + [CSS System] + [Component Library]

📊 AUDIT RESULTS
├─ Routes Scanned:        24
├─ Components Analyzed:   68
├─ Issues Found:          18
├─ ✅ Auto-Fixed:         11
├─ ⏸️ Deferred:           6
├─ 🚫 Blocked:           1
├─ Responsive Coverage:   78% → 94% (after fixes)
└─ Design Consistency:    ⚠️ 3 patterns need unification

───────────────────────────────────────────────────────────────────
✅ Fixed:
  DES-001 Added mobile grid fallback to /dashboard
  DES-003 Fixed horizontal overflow on /settings
  DES-005 Added responsive heading sizes (6 files)
  DES-008 Increased touch targets on table action buttons
  ... (7 more)

⏸️ Deferred (needs manual design work):
  DES-002 Convert modals to bottom sheets on mobile
  DES-007 Transform data tables to card stacks on mobile
  DES-012 Implement mobile bottom tab navigation
  ... (3 more)

🚫 Blocked:
  DES-015 Empty states broke page layout — reverted
───────────────────────────────────────────────────────────────────
Result: ✅ 11 FIXED | ⏸️ 6 DEFERRED | 🚫 1 BLOCKED — [duration]
Report: .design-reports/design-[timestamp].md
───────────────────────────────────────────────────────────────────
💡 Next: Address deferred items manually, then /design again
═══════════════════════════════════════════════════════════════════
```

---

## Phase 7: Suggest Next Action

| Result | Suggestion |
|--------|------------|
| No issues found | `/gh-ship` — design is polished, ship it |
| All issues auto-fixed | `/gh-ship` — commit fixes and push |
| DEFERRED items remain (CRITICAL/HIGH) | Address manually, then `/design` again |
| DEFERRED items remain (MED/LOW) | `/gh-ship` — ship with known design debt noted |
| Multiple design system inconsistencies | Consider creating a design tokens file, then `/design` again |

---

## Framework-Specific Reference

### Tailwind CSS Responsive Patterns

```tsx
// Mobile-first grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">

// Show/hide by viewport
<nav className="hidden md:flex">Desktop nav</nav>
<nav className="flex md:hidden">Mobile nav</nav>

// Responsive text
<h1 className="text-2xl md:text-4xl lg:text-5xl">

// Responsive spacing
<div className="p-4 md:p-6 lg:p-8">

// Modal → bottom sheet
<div className="fixed inset-x-0 bottom-0 md:inset-auto md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 md:max-w-lg">

// Container query (Tailwind v4+)
<div className="@container">
  <div className="flex flex-col @md:flex-row @md:items-center">
```

### CSS Module Responsive Patterns

```css
/* Mobile-first */
.grid { display: grid; grid-template-columns: 1fr; gap: 1rem; }
@media (min-width: 768px) { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (min-width: 1024px) { .grid { grid-template-columns: repeat(4, 1fr); } }

/* Show/hide */
.desktopNav { display: none; }
@media (min-width: 768px) { .desktopNav { display: flex; } }
.mobileNav { display: flex; }
@media (min-width: 768px) { .mobileNav { display: none; } }
```

### Modal → Bottom Sheet Pattern

```tsx
// Framework-agnostic responsive modal
function ResponsiveModal({ children, onClose }) {
  const isMobile = useMediaQuery('(max-width: 767px)');

  if (isMobile) {
    return (
      <BottomSheet onClose={onClose} snapPoints={[0.9, 0.5]}>
        {children}
      </BottomSheet>
    );
  }

  return (
    <CenteredOverlay onClose={onClose} maxWidth="600px">
      {children}
    </CenteredOverlay>
  );
}
```

### Table → Card Stack Pattern

```tsx
// Desktop: table | Mobile: card stack
function ResponsiveDataDisplay({ data, columns }) {
  return (
    <>
      {/* Desktop table */}
      <table className="hidden md:table w-full">
        <thead>
          <tr>{columns.map(col => <th key={col.key}>{col.label}</th>)}</tr>
        </thead>
        <tbody>
          {data.map(row => (
            <tr key={row.id}>
              {columns.map(col => <td key={col.key}>{row[col.key]}</td>)}
            </tr>
          ))}
        </tbody>
      </table>

      {/* Mobile cards */}
      <div className="flex flex-col gap-3 md:hidden">
        {data.map(row => (
          <div key={row.id} className="rounded-xl border p-4">
            {columns.slice(0, 3).map(col => (
              <div key={col.key} className="flex justify-between py-1">
                <span className="text-sm text-gray-400">{col.label}</span>
                <span className="text-sm font-medium">{row[col.key]}</span>
              </div>
            ))}
          </div>
        ))}
      </div>
    </>
  );
}
```

---

## What This Catches That Manual Review Misses

| Scenario | Manual Review | /design |
|----------|--------------|---------|
| Dashboard looks fine on your 1440px monitor | Sees desktop, assumes mobile is ok | Flags: no responsive grid, no mobile nav, table overflows |
| Modal works in Chrome dev tools | Checks 1-2 viewports quickly | Checks every modal at 6 viewports systematically |
| "I made it responsive" | Trusts the developer's word | Verifies: counts responsive classes, scores coverage per route |
| Touch targets "look fine" | Eyeballs it on a phone | Calculates: finds every element below 44px |
| Typography "seems readable" | Reads it once on desktop | Checks: fluid scaling, line length, contrast ratios, heading hierarchy |
| Empty states exist "somewhere" | Remembers adding a few | Scans every list/grid component for empty state handling |
| Images are optimized "probably" | Doesn't check every Image tag | Finds every `<Image>` without `sizes`, every `<img>` that should be `<Image>` |
| Buttons "have hover states" | Spot-checks a few buttons | Audits ALL 5 states (default, hover, active, focus, disabled) on every interactive element |
| "The spacing looks good" | Eyeballs once | Measures: detects inconsistent spacing scales, cramped layouts, missing section breaks |
| Animations "feel smooth" | Watches once on desktop | Checks: timing curves, exit animations, reduced-motion, scroll perf, mobile frame drops |
| Error messages work | Tests happy path | Flags: generic "Something went wrong", cold errors, missing undo, generic confirmations |
| "Users will figure it out" | Assumes intuitive | Checks: Hick's law (too many choices), Miller's law (overloaded lists), progressive disclosure gaps |
| App feels "native enough" | Uses it on own phone | Flags: missing safe areas, no haptic feedback, tap highlights, selectable UI text, status bar mismatches |
| Dark mode "looks fine" | Glances at a few pages | Detects: pure white on black halation, shadow-based elevation, oversaturated colors, bright images without scrim |

---

## Pipeline Position

```
/design → /a11y → /perf → /sec-ship → /redteam → /gh-ship
  Visual    A11y    Perf   Security    Active     Ship
  polish   audit   audit   analysis   exploit
```

- **/design**: "Does it look and feel great on every device?" (20-35 min)
- **/a11y**: "Is it accessible to everyone?" (15-25 min)
- **/perf**: "Is it fast?" (10-20 min)
- **/sec-ship**: "Is it secure?" (15-30 min)
- **/redteam**: "Can I break it?" (20-45 min)
- **/gh-ship**: "Ship it" (commit, push, PR, CI)

---

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### Design-Specific Cleanup

Resources this skill may create:
- Playwright browser instances (extensive screenshot sessions across viewports)
- Screenshots saved to `.design-reports/screenshots/`

Cleanup actions (run after final analysis, before report generation):
1. **Close all browser instances:** Call `browser_close` for every open Playwright session. This is the #1 cleanup priority — design audits open many pages across 3+ viewports
2. **Screenshots:** Keep screenshots in `.design-reports/screenshots/` (intended output for the report). Delete any screenshots in default Playwright output locations
3. **Verify no orphaned Chromium processes remain**
4. **Verify port used for dev server (if any) is released**
5. **Gitignore enforcement:** Ensure `.design-reports/` is in `.gitignore`
6. **Log cleanup results in the report**

Cleanup verification:
- `pgrep -f "chromium|playwright"` should match pre-skill baseline
- `browser_close` called explicitly (do NOT rely on process timeout)

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
