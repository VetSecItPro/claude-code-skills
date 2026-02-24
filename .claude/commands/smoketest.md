---
description: Quick smoke test — scan, auto-fix, report (~2-3 min)
## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

allowed-tools: Bash(npx *), Bash(npm *), Bash(pnpm *), Bash(yarn *), Bash(bun *), Bash(curl *), Bash(lsof *), Bash(cat *), Bash(ls *), Bash(grep *), Bash(find *), Bash(head *), Bash(tail *), Bash(wc *), Bash(echo *), Bash(kill *), Bash(pkill *), Bash(sleep *), Bash(PATH=*), Bash(export *), Bash(mkdir *), Bash(date *), Bash(jq *), Read, Write, Edit, Glob, Grep, Task
---

# /smoketest — Scan, Fix, Report

**One command. Find what's broken. Fix it. Prove it's fixed.**

**FIRE AND FORGET** — Execute autonomously. Fix safe issues. Report everything.

---

## Execution Rules (CRITICAL)

- **NO permission requests** — just execute
- **NO "should I proceed?" questions** — just do it
- **Run ALL checks regardless of failures** — never stop early
- **Auto-fix safe issues** — type errors, lint issues, missing imports, console.logs
- **Never break existing code** — verify build passes AFTER each fix
- **NEVER delete findings** — only update their status
- **Write report to disk AFTER EVERY PHASE** — the report is the single source of truth
- **UPDATE report as fixes happen** — each finding's status changes in real-time
- **Attempt fix up to 3 times** before marking BLOCKED
- **Display status report** — always show summary on screen
- **SITREP annotates everything** — what was fixed, what was deferred and WHY, what was blocked and WHY

---

## Report Persistence (CRITICAL — Survives Compaction/Restart)

The markdown report file is the **living document**. It must be self-contained and updated continuously.

### Finding Statuses

| Status | Meaning |
|--------|---------|
| `FOUND` | Issue discovered during scan, not yet addressed |
| `🔧 FIXING` | Currently being worked on |
| `✅ FIXED` | Fix applied and verified (build passes) |
| `⏸️ DEFERRED` | Intentionally skipping — too risky, needs architectural change, or business decision. Reason documented. |
| `🚫 BLOCKED` | Attempted fix up to 3 times, all failed or broke the build. Reason documented. |
| `NOT APPLICABLE` | Check skipped (e.g., no dev server for page probes) |

### Rules

1. **Write report at Phase 1** — file exists before any scans run, with header and empty tables
2. **Update after each scan layer** — findings added to table as discovered
3. **Update after each fix attempt** — status changes from FOUND → FIXING → FIXED/BLOCKED/DEFERRED
4. **Write to disk after every status change** — if session dies, report shows exactly where things stand
5. **Progress Log** — timestamped entries after each layer and each fix
6. **SITREP section** — for every DEFERRED or BLOCKED finding, document WHY and what was tried

### Resume Protocol

Before creating a new report, check for a recent incomplete one:

1. Find most recent `.smoketest-reports/smoke-*.md`
2. If < 30 minutes old AND not marked COMPLETE → resume it
3. Read Progress Log to find last completed step
4. Skip completed layers, continue from next incomplete step
5. Do NOT re-scan completed layers or re-fix already-FIXED findings

---

## Phase 1: Setup

### 1.1 Detect Environment

```bash
# Package manager
if [ -f "bun.lockb" ]; then PM="bun"; PM_RUN="bun run"
elif [ -f "pnpm-lock.yaml" ]; then PM="pnpm"; PM_RUN="pnpm run"
elif [ -f "yarn.lock" ]; then PM="yarn"; PM_RUN="yarn"
else PM="npm"; PM_RUN="npm run"; fi

# Project name
PROJECT_NAME=$(jq -r '.name // empty' package.json 2>/dev/null || basename "$PWD")

# Node path
export PATH="$HOME/.nvm/versions/node/v22.18.0/bin:$PATH"

# Framework
if grep -q '"next"' package.json 2>/dev/null; then FRAMEWORK="nextjs"
elif grep -q '"svelte"' package.json 2>/dev/null; then FRAMEWORK="sveltekit"
elif grep -q '"nuxt"' package.json 2>/dev/null; then FRAMEWORK="nuxt"
elif grep -q '"astro"' package.json 2>/dev/null; then FRAMEWORK="astro"
else FRAMEWORK="unknown"; fi

# Test runner
if grep -q '"vitest"' package.json 2>/dev/null; then TEST_RUNNER="vitest"
elif grep -q '"jest"' package.json 2>/dev/null; then TEST_RUNNER="jest"
else TEST_RUNNER="none"; fi

# Lint
HAS_LINT=false
if jq -e '.scripts.lint' package.json > /dev/null 2>&1; then HAS_LINT=true; fi

# Dev server
DEV_SERVER_UP=false
for port in 3000 3001 5173 5888 4321 8080; do
  if lsof -ti:$port > /dev/null 2>&1; then
    DEV_PORT=$port; DEV_SERVER_UP=true; break
  fi
done
```

### 1.2 Create Report Directory

```bash
mkdir -p .smoketest-reports
```

Ensure `.smoketest-reports/` is in `.gitignore`. If not, add it.

### 1.3 Check for Resumable Report

```bash
LATEST=$(ls -t .smoketest-reports/smoke-*.md 2>/dev/null | head -1)
if [ -n "$LATEST" ]; then
  AGE=$(( $(date +%s) - $(stat -f %m "$LATEST") ))
  if [ "$AGE" -lt 1800 ] && ! grep -q "COMPLETE" "$LATEST"; then
    REPORT_FILE="$LATEST"
    RESUMING=true
    # Read Progress Log, skip to next incomplete step
  fi
fi
```

If resuming: read the report, find last completed step in Progress Log, skip to next.

### 1.4 Initialize Report File (Skip if Resuming)

Filename: `.smoketest-reports/smoke-YYYY-MM-DD-HHMMSS.md`

**Write this to disk IMMEDIATELY:**

```markdown
# Smoke Test Report — [project-name]

**Date:** YYYY-MM-DD HH:MM
**Framework:** [framework]
**Package Manager:** [pm]
**Dev Server:** Running on :[port] / Not running
**Status:** 🔴 IN PROGRESS — Phase 1: Setup

---

## Progress Log

| Time | Phase | Action | Result |
|------|-------|--------|--------|
| [HH:MM] | Setup | Initialized | [framework] on [pm] |

---

## Findings

| # | Layer | Severity | Finding | File:Line | Status |
|---|-------|----------|---------|-----------|--------|

---

## Fix Log

| # | Finding | Action | Attempt | Verified |
|---|---------|--------|---------|----------|

---

## SITREP

_To be populated at conclusion_
```

**IMPORTANT:** This file is written to disk NOW, before any scans. Every subsequent phase updates this same file.

---

## Phase 2: Scan (All 9 Layers)

Run ALL layers. Collect every finding. Do NOT stop on failure.

**AFTER EACH LAYER completes:**
1. Add all findings from that layer to `## Findings` table (status = `FOUND`)
2. Append to `## Progress Log`: `| [HH:MM] | Layer N | [Name] | [X] findings |`
3. Update `**Status:**` to `🔴 IN PROGRESS — Layer [N+1]: [name]`
4. **Write the file to disk** — this is the checkpoint

### Layer 1: TypeScript Type Check

```bash
npx tsc --noEmit 2>&1
```

- Count errors via `grep -c "error TS"`
- For each error: extract file path, line number, error code, message
- **Severity:** HIGH (blocks build)
- **Auto-fixable:** Yes — missing imports, wrong types, unused vars

### Layer 2: Build

```bash
$PM_RUN build 2>&1
```

- Pass = exit code 0
- Fail = capture error output (last 20 lines)
- For Next.js: capture route summary, note any pages >200kB First Load JS
- **Severity:** CRITICAL (app won't deploy)
- **Auto-fixable:** Sometimes — depends on error type. Type errors yes, config errors usually no.

### Layer 3: Lint

Only if lint script exists in package.json.

```bash
$PM_RUN lint 2>&1
```

- Count warnings and errors separately
- For each error: extract file, line, rule name
- **Severity:** MEDIUM (errors), LOW (warnings)
- **Auto-fixable:** Yes — run `$PM_RUN lint --fix` for auto-fixable rules

### Layer 4: Unit Tests

Only if test runner detected.

```bash
# Prefer test:unit if it exists, otherwise test with --run
$PM_RUN test:unit 2>&1  # or: $PM_RUN test -- --run
```

- Capture pass/fail count
- For failures: extract test name, file, assertion error
- **Severity:** HIGH (failing test = regression)
- **Auto-fixable:** Rarely — only if the test itself has a trivial error (wrong import path). Otherwise MANUAL.

### Layer 5: Page Probes

Only if dev server is running.

```bash
PAGES=("/" "/login" "/signup" "/pricing" "/api/health")

for page in "${PAGES[@]}"; do
  RESPONSE=$(curl -s -o /dev/null -w "%{http_code}:%{time_total}" "http://localhost:$DEV_PORT$page" 2>/dev/null)
  HTTP_CODE=$(echo "$RESPONSE" | cut -d: -f1)
  RESPONSE_TIME=$(echo "$RESPONSE" | cut -d: -f2)
done
```

- 200, 301, 302, 307, 308 = pass
- 404 = warn (page may not exist in this project — skip, don't fail)
- 500 = CRITICAL finding
- Response time >3s = MEDIUM finding (slow page)
- **Auto-fixable:** No — runtime errors need investigation

### Layer 6: Security Headers

Only if dev server is running.

```bash
HEADERS=$(curl -sI "http://localhost:$DEV_PORT/" 2>/dev/null)
```

Check for:

| Header | Required | Severity if Missing |
|--------|----------|-------------------|
| `x-content-type-options: nosniff` | Yes | MEDIUM |
| `x-frame-options` or CSP `frame-ancestors` | Yes | MEDIUM |
| `referrer-policy` | Yes | LOW |
| `permissions-policy` | Nice to have | LOW |
| Absence of `x-powered-by` | Yes | LOW |
| `strict-transport-security` | Localhost exempt | Skip on localhost |

- **Auto-fixable:** Yes — add missing headers to `next.config` headers array or middleware

### Layer 7: Console.log in Server Code

Scan for `console.log` statements in server-side files that shouldn't ship to production.

```bash
# Search API routes, server actions, middleware, lib/ for console.log
grep -rn "console\.log" --include="*.ts" --include="*.tsx" \
  app/api/ lib/ middleware.ts 2>/dev/null | grep -v "// ok" | grep -v ".test."
```

- Exclude test files, files with `// ok` comment on same line
- **Severity:** LOW
- **Auto-fixable:** Yes — remove the console.log line (only if it's a standalone statement, not inside a ternary or return)

### Layer 8: Environment Variable Validation

Check that required env vars exist (not their values, just presence).

```bash
# Extract NEXT_PUBLIC_* and server vars referenced in code
grep -roh "process\.env\.\w\+" --include="*.ts" --include="*.tsx" app/ lib/ 2>/dev/null | sort -u
```

- Cross-reference with `.env.local` and `.env` files
- Missing var = MEDIUM finding
- `NEXT_PUBLIC_` var with sensitive-looking name (key, secret, password) = HIGH finding
- **Auto-fixable:** No — can't guess values. Report only.

### Layer 9: Dependency Audit

```bash
$PM audit --json 2>/dev/null
```

- Parse JSON output for vulnerabilities
- HIGH/CRITICAL npm advisories = HIGH finding
- MODERATE = MEDIUM finding
- LOW = LOW finding
- **Auto-fixable:** Partially — `$PM audit fix` for non-breaking fixes only

---

## Phase 3: Auto-Fix (Safe Only)

After ALL scans complete, fix safe issues. Rules:

### Fix Safety Rules

1. **NEVER fix something if you're not 100% certain it's correct**
2. **Verify build passes after EACH fix** — if build breaks, revert immediately
3. **One fix at a time** — don't batch fixes (makes reverts clean)
4. **Only fix these categories:**

| Category | Fix Strategy | Safe? |
|----------|-------------|-------|
| Type errors (missing import) | Add the import | Yes |
| Type errors (wrong type) | Fix the type annotation | Yes if obvious |
| Type errors (unused variable) | Prefix with `_` or remove if truly unused | Yes |
| Lint auto-fixable rules | `$PM_RUN lint --fix` | Yes |
| Console.log in server code | Remove the line | Yes if standalone statement |
| Missing security headers | Add to next.config or middleware | Yes |
| npm audit fix (non-breaking) | `$PM audit fix` | Yes |
| Failing unit test | **Do NOT auto-fix** | No — mark MANUAL |
| Build config errors | **Do NOT auto-fix** | No — mark MANUAL |
| Missing env vars | **Do NOT auto-fix** | No — mark MANUAL |
| 500 errors on pages | **Do NOT auto-fix** | No — mark MANUAL |

### Fix Process (Per Finding, Up to 3 Attempts)

For each fixable finding, in severity order (CRITICAL → HIGH → MEDIUM → LOW):

```
1. Update finding row → 🔧 FIXING
2. Update report Status → 🟡 FIXING — #[N] [description]
3. Write report to disk

4. ATTEMPT 1: Apply the minimal fix
   - Read the file, understand context
   - Apply fix
   - Verify: npx tsc --noEmit (or $PM_RUN build for build-related)
   - If passes → update row → ✅ FIXED, add to Fix Log, write to disk, next finding
   - If fails → revert fix

5. ATTEMPT 2: Try alternative approach
   - Different fix strategy based on the error from attempt 1
   - Verify again
   - If passes → ✅ FIXED, write to disk, next
   - If fails → revert

6. ATTEMPT 3: Last try, different angle
   - If passes → ✅ FIXED
   - If fails → revert, update row → 🚫 BLOCKED
   - Add to SITREP: "Blocked: [finding] — Tried: [approach 1], [approach 2], [approach 3]. Failed because: [reason]"

7. If fix is too risky or needs architectural change:
   - Update row → ⏸️ DEFERRED
   - Add to SITREP: "Deferred: [finding] — Reason: [why]"

8. Append to Progress Log: | [HH:MM] | Fix | #[N] | ✅ FIXED / 🚫 BLOCKED / ⏸️ DEFERRED |
9. Write report to disk — checkpoint after EVERY status change
10. Move to next finding
```

### After All Fixes

Run a final build check:

```bash
$PM_RUN build 2>&1
```

- If build passes → all fixes verified
- If build fails → identify which fix broke it, revert that fix, mark as 🚫 BLOCKED with reason

---

## Phase 4: Finalize Report

The report has been built incrementally through Phases 1-3. Now finalize it:

1. **Add Summary** after the header:

```markdown
## Summary

| Metric | Count |
|--------|-------|
| Total Findings | [X] |
| ✅ FIXED | [X] |
| ⏸️ DEFERRED | [X] |
| 🚫 BLOCKED | [X] |
| NOT APPLICABLE | [X] |
```

2. **Write SITREP section** — this is the narrative that survives compaction:

```markdown
## SITREP

### What Was Fixed
- #1 TS2345 in route.ts:42 — Changed argument type to match expected signature
- #3 no-unused-vars — Removed unused import from Header.tsx
- #7 CVE-2026-1234 — pnpm audit fix resolved lodash vulnerability

### What Was Deferred (and Why)
- #9 500 on /api/health — Requires running database connection. Cannot diagnose without checking Supabase project status. Recommend: verify SUPABASE_URL in .env.local
- #10 Missing RESEND_API_KEY — Cannot auto-create API keys. User must add to .env.local

### What Was Blocked (and Why)
- #8 Failing unit test — Attempted 3 fixes: (1) updated mock return type, (2) adjusted assertion, (3) rewrote test. All failed because service function was refactored but test was not updated to match new async pattern. Needs manual review of service contract.

### Build Verification
**Post-fix build:** ✅ Passed
**Post-fix types:** ✅ 0 errors
```

3. **Set final status:** `**Status:** 🟢 COMPLETE`
4. **Add duration** to header
5. **Final Progress Log entry:** `| [HH:MM] | Finalize | Report complete | [X] FIXED, [Y] DEFERRED, [Z] BLOCKED |`
6. **Write to disk** — final checkpoint

---

## Phase 5: Status Report (Display on Screen)

**ALWAYS display this on screen after writing the report file.**

### All Clear:

```
═══════════════════════════════════════════════════════════════
🔥 SMOKE TEST — [project-name]
═══════════════════════════════════════════════════════════════
├─ Types:     ✅ 0 errors
├─ Build:     ✅ [Xs]
├─ Lint:      ✅ 0 errors
├─ Units:     ✅ [X]/[X] passing
├─ Pages:     ✅ /, /login, /pricing → 200
├─ Headers:   ✅ All present
├─ Console:   ✅ No server console.logs
├─ Env Vars:  ✅ All referenced vars present
└─ Deps:      ✅ No known vulnerabilities
───────────────────────────────────────────────────────────────
Result: ✅ ALL CLEAR — [total duration]
Report: .smoketest-reports/smoke-[timestamp].md
───────────────────────────────────────────────────────────────
💡 Next: /gh-ship to commit and push
═══════════════════════════════════════════════════════════════
```

### With Fixes Applied:

```
═══════════════════════════════════════════════════════════════
🔥 SMOKE TEST — [project-name]
═══════════════════════════════════════════════════════════════
├─ Types:     🔧 2 errors → FIXED
├─ Build:     ✅ [Xs]
├─ Lint:      🔧 2 issues → FIXED
├─ Units:     ✅ [X]/[X] passing
├─ Pages:     ✅ /, /login → 200
├─ Headers:   🔧 1 missing → FIXED
├─ Console:   🔧 2 found → FIXED
├─ Env Vars:  ✅ All present
└─ Deps:      🔧 1 CVE → FIXED
───────────────────────────────────────────────────────────────
Result: 🔧 7 FIXED, 0 remaining — [total duration]
Report: .smoketest-reports/smoke-[timestamp].md
───────────────────────────────────────────────────────────────
💡 Next: /gh-ship to commit fixes and push
═══════════════════════════════════════════════════════════════
```

### With Manual Items:

```
═══════════════════════════════════════════════════════════════
🔥 SMOKE TEST — [project-name]
═══════════════════════════════════════════════════════════════
├─ Types:     🔧 2 errors → FIXED
├─ Build:     ✅ [Xs]
├─ Lint:      🔧 2 issues → FIXED
├─ Units:     ❌ 1 failing (MANUAL)
├─ Pages:     ❌ /api/health → 500 (MANUAL)
├─ Headers:   🔧 1 missing → FIXED
├─ Console:   🔧 2 found → FIXED
├─ Env Vars:  ⚠️  1 missing (MANUAL)
└─ Deps:      🔧 1 CVE → FIXED
───────────────────────────────────────────────────────────────
Result: ✅ 7 FIXED | ⏸️ 2 DEFERRED | 🚫 1 BLOCKED — [total duration]
Report: .smoketest-reports/smoke-[timestamp].md
───────────────────────────────────────────────────────────────
⏸️ Deferred:
  #9 /api/health 500 — needs DB connection check
  #10 Missing RESEND_API_KEY — user must add to .env.local
🚫 Blocked:
  #8 Failing test — 3 fix attempts failed, needs manual review
───────────────────────────────────────────────────────────────
💡 Next: Address deferred/blocked items, then /smoketest again
═══════════════════════════════════════════════════════════════
```

---

## Phase 6: Suggest Next Action

Based on results, recommend ONE next action:

| Result | Suggestion |
|--------|------------|
| All clear, no fixes needed | `/gh-ship` to commit and push |
| Fixes applied, all clear now | `/gh-ship` to commit fixes and push |
| DEFERRED/BLOCKED remain (CRIT/HIGH) | Address manually, then `/smoketest` again |
| DEFERRED/BLOCKED remain (MED/LOW) | `/gh-ship` — acceptable risk, ship with known issues |
| Everything broken | Check git status — did something go wrong? |

---

## What Each Layer Catches

| # | Layer | Catches | Time | Auto-Fix? |
|---|-------|---------|------|-----------|
| 1 | Types | Type errors, missing imports, wrong signatures | ~10-15s | Yes |
| 2 | Build | Compilation errors, config issues, bundle size | ~15-25s | Sometimes |
| 3 | Lint | Code style, unused vars, import order | ~5s | Yes |
| 4 | Units | Logic bugs, regressions in tested code | ~5-10s | Rarely |
| 5 | Pages | Broken routes, server crashes, slow responses | ~5s | No |
| 6 | Headers | Missing security headers, exposed server info | ~2s | Yes |
| 7 | Console | console.log in server code (shouldn't ship) | ~2s | Yes |
| 8 | Env Vars | Missing environment variables | ~2s | No |
| 9 | Deps | Known CVEs in dependencies | ~5s | Partially |

---

## What This Does NOT Catch (Use Other Skills)

| Gap | Skill |
|-----|-------|
| E2E user flows | `/test-ship` |
| Accessibility (WCAG) | `/a11y` |
| Performance deep-dive | `/perf` |
| Security audit | `/sec-ship` |
| Coverage gaps | `/test-ship` |
| Code quality / dead code | `/cleancode` |
| Database issues | `/db` |
| Dependency deep-dive | `/deps` |

---

## Usage

```bash
/smoketest              # Scan everything, auto-fix, report
```

No flags. No config. Auto-detects everything from package.json.

---

## Why This Design

1. **No fixtures needed** — checks build, types, public pages only
2. **No auth needed** — only probes unauthenticated routes
3. **Works on any project** — auto-detects PM, framework, test runner
4. **Fixes what it can** — don't just report problems, solve them
5. **Never breaks code** — verifies build after every fix, reverts if broken
6. **Persistent reports** — .smoketest-reports/ tracks history
7. **Actionable output** — every finding has a status and next step
8. **Complements the pipeline** — /smoketest (quick) → /test-ship (thorough) → /sec-ship (security) → /gh-ship (deploy)

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### Smoketest-Specific Cleanup

Cleanup actions:
1. **Dependency disclosure:** If `npm audit fix` modified dependencies, document changes in the report
2. **Gitignore enforcement:** Ensure `.smoketest-reports/` is in `.gitignore`
3. **No browser, server, or test data cleanup needed**

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
