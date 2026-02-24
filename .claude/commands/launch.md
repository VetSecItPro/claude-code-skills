---
description: Full launch readiness pipeline — orchestrates all skills + 5 unique checks, scored GO/NO-GO verdict

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

Status examples: "Running security audit (2/8 skills)...", "Checking SEO readiness...", "Aggregating scores...", "Generating launch report..."

---

allowed-tools: Bash(pnpm *), Bash(npx *), Bash(npm *), Bash(yarn *), Bash(cat *), Bash(ls *), Bash(grep *), Bash(find *), Bash(head *), Bash(tail *), Bash(wc *), Bash(echo *), Bash(mkdir *), Bash(date *), Bash(git *), Bash(du *), Read, Write, Edit, Glob, Grep, Task

---

# /launch — Full Launch Readiness Pipeline

**One command. Every skill. Every gap. Scored verdict. Ship with confidence.**

**FULL ORCHESTRATOR** — Spawns sub-agents to run existing skills (audit-only), then performs 5 unique domain checks no other skill covers. Aggregates everything into a unified scorecard with GO/NO-GO verdict.

---

## CRITICAL RULES

1. **AUDIT + AUTO-FIX** — Sub-agents scan, fix everything they safely can, and report what was fixed vs. deferred. Full skill runs, not audit-only.
2. **CONTEXT MANAGEMENT IS MANDATORY** — Follow the Context Management Protocol below. This skill WILL fail if context is not managed.
3. **SEQUENTIAL SKILL EXECUTION** — Run one skill sub-agent at a time. Process its summary. Write to disk. Move to the next.
4. **SUB-AGENT RETURN BUDGET: 500 TOKENS MAX** — Full reports go to disk. Only structured summaries come back to the orchestrator.
5. **DISK IS THE SOURCE OF TRUTH** — The scorecard lives on disk, not in memory. If context resets, read the scorecard to resume.
6. **SCORE HONESTLY** — A false GO is worse than a NO-GO. Users need truth before launch.
7. **WRITE TO DISK IMMEDIATELY** — Report file created before any scanning starts.
8. **DEFERRED TABLE IS MANDATORY** — Every finding that couldn't be auto-fixed gets a row in the Deferred Items table with a clear explanation of WHY it was deferred and WHAT needs to happen.
9. **REAL-TIME UPDATES** — The markdown report on disk reflects current state at ALL times. Fixed items, deferred items, and in-progress items are updated as they happen, never batched to the end.

---

## CONTEXT MANAGEMENT PROTOCOL (CRITICAL)

<!--
This is the most important section of this skill. The orchestrator will spawn 8 sub-agents
and perform 5 direct checks. Without context management, the orchestrator WILL run out
of context and the user will have to restart, losing all progress.
-->

### The Problem

When the orchestrator spawns sub-agents via the Task tool, each sub-agent returns results to the orchestrator's context. If 8 sub-agents each return detailed findings, the orchestrator's context fills up and the session dies. The user then has to `/clear` and restart, losing everything.

### The Solution: Disk-Based State + Lean Returns

**Rule 1: Sub-agents write full reports to disk, return only summaries.**

Every sub-agent gets this instruction in its prompt:
> "Return ONLY a structured summary under 500 tokens. Your full report is already on disk — do NOT echo it back."

**Rule 2: The orchestrator maintains a scorecard file on disk.**

After each skill completes, immediately update the scorecard file:
```
.launch-reports/scorecard-YYYYMMDD-HHMMSS.json
```

This file tracks:
```json
{
  "project": "my-app",
  "profile": "SaaS App",
  "started": "2026-02-12T10:00:00Z",
  "skills_completed": ["sec-ship", "deps"],
  "skills_remaining": ["compliance", "perf", "a11y", "cleancode", "test-ship", "docs"],
  "unique_domains_completed": [],
  "scores": {
    "sec-ship": { "score": 85, "critical": 0, "warnings": 3, "report": ".security-reports/..." },
    "deps": { "score": 92, "critical": 0, "warnings": 1, "report": ".deps-reports/..." }
  }
}
```

**Rule 3: If context feels heavy, checkpoint and continue.**

After every 3 skills, assess context usage. If the conversation is getting long:
- Ensure the scorecard JSON is up to date on disk
- Summarize what's been done so far in a brief message
- Continue with the next skill — do NOT stop and ask the user

**Rule 4: Resume protocol.**

At startup, check for an incomplete scorecard from the last 2 hours:
```bash
find .launch-reports/ -name "scorecard-*.json" -mmin -120
```

If found and `skills_remaining` is non-empty:
- Read the scorecard
- Report to the user: "Resuming launch check — X/13 checks already complete"
- Skip completed skills, resume from the next one
- This way, even if context runs out and the user restarts, NO work is lost

**Rule 5: NEVER read full sub-agent reports into orchestrator context.**

The sub-agent's `.security-reports/sec-ship-*.md` file might be 500+ lines. NEVER `Read` it into the orchestrator. The summary returned by the sub-agent (< 500 tokens) is all the orchestrator needs. Users read the full reports themselves from disk.

**Rule 6: Keep orchestrator messages lean.**

When presenting progress to the user, keep it short:
```
Security audit complete: 85/100 (0 critical, 3 warnings)
Full report: .security-reports/sec-ship-20260212-100500.md
Next: Dependencies...
```

Do NOT dump findings, file lists, or detailed results into the orchestrator's output. The scorecard and individual reports on disk have all the detail.

---

### Context Management Standard (For All Skills)

This pattern should be applied to ANY skill that spawns sub-agents:

1. **Sub-agents return < 500 tokens** — Full output goes to disk
2. **State lives on disk** — JSON checkpoint file updated after each step
3. **Resume from checkpoint** — Check for recent incomplete state at startup
4. **Never ingest sub-agent reports** — Only read summaries
5. **Sequential execution** — One sub-agent at a time unless truly independent
6. **Checkpoint every 3 steps** — Assess context, persist state

---

## REPORT PERSISTENCE

### Report Location
```
.launch-reports/
├── launch-YYYY-MM-DD-HHMMSS.md       # Final unified report
└── scorecard-YYYYMMDD-HHMMSS.json    # Machine-readable state (for resume)
```

### Rules
- Create `.launch-reports/` directory at start
- Write scorecard JSON AND the markdown report skeleton IMMEDIATELY (before any scanning)
- **THE MARKDOWN REPORT IS A LIVING DOCUMENT** — update it after EVERY skill/domain completes, not at the end
- After each check: fill in that row in the scorecard table, add findings to Critical Blockers/Warnings sections
- If context dies mid-run, the user can open the `.md` file and see exactly what's done and what's pending
- Sections not yet completed show `⏳ Pending` — they get filled in as work progresses
- Update the scorecard JSON in parallel (machine-readable state for resume)
- **NEVER delete previous reports** — they're historical snapshots

---

## EXECUTION FLOW

### Phase 0: Setup & Detection

**0.1 Create output directory:**
```bash
mkdir -p .launch-reports
```

**0.2 Check for resumable state:**
```bash
find .launch-reports/ -name "scorecard-*.json" -mmin -120
```
If found and incomplete, resume from where it left off.

**0.3 Detect project profile:**

Read `package.json` to determine:

| Signal | Profile |
|--------|---------|
| Has `@supabase/*` + auth pages + Stripe/Polar | **SaaS App** |
| No auth, no database, primarily pages | **Marketing Site** |
| Has `@capacitor/*` dependencies | **Mobile App** (also SaaS if has auth) |

**0.4 Initialize scorecard JSON:**

Write the initial scorecard with all skills listed in `skills_remaining`.

**0.5 Initialize the markdown report (LIVING DOCUMENT):**

Write the full report skeleton to `.launch-reports/launch-YYYY-MM-DD-HHMMSS.md` NOW — before any scanning starts. This includes:
- Header with project name, date, profile, branch
- VERDICT section showing `⏳ IN PROGRESS`
- Scorecard table with all 13 rows, each showing `⏳ Pending` for score/grade
- Empty Critical Blockers and Warnings tables
- All 5 Unique Domain Detail sections with `⏳ Pending` placeholders
- Skill Report Locations table (paths TBD)

This way, if the session dies at ANY point, the user can open the `.md` and see:
- Which checks completed (with real scores)
- Which checks are still pending
- All findings discovered so far

**0.6 Report project detection to user:**
```
Launch readiness check starting...
Project: [name]
Profile: [SaaS App / Marketing Site / Mobile App]
Checks: 8 skill audits + 5 unique domains = 13 total
```

---

### Phase 1: Skill Runs — Audit + Auto-Fix (Sub-agents, Sequential)

Run each skill as a sub-agent via the Task tool. **One at a time. Full run: scan, fix what's safe, defer what isn't.**

#### Skills to Run

| Order | Skill | Domain | Why Essential |
|-------|-------|--------|---------------|
| 1 | `/sec-ship` | Security | Non-negotiable for launch |
| 2 | `/deps` | Dependencies | Vulnerable deps block launch |
| 3 | `/compliance` | Legal/Privacy | Legal exposure blocks launch |
| 4 | `/perf` | Performance | Poor perf loses users on day 1 |
| 5 | `/a11y` | Accessibility | Legal requirement + ethics |
| 6 | `/cleancode` | Tech Debt | Code hygiene for maintainability |
| 7 | `/test-ship` | Testing | Test coverage for confidence |
| 8 | `/docs` | Documentation | Contributor/maintenance readiness |

#### Sub-agent Prompt Template

For each skill, spawn a Task agent with `subagent_type: "general-purpose"` and this prompt:

```
You are running a LAUNCH READINESS pass for the project at [PROJECT_PATH].

INSTRUCTIONS:
1. Read the skill methodology from ~/.claude/commands/[SKILL].md
2. Follow the skill's FULL pipeline — audit AND auto-fix everything that can be safely fixed
3. Write your full report to the skill's standard report directory (e.g., .security-reports/)
4. Track what you FIXED vs. what you DEFERRED (couldn't fix safely or needs human decision)
5. When complete, return ONLY this structured summary (MUST be under 500 tokens):

---
SKILL: [skill-name]
SCORE: [0-100, POST-fix score — reflects state after auto-fixes applied]
GRADE: [A/B/C/D/F]
FIXED: [count of issues auto-fixed]
DEFERRED: [count of issues that need manual work]
CRITICAL_BLOCKERS: [count remaining AFTER fixes]
WARNINGS: [count remaining AFTER fixes]
FIXED_ITEMS:
1. [one-line: what was fixed + file path]
2. [one-line]
3. [one-line]
DEFERRED_ITEMS:
1. [one-line: what + why deferred (e.g., "needs architectural change", "requires human decision")]
2. [one-line: what + why]
3. [one-line: what + why]
REPORT: [full path to report file on disk]
---

CRITICAL RULES:
- AUTO-FIX everything safe — the skill's methodology defines what's safe to auto-fix
- DEFER items that need architectural changes, human decisions, or would break functionality
- For each DEFERRED item: explain WHY in the summary (not just what)
- DO NOT return full findings lists, file contents, or verbose output
- Your full report is on disk — only return the summary above
- Score guide: 90-100=A (excellent), 80-89=B (good), 70-79=C (fair), 60-69=D (poor), 0-59=F (failing)
- CRITICAL_BLOCKERS = findings that STILL remain after auto-fix (must be fixed before launch)
- WARNINGS = findings that STILL remain after auto-fix (should fix but not launch-blocking)
```

#### After Each Sub-agent Returns

1. Parse the structured summary (SKILL, SCORE, FIXED, DEFERRED, etc.)
2. Update the scorecard JSON on disk:
   - Move skill from `skills_remaining` to `skills_completed`
   - Add score entry to `scores` object (including `fixed_count`, `deferred_count`, `deferred_items`)
3. **Update the markdown report on disk (MANDATORY — do this EVERY time):**
   - Replace that skill's `⏳ Pending` row in the scorecard table with real score/grade/findings
   - Add FIXED items to the Auto-Fixed Items table
   - Add DEFERRED items to the Deferred Items table (with WHY explanation)
   - Add any remaining critical blockers to the Critical Blockers table
   - Update the running totals (fixed count, deferred count, critical count, warning count)
   - The report on disk should ALWAYS reflect current state — never stale
4. Report progress to user (one line):
   ```
   [X/8] Security: 85/100 (B) — 5 fixed, 2 deferred, 0 critical remaining
   ```
5. Proceed to next skill

#### Parallelism

**Default: Sequential** (safest for context management).

If the user explicitly requests speed, these pairs CAN run in parallel (they scan different things):
- `/sec-ship` + `/deps` (security vs dependencies)
- `/perf` + `/a11y` (performance vs accessibility)
- `/cleancode` + `/docs` (code quality vs documentation)

**Never run more than 2 in parallel.** Always process results and update scorecard before spawning the next batch.

---

### Phase 2: Unique Domain Checks (Direct)

These 5 checks are performed directly by the orchestrator — no sub-agents needed. They're lightweight grep/glob checks.

#### Domain A: Content Readiness

**Checks (use Grep/Glob directly):**

1. No placeholder text — grep for (case-insensitive, exclude tests/node_modules/.md):
   - `lorem ipsum`, `placeholder text`, `\[TODO\]`, `\[TBD\]`, `\[INSERT\]`
2. 404 page exists — check for `app/not-found.tsx`
3. Error page exists — check for `app/error.tsx` or `app/global-error.tsx`
4. Favicon exists — check for `app/favicon.ico`, `app/icon.tsx`, `app/icon.png`
5. OG image exists — glob for `app/opengraph-image.*` or `public/og*`
6. No "coming soon" on live routes — grep production page files
7. Copyright year current — grep footer components for year

**Scoring (100 pts):**

| Check | Points | CRITICAL? |
|-------|--------|-----------|
| No placeholder text | 20 | No |
| 404 page exists | 20 | YES |
| Error page exists | 15 | No |
| Favicon exists | 15 | No |
| OG image exists | 15 | No |
| No "coming soon" | 10 | No |
| Current copyright year | 5 | No |

#### Domain B: SEO & Meta

**Checks:**

1. Root metadata — read `app/layout.tsx`, check for `metadata` export with `title` and `description`
2. Sitemap — glob for `app/sitemap.ts`, `app/sitemap.xml`, `public/sitemap.xml`
3. Robots — glob for `app/robots.ts`, `robots.txt`, `public/robots.txt`
4. OpenGraph metadata — check metadata export for `openGraph` configuration
5. Viewport — check for `viewport` export in root layout
6. Per-page metadata — sample 5 key route `page.tsx` files for `metadata` exports

**Scoring (100 pts):**

| Check | Points | CRITICAL? |
|-------|--------|-----------|
| Root metadata (title + desc) | 30 | YES |
| Sitemap exists | 20 | No |
| Robots.txt exists | 15 | No |
| OG metadata configured | 15 | No |
| Per-page metadata | 15 | No |
| Viewport configured | 5 | No |

#### Domain C: User Flow Completeness

**Checks:**

1. Auth pages exist — glob for login, signup, sign-in page files
2. Password reset flow — glob for forgot-password, reset-password pages
3. Loading states — count `loading.tsx` files vs total route directories
4. Error boundaries — count `error.tsx` files in route groups
5. Navigation — verify header/nav component exists and is in root layout
6. Pricing page — glob for `**/pricing**/page.tsx` (SaaS only)
7. Empty states — grep for empty state patterns in list/table components

**Scoring (100 pts):**

| Check | Points | CRITICAL? |
|-------|--------|-----------|
| Auth pages (login + signup) | 20 | YES (SaaS) |
| Password reset flow | 10 | No |
| Loading state coverage | 15 | No |
| Error boundaries | 15 | No |
| Navigation present | 15 | No |
| Pricing page (SaaS only) | 15 | No |
| Empty state handling | 10 | No |

#### Domain D: Infrastructure & Environment

**Checks:**

1. `.env.example` exists
2. Env var coverage — cross-reference `process.env.` references in code vs `.env.example` entries
3. Error monitoring — `@sentry/nextjs` in package.json AND config files exist (`sentry.*.config.*`)
4. Email service — `resend` or `@sendgrid/mail` in package.json
5. Vercel project linked — `.vercel/project.json` exists with correct team
6. Database migrations — `supabase/migrations/` exists with files
7. README with setup — `README.md` exists and has content

**Scoring (100 pts):**

| Check | Points | CRITICAL? |
|-------|--------|-----------|
| .env.example exists | 15 | No |
| Env vars documented | 15 | No |
| Error monitoring active | 20 | YES (SaaS) |
| Email service configured | 15 | No (SaaS only) |
| Vercel linked correctly | 10 | No |
| Migrations tracked | 15 | No |
| README present | 10 | No |

#### Domain E: Payment & Subscription

**Skip if no payment system detected** (no Stripe, Polar, or payment deps). Score as 100 with note "N/A".

**Checks (if payment detected):**

1. Webhook endpoint — glob for `**/webhook*` in `app/api/`
2. Subscription state management — grep for subscription context, `canAccess`, tier checks
3. Pricing page — glob for `**/pricing**/page.tsx`
4. Feature gating — grep for tier-based access patterns
5. Free tier available — check pricing data for $0/free
6. Upgrade prompts — grep for upgrade/subscribe patterns
7. Subscription management page — glob for billing/subscription settings

**Scoring (100 pts):**

| Check | Points | CRITICAL? |
|-------|--------|-----------|
| Webhook endpoint | 20 | YES |
| Subscription state | 15 | No |
| Pricing page | 15 | No |
| Feature gating | 15 | No |
| Free tier available | 10 | No |
| Upgrade prompts | 10 | No |
| Subscription management | 15 | No |

#### After Each Domain

Update scorecard JSON and report progress:
```
[10/13] SEO & Meta: 45/100 (F) — 1 critical (no root metadata)
```

---

### Phase 3: Aggregate & Score

After all 13 checks complete:

**3.1 Calculate weighted overall score:**

| Check | SaaS Weight | Marketing Weight | Mobile Weight |
|-------|-------------|------------------|---------------|
| sec-ship | 12% | 8% | 12% |
| deps | 8% | 6% | 8% |
| compliance | 8% | 8% | 8% |
| perf | 8% | 10% | 8% |
| a11y | 8% | 8% | 8% |
| cleancode | 5% | 5% | 5% |
| test-ship | 8% | 5% | 8% |
| docs | 3% | 3% | 3% |
| Content Readiness | 8% | 12% | 8% |
| SEO & Meta | 6% | 12% | 4% |
| User Flow | 10% | 5% | 10% |
| Infrastructure | 8% | 8% | 8% |
| Payment & Subs | 8% | 0% | 8% |
| **Total** | **100%** | **100%** | **100%** |

*Note: When Payment is N/A, redistribute its weight proportionally across other domains.*

**3.2 Determine verdict:**

- **GO**: Overall >= 75 AND zero critical blockers across ALL domains
- **CONDITIONAL**: Overall >= 50 AND <= 3 critical blockers (with clear fixes identified)
- **NO-GO**: Overall < 50 OR 4+ critical blockers OR build/type-check fails

**3.3 Grade scale:**

| Score | Grade |
|-------|-------|
| 90-100 | A |
| 80-89 | B |
| 70-79 | C |
| 60-69 | D |
| 0-59 | F |

---

### Phase 4: Finalize Report & Verdict

**4.1 Finalize the living markdown report (already on disk since Phase 0):**

The report has been updated incrementally after every check. Now finalize it:

```markdown
# Launch Readiness Report — [project-name]

**Date:** YYYY-MM-DD HH:MM
**Profile:** [SaaS App / Marketing Site / Mobile App]
**Branch:** [git branch]

---

## VERDICT: [emoji] [GO / CONDITIONAL / NO-GO]

**Overall Score: XX/100 (Grade X)**

**Critical Blockers:** [count]
**Warnings:** [count]

---

## Scorecard

| # | Domain | Score | Grade | Critical | Warnings | Report |
|---|--------|-------|-------|----------|----------|--------|
| 1 | Security (sec-ship) | XX | X | X | X | .security-reports/... |
| 2 | Dependencies (deps) | XX | X | X | X | .deps-reports/... |
| 3 | Legal/Privacy (compliance) | XX | X | X | X | .compliance-reports/... |
| 4 | Performance (perf) | XX | X | X | X | .perf-reports/... |
| 5 | Accessibility (a11y) | XX | X | X | X | .a11y-reports/... |
| 6 | Tech Debt (cleancode) | XX | X | X | X | .cleancode-reports/... |
| 7 | Testing (test-ship) | XX | X | X | X | .test-reports/... |
| 8 | Documentation (docs) | XX | X | X | X | .docs-reports/... |
| 9 | Content Readiness | XX | X | X | X | (in this report) |
| 10 | SEO & Meta | XX | X | X | X | (in this report) |
| 11 | User Flow | XX | X | X | X | (in this report) |
| 12 | Infrastructure | XX | X | X | X | (in this report) |
| 13 | Payment & Subs | XX | X | X | X | (in this report) |

---

## Auto-Fixed Items

| # | Domain | What Was Fixed | File(s) |
|---|--------|----------------|---------|
| 1 | [domain] | [description of fix] | [file path(s)] |

**Total auto-fixed: [count]**

---

## Deferred Items (Needs Manual Work)

| # | Domain | Finding | Why Deferred | What Needs to Happen | Status |
|---|--------|---------|-------------|----------------------|--------|
| 1 | [domain] | [what's wrong] | [architectural change / human decision / risk] | [specific action needed] | ⏳ PENDING |

**Total deferred: [count]**

When the user asks to work on deferred items, update the Status column in real-time:
- `⏳ PENDING` → `🔧 IN PROGRESS` → `✅ FIXED` or `🚫 BLOCKED`

---

## Critical Blockers (Remaining After Auto-Fix)

| # | Domain | Finding | Remediation |
|---|--------|---------|-------------|
| 1 | [domain] | [what's still wrong] | Run `/[skill]` or [manual action] |

---

## Warnings (Remaining After Auto-Fix)

| # | Domain | Finding | Remediation |
|---|--------|---------|-------------|
| 1 | [domain] | [what's wrong] | [recommendation] |

---

## Unique Domain Details

### Content Readiness — XX/100

| Check | Result | Points |
|-------|--------|--------|
| No placeholder text | [count] found | XX/20 |
| 404 page | [exists/missing] | XX/20 |
| Error page | [exists/missing] | XX/15 |
| Favicon | [exists/missing] | XX/15 |
| OG image | [exists/missing] | XX/15 |
| No "coming soon" | [count] found | XX/10 |
| Copyright year | [current/outdated] | XX/5 |

### SEO & Meta — XX/100

| Check | Result | Points |
|-------|--------|--------|
| Root metadata | [present/missing] | XX/30 |
| Sitemap | [exists/missing] | XX/20 |
| Robots.txt | [exists/missing] | XX/15 |
| OG metadata | [configured/missing] | XX/15 |
| Per-page metadata | [X/Y routes] | XX/15 |
| Viewport | [configured/missing] | XX/5 |

### User Flow — XX/100

[Same format]

### Infrastructure — XX/100

[Same format]

### Payment & Subs — XX/100

[Same format, or "N/A — No payment system detected"]

---

## Skill Report Locations

For detailed findings, read the individual skill reports:

| Skill | Report Path |
|-------|-------------|
| /sec-ship | .security-reports/[filename] |
| /deps | .deps-reports/[filename] |
| /compliance | .compliance-reports/[filename] |
| /perf | .perf-reports/[filename] |
| /a11y | .a11y-reports/[filename] |
| /cleancode | .cleancode-reports/[filename] |
| /test-ship | .test-reports/[filename] |
| /docs | .docs-reports/[filename] |

---

## Recommended Actions (Priority Order)

| # | Action | Skill | Impact |
|---|--------|-------|--------|
| 1 | [highest priority fix] | `/[skill]` | [score improvement estimate] |
| 2 | [next priority] | `/[skill]` or Manual | [impact] |
| 3 | [next priority] | `/[skill]` or Manual | [impact] |

---

## History

| Date | Score | Verdict | Notes |
|------|-------|---------|-------|
| [today] | XX/100 | [verdict] | [brief note] |
```

**4.2 Present visual summary to user:**

```
════════════════════════════════════════════════════════════════
  LAUNCH READINESS — [Project Name]
════════════════════════════════════════════════════════════════

  VERDICT:  [GO / CONDITIONAL / NO-GO]
  SCORE:    XX/100 (Grade X)
  PROFILE:  [SaaS App / Marketing Site / Mobile App]

────────────────────── SKILL AUDITS ────────────────────────────

  Security          XX/100  [bar]  [status]
  Dependencies      XX/100  [bar]  [status]
  Legal/Privacy     XX/100  [bar]  [status]
  Performance       XX/100  [bar]  [status]
  Accessibility     XX/100  [bar]  [status]
  Tech Debt         XX/100  [bar]  [status]
  Testing           XX/100  [bar]  [status]
  Documentation     XX/100  [bar]  [status]

────────────────── UNIQUE DOMAIN CHECKS ────────────────────────

  Content           XX/100  [bar]  [status]
  SEO & Meta        XX/100  [bar]  [status]
  User Flow         XX/100  [bar]  [status]
  Infrastructure    XX/100  [bar]  [status]
  Payments          XX/100  [bar]  [status]

────────────────────────────────────────────────────────────────

  CRITICAL BLOCKERS: [count]
  WARNINGS: [count]

  Top 3 Actions:
  1. [action] → /[skill]
  2. [action] → /[skill]
  3. [action] → Manual

  Full report: .launch-reports/launch-YYYY-MM-DD-HHMMSS.md
  Skill reports: .security-reports/, .deps-reports/, etc.

════════════════════════════════════════════════════════════════
```

Use visual bars: `████████████████████░` (95), `████████████████░░░░` (80), `████████░░░░░░░░░░░░` (40)

---

## WHAT THIS DOES NOT CHECK

Be transparent about limitations:

- **Does not verify production environment** — checks code-level readiness, not live infra.
- **Does not test payment processing end-to-end** — checks code exists, not that Stripe works.
- **Does not validate email deliverability** — checks service is configured, not that emails arrive.
- **Does not do visual design review** — `/design` requires visual inspection not possible in sub-agents.
- **Deferred items need human judgment** — items marked DEFERRED were intentionally skipped because they require architectural decisions, business logic changes, or carry risk. Review them before acting.

---

## SKIPPING SKILLS

If a skill was recently run (within the last 24 hours) and the user doesn't want to re-run it, they can say:

```
/launch --skip sec-ship,deps
```

The orchestrator will:
- Check for the most recent report from the skipped skill
- Read the report's summary/SITREP section for score and findings
- Use those results in the scorecard instead of re-running
- Note in the final report: "Score from previous run (YYYY-MM-DD)"

---

## USAGE

```bash
/launch                          # Full pipeline: audit + auto-fix + deferred table
/launch --skip sec-ship,deps     # Skip recently-run skills
/launch --audit-only             # Audit only, no fixes (old behavior)
/launch --fix-deferred           # Resume and work through deferred items from last run
```

No other arguments needed. Auto-detects project, profile, and applicable domains.

### Fix-Deferred Mode

When the user runs `/launch --fix-deferred` or says "fix the deferred items":

1. Read the most recent launch report from `.launch-reports/`
2. Parse the Deferred Items table
3. Work through each deferred item sequentially:
   - Update its status to `🔧 IN PROGRESS` in the report on disk
   - Attempt the fix (spawn a targeted sub-agent if needed)
   - Update status to `✅ FIXED` or `🚫 BLOCKED` with explanation
   - Update the scorecard scores to reflect the new state
4. After each item, update the report on disk immediately
5. When all deferred items are processed, recalculate the overall score and verdict

---

## WORKFLOW

```
Individual skill runs during development
    ↓
/launch                    ← Audit + auto-fix everything safe, scorecard + deferred table
    ↓
Review deferred items      ← Read the Deferred Items table in the report
    ↓
/launch --fix-deferred     ← Work through deferred items, update report in real-time
    ↓
/launch                    ← Re-run to verify (resumes fast with --skip)
    ↓
/gh-ship                   ← Ship when GO
```

---

## REMEMBER

- **Context management is NOT optional** — Sub-agents return < 500 tokens. Scorecard lives on disk. Resume from checkpoint.
- **Markdown report is a LIVING DOCUMENT** — Created at Phase 0 with placeholders, updated after EVERY check. Never wait until the end to write it.
- **Audit + auto-fix** — Sub-agents run the full skill pipeline: scan, fix what's safe, defer the rest
- **Deferred items table is mandatory** — Every unfixable finding gets a row with WHY it was deferred and WHAT needs to happen
- **Real-time updates** — When working on deferred items, update the status column (`PENDING` → `IN PROGRESS` → `FIXED`/`BLOCKED`) on disk immediately
- **Sequential by default** — One skill at a time. Max 2 parallel if user requests speed.
- **Disk is truth** — Scorecard JSON, markdown report, individual reports all on disk and always current
- **Resume protocol** — Check for incomplete scorecard at startup. Never redo completed work.
- **Score honestly** — A premature GO causes more damage than a NO-GO
- **Be specific** — "3 API routes missing rate limiting in app/api/" not "security could improve"
- **Be actionable** — Every finding maps to a skill or manual action
- **Keep orchestrator lean** — Short progress messages, no data dumps

---

<!-- 
  Claude Code Skill by Steel Motion LLC
  https://steelmotionllc.com/portfolio/software/claude-code-skills
  Licensed under MIT - see LICENSE
-->
