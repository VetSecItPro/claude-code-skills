---
description: "/monitor -- Post-Deploy Health Check: verify deployment health, check routes, functions, infrastructure, and error rates"
allowed-tools: Bash(cat:*), Bash(ls:*), Bash(find:*), Bash(wc:*), Bash(curl:*), Bash(dig:*), Bash(openssl:*), Bash(node:*), Bash(npx:*), Bash(npm:*), Bash(bun:*), Bash(pnpm:*), Bash(yarn:*), Bash(git:*), Bash(jq:*), Bash(date:*), Bash(mkdir:*), Bash(touch:*), Bash(head:*), Bash(tail:*), Bash(grep:*), Bash(sort:*), Bash(uniq:*), Read, Write, Edit, Glob, Grep, Task, WebFetch, WebSearch, mcp__vercel__list_deployments, mcp__vercel__get_deployment, mcp__vercel__get_deployment_build_logs, mcp__vercel__get_runtime_logs, mcp__vercel__list_projects, mcp__vercel__get_project, mcp__vercel__web_fetch_vercel_url
---

# /monitor — Post-Deploy Health Check

**Purpose:** Verify that a deployment is actually healthy after ship. Not "did CI pass" but "can real users reach every page and function without errors."

**Philosophy:**
1. Trust but verify — a green deploy doesn't mean a healthy app
2. Environment-agnostic core — HTTP health checks work everywhere
3. Platform-specific depth — use native APIs when available (Vercel MCP, etc.)
4. Non-destructive always — read-only checks, never modify deployment state
5. Baseline-aware — compare against established baselines, not arbitrary thresholds

<!--
## Design Rationale

### When to use /monitor
- After every production deploy (especially after /gh-ship)
- When users report issues but you can't reproduce locally
- After infrastructure changes (DNS, CDN, env vars)
- Periodic health checks on running deployments
- Before declaring an incident resolved (or delegate from /incident)

### When NOT to use /monitor
- For local development testing (use /qatest instead)
- For security scanning (use /sec-ship instead)
- For performance profiling (use /perf instead)
- For pre-deploy validation (use /qatest --changed instead)

### Flow
```
  /monitor
     │
     ▼
  ┌─────────────────────┐
  │  Stage 0: Init      │─── Detect platform, create report, check resume
  │  Stage 1: Routes    │─── Map all routes from framework + filesystem
  │  Stage 2: Endpoints │─── HTTP GET every route, status codes, timings
  │  Stage 3: Functions  │─── Test serverless/edge functions, error rates
  │  Stage 4: Env Vars   │─── Verify required env vars exist
  │  Stage 5: Infra      │─── DNS, SSL, CDN, cache headers
  │  Stage 6: Errors     │─── Runtime error logs, recent error spikes
  │  Stage 7: Score      │─── Calculate health score, write SITREP
  └─────────────────────┘
     │
     ▼
  .monitor-reports/MON-YYYYMMDD-HHMMSS.md
```

### Key Design Decisions
1. HTTP health checks are the universal foundation — every platform supports them
2. Platform detection unlocks deeper insights (Vercel runtime logs, etc.)
3. Baseline mode on first run; comparison mode on subsequent runs
4. Response time thresholds are relative to baseline, not absolute
5. Scoring weights emphasize route health (25%) because broken routes = broken app
-->

---

## STATUS UPDATES

> Reference: [Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)

Provide status updates every 30-60 seconds during active work. Use the standard format:

### Monitor-Specific Status Examples

```
📋 Stage 1/7 — Route Discovery
   ├─ Scanning filesystem routes...
   ├─ Found 23 page routes, 8 API routes
   └─ Building route manifest

🔍 Stage 2/7 — Endpoint Health [12/31 routes checked]
   ├─ ✅ 10 routes returning 200
   ├─ ⚠️ 1 route returning 301 (redirect)
   ├─ ❌ 1 route returning 500
   └─ Checking next batch...

✅ Stage 7/7 — Scoring Complete
   ├─ Health Score: 87/100 (B+)
   ├─ 30/31 routes healthy
   ├─ 1 function with elevated error rate
   └─ Report: .monitor-reports/MON-20260224-143022.md
```

---

## CONTEXT MANAGEMENT

> Reference: [Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)

### Monitor-Specific Context Rules

1. **Sub-agents return < 500 tokens.** Full results write to `.monitor-reports/` state file.
2. **State file:** `.monitor-reports/state-YYYYMMDD-HHMMSS.json` — updated after every stage.
3. **Resume protocol:** Check for incomplete state from last 2 hours before starting fresh.
4. **Max 2 parallel scout agents** for endpoint health checks. All other stages sequential.
5. **Orchestrator never reads full agent output** — only the return summary.
6. **Checkpoint after every stage** — if context resets, resume from last completed stage.

---

## AGENT ORCHESTRATION

> Reference: [Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)

### Model Selection Table

| Agent | Model | Rationale |
|-------|-------|-----------|
| Route Discovery | haiku | Literal pattern extraction from filesystem |
| Endpoint Health (batch 1) | sonnet | HTTP checks + response analysis |
| Endpoint Health (batch 2) | sonnet | HTTP checks + response analysis |
| Function Verification | sonnet | Log analysis + error pattern detection |
| Environment Validation | haiku | Literal env var name checking |
| Infrastructure Health | sonnet | DNS/SSL/CDN analysis requires reasoning |
| Error Tracking | sonnet | Log correlation + pattern detection |

### Agent Batching Rules

| Codebase Size | Routes | Batch Strategy |
|--------------|--------|----------------|
| Small (< 20 routes) | < 20 | Single agent for endpoint health |
| Medium (20-50 routes) | 20-50 | 2 parallel agents, split routes evenly |
| Large (50+ routes) | 50+ | 2 parallel agents, prioritize API routes first |

---

## REPORT PERSISTENCE

### Finding Lifecycle

Every finding follows this lifecycle:

```
FOUND → MONITORING → HEALTHY | DEGRADED | DOWN | DEFERRED
```

- **FOUND**: Issue detected during check
- **MONITORING**: Intermittent issue, needs more data
- **HEALTHY**: Check passed, no issues
- **DEGRADED**: Partially working but with issues (slow response, intermittent errors)
- **DOWN**: Completely non-functional
- **DEFERRED**: Cannot verify (e.g., requires auth, external dependency)

### Persistence Rules

1. Report file created at Stage 0, updated after every stage
2. State file updated after every stage with machine-readable results
3. Findings are NEVER deleted — status changes are appended
4. Each finding has a unique ID: `MON-[stage]-[seq]` (e.g., `MON-2-003`)
5. Previous run baselines are loaded from most recent state file
6. Report is the single source of truth — if it says DEGRADED, it's DEGRADED

### Resume Protocol

Before starting a new run, check for incomplete state:
1. Look for `.monitor-reports/state-*.json` from last 2 hours
2. If found with `status: "in_progress"`, offer to resume from last completed stage
3. If resuming, load existing findings and baselines
4. If starting fresh, archive previous state file

---

## HUMAN DECISION TRIGGERS

**Pause and ask the user before proceeding if:**

1. A SEV-1 issue is detected (full outage on critical route) — confirm before continuing checks
2. Multiple 5xx errors suggest active incident — recommend `/incident` instead
3. SSL certificate expires within 7 days — escalate immediately
4. Environment variables appear to be missing for critical services (DB, auth)
5. Deployment URL is unreachable (DNS failure) — confirm URL is correct
6. Previous baseline shows dramatic degradation (>50% slower) — confirm this is expected

---

## RUN MODES

```
/monitor                    # Full health check (default) — all 8 stages
/monitor --quick            # Routes + functions only (stages 0-3, 7) — skip infra/env/errors
/monitor --routes-only      # Just endpoint health (stages 0-2, 7)
/monitor --env-only         # Just environment variable validation (stages 0, 4, 7)
/monitor --baseline         # Establish baseline metrics for future comparison
/monitor [url]              # Check specific deployment URL (overrides auto-detection)
```

### Mode Behavior

| Mode | Stages Run | Estimated Time | Use Case |
|------|-----------|---------------|----------|
| Full (default) | 0-7 | 3-5 min | Post-deploy verification |
| `--quick` | 0-3, 7 | 1-2 min | Quick sanity check |
| `--routes-only` | 0-2, 7 | 30-60 sec | "Is the site up?" |
| `--env-only` | 0, 4, 7 | 15-30 sec | After env var changes |
| `--baseline` | 0-7 | 3-5 min | First run on new project |
| `[url]` | 0-7 | 3-5 min | Check specific deployment |

---

## CRITICAL RULES

1. **NEVER modify deployment state.** All checks are read-only. No POST/PUT/DELETE to production endpoints.
2. **NEVER expose or log environment variable VALUES.** Only check existence (is it set?), never the content.
3. **Report is the single source of truth.** Updated after every stage. If context resets, the report has all findings.
4. **Status updates every 30-60 seconds.** User must always know what stage you're in and what was found.
5. **Auto-detect platform but fall back gracefully.** If platform detection fails, HTTP checks still work.
6. **Respect rate limits.** Space HTTP requests at least 100ms apart. Never DDoS the deployment.
7. **Use Vercel MCP when available.** If Vercel is detected, use runtime logs and deployment info for deeper insight.
8. **Compare against baselines when available.** First run establishes baselines; subsequent runs compare.
9. **Report EVERY route, not a sample.** Check all discovered routes. No statistical sampling.
10. **Fail open on platform-specific checks.** If a platform API is unavailable, mark those checks as DEFERRED, not FAILED.
11. **Sub-agents return < 500 tokens.** Write full results to disk.
12. **NEVER ask the user for guidance mid-stage** unless a Human Decision Trigger fires.
13. **This is a GLOBAL skill.** Works on any project, any framework, any platform.
14. **Create `.monitor-reports/` directory and ensure it's gitignored.**

---

## SCORING SYSTEM

### Categories (Weighted)

| Category | Weight | What's Measured | Scoring Criteria |
|----------|--------|----------------|-----------------|
| Route Health | 25% | All routes return 2xx/3xx | 100: all pass. -10 per 4xx. -25 per 5xx. 0 if any critical route down |
| Response Time | 20% | P50/P95 within thresholds | 100: all under baseline×1.2. -5 per route over baseline×1.5. -15 per route over baseline×3 |
| Function Health | 15% | Serverless/edge functions execute | 100: all pass. -20 per function with errors. 0 if >50% failing |
| Environment Config | 15% | Required env vars present | 100: all present. -10 per missing non-critical. -25 per missing critical |
| Infrastructure | 15% | DNS, SSL, CDN, headers | 100: all pass. -15 per issue. -50 for SSL expiry <30 days. 0 for SSL expired |
| Error Rate | 10% | Runtime errors in last hour | 100: zero errors. -5 per unique error. -20 per recurring error. 0 if >10 unique errors |

### Grade Scale

| Score | Grade | Ship Decision |
|-------|-------|---------------|
| 95-100 | A+ | Healthy — no action needed |
| 90-94 | A | Healthy — minor items to watch |
| 85-89 | B+ | Mostly healthy — address flagged items |
| 80-84 | B | Acceptable — some degradation detected |
| 70-79 | C | Degraded — investigate soon |
| 60-69 | D | Unhealthy — investigate immediately |
| 0-59 | F | Critical — possible incident, consider /incident |

---

## ENVIRONMENT DETECTION

Detect hosting platform from project files:

| Indicator | Platform | Extra Capabilities |
|-----------|----------|--------------------|
| `vercel.json` or `.vercel/` directory | Vercel | Runtime logs via MCP, deployment info, build logs |
| `netlify.toml` or `.netlify/` | Netlify | Build logs (if CLI available) |
| `amplify.yml` or `buildspec.yml` | AWS Amplify | CloudWatch (if CLI available) |
| `fly.toml` | Fly.io | Fly CLI logs (if available) |
| `Dockerfile` + no platform config | Docker/Generic | Container health checks |
| `wrangler.toml` | Cloudflare Workers | Workers logs (if CLI available) |
| None of the above | Generic | HTTP checks only |

### Vercel-Specific Integration

When Vercel is detected, unlock these capabilities:
- `mcp__vercel__list_deployments` — Get latest deployment info
- `mcp__vercel__get_deployment` — Check deployment status and metadata
- `mcp__vercel__get_runtime_logs` — Check for runtime errors (last hour)
- `mcp__vercel__get_deployment_build_logs` — Verify build succeeded
- `mcp__vercel__web_fetch_vercel_url` — Fetch protected deployment URLs

### Deployment URL Resolution

Priority order for finding the deployment URL:
1. Explicit URL passed as argument (`/monitor https://example.com`)
2. Vercel MCP: latest production deployment URL
3. `vercel.json` → project name → `https://{project}.vercel.app`
4. `package.json` homepage field
5. Ask user for URL (Human Decision Trigger)

---

## PRE-FLIGHT CHECKS (Stage 0 Sub-Steps)

Before any monitoring begins:

1. **Detect project metadata** — framework (Next.js, Remix, etc.), package manager, project name
2. **Detect hosting platform** — scan for platform config files (see Environment Detection)
3. **Resolve deployment URL** — follow priority order above
4. **Verify URL is reachable** — single HEAD request to confirm connectivity
5. **Check for previous runs** — look for `.monitor-reports/state-*.json` within last 2 hours
6. **Load baseline** — if `baseline.json` exists in `.monitor-reports/`, load response time baselines
7. **Check for cross-skill context** — look for recent `/qatest`, `/sec-ship`, `/gh-ship` reports
8. **Create report infrastructure** — mkdir `.monitor-reports/`, initialize report file and state file
9. **Parse mode flags** — determine which stages to run based on flags

---

## STAGE 0: INITIALIZATION

### Actions

1. Parse any arguments (URL, flags)
2. Detect project framework and platform
3. Resolve deployment URL
4. Verify basic connectivity (HEAD request)
5. Check for resumable state (last 2 hours)
6. Load baselines if they exist
7. Create report infrastructure

### Report Template (Created at Stage 0)

```markdown
# Monitor Report: [PROJECT_NAME]
**Run ID:** MON-YYYYMMDD-HHMMSS
**Date:** YYYY-MM-DD HH:MM:SS
**Platform:** [Detected Platform]
**URL:** [Deployment URL]
**Mode:** [full | quick | routes-only | env-only | baseline]

## Progress
- [ ] Stage 1: Route Discovery
- [ ] Stage 2: Endpoint Health
- [ ] Stage 3: Function Verification
- [ ] Stage 4: Environment Validation
- [ ] Stage 5: Infrastructure Health
- [ ] Stage 6: Error Tracking
- [x] Stage 7: Scoring & Report

## Health Score
> Pending...

## Findings

### Route Health
> Pending...

### Response Times
> Pending...

### Function Health
> Pending...

### Environment Config
> Pending...

### Infrastructure
> Pending...

### Error Rate
> Pending...

## SITREP
> Pending...
```

### State File Template (Created at Stage 0)

```json
{
  "skill": "monitor",
  "runId": "MON-YYYYMMDD-HHMMSS",
  "project": "project-name",
  "platform": "vercel|netlify|aws|fly|docker|generic",
  "url": "https://deployment-url.com",
  "mode": "full|quick|routes-only|env-only|baseline",
  "started": "ISO-8601",
  "status": "in_progress",
  "stagesCompleted": [],
  "stagesRemaining": [1, 2, 3, 4, 5, 6, 7],
  "results": {},
  "baseline": null,
  "lastCheckpoint": "ISO-8601"
}
```

---

## STAGE 1: ROUTE DISCOVERY

**Agent:** 1 haiku agent for filesystem scanning

### Actions

1. **Scan filesystem routes:**
   - Next.js: `app/**/page.tsx`, `pages/**/*.tsx`
   - Remix: `app/routes/**/*.tsx`
   - SvelteKit: `src/routes/**/+page.svelte`
   - Astro: `src/pages/**/*.astro`
   - Generic: scan for route config files

2. **Scan API routes:**
   - Next.js: `app/api/**/route.ts`, `pages/api/**/*.ts`
   - Framework-specific API patterns
   - `serverless/`, `functions/`, `api/` directories

3. **Parse dynamic routes:**
   - Extract parameter patterns (`[id]`, `[...slug]`, etc.)
   - For dynamic routes, note they need sample values to test

4. **Check sitemap:**
   - Parse `sitemap.xml` or `sitemap.ts` for additional URLs
   - Cross-reference with filesystem routes

5. **Check redirects:**
   - Parse `next.config.js` redirects/rewrites
   - Parse `vercel.json` redirects
   - Parse `_redirects` file (Netlify)

6. **Build route manifest:**
   ```json
   {
     "pages": ["/", "/about", "/contact"],
     "api": ["/api/health", "/api/contact"],
     "dynamic": ["/blog/[slug]"],
     "redirects": [{"/old": "/new"}],
     "total": 31
   }
   ```

7. **Write manifest to state file**

### Output to Report

```markdown
### Route Discovery
- **Page Routes:** 23
- **API Routes:** 8
- **Dynamic Routes:** 5 (need sample values)
- **Redirects:** 3
- **Total Checkable:** 31

<details>
<summary>Full Route Manifest</summary>
[table of all routes]
</details>
```

---

## STAGE 2: ENDPOINT HEALTH

**Agents:** 1-2 sonnet agents (parallel if >20 routes)

### Actions

1. **HTTP GET every page route:**
   - Send GET request to each route on deployment URL
   - Record: status code, response time (ms), content-type, content-length
   - For 3xx: follow redirect, record final destination
   - For 4xx/5xx: record error, flag as finding

2. **HTTP GET/HEAD every API route:**
   - Send appropriate method (GET for read endpoints, HEAD for others)
   - Record: status code, response time, content-type
   - Note: POST/PUT/DELETE endpoints only get HEAD (non-destructive rule)

3. **Test dynamic routes with known values:**
   - Use values from sitemap, database slugs, or common patterns
   - If no values available, mark as DEFERRED

4. **Check redirect chains:**
   - Verify each redirect resolves correctly
   - Flag redirect loops or chains > 3 hops

5. **Measure response times:**
   - Record P50 (median) and P95 response times
   - Compare against baseline if available
   - Flag routes significantly slower than baseline (>1.5x)

6. **Rate limiting:** Space requests at least 100ms apart. If 429 received, back off exponentially.

### Findings Format

```markdown
| Route | Status | Time (ms) | vs Baseline | Finding |
|-------|--------|-----------|-------------|---------|
| / | 200 | 142 | +5% | HEALTHY |
| /about | 200 | 89 | -2% | HEALTHY |
| /api/contact | 500 | 2340 | N/A | DOWN — MON-2-001 |
| /blog/[slug] | — | — | — | DEFERRED — no test values |
```

---

## STAGE 3: FUNCTION VERIFICATION

**Agent:** 1 sonnet agent

### Actions

1. **Identify serverless/edge functions:**
   - API routes that return dynamic content
   - Server-side rendered pages
   - Edge middleware
   - Scheduled functions (cron jobs)

2. **Verify function execution:**
   - For API routes: check response is valid JSON/HTML (not error page)
   - For SSR pages: verify dynamic content is present (not static fallback)
   - For middleware: verify headers are being set correctly

3. **Check function error rates (Vercel-specific):**
   - Use `mcp__vercel__get_runtime_logs` with `level: ["error", "warning"]`
   - Count errors in last hour
   - Identify recurring error patterns

4. **Check cold start behavior:**
   - First request timing vs subsequent request timing
   - Flag functions with cold starts > 3 seconds

5. **Verify edge functions (if applicable):**
   - Check edge function routes respond from edge (check headers)
   - Verify geo-routing if configured

### Findings Format

```markdown
### Function Health
| Function | Type | Status | Error Rate | Cold Start | Finding |
|----------|------|--------|------------|------------|---------|
| /api/contact | Serverless | ✅ | 0% | 1.2s | HEALTHY |
| /api/auth | Edge | ⚠️ | 2.3% | N/A | DEGRADED — MON-3-001 |
```

---

## STAGE 4: ENVIRONMENT VALIDATION

**Agent:** 1 haiku agent

### Actions

1. **Identify required environment variables:**
   - Scan codebase for `process.env.VARIABLE_NAME` patterns
   - Scan for `env.VARIABLE_NAME` in server components
   - Check `.env.example`, `.env.local.example`, `.env.template` files
   - Check framework config files for env references

2. **Categorize variables by criticality:**
   - **Critical:** Database URLs, auth secrets, API keys for core features
   - **Important:** Analytics, email service, CDN config
   - **Optional:** Feature flags, debug settings

3. **Verify existence (NOT values):**
   - For Vercel: check if env vars are set via deployment metadata
   - For generic: check if app boots without errors related to missing env vars
   - Check for common misconfiguration: env var set in preview but not production

4. **Check for exposed secrets:**
   - Verify no env vars are leaked in client-side JavaScript bundles
   - Check that `NEXT_PUBLIC_` vars don't contain secrets
   - Verify `.env` files are not accessible via HTTP

### Findings Format

```markdown
### Environment Configuration
| Variable Pattern | Category | Status | Finding |
|-----------------|----------|--------|---------|
| DATABASE_URL | Critical | ✅ Set | HEALTHY |
| NEXT_PUBLIC_* (4 vars) | Important | ✅ Set | HEALTHY |
| SENTRY_DSN | Optional | ⚠️ Not set | DEGRADED — MON-4-001 |
```

**CRITICAL: Never log, display, or store actual environment variable values. Only report existence/absence.**

---

## STAGE 5: INFRASTRUCTURE HEALTH

**Agent:** 1 sonnet agent

### Actions

1. **DNS Resolution:**
   - Resolve deployment hostname
   - Check for CNAME/A record correctness
   - Verify DNS propagation (check from multiple resolvers if possible)
   - Flag DNS misconfiguration

2. **SSL Certificate:**
   - Check certificate validity dates
   - Flag if expiring within 30 days (warning) or 7 days (critical)
   - Verify certificate chain is complete
   - Check for mixed content warnings

3. **CDN & Cache Headers:**
   - Check `Cache-Control` headers on static assets
   - Verify CDN is serving content (check `x-vercel-cache`, `cf-cache-status`, etc.)
   - Check for proper `Vary` headers
   - Flag missing cache headers on static assets

4. **Security Headers:**
   - Check for: `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`
   - Check for: `Content-Security-Policy`, `Referrer-Policy`
   - Note: don't flag missing headers as critical — this is a health check, not sec-ship

5. **HTTP/2 or HTTP/3:**
   - Verify modern protocol support
   - Check for proper ALPN negotiation

6. **Compression:**
   - Verify gzip/brotli compression on text assets
   - Flag uncompressed large responses

### Findings Format

```markdown
### Infrastructure Health
| Check | Status | Details | Finding |
|-------|--------|---------|---------|
| DNS Resolution | ✅ | Resolves to 76.76.21.21 | HEALTHY |
| SSL Certificate | ⚠️ | Expires in 21 days | DEGRADED — MON-5-001 |
| CDN Cache | ✅ | 85% cache hit rate | HEALTHY |
| Security Headers | ✅ | 5/5 present | HEALTHY |
| HTTP/2 | ✅ | Supported | HEALTHY |
| Compression | ✅ | Brotli enabled | HEALTHY |
```

---

## STAGE 6: ERROR TRACKING

**Agent:** 1 sonnet agent

### Actions

1. **Platform-specific log analysis:**

   **Vercel (via MCP):**
   - `mcp__vercel__get_runtime_logs` — last hour, level: `["error", "fatal"]`
   - Count unique error messages
   - Identify error frequency and patterns
   - Check for error spikes (sudden increase)

   **Generic:**
   - Check application health endpoint (e.g., `/api/health`, `/healthz`)
   - Look for error patterns in HTTP responses
   - Check for 5xx response patterns from Stage 2 results

2. **Categorize errors:**
   - **Runtime errors:** Unhandled exceptions, crashes
   - **Integration errors:** Failed external API calls, database timeouts
   - **Configuration errors:** Missing env vars, invalid config
   - **User-facing errors:** 500 pages served to users

3. **Correlate with recent deploys:**
   - If errors started after latest deploy, flag as deployment-related
   - If errors existed before, flag as pre-existing
   - Use `mcp__vercel__list_deployments` for deploy timeline

4. **Error rate calculation:**
   - Total errors in last hour / total requests (if available)
   - Compare against baseline error rate

### Findings Format

```markdown
### Error Tracking (Last Hour)
| Error Pattern | Count | First Seen | Category | Finding |
|--------------|-------|------------|----------|---------|
| "Cannot read property 'id' of undefined" | 12 | 14:22 | Runtime | FOUND — MON-6-001 |
| "ETIMEDOUT: connection to db" | 3 | 14:45 | Integration | FOUND — MON-6-002 |
| **Total unique errors:** 2 | **Total occurrences:** 15 |
```

---

## STAGE 7: SCORING & REPORT

**No agent needed — orchestrator handles scoring and report finalization.**

### Actions

1. **Calculate category scores:**
   - Apply scoring criteria from each category
   - Apply weights to get weighted score
   - Calculate final health score (0-100)

2. **Determine grade and ship decision:**
   - Map score to grade using grade scale
   - Generate actionable recommendation

3. **Compare against baseline (if available):**
   - Show delta from previous run
   - Highlight improvements and regressions

4. **Generate SITREP:**
   - Before/After metrics table (if baseline exists)
   - What was checked
   - Issues found and severity
   - Recommendations
   - Historical context (if previous runs exist)

5. **Save baseline (if `--baseline` mode):**
   - Write response times and scores to `baseline.json`
   - Mark as reference for future comparisons

6. **Finalize report and state file**

7. **Console summary:**

```
📊 Monitor Health Check Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Health Score: 87/100 (B+)

Route Health:     92/100 (25%) ████████████████████████░░░░
Response Time:    85/100 (20%) ████████████████████░░░░░░░░
Function Health:  90/100 (15%) ██████████████████████░░░░░░
Environment:      100/100 (15%) ████████████████████████████
Infrastructure:   80/100 (15%) ████████████████████░░░░░░░░
Error Rate:       70/100 (10%) ██████████████████░░░░░░░░░░

Findings: 2 issues (1 degraded, 1 down)
Report: .monitor-reports/MON-20260224-143022.md
```

---

## SITREP REQUIREMENTS

The SITREP section of the report must include:

### 1. Before/After Metrics Table

| Metric | Baseline | Current | Delta |
|--------|----------|---------|-------|
| Routes Healthy | 31/31 | 30/31 | -1 |
| Avg Response (ms) | 120 | 145 | +21% |
| Function Error Rate | 0% | 2.3% | +2.3% |
| Infrastructure Score | 100 | 80 | -20 |
| Overall Health | 95 | 87 | -8 |

(If no baseline exists, show "Current" column only with "N/A — establishing baseline" for Before)

### 2. What Was Checked

Bullet list of all checks performed with counts.

### 3. Issues Found

Each issue with severity, description, and recommended action.

### 4. Deferred and Why

Items that couldn't be checked, with conditions under which they should be revisited.

### 5. Recommendations

Prioritized list of actions to improve health score.

### 6. Historical Context

If previous runs exist, show trend direction (improving, stable, degrading).

---

## DEFINITION OF DONE

A monitor run is complete when:

- [ ] All stages for the selected mode have executed
- [ ] Every discovered route has been checked (or explicitly DEFERRED with reason)
- [ ] Health score calculated with weighted categories
- [ ] Report file is complete with all sections populated
- [ ] State file is updated with `status: "completed"`
- [ ] SITREP section is written with all 6 required parts
- [ ] Console summary displayed to user
- [ ] Baseline saved (if `--baseline` mode)
- [ ] No stage left in `stagesRemaining` array

---

## OUTPUT STRUCTURE

```
.monitor-reports/
├── MON-YYYYMMDD-HHMMSS.md          # Main health report (living document)
├── state-YYYYMMDD-HHMMSS.json      # Machine-readable state for resume
├── baseline.json                     # Response time baselines (persists across runs)
└── history.json                      # Run history for trend tracking
```

### History File Format

```json
{
  "runs": [
    {
      "runId": "MON-YYYYMMDD-HHMMSS",
      "date": "ISO-8601",
      "score": 87,
      "grade": "B+",
      "routesHealthy": "30/31",
      "issuesFound": 2,
      "mode": "full"
    }
  ]
}
```

### Gitignore

Ensure `.monitor-reports/` is in `.gitignore`:
```bash
# Check if already gitignored
grep -q 'monitor-reports' .gitignore 2>/dev/null || echo '.monitor-reports/' >> .gitignore
```

---

## CROSS-SKILL INTEGRATION

### Skills That Feed Into /monitor
| Skill | What It Provides | Where to Check |
|-------|-----------------|----------------|
| `/gh-ship` | Recent deploy info, PR that was shipped | `.ship-reports/` |
| `/qatest` | Known issues, test results | `.qatest-reports/` |
| `/sec-ship` | Security findings that might affect health | `.security-reports/` |

### Skills That /monitor Feeds Into
| Skill | When to Recommend | Trigger |
|-------|------------------|---------|
| `/incident` | Health score < 60 or critical route down | Auto-recommend in report |
| `/qatest` | Degraded routes need deeper testing | Recommend in report |
| `/sec-ship` | Security headers missing or SSL issues | Recommend in report |
| `/perf` | Response times significantly degraded | Recommend in report |

---

## ERROR RECOVERY

### Deployment URL Unreachable
1. Verify URL is correct (check for typos)
2. Try with and without `www.` prefix
3. Check DNS resolution directly
4. If Vercel: check deployment status via MCP
5. If still unreachable: report as critical finding and complete remaining checks that don't require HTTP access

### Rate Limited (429)
1. Back off exponentially: 1s, 2s, 4s, 8s
2. After 4 retries, mark remaining routes as DEFERRED
3. Note rate limiting in findings

### Platform API Unavailable
1. Fall back to HTTP-only checks
2. Mark platform-specific checks as DEFERRED
3. Note reduced coverage in report

### Context Reset
1. Read state file from `.monitor-reports/state-*.json`
2. Resume from last completed stage
3. Re-read report file for existing findings
4. Continue pipeline from next stage

---

## INCREMENTAL CHECKS

When run after a previous baseline exists:

1. Load `baseline.json` for response time comparisons
2. Load `history.json` for trend analysis
3. Compare all metrics against baseline
4. Flag any metric that degraded by >20% as a finding
5. Update history with current run results

---

## REMEMBER

1. You are a health checker, not a fixer. Report issues, don't fix them. Recommend the right skill for fixing.
2. Every route gets checked. No sampling. No skipping.
3. Non-destructive always. GET and HEAD only. Never POST/PUT/DELETE to production.
4. Environment variable VALUES are never logged. Only check existence.
5. The report is the source of truth. Update it after every stage.
6. Compare against baselines when available. Absolute thresholds are less useful than relative change.
7. When health score < 60, recommend `/incident`. The user needs structured incident response, not more monitoring.
8. Platform-specific features are bonus depth, not requirements. HTTP checks are the foundation.
9. Rate limit your own requests. You are checking health, not creating a DDoS.
10. First run establishes the baseline. Tell the user to run `/monitor --baseline` on a known-good deployment.

---

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
