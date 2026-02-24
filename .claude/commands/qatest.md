---
description: "/qatest — Autonomous QA/UAT: crawl all pages, test all interactions, validate all APIs, autofix issues, ship with confidence"
allowed-tools: Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Bash(bun *), Bash(curl *), Bash(mkdir *), Bash(date *), Bash(ls *), Bash(cat *), Bash(lsof *), Bash(kill *), Bash(git *), Bash(find *), Bash(grep *), Bash(node *), Read, Write, Edit, Glob, Grep, Task, WebSearch, WebFetch, mcp__playwright__browser_navigate, mcp__playwright__browser_screenshot, mcp__playwright__browser_click, mcp__playwright__browser_type, mcp__playwright__browser_hover, mcp__playwright__browser_select_option, mcp__playwright__browser_evaluate, mcp__playwright__browser_close, mcp__playwright__browser_wait_for, mcp__playwright__browser_go_back, mcp__playwright__browser_go_forward, mcp__playwright__browser_press_key, mcp__playwright__browser_drag, mcp__playwright__browser_resize, mcp__playwright__browser_snapshot, mcp__playwright__browser_tab_list, mcp__playwright__browser_tab_new, mcp__playwright__browser_tab_select, mcp__playwright__browser_tab_close
---

# /qatest — Autonomous QA & UAT Engine

**The penultimate gate before shipping.** This skill acts as a senior QA tester — crawling every page, clicking every button, submitting every form, hitting every API endpoint, and validating that your web app works exactly as a real user expects. What can be auto-fixed gets fixed. What can't gets documented with full context for human decision.

This is NOT a unit test runner or a linter. This is **functional validation** — does the app actually work?

---

## Philosophy

1. **Test like a real user, think like an engineer.** Navigate the app as a user would, but analyze failures with engineering precision.
2. **Fix what's safe, defer what's not.** Missing `alt` text? Fix it. Broken business logic? Document it with root cause analysis.
3. **Full by default, incremental by choice.** The pre-ship gate is always a full scan. Incremental is for developer iteration.
4. **Leave the codebase better than you found it.** Every run should either fix issues or document exactly why they can't be fixed yet.
5. **Never break what works.** Every autofix is verified against the build. If a fix breaks the build, it's rolled back instantly.

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

### Skill-Specific Status Examples

```
🚀 /qatest Started
   Project: steelmotion
   Mode: full
   Base URL: http://localhost:3000

🔍 Route Discovery...
   ├─ Filesystem: 19 page routes found
   ├─ API: 2 endpoints found (POST /api/contact, POST /api/partnerships)
   ├─ Sitemap: 15 URLs confirmed
   └─ ✅ Discovery complete — 19 pages, 2 API routes

🧪 Page Health Scan... (19 pages)
   ├─ [1/19] / — 200 OK, 0 console errors
   ├─ [5/19] /services/ai-transformation — 200 OK, 0 console errors
   ├─ [10/19] /portfolio/creative — 200 OK, 1 console warning
   ├─ [19/19] /terms — 200 OK, 0 console errors
   └─ ✅ Page Health complete — 19/19 passed, 0 errors, 1 warning

🔧 Autofix Phase...
   ├─ QA-003: Added alt="" to decorative image — ✅ build passes
   ├─ QA-007: Added rel="noopener" to external link — ✅ build passes
   └─ ✅ Auto-fixed 2/2 safe issues

📊 QA Score: 94/100 (A)
   Pages: 19/19 healthy
   API: 2/2 passing
   A11y: 47/52 passing (5 deferred)
   Interactive: 38/38 elements working
```

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Sub-agents return < 500 tokens (full findings written to `.qatest-reports/`)
- State file: `.qatest-reports/state-YYYYMMDD-HHMMSS.json`
- Resume from checkpoint if context resets — skip completed phases
- Max 2 parallel scout agents; fix agents run sequentially
- Orchestrator NEVER reads full agent report files into context
- Checkpoint after every phase completion

---

## AGENT ORCHESTRATION

This skill follows the **[Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)**.

### Model Selection for This Skill

| Agent | Model | Task |
|-------|-------|------|
| Route & endpoint discovery | `sonnet` | Enumerate filesystem routes, parse configs, trace imports, read content dirs — needs code understanding |
| Page health scout | `sonnet` | Evaluate page load, console errors, hydration, DOM validation, SEO — needs judgment on severity |
| Static asset & header verifier | `sonnet` | Verify static assets serve correctly, validate security headers, test redirects — needs HTTP knowledge |
| Interactive testing scout | `sonnet` | Click every element, fill every form, test every widget — needs UI understanding |
| User journey tester | `sonnet` | Simulate complete user flows across multiple pages — needs app-level understanding |
| API route tester | `sonnet` | Craft valid/invalid payloads, test all methods, validate security — needs reasoning |
| Third-party integration checker | `sonnet` | Verify external service connectivity, env var completeness — needs integration knowledge |
| Accessibility scanner | `sonnet` | Run and interpret axe-core results, keyboard nav, screen reader simulation — needs a11y expertise |
| Responsive viewport scout | `sonnet` | Evaluate layout at 3 viewports, detect overflow/overlap, test mobile nav — needs visual reasoning |
| Performance baseline agent | `sonnet` | Measure page load times, CWV spot check, detect memory leaks — needs perf knowledge |
| Autofix agent | `sonnet` | Modify code safely, verify build after each fix — must write correct code |
| Report synthesizer | `sonnet` | Create coherent narrative from all findings across all phases — needs analytical writing |

### Agent Batching

| Pages in App | Pages Per Scout Agent | Max Parallel Scouts |
|-------------|----------------------|-------------------|
| < 20 pages | All in 1 agent | 1 |
| 20-50 pages | 10-15 per agent | 2 |
| 50-100 pages | 15-20 per agent | 2 |
| 100+ pages | 20-25 per agent | 2 |

---

## MODES

```bash
/qatest                    # Full comprehensive QA scan (default)
/qatest --changed          # Incremental — only test pages/routes affected by recent changes
/qatest --api-only         # Only test API routes
/qatest --pages-only       # Only test pages/UI (skip API testing)
/qatest --retest           # Re-test previously FAILED/DEFERRED items from last report
/qatest --no-fix           # Scan only, no autofix phase
```

### Mode Details

**Full (default):** Crawl every page, test every interactive element, hit every API endpoint, run accessibility, check responsive viewports. This is the pre-ship gate — always run this before deploying.

**Incremental (`--changed`):** Compare current branch against `main` (or base branch). Only test pages whose source files changed. Useful during development iteration. NEVER use this as the final pre-ship check.

**API Only (`--api-only`):** Skip browser testing entirely. Test all API routes with valid/invalid payloads, check response codes, headers, rate limiting, CORS. Useful after backend-only changes.

**Pages Only (`--pages-only`):** Skip API testing. Crawl and interact with all pages. Useful after frontend-only changes.

**Retest (`--retest`):** Read the last `.qatest-reports/` report, find all FAILED and DEFERRED items, re-test only those. Useful after manual fixes.

**No Fix (`--no-fix`):** Run all scanning phases but skip the autofix phase. Produces a report-only audit.

---

## CRITICAL RULES

### Rule 1: Framework Detection First
Before any testing, detect the project's framework, package manager, and dev server command. NEVER hard-code assumptions.

### Rule 2: Dev Server Must Be Running
The skill needs a running dev server to test against. If none is detected, attempt to start one. If it can't be started, abort with clear instructions.

### Rule 3: Never Modify Test Data in Production
All test data created during QA (test users, form submissions, etc.) must be clearly identifiable and cleaned up after testing. Use test prefixes like `qatest_` for any created data.

### Rule 4: Build Verification After Every Fix
After every autofix, run the project's build command. If the build breaks, immediately revert the fix and mark the finding as BLOCKED.

### Rule 5: Rate Limiter Awareness
Many apps have rate limiting. The skill must detect rate limiters and either: (a) use appropriate delays between requests, or (b) test rate limiting as a feature ("does it actually rate-limit?"), or (c) use a test bypass if one exists.

### Rule 6: No Destructive Operations on Real Data
When testing CRUD operations, the skill creates its own test data, validates operations, then cleans up. It NEVER modifies or deletes existing user data.

### Rule 7: Screenshot Everything
Take a screenshot of every page at desktop viewport. These go into `.qatest-reports/screenshots/` for visual reference.

### Rule 8: Console Is Sacred
Every `console.error` and unhandled exception is a finding. `console.warn` is logged but scored lower. `console.log` in production is noted as a code quality issue.

### Rule 9: Global Skill — No Project Assumptions
This skill works across any web project. It detects everything dynamically: framework, routes, API endpoints, auth system, dev server command. Never assume Next.js, React, or any specific technology.

### Rule 10: Report Is the Source of Truth
The timestamped markdown report in `.qatest-reports/` is the definitive record. It includes every finding, every fix, every deferral with reasoning, and the final QA score.

---

## EXECUTION PHASES

### Phase 0: Pre-Flight

**Purpose:** Detect project configuration, verify prerequisites, validate environment, check for previous runs.

**Steps:**

1. **Detect project metadata:**
   ```
   - Framework: Check package.json for next, nuxt, remix, svelte, astro, vite, etc.
   - Package manager: Check for pnpm-lock.yaml, yarn.lock, package-lock.json, bun.lockb
   - Dev command: Read scripts from package.json (dev, start, serve)
   - Build command: Read scripts from package.json (build)
   - Test command: Read scripts from package.json (test, test:run)
   - TypeScript: Check for tsconfig.json
   - Base URL: Check for $BASE_URL env var, default to http://localhost:3000
   - Auth system: Check for next-auth, @supabase/auth, clerk, lucia, etc.
   - Database: Check for prisma, drizzle, supabase, mongoose, etc.
   - Email service: Check for resend, sendgrid, nodemailer, etc.
   - Rate limiting: Check for @upstash/ratelimit, express-rate-limit, etc.
   ```

2. **Verify prerequisites:**
   ```
   - Dependencies installed? (check node_modules exists, run install if not)
   - Build passes? (run build command — if build is broken, ABORT)
   - Git state clean? (warn if uncommitted changes)
   - TypeScript compiles? (run typecheck if available — catch type errors early)
   ```

3. **Environment variable validation:**
   ```
   - Scan all source files for process.env.* and import.meta.env.* references
   - Check .env.local, .env, .env.development for defined variables
   - Cross-reference: Are all referenced env vars actually defined?
   - Flag any missing required env vars as CRITICAL (they cause runtime failures)
   - Check .env.example if it exists — compare against actual .env files
   - NEVER log or report actual env var VALUES (security risk) — only report NAMES
   ```

4. **Check for dev server:**
   ```
   - Is something running on the expected port? (lsof -i :3000)
   - If yes: use it
   - If no: start the dev server in background, wait for it to be ready
   - Verify the server responds with HTTP 200 on the base URL
   - Record server start time for performance baseline
   ```

5. **Check for previous runs:**
   ```
   - Look for .qatest-reports/state-*.json from the last 2 hours
   - If found and incomplete: offer to resume
   - Check last 3 completed reports for historical context
   ```

6. **Check cross-skill reports:**
   ```
   - .security-reports/ — note any recent security findings
   - .a11y-reports/ — note any recent accessibility findings
   - .perf-reports/ — note any recent performance findings
   - .test-reports/ — note any recent test failures
   - .cleancode-reports/ — note any dead code or quality issues
   ```

7. **Detect app capabilities (for targeted testing):**
   ```
   - Has authentication? → Enable auth flow testing
   - Has forms? → Enable form submission testing
   - Has dynamic routes? → Enable content enumeration
   - Has middleware? → Enable middleware/header testing
   - Has redirects (next.config)? → Enable redirect testing
   - Has cookie consent? → Enable cookie banner testing
   - Has dark mode/themes? → Enable theme testing
   - Has search? → Enable search testing
   - Has pagination? → Enable pagination testing
   - Has i18n/localization? → Enable locale testing
   - Has webhooks? → Enable webhook testing
   - Has WebSocket/real-time? → Enable real-time testing
   ```

8. **Create report infrastructure:**
   ```
   - mkdir -p .qatest-reports/screenshots .qatest-reports/evidence
   - Create state file: .qatest-reports/state-YYYYMMDD-HHMMSS.json
   - Create report skeleton: .qatest-reports/qatest-YYYYMMDD-HHMMSS.md
   - Ensure .qatest-reports/ is in .gitignore (add if missing)
   ```

9. **Parse mode flags:**
   ```
   - --changed: Identify changed files vs main branch, map to affected routes
   - --api-only: Skip browser phases (page health, interactive, responsive)
   - --pages-only: Skip phase 4 (API testing)
   - --retest: Load last report, extract FAILED/DEFERRED items, test only those
   - --no-fix: Set autofix=false, skip phase 8
   ```

**Output:** Project configuration object, capability flags, list of routes to test, state file initialized.

---

### Phase 1: Route & Endpoint Discovery

**Purpose:** Build a COMPLETE map of every testable surface in the app. Every page, every API route, every static asset, every redirect, every dynamic slug — nothing is left undiscovered.

**Agent:** `sonnet` — Route Discovery Agent (upgraded from haiku — needs to trace imports, read content directories, parse configs)

**Steps:**

1. **Filesystem route discovery (EXHAUSTIVE):**
   - For Next.js App Router: Glob `app/**/page.{tsx,jsx,ts,js}`, extract route paths
   - For Next.js Pages Router: Glob `pages/**/*.{tsx,jsx,ts,js}`, exclude `_app`, `_document`
   - For other frameworks: Adapt based on detected framework conventions
   - Handle dynamic routes: `[slug]` → note as parameterized, enumerate ALL available slugs (see step 4)
   - Handle route groups: `(group)` → strip from URL path
   - Handle catch-all routes: `[...slug]` → test with sample paths AND edge cases
   - Handle parallel routes: `@modal`, `@sidebar` → note layout compositions
   - Handle intercepting routes: `(.)photo`, `(..)modal` → note intercept patterns
   - Also discover: `not-found.tsx`, `error.tsx`, `loading.tsx`, `global-error.tsx` — these are testable pages too
   - Also discover: `layout.tsx` files — note which layouts wrap which routes (for shared component tracking)
   - Also discover: `template.tsx` files — re-render behavior on navigation

2. **API endpoint discovery (EXHAUSTIVE):**
   - For Next.js: Glob `app/api/**/route.{ts,js}`, extract paths and exported HTTP methods
   - For other frameworks: Adapt to framework conventions
   - Parse each route file to determine: ALL exported methods (GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS)
   - Read the file to identify:
     - Zod schemas / validation rules (for payload generation)
     - Rate limiter configuration
     - Auth requirements (middleware, session checks)
     - Request size limits
     - Response types (JSON, redirect, stream)
   - Also discover: server actions (Next.js `"use server"` functions) — these are API-like and testable
   - Also discover: middleware.ts — global request processing

3. **Static asset discovery:**
   - Check `/public/` directory for all static files
   - Expected assets to validate:
     - `/robots.txt` — must be accessible and well-formed
     - `/sitemap.xml` — must be accessible and list all public routes
     - `/manifest.webmanifest` or `/manifest.json` — PWA manifest
     - `/favicon.ico` and `/icon.png` variants — must resolve
     - `/apple-touch-icon.png` — if referenced in layout
     - `/opengraph-image.*` or OG images — must be valid images
     - Any other public assets referenced in HTML
   - Verify each asset returns 200 and correct Content-Type

4. **Dynamic content discovery (ALL slugs, not a sample):**
   - Check for MDX/content directories: `content/`, `posts/`, `blog/`, `data/`, `.velite/`
   - For Velite: Read `.velite/` output to enumerate all content slugs
   - For MDX: Glob `content/**/*.mdx` and extract slugs from filenames/frontmatter
   - For CMS-driven content: Check for `generateStaticParams()` in dynamic route files — this tells you all valid slugs
   - For database-driven content: Note that dynamic slugs come from DB (test with sitemap URLs)
   - **TEST EVERY AVAILABLE SLUG** — not 2-3, ALL of them. Each slug might render different content with different components, images, and layouts
   - Also generate INVALID slugs to test 404 behavior: `/articles/this-does-not-exist-12345`

5. **Redirect discovery:**
   - Parse `next.config.ts` / `next.config.js` for `redirects()` configuration
   - Parse middleware.ts for `NextResponse.redirect()` calls
   - Also check for meta refresh tags and JavaScript-based redirects
   - Build redirect test map: `{ from: "/old-path", to: "/new-path", status: 301|302|307|308 }`
   - Every configured redirect must be tested for correct destination AND status code

6. **Middleware & header discovery:**
   - Read `middleware.ts` to identify:
     - CSP (Content Security Policy) headers being set
     - CORS configuration
     - Bot blocking rules
     - Cache-control headers
     - Custom header additions
     - Request rewriting rules
     - Geolocation or edge-based routing
   - Read `next.config.ts` for `headers()` configuration
   - Build a header expectation map per route pattern

7. **Sitemap cross-reference:**
   - Fetch `/sitemap.xml` from running server
   - Parse ALL URLs in the sitemap
   - Compare against filesystem routes:
     - Routes in sitemap but NOT in filesystem → FINDING: orphaned sitemap URL
     - Routes in filesystem but NOT in sitemap → FINDING: missing from sitemap (SEO issue)
     - Dynamic routes: verify all slugs are in sitemap
   - Verify `lastmod` dates are reasonable (not in the future, not ancient)

8. **Third-party integration discovery:**
   - Scan for external service connections:
     - Supabase: Check for `SUPABASE_URL`, `SUPABASE_ANON_KEY` env vars
     - Resend: Check for `RESEND_API_KEY`
     - Stripe: Check for `STRIPE_SECRET_KEY`, webhook endpoints
     - Redis/Upstash: Check for `UPSTASH_REDIS_REST_URL`
     - Analytics: Check for GA, Vercel Analytics, PostHog, etc.
   - Note which services are required for app functionality vs optional

9. **Build route manifest:**
   ```json
   {
     "pages": [
       { "path": "/", "source": "app/page.tsx", "dynamic": false, "layout": "app/layout.tsx" },
       { "path": "/about", "source": "app/about/page.tsx", "dynamic": false },
       { "path": "/articles/[slug]", "source": "app/articles/[slug]/page.tsx", "dynamic": true, "testSlugs": ["all", "available", "slugs", "listed"] }
     ],
     "api": [
       { "path": "/api/contact", "source": "app/api/contact/route.ts", "methods": ["POST"], "hasRateLimiter": true, "hasValidation": true }
     ],
     "staticAssets": ["/robots.txt", "/sitemap.xml", "/manifest.webmanifest", "/favicon.ico", "/opengraph-image.png"],
     "redirects": [
       { "from": "/old", "to": "/new", "status": 301 }
     ],
     "errorPages": ["/not-found", "/global-error"],
     "middleware": { "hasCSP": true, "hasBotBlocking": true, "hasCaching": true },
     "thirdParty": ["supabase", "resend", "upstash"],
     "totalTestableURLs": 42
   }
   ```

10. **Write manifest to disk:** `.qatest-reports/route-manifest.json`

**Output to orchestrator:** Route count summary (pages, API, static assets, redirects, dynamic slugs), manifest file path.

---

### Phase 2: Page Health & Infrastructure Scan

**Purpose:** Visit EVERY page (including every dynamic slug, error pages, and static assets), verify it loads correctly, capture console output, validate infrastructure, take screenshots.

**Agent:** `sonnet` — Page Health Scout (one agent per batch of pages)

**IMPORTANT: "Every page" means EVERY page. Not a sample. Not critical routes only. EVERY route in the manifest, including every dynamic slug, every static asset, every redirect, every error page.**

#### 2A. Page-by-Page Health Check

**For each page in the route manifest:**

1. **Deep-link entry test (COLD ENTRY):**
   - Navigate DIRECTLY to the page URL (not from homepage) — this tests SSR, data fetching, and layout hydration when the page is the entry point
   - Many bugs only appear on cold entry (missing context, failed data fetches, layout shifts)

2. **Wait for full load:**
   - Use `browser_wait_for` for network idle
   - Set a generous timeout (15 seconds for SSR pages, 30 seconds for pages with external data)
   - If timeout: FINDING (severity: HIGH — page is too slow or hanging)

3. **HTTP Status:** Verify correct status code (200 for pages, 301/302 for redirects, 404 for invalid routes)

4. **Hydration error detection (SSR/Next.js specific):**
   - Use `browser_evaluate` to check for:
     - React hydration mismatch warnings in console
     - `Warning: Text content did not match` messages
     - `Warning: Expected server HTML to contain` messages
     - `Hydration failed because` messages
   - These are CRITICAL findings — they mean the server and client render differently

5. **Console capture (COMPLETE):**
   - Install console interceptor via `browser_evaluate` BEFORE navigation:
     ```javascript
     window.__qatest_console = { errors: [], warnings: [], logs: [] };
     const origError = console.error;
     console.error = (...args) => { window.__qatest_console.errors.push(args.join(' ')); origError(...args); };
     // Same for warn and log
     ```
   - After page load, collect ALL console output:
     - `console.error` → FINDING (severity: HIGH)
     - Unhandled promise rejections → FINDING (severity: HIGH)
     - Unhandled exceptions → FINDING (severity: CRITICAL)
     - `console.warn` → FINDING (severity: LOW)
     - `console.log` in production → FINDING (severity: INFO, code quality)
     - React development-mode warnings → FINDING (severity: MEDIUM — means prod build isn't optimized)

6. **Missing resources (EXHAUSTIVE):**
   - Check for ANY failed network request via `browser_evaluate` with PerformanceObserver:
     - 404 images → FINDING (severity: HIGH)
     - 404 scripts → FINDING (severity: CRITICAL — broken functionality)
     - 404 stylesheets → FINDING (severity: HIGH — broken layout)
     - 404 fonts → FINDING (severity: MEDIUM — FOUT/fallback rendering)
     - 404 API calls → FINDING (severity: HIGH — missing data)
     - Mixed content (HTTP on HTTPS page) → FINDING (severity: HIGH — security)

7. **Screenshot:** Use `browser_screenshot` → save to `.qatest-reports/screenshots/{route-slug}.png`

8. **Comprehensive DOM validation:**
   - **SEO fundamentals:**
     - `<title>` tag exists, non-empty, unique across pages, reasonable length (30-60 chars)
     - `<meta name="description">` exists, non-empty, unique across pages, reasonable length (120-160 chars)
     - `<html lang="...">` attribute exists and is valid
     - Canonical URL: `<link rel="canonical">` exists and points to correct URL
     - No duplicate `<title>` or `<meta name="description">` tags
   - **Structured data (JSON-LD):**
     - Check for `<script type="application/ld+json">` tags
     - Validate JSON is parseable
     - Check for required fields based on schema type (Organization, WebSite, Article, etc.)
   - **Open Graph tags:**
     - `og:title`, `og:description`, `og:image`, `og:url` present
     - `og:image` URL actually resolves to a valid image
     - Twitter card tags: `twitter:card`, `twitter:title`, `twitter:description`
   - **Heading structure:**
     - Exactly one `<h1>` per page
     - Headings are sequential (h1 → h2 → h3, no skipping)
     - No empty headings
   - **Image validation:**
     - ALL images have `alt` attributes (empty `alt=""` is valid for decorative only)
     - Images actually load (no broken images)
     - Images have reasonable file sizes (warn if > 500KB unoptimized)
     - Next.js: Images use `<Image>` component (not raw `<img>`)
   - **Link validation:**
     - All `<a href="...">` links point to valid destinations
     - Internal links: match against route manifest
     - External links: verify `target="_blank"` has `rel="noopener noreferrer"`
     - Anchor links: verify target `#id` exists on the page
     - No empty `href=""` or `href="#"` links (except intentional scroll-to-top)
     - No `javascript:void(0)` links (accessibility anti-pattern)
   - **Placeholder content detection:**
     - Scan page text for: "Lorem ipsum", "TODO", "FIXME", "placeholder", "coming soon" (if suspicious), "test", "example.com" in visible content
     - Scan for placeholder images (1x1 pixels, base64 gray squares, picsum.photos, placeholder.com)
     - These indicate unfinished content → FINDING (severity: MEDIUM)

9. **Loading & empty state detection:**
   - After page load, check for:
     - Persistent loading spinners that never resolve (stuck loading state) → FINDING (severity: HIGH)
     - Skeleton screens that never fill with content → FINDING (severity: HIGH)
     - Empty containers that should have content (e.g., empty blog list, empty portfolio grid)
     - "No results" or empty state messages where content is expected
   - Note: Some empty states are valid (e.g., search results before query). Use page context to judge.

10. **Cookie & storage behavior:**
    - After page load, catalog:
      - All cookies set by the page (name, domain, httpOnly, secure, sameSite, expiry)
      - All localStorage keys written
      - All sessionStorage keys written
    - Flag: Cookies without `Secure` flag on HTTPS → FINDING (severity: MEDIUM)
    - Flag: Cookies without `HttpOnly` flag storing sensitive data → FINDING (severity: HIGH)
    - Flag: localStorage storing sensitive data (tokens, PII) → FINDING (severity: HIGH)

#### 2B. Static Asset Verification

**For each static asset in the manifest:**

1. **Fetch and verify:**
   - `robots.txt` → 200, valid directives, references sitemap
   - `sitemap.xml` → 200, valid XML, lists all public routes
   - `manifest.webmanifest` → 200, valid JSON, has required fields (name, icons, start_url)
   - `favicon.ico` → 200 or valid alternatives referenced in `<link rel="icon">`
   - OG images → 200, valid image format, reasonable dimensions (1200x630 recommended)
   - Any other public assets referenced in HTML

2. **Font loading verification:**
   - Check for custom font declarations in CSS
   - Verify font files load successfully (no 404)
   - Check for `font-display` CSS property (should be `swap` or `optional` for performance)
   - Detect FOUT (Flash of Unstyled Text) — if visible, note as INFO

#### 2C. Redirect Verification

**For each redirect in the manifest:**

1. Send request to the `from` URL
2. Verify it redirects to the correct `to` URL
3. Verify the status code matches expectation (301 permanent, 302 temporary, etc.)
4. Verify the final destination returns 200
5. Check for redirect chains (A → B → C) — flag chains > 2 hops as MEDIUM finding

#### 2D. Error Page Testing

1. **404 page:** Navigate to a non-existent URL (e.g., `/this-page-does-not-exist-qatest-12345`)
   - Verify: Custom 404 page renders (not a blank page or default Next.js error)
   - Verify: 404 page has navigation back to the main site
   - Verify: HTTP status is actually 404 (not 200 with error content)
   - Screenshot the 404 page

2. **Error boundary:** If `global-error.tsx` or `error.tsx` exists:
   - Note its existence and structure (it will be tested if triggered naturally)
   - Verify it includes a retry/recovery mechanism
   - Verify it doesn't expose stack traces or internal details

#### 2E. Middleware & Header Verification

**For each route, verify expected headers from middleware:**

1. **Security headers:**
   - `Content-Security-Policy` — present and reasonably restrictive
   - `X-Content-Type-Options: nosniff` — present
   - `X-Frame-Options` — present (DENY or SAMEORIGIN)
   - `Strict-Transport-Security` — present on HTTPS
   - `Referrer-Policy` — present
   - `Permissions-Policy` — present (controls camera, microphone, geolocation, etc.)

2. **Caching headers:**
   - Static assets: Should have `Cache-Control` with long max-age
   - Dynamic pages: Should have appropriate caching (or no-cache)
   - API routes: Should have `Cache-Control: no-store` (prevent caching of mutations)

3. **Bot blocking (if configured):**
   - Send request with a blocked User-Agent (e.g., `GPTBot`)
   - Verify: 403 or redirect response (bot is blocked)
   - Send request with normal User-Agent
   - Verify: 200 response (real users aren't blocked)

**Finding severity scale:**
- **CRITICAL:** Page returns 500, page crashes, white screen, unhandled exceptions, missing critical scripts
- **HIGH:** Console errors, hydration mismatches, missing resources, broken internal links, stuck loading states, sensitive data in localStorage
- **MEDIUM:** Missing SEO tags, placeholder content, missing security headers, console warnings, font loading issues
- **LOW:** Console.log in production, minor HTML issues, missing OG tags
- **INFO:** Observations that aren't problems (e.g., "uses client-side rendering", "3 external links found")

**Output:** Findings list per page, screenshot paths, static asset results, redirect results, header results, health score.

---

### Phase 3: Interactive Element Testing

**Purpose:** Test every clickable, typeable, and submittable element like a real user would.

**Agent:** `sonnet` — Interactive Testing Scout (via MCP browser tools)

**For each page:**

1. **Element inventory:** Use `browser_snapshot` (accessibility tree) to identify:
   - All buttons (`<button>`, `[role="button"]`)
   - All links (`<a>`)
   - All form inputs (`<input>`, `<textarea>`, `<select>`)
   - All forms (`<form>`)
   - Interactive widgets (dropdowns, modals, tabs, accordions)

2. **Navigation testing:**
   - Click each internal navigation link
   - Verify it navigates to the correct page (URL changes as expected)
   - Verify no errors after navigation
   - Test browser back button after each navigation
   - Test all navbar/menu links
   - Test footer links
   - Test any breadcrumb navigation

3. **Button testing:**
   - Click each non-navigation button
   - Observe: Does something happen? (modal opens, state changes, animation triggers)
   - Check for console errors after click
   - For toggle buttons: verify they toggle
   - For buttons that open modals/drawers: verify the modal opens and can be closed

4. **Form testing:**
   For each form found:

   a. **Valid submission test:**
      - Fill all required fields with valid test data
      - Use test prefix: email `qatest_${timestamp}@example.com`, name `QA Test User`
      - Submit the form
      - Verify: success feedback shown (toast, message, redirect)
      - Verify: no console errors
      - Note: form endpoint and method for API phase

   b. **Empty submission test:**
      - Submit the form with no data
      - Verify: validation errors appear
      - Verify: form does NOT submit (no network request)

   c. **Invalid data test:**
      - Fill email fields with "not-an-email"
      - Fill required fields with empty strings
      - Submit and verify validation messages

   d. **Security input test:**
      - Try `<script>alert('xss')</script>` in text fields
      - Try SQL injection patterns: `'; DROP TABLE users; --`
      - Verify: inputs are sanitized (no XSS, no SQL injection)
      - These should NOT trigger real errors — they test input sanitization

5. **Scroll and viewport testing:**
   - Scroll to bottom of each page
   - Verify lazy-loaded content appears
   - Check for fixed/sticky elements (navbar, CTA buttons)
   - Verify smooth scroll behavior if implemented

6. **Keyboard navigation:**
   - Tab through the page using `browser_press_key`
   - Verify focus indicators are visible
   - Verify all interactive elements are reachable via keyboard
   - Test Enter/Space on focused buttons
   - Test Escape to close modals

7. **Cookie consent / GDPR banner testing (if detected):**
   - Verify the cookie banner appears on first visit
   - Test "Accept" — verify cookies are set appropriately
   - Test "Reject" / "Decline" — verify tracking cookies are NOT set
   - Verify banner doesn't reappear after accepting (cookie/localStorage persistence)
   - Verify banner reappears in incognito/fresh session

8. **Theme / dark mode testing (if detected):**
   - If the app has a theme toggle:
     - Toggle to dark mode → screenshot → verify no white flashes
     - Toggle back to light mode → verify clean transition
     - Check that ALL components adapt (no "white boxes" in dark mode)
     - Verify theme persists across page navigation
     - Verify theme persists on reload (localStorage/cookie)
   - Check `prefers-color-scheme` media query support

9. **Search functionality testing (if detected):**
   - If the app has a search feature:
     - Search with a valid query → verify results appear
     - Search with empty query → verify appropriate behavior
     - Search with special characters (`<script>`, `%20`, `"quotes"`) → verify sanitization
     - Search with very long query (1000+ chars) → verify no crash
     - Search for something that returns no results → verify empty state message

10. **Pagination / infinite scroll testing (if detected):**
    - If pagination exists:
      - Navigate to page 1 → verify content
      - Navigate to page 2 → verify different content
      - Navigate to last page → verify content loads
      - Navigate to page 0 or negative → verify error handling
      - Navigate to page beyond last → verify error handling
    - If infinite scroll:
      - Scroll to trigger first batch load → verify new content appears
      - Continue scrolling → verify subsequent batches load
      - Verify "end of content" indicator appears when no more data

11. **State persistence across navigation:**
    - If the app has client-side state (form data, filters, scroll position):
      - Set some state (e.g., fill a form partially, select a filter)
      - Navigate away then navigate back (browser back button)
      - Verify: Does state persist? (If it should, it should. If it shouldn't, it shouldn't.)
      - This tests React context, URL state, localStorage, and session management

12. **Multi-tab behavior:**
    - Open the same page in a new tab (`browser_tab_new`)
    - Verify: Both tabs render correctly
    - If the app has real-time features, verify both tabs stay in sync
    - Close the extra tab

**Output:** Interaction findings, form test results, keyboard nav results, cookie consent results, theme results.

---

### Phase 3.5: User Journey / Flow Testing

**Purpose:** Simulate complete end-to-end user journeys — not just individual pages, but realistic paths through the app. This catches bugs that only appear when pages interact: missing context, broken navigation chains, state that doesn't carry across pages.

**Agent:** `sonnet` — Journey Testing Scout (via MCP browser tools)

**IMPORTANT:** This phase tests the app as a REAL USER would experience it — starting at the homepage (or a landing page), navigating through the site, performing actions, and completing goals.

#### Journey 1: First-Time Visitor Flow
```
Homepage → Scroll full page → Click services link → Read service page →
Navigate back → Click another service → Go to About → Go to Contact →
Fill form → Submit → See confirmation
```
**Verify at each step:**
- Navigation works (correct page loads)
- No console errors between navigations
- Back button works correctly at every step
- Page content loads fully (no stuck spinners)
- Scroll position resets on new page (unless anchor navigation)

#### Journey 2: Content Explorer Flow
```
Homepage → Blog/Articles listing → Click first article → Read article →
Click related article link or category → Back to listing →
Click another article → Navigate to portfolio → Click portfolio item →
Back to portfolio listing
```
**Verify at each step:**
- All dynamic content renders (MDX, images, code blocks)
- Navigation between dynamic routes works
- Category/tag filtering works (if available)
- No layout shifts after content loads

#### Journey 3: Business Inquiry Flow
```
Homepage → Services page → Click CTA for inquiry → Partnership page →
Fill partnership form → Submit → See confirmation →
Navigate to contact → Fill contact form → Submit → See confirmation
```
**Verify at each step:**
- CTAs navigate to correct pages
- Forms pre-fill any available data (if applicable)
- Form submission works
- Success/error states display correctly
- User can navigate away after submission

#### Journey 4: Mobile Navigation Flow (at 375px viewport)
```
Homepage → Open hamburger menu → Click link → Close menu (if auto-close) →
Scroll page → Open menu again → Navigate to deep page → Back button →
Verify menu state
```
**Verify at each step:**
- Hamburger menu opens/closes correctly
- Menu links navigate correctly
- Menu auto-closes on navigation (if expected)
- No content trapped behind the menu
- Touch interactions work

#### Journey 5: Deep Link Integrity Flow
```
Enter at /articles/[random-slug] directly → Verify full page renders →
Navigate to homepage via nav → Navigate to another deep page →
Enter at /services/ai-transformation directly → Verify full render →
Try a 404 URL → Verify 404 page → Click home link from 404 → Verify
```
**Verify:**
- Every deep link entry renders the full page with layout, navigation, and content
- No "missing context" errors from entering mid-flow
- 404 page provides a path back to the main site

#### Journey 6: Rapid Navigation Stress Test
```
Click 10+ links in rapid succession (< 1 second between clicks) →
Verify final page renders correctly → No console errors →
No memory leaks → No stuck loading states
```
**Verify:**
- App handles rapid navigation without crashing
- Route transitions don't stack or conflict
- No zombie network requests from cancelled navigations

#### Dynamic Journey Generation
For apps with auth, e-commerce, dashboards, or other complex flows:
- The skill should analyze the app's capabilities (from Phase 0 detection) and generate additional journeys specific to the app's functionality
- Example for a SaaS app with auth: signup → onboarding → dashboard → settings → logout → login
- Example for e-commerce: browse → add to cart → checkout → payment → confirmation

**Output:** Journey test results, per-step findings, flow completion status.

---

### Phase 4: API Route Testing

**Purpose:** Validate every API endpoint with valid, invalid, and edge-case payloads.

**Agent:** `sonnet` — API Testing Agent (via Bash curl/fetch, NOT browser)

**For each API endpoint discovered in Phase 1:**

1. **Method validation:**
   - Send requests with each expected HTTP method → verify success
   - Send requests with unexpected methods (GET on POST-only route) → verify 405 Method Not Allowed
   - Send OPTIONS request → verify CORS headers if applicable

2. **Valid payload testing:**
   - Construct a valid payload based on the route's Zod schema or type definitions
   - Send the request with valid data
   - Verify: 200/201 response, correct response body structure
   - Note: save the response for comparison

3. **Invalid payload testing:**
   - Send empty body → verify 400 Bad Request with validation error
   - Send malformed JSON → verify 400
   - Send missing required fields → verify 400 with field-specific errors
   - Send extra unexpected fields → verify they're ignored or rejected

4. **Security header validation:**
   - Check response headers:
     - `Content-Type` is appropriate
     - `X-Content-Type-Options: nosniff` present
     - No sensitive headers leaked (e.g., server version)
   - Check CORS headers match configuration
   - Check CSP headers on responses

5. **Rate limiting validation:**
   - If rate limiting is detected (via code analysis):
     - Send rapid sequential requests
     - Verify: rate limiter triggers (429 Too Many Requests)
     - Verify: appropriate retry-after or error message
   - If no rate limiting is detected on a write endpoint: flag as finding

6. **Input sanitization validation:**
   - Send XSS payloads in string fields
   - Send SQL injection patterns
   - Send extremely long strings (test input size limits)
   - Verify: inputs are sanitized in response/database
   - Verify: appropriate error messages for oversized input

7. **Error handling validation:**
   - Send requests that should trigger server errors (if testable)
   - Verify: errors return appropriate status codes (not 200 with error body)
   - Verify: error responses don't leak stack traces or internal details

8. **Webhook testing (if applicable):**
   - Identify webhook endpoints (common patterns: `/api/webhook/*`, `/api/stripe/*`)
   - Verify they validate webhook signatures
   - Send requests without valid signatures → verify rejection

9. **Honeypot field testing (if detected):**
   - If the API route has honeypot fields (common: `fax`, `website`, `url`, hidden fields):
     - Submit with honeypot field EMPTY → verify request succeeds
     - Submit with honeypot field FILLED (like a bot would) → verify request is rejected/silently ignored
     - This validates bot protection is working

10. **Server action testing (Next.js "use server"):**
    - If the app uses server actions:
      - Identify all exported server action functions
      - Test each with valid inputs → verify expected behavior
      - Test each with invalid inputs → verify validation
      - Server actions are API-like but invoked differently — they need separate testing

11. **Content-Type boundary testing:**
    - Send request with wrong Content-Type (e.g., `text/plain` instead of `application/json`) → verify appropriate error
    - Send request with no Content-Type header → verify handling
    - Send multipart/form-data if the endpoint doesn't expect it → verify rejection

12. **Concurrent request testing:**
    - Send 5 identical valid requests simultaneously
    - Verify: All return correct responses (no race conditions)
    - Verify: If the operation is idempotent, responses are consistent
    - Verify: If the operation creates resources, duplicates are handled

**Output:** API test results per endpoint, security findings, rate limit status, honeypot results, server action results.

---

### Phase 4.5: Third-Party Integration Health Check

**Purpose:** Verify that external services the app depends on are reachable and configured correctly. A beautiful UI means nothing if Supabase is unreachable or Resend can't send emails.

**Agent:** `sonnet` — Integration Health Agent (via Bash)

**IMPORTANT:** This phase does NOT test external service functionality deeply (that's their responsibility). It validates that the app's CONNECTION to these services works.

**For each detected third-party service:**

1. **Supabase (if detected):**
   - Verify `SUPABASE_URL` is set and reachable (HEAD request)
   - Verify `SUPABASE_ANON_KEY` or `SUPABASE_SERVICE_ROLE_KEY` is set
   - If the app has a health check endpoint, hit it
   - Verify: Tables referenced in code actually exist (via anon key query if possible)
   - Check: RLS policies exist on tables that store user data

2. **Resend / email service (if detected):**
   - Verify `RESEND_API_KEY` is set
   - Verify the API key is valid by checking API status (Resend has a `/emails` endpoint that returns 401 vs 403)
   - Do NOT send a test email — just verify connectivity
   - Check: From email domain is configured correctly

3. **Upstash Redis (if detected):**
   - Verify `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN` are set
   - Verify connectivity (simple GET to the REST API)
   - Check: Rate limiter is using Redis, not the mock fallback

4. **Stripe (if detected):**
   - Verify API keys are set
   - Verify webhook signing secret is set
   - Check webhook endpoint exists and responds
   - Do NOT make any payment-related API calls

5. **Analytics (if detected):**
   - Verify tracking scripts are loaded but not blocking rendering
   - Check for `NEXT_PUBLIC_*` analytics env vars
   - Verify scripts are loaded with appropriate `defer` or `async`

6. **Generic health check:**
   - If the app has a `/api/health` or `/api/status` endpoint, hit it
   - Verify it returns 200 with appropriate status information
   - If no health endpoint exists, note as a MEDIUM finding (apps should have health checks)

**Output:** Integration health status per service, connectivity results, missing configuration findings.

---

### Phase 5: Accessibility Scan

**Purpose:** Run automated WCAG 2.1 AA compliance checks on every page.

**Agent:** `sonnet` — Accessibility Scanner

**Approach:** Use `browser_evaluate` to inject and run axe-core on each page. If axe-core is not available in the project, use the Playwright MCP snapshot (accessibility tree) for manual checks.

**For each page:**

1. **axe-core scan (primary method):**
   - Inject axe-core via CDN: `browser_evaluate` with `<script src="https://cdn.jsdelivr.net/npm/axe-core@latest/axe.min.js">`
   - Run `axe.run()` with WCAG 2.1 AA ruleset
   - Capture violations, passes, and incomplete checks
   - Map violations to severity:
     - `critical` → CRITICAL finding
     - `serious` → HIGH finding
     - `moderate` → MEDIUM finding
     - `minor` → LOW finding

2. **Manual accessibility checks (supplement axe-core):**
   - Color contrast: Check text meets 4.5:1 ratio (AA) via computed styles
   - Focus management: Tab through page, verify logical focus order
   - Skip navigation: Check for skip-to-content link
   - Heading hierarchy: Verify headings are sequential (h1 → h2 → h3, no skipping)
   - Image alt text: Verify meaningful images have descriptive alt (not just `alt=""`)
   - Form labels: Verify all inputs have associated `<label>` elements
   - ARIA landmarks: Check for `<main>`, `<nav>`, `<header>`, `<footer>`
   - Motion: Check for `prefers-reduced-motion` media query support
   - Touch targets: Verify interactive elements are at least 44x44px on mobile

3. **Screen reader simulation:**
   - Use the Playwright accessibility tree (`browser_snapshot`) to verify:
     - All interactive elements have accessible names
     - ARIA roles are used correctly
     - Live regions are properly announced
     - Dynamic content changes are communicated

**Output:** Accessibility findings per page with WCAG rule references, axe-core scores.

---

### Phase 6: Responsive Viewport Testing

**Purpose:** Verify the app works correctly at mobile, tablet, and desktop viewports.

**Agent:** `sonnet` — Responsive Testing Scout (via MCP browser tools)

**Viewports:**
- Mobile: 375x812 (iPhone SE/13 mini)
- Tablet: 768x1024 (iPad)
- Desktop: 1280x800 (standard laptop)

**For each page at each viewport:**

1. **Resize and screenshot:**
   - Use `browser_resize` to set viewport
   - Navigate to the page
   - Take screenshot → `.qatest-reports/screenshots/{route-slug}-{viewport}.png`

2. **Layout checks via DOM evaluation:**
   - Check for horizontal overflow: `document.documentElement.scrollWidth > document.documentElement.clientWidth`
   - Check for elements extending beyond viewport
   - Check for text truncation or overlap (elements with `overflow: hidden` cutting off content)

3. **Mobile-specific checks:**
   - Touch target size: All interactive elements >= 44x44px
   - Font size: Body text >= 16px (prevents iOS zoom on input focus)
   - Viewport meta tag: `<meta name="viewport" content="width=device-width, initial-scale=1">`
   - No horizontal scroll
   - Mobile navigation works (hamburger menu opens/closes)
   - Forms are usable (inputs aren't too small, keyboard doesn't obscure)

4. **Tablet-specific checks:**
   - Layout adapts appropriately (not just stretched mobile or squished desktop)
   - Grid layouts adjust columns
   - Navigation is appropriate for the viewport

5. **Desktop-specific checks:**
   - Content doesn't stretch to uncomfortable reading widths (max-width on text)
   - Hover states work on interactive elements
   - Full navigation is visible (not hidden in hamburger)

**Output:** Responsive findings per viewport, screenshot paths, mobile usability score.

---

### Phase 6.5: Performance Baseline & Memory Check

**Purpose:** Establish performance baselines and detect memory leaks. This is NOT a full performance audit (that's `/perf`), but catches show-stopping performance issues that affect user experience.

**Agent:** `sonnet` — Performance Baseline Agent (via MCP browser tools)

1. **Page load timing (every page):**
   - Use `browser_evaluate` with `performance.getEntriesByType('navigation')` to capture:
     - `domContentLoadedEventEnd` — when DOM is ready
     - `loadEventEnd` — when page is fully loaded
   - Thresholds:
     - < 3 seconds: PASS
     - 3-5 seconds: FINDING (severity: MEDIUM — slow but functional)
     - 5-10 seconds: FINDING (severity: HIGH — poor user experience)
     - > 10 seconds: FINDING (severity: CRITICAL — unusable)

2. **Core Web Vitals spot check (homepage + 2 key pages):**
   - Largest Contentful Paint (LCP): Should be < 2.5s
   - Cumulative Layout Shift (CLS): Should be < 0.1
   - Use `browser_evaluate` with `PerformanceObserver` to capture these
   - This is a spot check, not a full audit — flag any obvious violations

3. **Memory leak detection:**
   - Get baseline memory: `browser_evaluate` with `performance.memory.usedJSHeapSize`
   - Navigate through 10+ pages sequentially (simulating browsing session)
   - Get final memory: check heap size again
   - If memory grew > 50% from baseline: FINDING (severity: HIGH — likely memory leak)
   - If memory grew > 100%: FINDING (severity: CRITICAL — definite memory leak)
   - Note: This is a rough check — `/perf` does deeper profiling

4. **JavaScript bundle impact:**
   - Use `browser_evaluate` with `performance.getEntriesByType('resource')` to list all loaded JS files
   - Sum total JS transferred
   - If > 1MB total JS: FINDING (severity: MEDIUM — heavy page)
   - If > 2MB total JS: FINDING (severity: HIGH — very heavy page)
   - List the top 5 largest JS files for reference

**Output:** Page load times, CWV spot check, memory trend, bundle size summary.

---

### Phase 7: Analysis & Scoring

**Purpose:** Synthesize all findings into a cohesive QA score and prioritized finding list.

**Agent:** Orchestrator performs this directly (lightweight synthesis, no sub-agent needed)

**Scoring Formula:**

```
QA Score = weighted average of category scores

Categories (weights):
- Page Health & Infrastructure: 20%  (pages loading, no errors, no broken links, static assets, redirects, headers, SEO)
- Interactivity & Journeys:    20%  (forms work, buttons work, navigation works, user flows complete)
- API Correctness:             15%  (endpoints respond correctly, validate input, handle errors, honeypots work)
- Accessibility:               15%  (WCAG 2.1 AA compliance, keyboard nav, screen reader)
- Responsive:                  10%  (mobile/tablet/desktop usability, touch targets, viewport)
- Integration Health:          10%  (third-party services reachable, env vars set, connections healthy)
- Performance Baseline:        10%  (page load times, CWV spot check, no memory leaks)

Per-category scoring:
- Start at 100
- CRITICAL finding: -25 points
- HIGH finding: -10 points
- MEDIUM finding: -5 points
- LOW finding: -2 points
- INFO finding: -0 points (documented but doesn't affect score)

Floor: 0 (never negative)
```

**Grade scale:**
- 95-100: A+ (Ship with confidence)
- 90-94: A (Ship — minor issues only)
- 80-89: B (Ship with caution — review medium findings)
- 70-79: C (Fix HIGH findings before shipping)
- 60-69: D (Significant issues — do not ship)
- 0-59: F (Critical failures — do not ship)

**Ship decision:**
- A+ / A: SHIP
- B: SHIP WITH REVIEW (human should review medium findings)
- C or below: DO NOT SHIP (fix issues and re-run)

**Prioritized finding list:**
Order all findings by: severity (CRITICAL → HIGH → MEDIUM → LOW → INFO), then by category, then by page.

**Autofix candidates:**
Mark each finding as:
- `AUTOFIX` — safe to fix automatically
- `DEFERRED` — requires human judgment
- `BLOCKED` — cannot be fixed due to external dependency

---

### Phase 8: Autofix

**Purpose:** Fix all deterministic, safe issues automatically.

**Agent:** `sonnet` — Autofix Agent (sequential, one fix at a time)

**IMPORTANT:** Skip this phase if `--no-fix` flag is set.

### Safe to Auto-Fix (AUTOFIX candidates)

| Finding Type | Fix | Risk |
|-------------|-----|------|
| Missing `alt=""` on decorative `<img>` (images inside links/buttons with text) | Add `alt=""` attribute | Very Low |
| Missing `rel="noopener noreferrer"` on external `<a target="_blank">` | Add the rel attribute | Very Low |
| Missing `<html lang="...">` attribute | Add `lang="en"` (or detect from content) | Very Low |
| Missing viewport meta tag | Add standard viewport meta | Very Low |
| Missing `aria-label` on icon-only buttons | Add descriptive aria-label based on icon name/context | Low |
| Broken internal links (href points to non-existent route) | Update href to correct route (if deterministic) | Low |
| Missing form `<label>` associations | Add `id` to input and `htmlFor` to label, or wrap input in label | Low |
| `console.log` statements in production code | Remove console.log (but NOT console.error/warn) | Low |
| Missing `<title>` tag | Generate from route path or page heading | Low |
| Missing `<meta name="description">` | Generate from first paragraph of page content | Low-Medium |
| Heading hierarchy gaps (h1 → h3, missing h2) | Adjust heading levels to be sequential | Low |
| Missing `rel="canonical"` link tag | Add canonical URL based on route path | Very Low |
| Missing `X-Content-Type-Options: nosniff` header | Add to middleware/next.config headers | Very Low |
| Missing `Referrer-Policy` header | Add `strict-origin-when-cross-origin` to headers config | Very Low |
| Missing `Permissions-Policy` header | Add with sensible defaults to headers config | Very Low |
| Empty `href=""` or `href="#"` on anchor tags (non-intentional) | Add correct href destination or convert to button | Low |
| `javascript:void(0)` in href | Convert to button element with onClick | Low |
| Missing `.qatest-reports/` in `.gitignore` | Append to .gitignore | Very Low |
| Missing `font-display: swap` on @font-face declarations | Add `font-display: swap` | Very Low |
| Missing `async` or `defer` on third-party scripts | Add `async` attribute | Low |
| Raw `<img>` tags in Next.js (should be `<Image>`) | Convert to Next.js `<Image>` component | Low-Medium |
| Sitemap missing routes that exist in filesystem | Add missing routes to sitemap generation | Low |
| Missing OG image meta tag | Add og:image pointing to existing OG image file (if one exists) | Low |

### Autofix Process (for each fixable finding)

```
1. READ the source file containing the issue
2. IDENTIFY the exact line(s) to change
3. APPLY the fix using the Edit tool
4. VERIFY the build still passes:
   - Run: pnpm build (or equivalent)
   - If build PASSES: mark finding as FIXED ✅
   - If build FAILS: immediately REVERT the change, mark as BLOCKED
5. UPDATE the state file and report
```

### NEVER Auto-Fix (always DEFERRED)

| Finding Type | Why Not |
|-------------|---------|
| Descriptive alt text for meaningful images | Requires understanding image content — human judgment |
| Layout/CSS issues | Cascading side effects unpredictable |
| Business logic errors | Requires understanding product intent |
| Broken external links | External site may be temporarily down |
| Complex accessibility violations | May require architectural changes |
| Performance issues | Optimization can break functionality |
| Content/copy issues | Product/marketing decision |
| Authentication/authorization failures | Security-sensitive, needs careful review |
| API response format issues | May break client contracts |
| Rate limiter configuration | Product decision on thresholds |

---

### Phase 9: Validation

**Purpose:** Re-test everything that was auto-fixed to confirm no regressions.

**Steps:**

1. **Build verification:**
   ```
   - Run full build: pnpm build (or equivalent)
   - Run typecheck: pnpm typecheck (if available)
   - Run lint: pnpm lint (if available)
   - Run existing tests: pnpm test:run (if available)
   - ALL must pass. If any fail, identify which autofix caused the failure and revert it.
   ```

2. **Re-test fixed pages:**
   - For each page that had an autofix applied:
     - Navigate to the page via MCP browser
     - Verify the fix is visible/working
     - Verify no new console errors
     - Take a fresh screenshot

3. **Re-run accessibility on fixed pages:**
   - For each page with accessibility fixes:
     - Re-run axe-core
     - Verify the violation is resolved
     - Verify no new violations introduced

4. **Score recalculation:**
   - Recalculate QA score with fixes applied
   - Report before/after comparison

---

### Phase 10: Report Generation

**Purpose:** Write the final, comprehensive QA report.

**Report file:** `.qatest-reports/qatest-YYYYMMDD-HHMMSS.md`

**Report structure:**

```markdown
# QA Test Report — [Project Name]

**Date:** YYYY-MM-DD HH:MM
**Project:** [project-name]
**Framework:** [detected framework]
**Base URL:** [base URL tested against]
**Mode:** [full / changed / api-only / pages-only / retest]
**Duration:** [total time]

---

## QA Score: [XX]/100 ([Grade])

**Ship Decision:** [SHIP / SHIP WITH REVIEW / DO NOT SHIP]

| Category | Weight | Score | Findings |
|----------|--------|-------|----------|
| Page Health & Infrastructure | 20% | XX/100 | X critical, X high, X medium, X low |
| Interactivity & Journeys | 20% | XX/100 | X critical, X high, X medium, X low |
| API Correctness | 15% | XX/100 | X critical, X high, X medium, X low |
| Accessibility | 15% | XX/100 | X critical, X high, X medium, X low |
| Responsive | 10% | XX/100 | X critical, X high, X medium, X low |
| Integration Health | 10% | XX/100 | X critical, X high, X medium, X low |
| Performance Baseline | 10% | XX/100 | X critical, X high, X medium, X low |

---

## Route Manifest

| # | Route | Type | Status |
|---|-------|------|--------|
| 1 | / | page | ✅ 200 |
| 2 | /about | page | ✅ 200 |
| ... | ... | ... | ... |

---

## Findings Summary

| # | Severity | Category | Page/Endpoint | Finding | Status |
|---|----------|----------|---------------|---------|--------|
| QA-001 | CRITICAL | Page Health | /checkout | Page returns 500 | DEFERRED |
| QA-002 | HIGH | API | POST /api/contact | No rate limiting | DEFERRED |
| QA-003 | MEDIUM | A11y | / | Missing alt text on hero image | FIXED ✅ |
| ... | ... | ... | ... | ... | ... |

---

## Detailed Findings

### QA-001: [Finding title]
- **Severity:** CRITICAL
- **Category:** Page Health
- **Page:** /checkout
- **Status:** DEFERRED
- **Description:** [Detailed description of the issue]
- **Expected:** [What should happen]
- **Actual:** [What actually happened]
- **Evidence:** [Screenshot path, console output, etc.]
- **Root Cause:** [If determinable]
- **Recommendation:** [How to fix]

[Repeat for each finding]

---

## Auto-Fix Summary

| # | Finding | Fix Applied | Build Status | Final Status |
|---|---------|-------------|-------------|-------------|
| QA-003 | Missing alt text | Added alt="" | ✅ Pass | FIXED |
| QA-007 | Missing rel="noopener" | Added attribute | ✅ Pass | FIXED |

**Total:** X issues auto-fixed, X deferred, X blocked

---

## Files Modified by Autofix

| File | Changes |
|------|---------|
| src/components/Hero.tsx | Added alt="" to decorative image (line 42) |
| src/components/Footer.tsx | Added rel="noopener noreferrer" to external links (lines 15, 23) |

---

## Before/After State

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| QA Score | XX/100 | XX/100 | +X |
| Critical Findings | X | X | -X |
| High Findings | X | X | -X |
| Total Findings | X | X | -X |
| Pages Passing | X/Y | X/Y | +X |
| API Endpoints Passing | X/Y | X/Y | +X |
| A11y Violations | X | X | -X |

---

## Deferred Items

| ID | Finding | Why Deferred | Conditions to Revisit |
|----|---------|-------------|----------------------|
| QA-001 | Page /checkout returns 500 | Server error in API handler — requires debugging | Fix the API route handler, then run /qatest --retest |
| QA-002 | No rate limiting on POST /api/contact | Product decision on rate limit thresholds | Discuss with team, implement rate limiting, re-test |

---

## Blocked Items

| ID | Finding | Blocker | What Needs to Happen |
|----|---------|---------|---------------------|
| QA-010 | Heading hierarchy fix broke layout | CSS depends on h3 selector | Refactor CSS to not depend on heading elements |

---

## Screenshots

All screenshots saved to: `.qatest-reports/screenshots/`

| Page | Desktop | Mobile | Tablet |
|------|---------|--------|--------|
| / | [homepage.png] | [homepage-mobile.png] | [homepage-tablet.png] |
| /about | [about.png] | [about-mobile.png] | [about-tablet.png] |

---

## Historical Context

[If previous runs exist]
- Run #N (date): X findings, Y fixed, Z deferred
- Trend: [improving / stable / declining]

---

## Recommendations

1. [Prioritized recommendation with finding reference]
2. [Next recommendation]
3. Run `/gh-ship` to commit and deploy after fixes

---

## SITREP

**Status:** [COMPLETE / PARTIAL / BLOCKED]
**QA Score:** [XX/100] ([Grade])
**Ship Decision:** [SHIP / SHIP WITH REVIEW / DO NOT SHIP]
**Findings:** X total (X critical, X high, X medium, X low, X info)
**Auto-Fixed:** X issues
**Deferred:** X issues (require human judgment)
**Blocked:** X issues (external dependencies)
**Duration:** X minutes
**Pages Tested:** X/Y
**API Endpoints Tested:** X/Y
```

---

## DATA CLEANUP

After all testing is complete, clean up any test data created during the QA run:

1. **Form submissions:** If the skill submitted test forms (with `qatest_` prefix), note them in the report for manual cleanup if they went to a real database. The skill does NOT delete from databases directly — it documents what was created.

2. **Browser state:** Close all browser tabs/contexts opened during testing via `browser_close`.

3. **Dev server:** If the skill started a dev server, note it in the report. Do NOT kill it — the user may want it running.

4. **Test files:** No temporary test files are created (unlike COA 3). Nothing to clean up.

---

## INCREMENTAL MODE (--changed)

When `--changed` flag is set:

1. **Detect changed files:**
   ```bash
   git diff --name-only main...HEAD
   ```

2. **Map files to routes:**
   - Changed `app/about/page.tsx` → test `/about`
   - Changed `src/components/navbar.tsx` → test ALL pages (shared component)
   - Changed `app/api/contact/route.ts` → test `POST /api/contact`
   - Changed `src/lib/validations/contact.ts` → test `/api/contact` + any page with contact form
   - Changed `tailwind.config.ts` or global CSS → test ALL pages (global style change)
   - Changed `src/components/ui/*` → test all pages using that component

3. **Shared component detection:**
   If a changed file is imported by multiple pages, ALL those pages are tested. The skill traces the import graph to determine the blast radius.

4. **Minimum scope:**
   Even in incremental mode, always test:
   - The homepage (`/`)
   - Any page with forms (critical user interaction)
   - All API routes (fast to test, high impact)

---

## ERROR RECOVERY

### Dev server won't start
```
❌ Error: Dev server failed to start
   Port: 3000
   Reason: [error message]

   🔧 Suggested fixes:
   1. Check if port 3000 is already in use: lsof -i :3000
   2. Try a different port: PORT=3001 pnpm dev
   3. Check for build errors: pnpm build
   4. Check node_modules: pnpm install
```

### Build fails during autofix
```
1. Immediately revert the change (git checkout on the file)
2. Mark the finding as BLOCKED
3. Log the build error in the report
4. Continue to next autofix candidate
5. NEVER leave the codebase in a broken state
```

### Browser/MCP timeout
```
1. Close the browser: browser_close
2. Wait 5 seconds
3. Retry the page navigation once
4. If still fails: mark page as BLOCKED, continue to next page
5. NEVER stop the entire pipeline for one page failure
```

### Rate limiter blocks testing
```
1. Detect 429 responses
2. Read the retry-after header
3. Wait the specified time (or 30 seconds if no header)
4. Retry once
5. If still blocked: note rate limiter is working (that's actually a PASS), continue
```

### Context getting heavy
```
1. Check state file — how many phases are complete?
2. If > 60% complete: push through to finish
3. If < 60% complete: write comprehensive checkpoint to state file
4. State file allows resume from any point
```

---

## CROSS-SKILL INTEGRATION

### Skills that should run BEFORE /qatest
- `/deps` — Ensure dependencies are healthy
- `/sec-ship` — Fix security vulnerabilities first
- `/a11y` — Deep accessibility audit (qatest does lighter a11y)

### Skills that should run AFTER /qatest
- `/gh-ship` — Commit fixes and create PR
- `/perf` — Performance optimization (after functional correctness is confirmed)

### Cross-skill awareness
At startup, check for recent reports from:
- `.security-reports/` — If sec-ship found auth issues, qatest should verify they're fixed
- `.a11y-reports/` — If a11y found violations, qatest should verify basic compliance
- `.perf-reports/` — If perf found slow pages, qatest should set longer timeouts for those pages

---

## REPORT DIRECTORY & GITIGNORE

**Report directory:** `.qatest-reports/`

**Contents:**
```
.qatest-reports/
├── qatest-YYYYMMDD-HHMMSS.md       # Main report
├── state-YYYYMMDD-HHMMSS.json      # State file (for resume)
├── route-manifest.json              # Discovered routes
└── screenshots/                     # Page screenshots
    ├── homepage.png
    ├── homepage-mobile.png
    ├── about.png
    └── ...
```

**Gitignore:** At startup, verify `.qatest-reports/` is in `.gitignore`. If not, add it:
```bash
echo '\n# QA test reports\n.qatest-reports/' >> .gitignore
```

This is a GLOBAL skill — every project it runs in should have `.qatest-reports/` gitignored.

---

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### QATest-Specific Cleanup

Resources this skill may create:
- Playwright browser instances (screenshots, page navigation)
- Dev server (if started by skill)
- Test form submissions with `qatest_` prefix in databases
- Screenshots in `.qatest-reports/screenshots/`

Cleanup actions (run after Phase 9 Validation, before Phase 10 Report):
1. **Close all browser instances:** Call `browser_close` for every open Playwright session
2. **Stop dev server (if started by this skill):** Kill the PID tracked at Phase 0. Verify port is released. If the user started the server before the skill, leave it running
3. **Delete test data:** Query for records with `qatest_` prefix and DELETE them. Log any records that couldn't be deleted
4. **Screenshots:** Keep screenshots in `.qatest-reports/screenshots/` (intended output). Delete any screenshots in `/tmp/` or working directory
5. **Verify no orphaned Chromium/Playwright processes remain**
6. **Log cleanup results in the report**

Cleanup verification:
- `pgrep -f "chromium|playwright"` should match pre-skill baseline
- `lsof -ti:3000` should be empty (if skill started the server)
- Database query for `qatest_*` records should return 0

---

## REMEMBER

- **You are the QA lead, not the developer.** Your job is to find and document issues, fix the easy ones, and give a clear go/no-go decision.
- **Full scan by default.** The pre-ship gate is always comprehensive. Use `--changed` only for dev iteration.
- **Fix what's safe, document what's not.** Every deferred item gets a reason and a condition to revisit.
- **Never break the build.** Every fix is verified. If it breaks, it's reverted instantly.
- **The report is the deliverable.** A clear, timestamped, comprehensive QA report that tells the team exactly where things stand.
- **Manage your context.** Delegate heavy work to sub-agents. Keep the orchestrator lean. Use state files for persistence.
- **Clean up after yourself.** Close browsers, document test data, don't leave the codebase in a worse state than you found it.
- **Ship decision is binary.** Either the app is ready to ship or it's not. Be clear and decisive.

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
