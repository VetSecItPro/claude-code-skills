# /test-ship — Comprehensive Test Audit, Fix & Ship Pipeline

**One skill. Fully autonomous. Audit → Fix → Verify → Ship.**

**FIRE AND FORGET** — Execute the entire pipeline without waiting for user input. Status updates every 5 minutes. Human intervention only for truly dangerous operations.

---

## Execution Rules (CRITICAL)

- **NO permission requests** — just execute
- **NO "should I proceed?" questions** — just do it
- **NO waiting for user confirmation** — work continuously
- **REPORT IS THE SINGLE SOURCE OF TRUTH** — written to disk after every status change
- **UPDATE REPORT AS FIXES HAPPEN** — each finding's status changes in real-time in the markdown
- **NEVER DELETE findings** — only update their status
- **ATTEMPT FIX UP TO 3 TIMES** before marking BLOCKED — document each attempt
- **SITREP ANNOTATES EVERYTHING** — what was fixed, what was deferred and WHY, what was blocked and WHY
- **Status updates every 5 minutes** — output progress without waiting for response
- **Self-healing loops** — if a fix reveals new issues, fix those too
- **All tools pre-approved** — no permission prompts during execution
- **Parallel agents** — 6 audit agents, 4 fix agents, maximize throughput

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

### Test Pipeline Status Flow

```markdown
🚀 Test Pipeline Started
   Mode: [full/audit/quick/changed]
   Project: [name]

🔍 Phase 1: Test Coverage Audit
   ├─ Scanning for missing unit tests
   ├─ Checking integration test coverage
   ├─ Analyzing E2E test coverage
   ├─ Reviewing test quality and patterns
   └─ ✅ Audit complete ([X] findings)

📊 Coverage Summary:
   • Unit tests: [X]% coverage
   • Integration tests: [X]% coverage
   • E2E tests: [X]% coverage
   • Missing tests: [X] files

🔨 Phase 2: Test Generation & Fixes
   ├─ [30s] Writing unit tests for user service
   ├─ [60s] Adding integration tests for API routes
   ├─ [90s] Creating E2E tests for auth flow
   └─ ✅ Tests generated ([X] new tests)

🧪 Phase 3: Test Execution
   ├─ Running unit tests ([X]/[Y] passing)
   ├─ Running integration tests ([X]/[Y] passing)
   ├─ Running E2E tests ([X]/[Y] passing)
   └─ ✅ Tests complete

📊 Test Results:
   • ✅ PASSING: [X] tests
   • ❌ FAILING: [X] tests
   • 🔒 QUARANTINED: [X] flaky tests
   • Coverage: [old]% → [new]%

📝 Phase 4: Report Generation
   └─ ✅ Report: .test-reports/test-2026-02-08.md

✅ Test Pipeline Complete
   Duration: [X] minutes
   Tests Added: [X]
   Coverage: [old]% → [new]%
   Status: [COMPLETE/NEEDS_REVIEW]
```

---

## Report Persistence (CRITICAL — Survives Compaction/Restart)

The markdown report file is the **living document**. It must be self-contained and updated continuously.

### Finding Statuses

| Status | Meaning |
|--------|---------|
| `FOUND` | Issue discovered during audit, not yet addressed |
| `🔧 FIXING` | Currently being worked on |
| `✅ FIXED` | Fix applied, test passes, build verified |
| `⏸️ DEFERRED` | Intentionally skipping — needs architectural change, business decision, or is low-priority. Reason documented in SITREP. |
| `🚫 BLOCKED` | Attempted fix up to 3 times, all failed. Each attempt documented in SITREP. |
| `🔒 QUARANTINED` | Flaky test — inconsistent results, moved to quarantine folder |

### Rules

1. **Write report at Phase 0** — file exists before any audits, with header and empty sections
2. **Update after each audit agent completes** — findings added to task list as discovered
3. **Update after each fix** — status changes from FOUND → FIXING → FIXED/BLOCKED/DEFERRED
4. **Write to disk after every status change** — if session dies, report shows exactly where things stand
5. **Progress Log** — timestamped entries after each phase, each agent, and each fix
6. **SITREP section** — for every DEFERRED or BLOCKED finding, document:
   - What was tried (all attempts)
   - Why it failed
   - What would be needed to fix it

### Resume Protocol

Before creating a new report, check for a recent incomplete one:

1. Find most recent `.test-reports/test-*.md`
2. If < 1 hour old AND Status is not `🟢 COMPLETE` → resume it
3. Read `## Progress Log` to find last completed step
4. Skip completed phases/agents, continue from next incomplete step
5. Do NOT re-audit completed agents or re-fix already-FIXED findings

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Audit agents return < 500 tokens each (full test analysis written to .test-reports/)
- Fix agents return < 500 tokens each (list of files modified + test results summary)
- State file `.test-reports/state-YYYYMMDD-HHMMSS.json` tracks which audit/fix phases are complete
- Resume from checkpoint if context resets — skip completed audits and fixes
- Max 2 parallel audit agents; fix agents run sequentially (they modify code)
- Orchestrator messages stay lean: "Audit 3/6 complete: 15 missing tests found in lib/services/"

---

## AGENT ORCHESTRATION

This skill follows the **[Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)**.

The orchestrator coordinates agents but NEVER runs audits, writes tests, or fixes code directly. All heavy work is delegated to focused agents that write results to disk.

### Model Selection for This Skill

| Agent Type | Model | Why |
|-----------|-------|-----|
| Test file inventory | `haiku` | Listing existing test files, counting coverage — no judgment |
| Test runner | `haiku` | Executing test suites and collecting pass/fail results — no judgment |
| Coverage gap analyzer | `sonnet` | Must understand code paths, identify untested critical logic, prioritize what needs tests |
| Test quality analyzer | `sonnet` | Must evaluate if existing tests are meaningful or just hitting coverage numbers |
| Test writer | `sonnet` | Must understand the code under test, write correct assertions, handle edge cases |
| Test fixer | `sonnet` | Must diagnose why tests fail, fix assertions or code, verify passing |
| Flaky test analyzer | `sonnet` | Must identify timing issues, race conditions, environment dependencies |
| Integration test designer | `opus` | Must understand system-wide data flow to design meaningful integration tests |
| Report synthesizer | `sonnet` | Must compile coverage stats, test health scores, and prioritized gaps |

### Agent Batching

- Inventory and runner agents can run 2 in parallel (read-only)
- Test writers handle up to 3-5 test files each (tests are complex to write correctly)
- Test writers run SEQUENTIALLY (each may affect shared test utilities)
- Fix agents handle up to 5 test fixes each, run SEQUENTIALLY

---

## Usage

```bash
/test-ship                    # Full pipeline: audit → fix → verify → report
/test-ship --changed          # Only test files changed since last commit
/test-ship --since=HEAD~5     # Only test files changed in last 5 commits
/test-ship --resume           # Resume interrupted run from saved state
/test-ship --audit-only       # Skip fix phase, just report findings
/test-ship --quick            # Skip E2E and performance (fast mode)
```

---

## Output Files (All Gitignored)

| File | Purpose |
|------|---------|
| `.test-reports/test-YYYY-MM-DD-HHMMSS.md` | Human-readable final report |
| `.test-ship-state.json` | Resume state (for interrupted runs) |
| `.test-ship-history.json` | Historical trend data |
| `.test-ship-quarantine.json` | Flaky tests quarantined |
| `.test-audit.json` | Machine-readable findings |
| `.test-reports/coverage/` | HTML coverage report |
| `.test-reports/lighthouse/` | Lighthouse HTML reports |
| `.test-reports/screenshots/` | E2E failure screenshots |

**FIRST ACTION:** Ensure these are gitignored:
```bash
# Add to .gitignore if not present
for pattern in ".test-reports/" ".test-ship-state.json" ".test-ship-history.json" ".test-ship-quarantine.json" ".test-audit.json"; do
  grep -qxF "$pattern" .gitignore 2>/dev/null || echo "$pattern" >> .gitignore
done
```

---

## Architecture Overview

```
═══════════════════════════════════════════════════════════════════════════════
                              /test-ship PIPELINE
═══════════════════════════════════════════════════════════════════════════════

PHASE 0: SETUP
├── Git snapshot (rollback point)
├── Check for resume state
├── Baseline metrics capture
├── Time/cost estimation
├── Install missing tools (vitest, playwright)
└── Start dev server (background)

PHASE 1: AUDIT (6 parallel agents)
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│  UNIT   │ │ INTEGR  │ │   E2E   │ │  PERF   │ │  A11Y   │ │ QUALITY │
│ Agent 1 │ │ Agent 2 │ │ Agent 3 │ │ Agent 4 │ │ Agent 5 │ │ Agent 6 │
└────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘
     └───────────┴───────────┴─────┬─────┴───────────┴───────────┘
                                   │
                            [MERGE FINDINGS]
                            [STATUS UPDATE]

PHASE 2: FIX (4 parallel agents by file ownership)
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ FIX Agent A │ │ FIX Agent B │ │ FIX Agent C │ │ FIX Agent D │
│ (API routes)│ │(Components) │ │   (Tests)   │ │(Lib/Config) │
└──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
       └───────────────┴───────┬───────┴───────────────┘
                               │
                        [VERIFY EACH FIX]
                        [STATUS UPDATE]

PHASE 3: VERIFY & SELF-HEAL
                    ┌──────────────┐
                    │  RUN TESTS   │
                    └───────┬──────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
         [ALL PASS]              [NEW ISSUES FOUND]
              │                           │
              │                    [LOOP BACK TO FIX]
              │                    (until truly stuck)
              │                           │
              └─────────────┬─────────────┘
                            │
                     [STATUS UPDATE]

PHASE 4: FINAL REPORT
├── Before/after delta comparison
├── Full findings table with severity
├── What was fixed
├── What remains (with reasons)
├── Historical trend
├── Recommendations
└── Save to .test-reports/

PHASE 5: CLEANUP
├── Stop dev server
├── Clear temp files
├── Release ports
├── Update history log
└── Save state for future resume

═══════════════════════════════════════════════════════════════════════════════
```

---

## PHASE 0: SETUP

### 0.1 Check Resume State

```bash
if [ -f ".test-ship-state.json" ]; then
  LAST_PHASE=$(jq -r '.lastPhase' .test-ship-state.json)
  LAST_STEP=$(jq -r '.lastStep' .test-ship-state.json)
  TIMESTAMP=$(jq -r '.timestamp' .test-ship-state.json)

  echo "📋 Previous run interrupted at Phase $LAST_PHASE, Step $LAST_STEP"
  echo "   Timestamp: $TIMESTAMP"

  if [ "$1" = "--resume" ]; then
    echo "🔄 Resuming from saved state..."
    # Load state and continue from where we left off
  else
    echo "🆕 Starting fresh run (use --resume to continue previous)"
    rm .test-ship-state.json
  fi
fi
```

### 0.2 Git Snapshot (Rollback Point)

```bash
# Save current state for potential rollback
git stash push -m "test-ship-snapshot-$(date +%Y%m%d-%H%M%S)" --include-untracked 2>/dev/null || true
git stash pop 2>/dev/null || true
git add -A && git commit -m "chore: pre-test-ship snapshot" --allow-empty 2>/dev/null || true

TEST_SHIP_BASE=$(git rev-parse HEAD)
echo "📌 Rollback point: $TEST_SHIP_BASE"
```

### 0.3 Detect Environment

```bash
# Package manager
if [ -f "bun.lockb" ]; then
  PM="bun"; PM_RUN="bun run"; PM_INSTALL="bun install"; PM_ADD="bun add -D"
elif [ -f "pnpm-lock.yaml" ]; then
  PM="pnpm"; PM_RUN="pnpm run"; PM_INSTALL="pnpm install"; PM_ADD="pnpm add -D"
elif [ -f "yarn.lock" ]; then
  PM="yarn"; PM_RUN="yarn"; PM_INSTALL="yarn install"; PM_ADD="yarn add -D"
else
  PM="npm"; PM_RUN="npm run"; PM_INSTALL="npm install"; PM_ADD="npm install -D"
fi

# Test framework
if grep -q '"vitest"' package.json 2>/dev/null; then TEST_RUNNER="vitest"
elif grep -q '"jest"' package.json 2>/dev/null; then TEST_RUNNER="jest"
else TEST_RUNNER="none"; fi

# E2E framework
if grep -q '"@playwright/test"' package.json 2>/dev/null; then E2E_RUNNER="playwright"
elif grep -q '"cypress"' package.json 2>/dev/null; then E2E_RUNNER="cypress"
else E2E_RUNNER="none"; fi

# Framework
if grep -q '"next"' package.json 2>/dev/null; then FRAMEWORK="nextjs"
elif grep -q '"svelte"' package.json 2>/dev/null; then FRAMEWORK="sveltekit"
elif grep -q '"nuxt"' package.json 2>/dev/null; then FRAMEWORK="nuxt"
elif grep -q '"astro"' package.json 2>/dev/null; then FRAMEWORK="astro"
else FRAMEWORK="unknown"; fi
```

### 0.4 Install Missing Test Infrastructure

```bash
# Install vitest if no test runner
if [ "$TEST_RUNNER" = "none" ]; then
  echo "📦 Installing vitest..."
  $PM_ADD vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom @vitejs/plugin-react

  # Create vitest.config.ts
  cat > vitest.config.ts << 'EOF'
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    include: ['**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      reportsDirectory: '.test-reports/coverage',
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './'),
    },
  },
})
EOF

  # Create setup file
  cat > vitest.setup.ts << 'EOF'
import '@testing-library/jest-dom/vitest'
EOF

  TEST_RUNNER="vitest"
fi

# Install playwright if no E2E runner
if [ "$E2E_RUNNER" = "none" ]; then
  echo "📦 Installing Playwright..."
  $PM_ADD @playwright/test
  npx playwright install chromium webkit firefox

  # Create playwright.config.ts
  cat > playwright.config.ts << 'EOF'
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html', { outputFolder: '.test-reports/playwright' }]],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    // Desktop
    { name: 'Desktop Chrome', use: { ...devices['Desktop Chrome'] } },
    { name: 'Desktop Safari', use: { ...devices['Desktop Safari'] } },
    { name: 'Desktop Firefox', use: { ...devices['Desktop Firefox'] } },
    // Mobile
    { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 14'] } },
    { name: 'iPhone SE', use: { ...devices['iPhone SE'] } },
    // Tablet
    { name: 'iPad', use: { ...devices['iPad Pro 11'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
EOF

  mkdir -p e2e
  E2E_RUNNER="playwright"
fi
```

### 0.5 Baseline Metrics Capture

**Capture current state BEFORE any changes:**

```bash
echo "📊 Capturing baseline metrics..."

# Test counts
BASELINE_TEST_FILES=$(find . -name "*.test.ts" -o -name "*.test.tsx" -o -name "*.spec.ts" | wc -l)
BASELINE_SOURCE_FILES=$(find app lib components hooks -name "*.ts" -o -name "*.tsx" 2>/dev/null | grep -v test | grep -v spec | wc -l)

# Coverage (if available)
if [ -f ".test-reports/coverage/coverage-summary.json" ]; then
  BASELINE_COVERAGE=$(jq '.total.lines.pct' .test-reports/coverage/coverage-summary.json 2>/dev/null || echo "0")
else
  BASELINE_COVERAGE="unknown"
fi

# Bundle size (if Next.js)
if [ "$FRAMEWORK" = "nextjs" ] && [ -d ".next" ]; then
  BASELINE_BUNDLE=$(du -sh .next 2>/dev/null | cut -f1 || echo "unknown")
else
  BASELINE_BUNDLE="unknown"
fi

# Type errors
BASELINE_TYPE_ERRORS=$(npx tsc --noEmit 2>&1 | grep -c "error TS" || echo "0")

# TODO/FIXME count
BASELINE_TODOS=$(grep -r "TODO\|FIXME\|HACK\|XXX" --include="*.ts" --include="*.tsx" . 2>/dev/null | wc -l || echo "0")

# Save baseline
cat > .test-ship-baseline.json << EOF
{
  "timestamp": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
  "testFiles": $BASELINE_TEST_FILES,
  "sourceFiles": $BASELINE_SOURCE_FILES,
  "coverage": "$BASELINE_COVERAGE",
  "bundleSize": "$BASELINE_BUNDLE",
  "typeErrors": $BASELINE_TYPE_ERRORS,
  "todos": $BASELINE_TODOS
}
EOF

echo "✅ Baseline captured"
```

### 0.6 Time/Cost Estimation

```bash
echo "⏱️ Estimating duration..."

# Count files to estimate time
SOURCE_COUNT=$(find app lib components hooks -name "*.ts" -o -name "*.tsx" 2>/dev/null | wc -l)
TEST_COUNT=$(find . -name "*.test.ts" -o -name "*.test.tsx" -o -name "*.spec.ts" | wc -l)
ROUTE_COUNT=$(find app/api -name "route.ts" 2>/dev/null | wc -l)
PAGE_COUNT=$(find app -name "page.tsx" 2>/dev/null | wc -l)

# Estimation formula (rough)
AUDIT_MINUTES=$((SOURCE_COUNT / 20 + TEST_COUNT / 10 + 5))
FIX_MINUTES=$((AUDIT_MINUTES / 2 + 5))
VERIFY_MINUTES=$((TEST_COUNT / 20 + 3))
TOTAL_ESTIMATE=$((AUDIT_MINUTES + FIX_MINUTES + VERIFY_MINUTES))

cat << EOF

📊 Codebase Analysis
   • Source files: $SOURCE_COUNT
   • Test files: $TEST_COUNT
   • API routes: $ROUTE_COUNT
   • Pages: $PAGE_COUNT

⏱️ Estimated Duration
   • Audit phase: ~${AUDIT_MINUTES} minutes
   • Fix phase: ~${FIX_MINUTES} minutes (depends on findings)
   • Verification: ~${VERIFY_MINUTES} minutes
   • Total: ~${TOTAL_ESTIMATE} minutes

EOF
```

### 0.7 Start Dev Server

```bash
echo "🚀 Starting dev server..."
$PM_RUN dev > /dev/null 2>&1 &
DEV_SERVER_PID=$!

# Wait for server to be ready
MAX_WAIT=60
ELAPSED=0
while ! curl -s http://localhost:3000 > /dev/null 2>&1; do
  sleep 2
  ELAPSED=$((ELAPSED + 2))
  if [ $ELAPSED -ge $MAX_WAIT ]; then
    echo "⚠️ Dev server took too long to start, continuing anyway..."
    break
  fi
done

echo "✅ Dev server running (PID: $DEV_SERVER_PID)"
```

### 0.7b Seed E2E Test Users

If the project has E2E tests and a seed script exists, seed test users before running tests:

```bash
# Seed E2E test users (if seed script exists)
if [ -f "tests/e2e/setup/seed-test-users.ts" ]; then
  echo "🌱 Seeding E2E test users..."
  npx tsx tests/e2e/setup/seed-test-users.ts
  if [ $? -eq 0 ]; then
    echo "✅ E2E test users seeded"
    E2E_USERS_SEEDED=true
  else
    echo "⚠️ E2E test user seeding failed — E2E tests may skip"
    E2E_USERS_SEEDED=false
  fi
fi
```

**Note:** Playwright's `setup` project will also run this, but running it early in Phase 0 catches seed failures before the full test suite runs.

### 0.8 Create Output Directories

```bash
mkdir -p .test-reports/coverage .test-reports/lighthouse .test-reports/screenshots .test-reports/playwright
```

### 0.9 Check for Resumable Report

```bash
LATEST=$(ls -t .test-reports/test-*.md 2>/dev/null | head -1)
if [ -n "$LATEST" ]; then
  AGE=$(( $(date +%s) - $(stat -f %m "$LATEST") ))
  if [ "$AGE" -lt 3600 ] && ! grep -q "🟢 COMPLETE" "$LATEST"; then
    REPORT_FILE="$LATEST"
    RESUMING=true
    # Read Progress Log, skip completed steps
  fi
fi
```

### 0.10 Initialize Report File (Skip if Resuming)

**Write this to disk IMMEDIATELY:**

```markdown
# Test Audit Report — [PROJECT_NAME]

**Date:** YYYY-MM-DD HH:MM
**Status:** 🔴 IN PROGRESS — Phase 0: Setup
**Framework:** [framework]
**Test Runner:** [vitest/jest]
**E2E Runner:** [playwright/cypress]

---

## Progress Log

| Time | Phase | Action | Result |
|------|-------|--------|--------|
| [HH:MM] | Phase 0 | Setup | Baseline captured |

---

## Findings

| ID | Severity | Domain | Finding | File:Line | Status |
|----|----------|--------|---------|-----------|--------|

---

## Fix Log

| ID | Finding | Action | Attempt | Build | Test |
|----|---------|--------|---------|-------|------|

---

## SITREP

_To be populated as fixes are applied_

### What Was Fixed
_Updated in real-time_

### What Was Deferred (and Why)
_Each deferred item with full explanation_

### What Was Blocked (and Why)
_Each blocked item with all attempts documented_

### Quarantined Tests
_Flaky tests with reason and failure pattern_
```

### 0.11 Save Initial State

```javascript
// .test-ship-state.json (supplementary — report file is the primary source of truth)
{
  "startTime": "ISO timestamp",
  "lastPhase": 0,
  "lastStep": "setup-complete",
  "devServerPid": 12345,
  "reportFile": ".test-reports/test-YYYY-MM-DD-HHMMSS.md"
}
```

---

## PHASE 1: AUDIT (6 Parallel Agents)

### Status Update: Start

```
═══════════════════════════════════════════════════════════════
🚀 PHASE 1: AUDIT STARTED
═══════════════════════════════════════════════════════════════
⏱️ Time: 0:00
📊 Spawning 6 audit agents in parallel...

   Agent 1: Unit Tests
   Agent 2: Integration Tests
   Agent 3: E2E Tests (7 viewports)
   Agent 4: Performance (Lighthouse + Vitals)
   Agent 5: Accessibility (WCAG 2.1 AA)
   Agent 6: Code Quality

Working... Status updates every 5 minutes.
═══════════════════════════════════════════════════════════════
```

Launch **6 Task agents in parallel** using the Task tool. Each agent runs independently and returns findings. Return ONLY a structured summary under 500 tokens. Full findings go to disk.

---

### AGENT 1: Unit Tests

**Responsibilities:**
1. Run all unit tests
2. Collect coverage report
3. Identify untested functions
4. Check for flaky tests
5. Detect console errors during tests

**Scan for:**

| Check | Severity if Failed |
|-------|-------------------|
| Test execution failure | CRITICAL |
| Coverage <50% on file | HIGH |
| Coverage <80% on file | MEDIUM |
| Exported function without test | MEDIUM |
| Console.error during test | MEDIUM |
| Skipped test | LOW |
| Test without assertions | LOW |
| Flaky test (inconsistent) | LOW (quarantine) |

**Commands:**
```bash
# Run tests with coverage
$PM_RUN test --coverage --reporter=json --outputFile=.test-reports/unit-results.json

# Check for console errors
$PM_RUN test 2>&1 | grep -i "console.error\|console.warn" > .test-reports/console-errors.txt
```

**Finding format:**
```json
{
  "id": "TEST-001",
  "domain": "unit",
  "category": "coverage",
  "severity": "high",
  "title": "Low coverage on app/api/chat/route.ts",
  "file": "app/api/chat/route.ts",
  "metric": "32% lines covered",
  "target": "80%",
  "remediation": "Add tests for POST handler, error cases, and edge cases"
}
```

---

### AGENT 2: Integration Tests

**Responsibilities:**
1. Test all API routes
2. Test database operations
3. Test authentication flows
4. Test webhook handlers
5. Test external API integrations (mocked)

**For each API route (`app/api/**/route.ts`):**

| HTTP Method | Required Tests |
|-------------|----------------|
| GET | 200 success, 401 unauth, 404 not found, query param validation |
| POST | 200/201 success, 400 validation, 401 unauth, 409 conflict |
| PUT/PATCH | 200 success, 400, 401, 403 forbidden, 404 |
| DELETE | 200/204 success, 401, 403, 404 |

**Check for:**
- Missing API route tests → HIGH
- Missing auth test → HIGH
- Missing error case test → MEDIUM
- Missing rate limit test → LOW

**Commands:**
```bash
# List all API routes
find app/api -name "route.ts" -exec echo "Route: {}" \;

# List corresponding test files
find . -path "*api*" -name "*.test.ts"

# Run integration tests only
$PM_RUN test --testPathPattern="api|integration"
```

---

### AGENT 3: E2E Tests

**Auth Pre-Condition:** E2E test users are seeded in Phase 0 and authenticated via Playwright's `setup` project. Test files use `test.use({ storageState })` for pre-authenticated sessions. The teardown project deletes all test users after tests complete.

**Responsibilities:**
1. Run E2E tests on ALL viewports
2. Test critical user flows
3. Test navigation (no dead links)
4. Test forms (validation, submission)
5. Test error states
6. Take screenshots of failures

**Viewport Matrix:**

| Viewport | Dimensions | Browser | Priority |
|----------|------------|---------|----------|
| Desktop Chrome | 1920x1080 | Chromium | CRITICAL |
| Desktop Safari | 1920x1080 | WebKit | HIGH |
| Desktop Firefox | 1920x1080 | Firefox | MEDIUM |
| Laptop | 1366x768 | Chromium | MEDIUM |
| Tablet (iPad) | 768x1024 | WebKit | MEDIUM |
| Mobile (iPhone 14) | 390x844 | WebKit | CRITICAL |
| Mobile (iPhone SE) | 375x667 | WebKit | HIGH |
| Mobile (Pixel 5) | 393x851 | Chromium | MEDIUM |

**Critical Flows (must pass on Desktop + Mobile):**

```typescript
// FLOW 1: Authentication
signup → email verify → login → session → logout

// FLOW 2: Core Feature (content analysis)
login → paste URL → analyze → wait → view results

// FLOW 3: Interaction
open content → open chat → send message → receive response

// FLOW 4: Management
view library → filter → open item → add tags → create collection

// FLOW 5: Export
open content → export → select format → download

// FLOW 6: Payment
pricing → select plan → checkout → complete → upgraded tier

// FLOW 7: Admin (if applicable)
admin panel → view metrics → moderate content → action succeeds
```

**Check for:**
- Flow fails on any viewport → CRITICAL
- Flow fails on mobile only → HIGH
- Navigation dead link → HIGH
- Form validation broken → HIGH
- Error state not handled → MEDIUM
- Slow interaction (>3s) → MEDIUM
- Console error during E2E → MEDIUM

**Commands:**
```bash
# Run all E2E tests
npx playwright test --reporter=html

# Run specific viewport
npx playwright test --project="Mobile Safari"

# Run with screenshots
npx playwright test --screenshot=on
```

---

### AGENT 4: Performance Tests

**Responsibilities:**
1. Run Lighthouse on key pages (desktop + mobile)
2. Measure Core Web Vitals
3. Analyze bundle sizes
4. Measure API latency
5. Check for memory leaks
6. Verify caching headers

**Lighthouse Audits:**

| Page | Desktop Target | Mobile Target |
|------|----------------|---------------|
| Homepage (/) | 90+ all | 80+ all |
| Pricing | 95+ all | 90+ all |
| Library | 80+ perf | 70+ perf |
| Item detail | 75+ perf | 65+ perf |
| Dashboard | 85+ all | 75+ all |

**Core Web Vitals Thresholds:**

| Metric | Good | Needs Work | Poor |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | <2.5s | 2.5-4s | >4s |
| INP (Interaction to Next Paint) | <200ms | 200-500ms | >500ms |
| CLS (Cumulative Layout Shift) | <0.1 | 0.1-0.25 | >0.25 |
| TTFB (Time to First Byte) | <800ms | 800-1800ms | >1800ms |

**Bundle Size Thresholds:**

| Metric | Warning | Error |
|--------|---------|-------|
| Page First Load JS | >200kB | >300kB |
| Shared chunk | >150kB | >200kB |
| Total JS | >400kB | >500kB |

**Check for:**
- Lighthouse <70 on any category → HIGH
- LCP >4s → HIGH
- CLS >0.25 → HIGH
- Bundle >300kB → HIGH
- Memory leak detected → HIGH
- No cache headers on static assets → MEDIUM
- Image not optimized → MEDIUM

**Commands:**
```bash
# Lighthouse
npx lighthouse http://localhost:3000 --output=html,json --output-path=.test-reports/lighthouse/home --preset=desktop
npx lighthouse http://localhost:3000 --output=html,json --output-path=.test-reports/lighthouse/home-mobile --preset=mobile

# Bundle analysis
$PM_RUN build 2>&1 | tee .test-reports/build-output.txt
npx @next/bundle-analyzer

# API latency
for i in {1..20}; do curl -s -o /dev/null -w "%{time_total}\n" http://localhost:3000/api/health; done | awk '{sum+=$1} END {print "Avg:", sum/NR}'
```

---

### AGENT 5: Accessibility Tests

**Responsibilities:**
1. Run axe-core on all pages
2. Check WCAG 2.1 AA compliance
3. Test keyboard navigation
4. Check color contrast
5. Verify ARIA usage
6. Test screen reader compatibility

**WCAG 2.1 AA Requirements:**

| Principle | Checks |
|-----------|--------|
| Perceivable | Alt text, contrast 4.5:1, resize to 200%, captions |
| Operable | Keyboard accessible, no traps, skip links, focus visible |
| Understandable | Labels, error identification, consistent navigation |
| Robust | Valid HTML, ARIA attributes, name/role/value |

**Check for:**
- Missing form label → HIGH
- Contrast <4.5:1 → HIGH
- No keyboard access → HIGH
- Missing alt text → HIGH
- ARIA misuse → MEDIUM
- Focus not visible → MEDIUM
- Touch target <44px → MEDIUM
- Skip link missing → LOW

**Commands:**
```bash
# Run axe-core via Playwright
npx playwright test --grep="accessibility" --reporter=json

# Check contrast
npx pa11y http://localhost:3000 --reporter=json

# Validate HTML
npx html-validate "**/*.html"
```

---

### AGENT 6: Code Quality Tests

**Responsibilities:**
1. Check TypeScript strictness (no `any`)
2. Measure test coverage
3. Detect dead code
4. Analyze complexity
5. Find code duplication
6. Check dependency health
7. Audit TODO/FIXME comments
8. Verify API contracts match types

**Type Coverage:**
```bash
# Strict type check
npx tsc --noEmit --strict 2>&1 | tee .test-reports/type-errors.txt

# Count any types
grep -r ": any" --include="*.ts" --include="*.tsx" . | wc -l
```

**Dead Code:**
```bash
npx ts-prune 2>&1 | grep -v "used in module" > .test-reports/dead-code.txt
```

**Complexity:**
```bash
npx eslint --rule 'complexity: [error, 15]' . 2>&1 | tee .test-reports/complexity.txt
```

**Duplication:**
```bash
npx jscpd --min-lines 5 --reporters json --output .test-reports/duplication
```

**TODO/FIXME Audit:**
```bash
grep -rn "TODO\|FIXME\|HACK\|XXX\|@ts-ignore\|@ts-expect-error\|eslint-disable" \
  --include="*.ts" --include="*.tsx" . > .test-reports/todos.txt
```

**Check for:**
- Type error → HIGH
- `any` type in source → MEDIUM
- `@ts-ignore` → MEDIUM
- Complexity >20 → HIGH
- Complexity >15 → MEDIUM
- Duplication >10% → HIGH
- Dead export → LOW
- Outdated dependency (major) → MEDIUM
- TODO older than 6 months → LOW

---

### Status Update: Every 5 Minutes

```
═══════════════════════════════════════════════════════════════
⏳ AUDIT IN PROGRESS — 5:00 elapsed
═══════════════════════════════════════════════════════════════
Agent 1 (Unit):        ████████░░ 80% — 45/50 tests run
Agent 2 (Integration): ██████░░░░ 60% — 15/26 routes tested
Agent 3 (E2E):         ████░░░░░░ 40% — 8/20 flows, 3/7 viewports
Agent 4 (Performance): ██████████ 100% — Complete
Agent 5 (Accessibility): ████████░░ 80% — 12/15 pages scanned
Agent 6 (Quality):     ███████░░░ 70% — Type check complete

Findings so far: 8 (1 CRIT, 2 HIGH, 3 MED, 2 LOW)
═══════════════════════════════════════════════════════════════
```

---

### Merge Findings

After all 6 agents complete:

1. Collect all findings from each agent
2. Assign unique TEST-XXX IDs
3. Deduplicate overlapping findings
4. Sort by severity (CRITICAL → HIGH → MEDIUM → LOW)
5. Group by file for efficient fixing
6. Build dependency graph for fix ordering
7. **Add ALL findings to report `## Findings` table** with status `FOUND`
8. **Append to Progress Log:** `| [HH:MM] | Phase 1 | Audit complete | [X] findings across [Y] agents |`
9. **Update Status:** `🔴 IN PROGRESS — Phase 2: Fix`
10. **Write report to disk** — this is the checkpoint between audit and fix

**Severity Assignment:**

| Level | Criteria |
|-------|----------|
| CRITICAL | Test failing, production crash risk, security issue |
| HIGH | Feature broken, <50% coverage on critical path, a11y violation |
| MEDIUM | Edge case uncovered, performance regression, type error |
| LOW | Missing test, code smell, minor optimization |
| INFO | Suggestion, best practice, cosmetic |

---

### Status Update: Audit Complete

```
═══════════════════════════════════════════════════════════════
✅ PHASE 1: AUDIT COMPLETE
═══════════════════════════════════════════════════════════════
⏱️ Duration: 12:34

📊 Findings Summary
   CRITICAL:  2
   HIGH:      5
   MEDIUM:    12
   LOW:       8
   INFO:      3
   ─────────────
   TOTAL:     30

🔍 By Domain
   Unit:          8 findings
   Integration:   6 findings
   E2E:           4 findings
   Performance:   5 findings
   Accessibility: 4 findings
   Quality:       3 findings

Proceeding to PHASE 2: FIX...
═══════════════════════════════════════════════════════════════
```

---

## PHASE 2: FIX (Parallel by File Ownership)

### Fix Strategy

**DO NOT fix randomly.** Fix in intelligent order:

1. **Dependencies first** — Fix shared utilities before components that use them
2. **Severity order** — CRITICAL before HIGH before MEDIUM
3. **Non-conflicting in parallel** — Multiple agents fix different files simultaneously
4. **Verify after each fix** — Run relevant tests to catch regressions immediately

**Living Report Update Cycle (per finding):**

```
1. Update finding row → 🔧 FIXING
2. Update report header Status → 🟡 FIXING — TEST-XXX [description]
3. Write report to disk

4. ATTEMPT 1: Apply fix, verify (build + test)
   - If passes → ✅ FIXED, add to Fix Log + SITREP "What Was Fixed", write to disk
   - If fails → revert

5. ATTEMPT 2: Alternative approach
   - If passes → ✅ FIXED, write to disk
   - If fails → revert

6. ATTEMPT 3: Different angle
   - If passes → ✅ FIXED
   - If fails → revert, decide:
     a. Needs human decision → ⏸️ DEFERRED, add to SITREP with reason
     b. 3 approaches exhausted → 🚫 BLOCKED, add to SITREP with all 3 attempts documented
     c. Flaky test → 🔒 QUARANTINED, move to quarantine folder

7. Append to Progress Log: | [HH:MM] | Phase 2 | TEST-XXX | ✅/⏸️/🚫/🔒 |
8. Write report to disk — checkpoint after EVERY status change
9. Move to next finding
```

**File Ownership for Parallel Fixing:**

| Agent | Owns | Description |
|-------|------|-------------|
| Agent A | `app/api/**` | API routes, server actions |
| Agent B | `components/**`, `app/**/page.tsx` | UI components, pages |
| Agent C | `__tests__/**`, `*.test.ts`, `*.spec.ts`, `e2e/**` | Test files |
| Agent D | `lib/**`, `hooks/**`, `*.config.*`, `*.json` | Utilities, config |

If a fix requires touching files owned by another agent, it queues the change.

---

### Fix Templates by Category

#### Unit Test: Generate Missing Test

**Analyze existing tests to match style:**
```typescript
// Detect project's test style
const testStyle = detectTestStyle() // 'describe/it', 'test', 'vitest', 'jest'
const importStyle = detectImportStyle() // named, default, namespace
const assertStyle = detectAssertStyle() // expect, assert, chai
```

**Generate test matching project conventions:**
```typescript
import { describe, it, expect, vi } from 'vitest'
import { functionName } from '@/lib/module'

describe('functionName', () => {
  it('handles normal input', () => {
    const result = functionName('valid input')
    expect(result).toBeDefined()
  })

  it('handles edge cases', () => {
    expect(functionName('')).toBe(/* expected */)
    expect(functionName(null)).toBe(/* expected */)
    expect(() => functionName(undefined)).toThrow()
  })

  it('handles errors gracefully', () => {
    // Test error paths
  })
})
```

---

#### Integration Test: Generate API Route Test

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { GET, POST } from './route'
import { NextRequest } from 'next/server'
import { createClient } from '@/lib/supabase/server'

vi.mock('@/lib/supabase/server')

describe('API /api/[route]', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  describe('GET', () => {
    it('returns 401 when unauthenticated', async () => {
      vi.mocked(createClient).mockResolvedValue({
        auth: { getUser: vi.fn().mockResolvedValue({ data: { user: null } }) }
      } as any)

      const req = new NextRequest('http://localhost/api/route')
      const res = await GET(req)
      expect(res.status).toBe(401)
    })

    it('returns data when authenticated', async () => {
      vi.mocked(createClient).mockResolvedValue({
        auth: { getUser: vi.fn().mockResolvedValue({ data: { user: { id: 'test-user' } } }) },
        from: vi.fn().mockReturnValue({
          select: vi.fn().mockReturnThis(),
          eq: vi.fn().mockResolvedValue({ data: [{ id: 1 }], error: null })
        })
      } as any)

      const req = new NextRequest('http://localhost/api/route')
      const res = await GET(req)
      expect(res.status).toBe(200)
    })
  })
})
```

---

#### E2E Test: Fix Touch Target

```typescript
// Finding: Button touch target <44px on mobile
// File: components/button.tsx

// BEFORE
<button className="p-1 text-sm">
  <Icon className="w-4 h-4" />
</button>

// AFTER — minimum 44x44px touch target
<button className="p-3 min-h-[44px] min-w-[44px] flex items-center justify-center text-sm">
  <Icon className="w-4 h-4" />
</button>
```

---

#### Performance: Fix Large Bundle

```typescript
// Finding: Page bundle >200kB
// Cause: Heavy component loaded synchronously

// BEFORE
import { HeavyChart } from '@/components/charts/heavy-chart'

// AFTER — dynamic import
import dynamic from 'next/dynamic'
const HeavyChart = dynamic(
  () => import('@/components/charts/heavy-chart').then(m => m.HeavyChart),
  { ssr: false, loading: () => <ChartSkeleton /> }
)
```

---

#### Performance: Fix Slow LCP

```typescript
// Finding: LCP >2.5s on homepage

// Fix 1: Add priority to hero image
<Image src="/hero.jpg" priority alt="Hero" />

// Fix 2: Preload critical font
// In layout.tsx
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossOrigin="anonymous" />

// Fix 3: Reduce blocking JS
// Move non-critical scripts to bottom or dynamic import
```

---

#### Accessibility: Fix Missing Label

```typescript
// Finding: Form input without label

// BEFORE
<input type="email" placeholder="Email" />

// AFTER — Option 1: Visible label
<label htmlFor="email" className="sr-only">Email address</label>
<input id="email" type="email" placeholder="Email" aria-describedby="email-hint" />

// AFTER — Option 2: aria-label
<input type="email" aria-label="Email address" placeholder="Email" />
```

---

#### Accessibility: Fix Low Contrast

```typescript
// Finding: Text contrast <4.5:1

// BEFORE — gray-400 on white = 2.9:1
<p className="text-gray-400">Subtitle text</p>

// AFTER — gray-600 on white = 5.7:1
<p className="text-gray-600">Subtitle text</p>

// Color contrast reference:
// gray-400 (#9CA3AF) on white = 2.9:1 ❌
// gray-500 (#6B7280) on white = 4.6:1 ✅
// gray-600 (#4B5563) on white = 7.0:1 ✅✅
```

---

#### Quality: Fix Type Error

```typescript
// Finding: Type 'any' used

// BEFORE
function processData(data: any) {
  return data.items.map((item: any) => item.name)
}

// AFTER — proper types
interface DataItem {
  name: string
  id: string
}

interface Data {
  items: DataItem[]
}

function processData(data: Data): string[] {
  return data.items.map((item) => item.name)
}
```

---

#### Quality: Fix High Complexity

```typescript
// Finding: Function complexity >15

// BEFORE — monolithic function
function processRequest(req: Request) {
  // 80 lines of nested conditionals
}

// AFTER — split into focused functions
function validateRequest(req: Request): ValidationResult { /* ... */ }
function extractData(req: Request): RequestData { /* ... */ }
function transformData(data: RequestData): ProcessedData { /* ... */ }
function saveData(data: ProcessedData): Promise<void> { /* ... */ }

function processRequest(req: Request) {
  const validation = validateRequest(req)
  if (!validation.valid) throw new ValidationError(validation.errors)

  const data = extractData(req)
  const transformed = transformData(data)
  return saveData(transformed)
}
```

---

### Flaky Test Quarantine

If a test fails inconsistently (passes sometimes, fails sometimes):

1. Run the test 3 times
2. If results are mixed → flaky
3. Move to quarantine:

```bash
mkdir -p __tests__/quarantine
mv __tests__/flaky.test.ts __tests__/quarantine/
```

4. Add to quarantine log:
```json
// .test-ship-quarantine.json
{
  "quarantined": [
    {
      "file": "__tests__/quarantine/flaky.test.ts",
      "originalPath": "__tests__/flaky.test.ts",
      "reason": "Inconsistent results (2 pass, 1 fail in 3 runs)",
      "quarantinedAt": "2026-02-05T23:00:00Z",
      "failurePattern": "Timeout on async operation"
    }
  ]
}
```

5. Report separately in final summary

---

### Smart Retry Logic

| Failure Type | Retry Strategy |
|--------------|----------------|
| Network timeout | 2s → 5s → 10s (exponential backoff) |
| Rate limit (429) | Wait 60s, retry once |
| Assertion failure | No retry (real failure) |
| Flaky (inconsistent) | Run 3x, majority wins |
| File locked | Wait 1s, retry 5x |
| OOM/crash | Skip, mark as manual |
| Syntax error from fix | Revert immediately |

---

### Breaking Change Detection

Before applying a fix, check if it changes:
- Exported function signature → **WARN**
- Exported type definition → **WARN**
- API response shape → **WARN**
- Component props → **WARN**
- Database schema → **STOP, ask human**

```
⚠️ BREAKING CHANGE DETECTED
   File: lib/api-client.ts
   Change: Function signature modified

   BEFORE: fetchData(id: string): Promise<Data>
   AFTER:  fetchData(id: string, options?: Options): Promise<Data>

   This is backwards-compatible (optional param). Proceeding...
```

If NOT backwards-compatible, flag for human review but continue with other fixes.

---

### Status Update: Every 5 Minutes

```
═══════════════════════════════════════════════════════════════
⏳ FIX IN PROGRESS — 20:00 elapsed
═══════════════════════════════════════════════════════════════
Fix Agent A (API):       ████████░░ 80% — 8/10 fixes applied
Fix Agent B (Components): ██████░░░░ 60% — 6/10 fixes applied
Fix Agent C (Tests):     ████████████ 100% — 5/5 fixes applied
Fix Agent D (Lib/Config): ███████░░░ 70% — 3/5 fixes applied

Progress: 22/30 fixed, 3 in progress, 5 remaining
Regressions caught: 1 (reverted)
Quarantined: 2 flaky tests
═══════════════════════════════════════════════════════════════
```

---

## PHASE 3: VERIFY & SELF-HEAL

### Verification Loop

After all fixes are applied:

```
┌─────────────────────────────────────────────────────────────┐
│                    VERIFICATION LOOP                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌──────────────┐                                         │
│    │  RUN TESTS   │                                         │
│    └───────┬──────┘                                         │
│            │                                                │
│  ┌─────────┴─────────┐                                      │
│  │                   │                                      │
│  ▼                   ▼                                      │
│ [ALL PASS]     [FAILURES]                                   │
│  │                   │                                      │
│  │            ┌──────┴──────┐                               │
│  │            │             │                               │
│  │            ▼             ▼                               │
│  │      [FROM FIX]    [NEW ISSUE]                           │
│  │            │             │                               │
│  │            ▼             ▼                               │
│  │      [REVERT FIX]  [ADD TO QUEUE]                        │
│  │            │             │                               │
│  │            └──────┬──────┘                               │
│  │                   │                                      │
│  │                   ▼                                      │
│  │            [ATTEMPT FIX]                                 │
│  │                   │                                      │
│  │            ┌──────┴──────┐                               │
│  │            │             │                               │
│  │            ▼             ▼                               │
│  │       [SUCCESS]    [FAIL x2]                             │
│  │            │             │                               │
│  │            │             ▼                               │
│  │            │      [ASSESS & STRATEGIZE]                  │
│  │            │             │                               │
│  │            │      ┌──────┴──────┐                        │
│  │            │      │             │                        │
│  │            │      ▼             ▼                        │
│  │            │ [NEW APPROACH] [TRULY STUCK]                │
│  │            │      │             │                        │
│  │            │      │             ▼                        │
│  │            │      │      [MARK MANUAL]                   │
│  │            │      │             │                        │
│  │            └──────┴─────────────┘                        │
│  │                   │                                      │
│  │                   ▼                                      │
│  │            [LOOP BACK]                                   │
│  │                                                          │
│  └──────────────────────────────────────────────────────────│
│                                                             │
│                    [DONE]                                   │
└─────────────────────────────────────────────────────────────┘
```

### "Assess & Strategize" Before Defeat

After 2 failed fix attempts, **DO NOT immediately give up**. Instead:

1. **Analyze the failure pattern:**
   - Is it a dependency issue?
   - Is it a type mismatch?
   - Is it a timing/async issue?
   - Is it environment-specific?

2. **Try alternative approaches:**
   - Different fix strategy
   - Fix upstream dependency first
   - Adjust test instead of code (if test is wrong)
   - Add retry/wait logic for timing issues

3. **Only declare "truly stuck" if:**
   - 3+ different approaches failed
   - Fix requires architectural change
   - Fix requires human decision (business logic)
   - Fix would break backwards compatibility in unacceptable way

```
═══════════════════════════════════════════════════════════════
🔍 ASSESSING STUCK ISSUE: TEST-015
═══════════════════════════════════════════════════════════════
Issue: Integration test for /api/chat times out
Failed attempts:
  1. Increased timeout to 30s → Still fails
  2. Mocked external API → Different error

Analysis:
  • Test requires real OpenRouter connection
  • CI environment may have network restrictions
  • This is an environment issue, not code issue

Strategy:
  • Skip this test in CI (add conditional)
  • Add mock for CI environment
  • Keep real test for local development

Applying new strategy...
═══════════════════════════════════════════════════════════════
```

---

### Dependency Graph Verification

After fixing `lib/utils.ts`, automatically re-test all files that import it:

```bash
# Find all files that import from the fixed file
DEPENDENTS=$(grep -rl "from.*lib/utils" --include="*.ts" --include="*.tsx" .)

# Run tests for each dependent
for file in $DEPENDENTS; do
  testFile="${file%.ts}.test.ts"
  if [ -f "$testFile" ]; then
    $PM_RUN test "$testFile"
  fi
done
```

---

### Status Update: Verification

```
═══════════════════════════════════════════════════════════════
🔍 PHASE 3: VERIFICATION
═══════════════════════════════════════════════════════════════
⏱️ Time: 28:00

Running full test suite...

Unit Tests:        ████████████ 50/50 ✅
Integration Tests: ████████████ 26/26 ✅
E2E Tests:         ██████████░░ 18/20 (2 flaky, quarantined)

New issues discovered: 1
  • TEST-031: Type error introduced by TEST-015 fix

Applying fix for TEST-031...
═══════════════════════════════════════════════════════════════
```

---

## PHASE 4: FINAL REPORT

### Before/After Delta Comparison

```
═══════════════════════════════════════════════════════════════
📊 IMPROVEMENT SUMMARY
═══════════════════════════════════════════════════════════════

                        BEFORE          AFTER           DELTA
─────────────────────────────────────────────────────────────
Test Coverage           67%             84%             +17% ✅
Failing Tests           5               0               -5   ✅
Type Errors             12              0               -12  ✅
Lighthouse (Desktop)    72              89              +17  ✅
Lighthouse (Mobile)     58              78              +20  ✅
Bundle Size             245kB           198kB           -47kB ✅
A11y Violations         8               0               -8   ✅
TODO/FIXME Count        45              42              -3
─────────────────────────────────────────────────────────────
```

---

### Full Findings Table

```
═══════════════════════════════════════════════════════════════
📋 ALL FINDINGS
═══════════════════════════════════════════════════════════════

| # | ID | Severity | Domain | Title | Status |
|---|-----|----------|--------|-------|--------|
| 1 | TEST-001 | CRITICAL | E2E | Login fails on mobile | ✅ Fixed |
| 2 | TEST-002 | CRITICAL | Unit | API route throws unhandled | ✅ Fixed |
| 3 | TEST-003 | HIGH | Integration | Missing auth test | ✅ Fixed |
| 4 | TEST-004 | HIGH | Perf | LCP >4s on /library | ✅ Fixed |
| 5 | TEST-005 | HIGH | A11y | Form missing labels | ✅ Fixed |
| 6 | TEST-006 | HIGH | Quality | 15 type errors | ✅ Fixed |
| 7 | TEST-007 | HIGH | Unit | 0% coverage on route | ✅ Fixed |
| 8 | TEST-008 | MEDIUM | E2E | Slow transition | ✅ Fixed |
| 9 | TEST-009 | MEDIUM | Perf | Bundle 280kB | ✅ Fixed |
| 10 | TEST-010 | MEDIUM | A11y | Low contrast text | ✅ Fixed |
| ... | ... | ... | ... | ... | ... |
| 28 | TEST-028 | LOW | Quality | Unused export | ✅ Fixed |
| 29 | TEST-029 | LOW | Unit | Skipped test | ⏭️ Skipped |
| 30 | TEST-030 | LOW | Quality | TODO comment | ⏭️ Skipped |

─────────────────────────────────────────────────────────────
SUMMARY: 28 Fixed | 2 Skipped | 2 Quarantined (flaky)
═══════════════════════════════════════════════════════════════
```

---

### Quarantined Tests

```
═══════════════════════════════════════════════════════════════
🔒 QUARANTINED (FLAKY TESTS)
═══════════════════════════════════════════════════════════════

| Test File | Reason | Moved To |
|-----------|--------|----------|
| auth.test.ts | 2/3 pass, timing issue | __tests__/quarantine/ |
| api-rate.test.ts | Network dependent | __tests__/quarantine/ |

These tests pass inconsistently and have been moved to quarantine
to prevent blocking the pipeline. Review and fix the root cause
when time permits.
═══════════════════════════════════════════════════════════════
```

---

### Skipped (Manual Needed)

```
═══════════════════════════════════════════════════════════════
⏭️ SKIPPED (REQUIRES MANUAL INTERVENTION)
═══════════════════════════════════════════════════════════════

| ID | Title | Reason |
|----|-------|--------|
| TEST-029 | Skipped test in auth.test.ts | Intentionally skipped by developer (// skip: WIP) |
| TEST-030 | TODO in lib/utils.ts | Business decision needed: "TODO: decide on retry strategy" |

These items require human decision or are intentionally deferred.
═══════════════════════════════════════════════════════════════
```

---

### Historical Trend

```
═══════════════════════════════════════════════════════════════
📈 HISTORICAL TREND
═══════════════════════════════════════════════════════════════

Date        Coverage   Findings   Fixed    Lighthouse
──────────────────────────────────────────────────────
2026-01-15     52%        42        35         62
2026-01-22     61%        28        25         71
2026-02-01     67%        24        20         72
2026-02-05     84%        30        28         89  ← TODAY

Overall improvement since first run:
  • Coverage: +32% (52% → 84%)
  • Lighthouse: +27 points (62 → 89)
  • Findings addressed: 108 total
═══════════════════════════════════════════════════════════════
```

---

### Recommendations

```
═══════════════════════════════════════════════════════════════
💡 RECOMMENDATIONS
═══════════════════════════════════════════════════════════════

1. HIGH PRIORITY
   • Fix the 2 quarantined flaky tests — they indicate timing issues
     that could cause intermittent production bugs

2. MEDIUM PRIORITY
   • Address 42 remaining TODO/FIXME comments — technical debt accumulating
   • Consider adding E2E tests for admin flows (currently 0 coverage)

3. LOW PRIORITY
   • Mobile Lighthouse is 78 (target 85) — optimize images and reduce JS
   • Consider adding visual regression testing for UI-heavy pages

4. MAINTENANCE
   • Run /test-ship weekly to catch regressions early
   • Add pre-commit hook to run /test-ship --changed
═══════════════════════════════════════════════════════════════
```

---

### Save Report

Save to `.test-reports/test-YYYY-MM-DD-HHMMSS.md` with full content.

Update `.test-ship-history.json`:
```json
{
  "runs": [
    {
      "date": "2026-02-05T23:45:00Z",
      "duration": "32:15",
      "coverage": 84,
      "findings": 30,
      "fixed": 28,
      "skipped": 2,
      "quarantined": 2,
      "lighthouse": { "desktop": 89, "mobile": 78 },
      "typeErrors": 0,
      "testsPassing": 96,
      "testsFailing": 0
    }
  ]
}
```

---

## PHASE 5: CLEANUP

### E2E Test User Teardown (Guaranteed)

Delete all test users and cascaded resources created during seeding. This MUST run even if tests fail to prevent test data accumulation.

```bash
# Teardown E2E test users
if [ "$E2E_USERS_SEEDED" = "true" ] || [ -f "tests/e2e/setup/teardown-test-users.ts" ]; then
  echo "🧹 Tearing down E2E test users..."
  npx tsx tests/e2e/setup/teardown-test-users.ts
  if [ $? -eq 0 ]; then
    echo "✅ E2E test users deleted"
  else
    echo "⚠️ E2E test user teardown failed — manual cleanup may be needed"
  fi
fi

# Delete auth storage state files
rm -rf tests/e2e/.auth/*.json 2>/dev/null || true
```

### Resource Cleanup (Guaranteed)

```bash
# Stop dev server
if [ -n "$DEV_SERVER_PID" ]; then
  kill $DEV_SERVER_PID 2>/dev/null || true
  echo "✅ Dev server stopped"
fi

# Kill any orphaned processes
pkill -f "next dev" 2>/dev/null || true
pkill -f "playwright" 2>/dev/null || true

# Release ports
lsof -ti:3000 | xargs kill -9 2>/dev/null || true

# Clean temp files
rm -rf .test-ship-state.json 2>/dev/null || true
rm -rf /tmp/playwright* 2>/dev/null || true

# Clear test caches (optional, saves disk)
rm -rf node_modules/.cache/vitest 2>/dev/null || true
```

### Final State Save (For Next Run Comparison)

```bash
# Keep baseline for next run comparison
mv .test-ship-baseline.json .test-ship-last-baseline.json 2>/dev/null || true
```

---

## FINAL OUTPUT (Mandatory)

```
═══════════════════════════════════════════════════════════════════════════════
🎉 /test-ship COMPLETE
═══════════════════════════════════════════════════════════════════════════════

⏱️ Total Duration: 32 minutes 15 seconds

📊 RESULTS SUMMARY
─────────────────────────────────────────────────────────────────────────────

| Metric              | Before    | After     | Change        |
|---------------------|-----------|-----------|---------------|
| Test Coverage       | 67%       | 84%       | +17% ✅       |
| Failing Tests       | 5         | 0         | -5 ✅         |
| Type Errors         | 12        | 0         | -12 ✅        |
| Lighthouse Desktop  | 72        | 89        | +17 ✅        |
| Lighthouse Mobile   | 58        | 78        | +20 ✅        |
| Bundle Size         | 245kB     | 198kB     | -47kB ✅      |
| A11y Violations     | 8         | 0         | -8 ✅         |

─────────────────────────────────────────────────────────────────────────────

📋 FINDINGS: 30 total
   ✅ Fixed:       28
   ⏭️ Skipped:     2 (manual needed)
   🔒 Quarantined: 2 (flaky tests)

🧪 TEST STATUS
   Unit:        50/50 passing ✅
   Integration: 26/26 passing ✅
   E2E:         18/18 passing ✅ (2 quarantined)

🏗️ BUILD STATUS: Passing ✅

📁 REPORTS
   • Full report:  .test-reports/test-2026-02-05-234500.md
   • Coverage:     .test-reports/coverage/index.html
   • Lighthouse:   .test-reports/lighthouse/
   • Screenshots:  .test-reports/screenshots/

💡 NEXT STEPS
   1. Review the 2 quarantined flaky tests
   2. Address 2 skipped items requiring manual decision
   3. Run /gh-ship to push improvements to GitHub

═══════════════════════════════════════════════════════════════════════════════
```

---

## Human Intervention Triggers

The skill will **STOP and ask** only for these truly dangerous operations:

| Trigger | Why Dangerous |
|---------|---------------|
| Delete production database table | Irreversible data loss |
| Modify auth/security configuration | Could lock out users |
| Major dependency upgrade (semver major) | Could break production |
| Remove public API endpoint | Could break external consumers |
| Modify billing/payment code | Financial risk |
| Change database schema (column drop) | Data loss risk |

For everything else, proceed automatically.

---

## Rollback Procedure

If catastrophic failure:

```bash
echo "🔄 Rolling back to pre-test-ship state..."

git reset --hard $TEST_SHIP_BASE
git clean -fd

# Restore quarantine state
git checkout -- .test-ship-quarantine.json 2>/dev/null || true

echo "✅ Rolled back to: $TEST_SHIP_BASE"
echo "All changes from this /test-ship run have been reverted."
```

---

## Important Notes

- This skill is **FULLY AUTONOMOUS** — it runs from start to finish without waiting
- **Report is the living task list** — updated after every status change, written to disk constantly
- **Never delete findings** — only update status: ✅ FIXED / ⏸️ DEFERRED / 🚫 BLOCKED / 🔒 QUARANTINED
- **3 attempts before defeat** — try 3 different approaches, document each, then mark BLOCKED
- **SITREP annotates everything** — what was fixed, what was deferred (WHY), what was blocked (WHY + all attempts)
- **Report survives session restart** — resume from where it left off via Progress Log
- **Status updates every 5 minutes** keep you informed without requiring response
- **Self-healing loops** continue until truly stuck (not just first failure)
- **Parallel execution** maximizes speed (6 audit agents, 4 fix agents)
- **Historical tracking** shows improvement over time
- **All outputs gitignored** — reports don't pollute your repo
- Run `/test-ship --changed` for quick pre-commit validation

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
