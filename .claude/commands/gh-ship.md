# /gh-ship — Full Git Pipeline: Commit → Push → PR → CI → Fix → Merge → Deploy → Cleanup

You are a Git automation specialist. Execute the COMPLETE shipping pipeline autonomously from start to finish. **No stale branches, no uncommitted work, no half-finished PRs.** Everything ships professionally or rolls back cleanly.

## CRITICAL RULES

1. **NEVER ask for permission** - just do it
2. **NEVER pause for confirmation** - proceed automatically
3. **NEVER skip steps** - complete the full pipeline
4. **FIX issues automatically** - don't report and stop, FIX and continue
5. **Generate smart commit messages** - analyze the changes, write proper titles
6. **Retry on failure** - up to 3 attempts for any fixable issue
7. **Rollback on catastrophic failure** - restore original state if unrecoverable
8. **Verify deployments** - check both preview and production are live
9. **ALWAYS cleanup** - delete branches, close PRs, verify clean state
10. **NEVER leave stale work** - if it can't ship, it must rollback

---

## CI/CD WORKFLOW ARCHITECTURE

**IMPORTANT:** Understand the deployment pipeline before proceeding.

### Workflow Structure

```
┌─────────────────────┐
│  Feature Branch     │
│  (your changes)     │
└──────────┬──────────┘
           │
           │ git push
           ↓
┌─────────────────────┐
│  Pull Request       │
│  (created by gh)    │
└──────────┬──────────┘
           │
           │ triggers
           ↓
┌─────────────────────┐
│  GitHub Actions: CI │──── Lint, TypeCheck, Security, Build, E2E Tests
└──────────┬──────────┘
           │
           │ CI passes
           ↓
┌─────────────────────┐
│  vercel[bot]        │──── Preview deployment (automatic)
│  Preview Deploy     │
└─────────────────────┘
           │
           │ PR merged
           ↓
┌─────────────────────┐
│  Main Branch        │
└──────────┬──────────┘
           │
           │ triggers BOTH
           ↓
     ┌─────┴─────┐
     │           │
     ↓           ↓
┌────────┐  ┌───────────────┐
│   CI   │  │  Migrations   │
│ (again)│  │  Workflow     │
└────┬───┘  └───────┬───────┘
     │              │
     │ passes       │ applies migrations
     └──────┬───────┘
            ↓
┌─────────────────────┐
│  vercel[bot]        │──── Production deployment (automatic)
│  Production Deploy  │      (waits for CI if configured)
└─────────────────────┘
            │
            ↓
┌─────────────────────┐
│  CLEANUP            │──── Delete branch, close PR, verify clean
└─────────────────────┘
```

### Key Points

1. **CI runs on BOTH pull_request AND push to main**
   - This prevents broken code from bypassing checks via direct push
   - CI must pass before production deployment

2. **Database migrations run automatically**
   - Separate workflow (`.github/workflows/deploy.yml`)
   - Applies Supabase migrations on push to main
   - Checks for new migrations in `supabase/migrations/`

3. **Vercel deploys via GitHub bot**
   - Preview: Automatic on PR creation/update
   - Production: Automatic after merge to main
   - **No manual Vercel CLI deployment needed**

4. **Deployment Protection (RECOMMENDED)**
   - Configure in Vercel dashboard: Settings → Git → "Wait for Checks"
   - Select required checks: CI / Lint, CI / Build, CI / E2E Tests
   - Vercel will NOT deploy if CI fails

5. **Branch Protection (RECOMMENDED)**
   - Configure in GitHub: Settings → Branches → main
   - Require status checks to pass before merging
   - Prevents direct push to main without PR review

6. **Cleanup is MANDATORY**
   - Branch deleted remotely and locally after merge
   - PR closed and verified
   - Local state clean (no uncommitted work)
   - Status verified (no stale branches remain)

### Manual Configuration Required

These settings cannot be automated and must be configured manually:

**Vercel Dashboard** (per project):
- Go to: Vercel Dashboard → [Project] → Settings → Git
- Enable: "Wait for Checks"
- Select: CI workflow checks to wait for

**GitHub Repository** (per project):
- Go to: GitHub → [Repo] → Settings → Branches
- Enable branch protection for `main`
- Require: Status checks to pass before merging

---

## STAGE 0: SAVE STATE & PRE-FLIGHT CHECKS

### 0.1 Save Rollback State
```bash
START_TIME=$(date +%s)
ORIGINAL_BRANCH=$(git branch --show-current)
ORIGINAL_COMMIT=$(git rev-parse HEAD)
ORIGINAL_STASH=""
```
Store these for rollback if needed.

### 0.2 Detect Stale Branches & Uncommitted Work

**Check for stale local branches:**
```bash
# Find merged branches that weren't deleted
STALE_LOCAL=$(git branch --merged main | grep -v "main\|master\|\*" | xargs)
if [ -n "$STALE_LOCAL" ]; then
  echo "🧹 Cleaning up $(echo "$STALE_LOCAL" | wc -w) stale local branches..."
  echo "$STALE_LOCAL" | xargs git branch -d
  echo "✅ Deleted stale branches: $STALE_LOCAL"
fi
```

**Check for uncommitted changes in current branch:**
```bash
if [ -n "$(git status --porcelain)" ]; then
  echo "📝 Found uncommitted work - will commit and ship"
else
  echo "❌ Nothing to ship - no changes detected"
  exit 0
fi
```

**Check for unmerged remote branches (warning only):**
```bash
# Warn about other unmerged branches
REMOTE_BRANCHES=$(git branch -r | grep -v "HEAD\|main\|master" | wc -l)
if [ "$REMOTE_BRANCHES" -gt 0 ]; then
  echo "⚠️ Warning: $REMOTE_BRANCHES unmerged remote branches exist"
  echo "   Run this again from those branches to ship them"
fi
```

### 0.3 Directory Exclusion Check
Check if current directory should NOT be a git repo (per CLAUDE.md rules):
```bash
# Read CLAUDE.md and check for "NO GIT" or "NEVER initialize a git repo" rules
# If this directory is excluded, abort immediately with explanation
```

**Known excluded directories:**
- `/Users/airborneshellback/vibecode-projects/n8n-workflows/` — managed via n8n UI, not git

If in excluded directory → **ABORT** with message: "This directory is not version controlled per CLAUDE.md rules."

### 0.4 Environment Checks
Run ALL of these. If any fail, fix or abort:

```bash
# Must be in a git repo
git rev-parse --git-dir > /dev/null 2>&1 || { echo "❌ Not a git repo"; exit 1; }

# Must have a remote
git remote get-url origin > /dev/null 2>&1 || { echo "❌ No remote configured"; exit 1; }

# GitHub CLI must be authenticated
gh auth status > /dev/null 2>&1 || { echo "❌ GitHub CLI not authenticated. Run: gh auth login"; exit 1; }

# Check for uncommitted changes
if [ -z "$(git status --porcelain)" ]; then
  echo "❌ Nothing to ship - no changes detected"
  exit 0
fi
```

### 0.5 Detect Package Manager
```bash
if [ -f "bun.lockb" ]; then
  PM="bun"; PM_RUN="bun run"; PM_INSTALL="bun install"
elif [ -f "pnpm-lock.yaml" ]; then
  PM="pnpm"; PM_RUN="pnpm run"; PM_INSTALL="pnpm install"
elif [ -f "yarn.lock" ]; then
  PM="yarn"; PM_RUN="yarn"; PM_INSTALL="yarn install"
else
  PM="npm"; PM_RUN="npm run"; PM_INSTALL="npm install"
fi
```

### 0.6 Detect Linter/Formatter
```bash
if [ -f "biome.json" ] || [ -f "biome.jsonc" ]; then
  LINTER="biome"; LINTER_FIX="npx @biomejs/biome check --write ."
elif [ -f "deno.json" ] || [ -f "deno.jsonc" ]; then
  LINTER="deno"; LINTER_FIX="deno fmt && deno lint --fix"
else
  LINTER="eslint"; LINTER_FIX="npx eslint --fix . && npx prettier --write ."
fi
```

### 0.7 Stash Unstaged Changes
```bash
# If there are unstaged changes (modified but not added), stash them
UNSTAGED=$(git diff --name-only)
if [ -n "$UNSTAGED" ]; then
  git stash push -m "gh-ship-auto-stash-$(date +%s)"
  ORIGINAL_STASH="yes"
  echo "📦 Stashed unstaged changes for safety"
fi
```

### 0.8 Safety Scans

#### Comprehensive Secrets Detection
```bash
# Extended secrets pattern - catches most common API keys and tokens
SECRETS_PATTERN='(
  API_KEY|SECRET_KEY|PRIVATE_KEY|PASSWORD|ACCESS_TOKEN|CLIENT_SECRET|
  DATABASE_URL|SUPABASE_SERVICE_ROLE|SUPABASE_ANON_KEY|
  sk-[a-zA-Z0-9]{32,}|
  sk_live_[a-zA-Z0-9]{24,}|sk_test_[a-zA-Z0-9]{24,}|
  pk_live_[a-zA-Z0-9]{24,}|pk_test_[a-zA-Z0-9]{24,}|
  ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|
  github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}|
  xox[baprs]-[a-zA-Z0-9-]{10,}|
  AKIA[0-9A-Z]{16}|
  AIza[0-9A-Za-z_-]{35}|
  SG\.[a-zA-Z0-9_-]{22}\.[a-zA-Z0-9_-]{43}|
  SK[a-z0-9]{32}|
  eyJ[a-zA-Z0-9_-]*\.eyJ[a-zA-Z0-9_-]*|
  -----BEGIN (RSA|DSA|EC|OPENSSH|PGP) PRIVATE KEY-----
)'

SECRETS_FOUND=$(git diff --cached --name-only | xargs grep -l -E "$SECRETS_PATTERN" 2>/dev/null || true)
```

**If secrets found → AUTO-FIX:**
1. Identify the file(s)
2. Check if it's a `.env` file → Add `.env*` to `.gitignore`, unstage it
3. Check if it's a config file with hardcoded secrets → Replace with `process.env.VAR_NAME`, create/update `.env.example`
4. Unstage the problematic file: `git reset HEAD <file>`
5. Log: "🔒 Removed secrets from commit: <file>"

#### Large File Detection
```bash
# Find files over 50MB
LARGE_FILES=$(find . -path ./.git -prune -o -type f -size +50M -print 2>/dev/null)
```

**If large files found → AUTO-FIX:**
1. Check file type:
   - **Binary (images, videos, etc.):**
     - Add to `.gitignore`
     - Unstage: `git reset HEAD <file>`
     - Log: "📦 Large binary excluded: <file> - consider Git LFS"
   - **Data files (JSON, CSV, SQL dumps):**
     - Add to `.gitignore`
     - Unstage
     - Log: "📦 Large data file excluded: <file>"
   - **Compiled/build artifacts:**
     - Add to `.gitignore`
     - Unstage
     - Log: "🗑️ Build artifact excluded: <file>"

#### .env Files Check
```bash
# Never commit .env files
ENV_FILES=$(git diff --cached --name-only | grep -E '^\.env' || true)
```

**If .env files staged → AUTO-FIX:**
1. Add `.env*` to `.gitignore` (if not already)
2. Unstage: `git reset HEAD .env*`
3. Create `.env.example` with placeholder keys (no values)
4. Stage `.gitignore` and `.env.example`

#### node_modules / Build Artifacts Check
```bash
# Never commit these
EXCLUDED_DIRS="node_modules/|\.next/|dist/|build/|\.turbo/|\.vercel/"
if git diff --cached --name-only | grep -qE "$EXCLUDED_DIRS"; then
  echo "Build artifacts or node_modules detected"
fi
```

**If found → AUTO-FIX:**
1. Add to `.gitignore`
2. Unstage: `git reset HEAD <dir>/`
3. Stage `.gitignore`

---

## STAGE 1: Pre-Push Local Validation

Before pushing, run local checks to catch errors early:

```bash
# Only if package.json exists
if [ -f "package.json" ]; then
  # Type check (if script exists)
  $PM_RUN typecheck --if-present 2>/dev/null || $PM_RUN tsc --noEmit 2>/dev/null || true

  # Lint check (if script exists)
  $PM_RUN lint --if-present 2>/dev/null || true

  # Build check (for Next.js/Vite projects)
  # Skip full build, just check for obvious errors
fi
```

**If local validation fails → AUTO-FIX:**
- Run `$LINTER_FIX` to auto-fix linting issues
- For TypeScript errors: attempt to fix, or log and continue

---

## STAGE 2: Analyze Changes

```bash
git status --short
git diff --staged --stat
git diff --stat
```

- Count files changed
- Identify types: source code, config, docs, tests, assets
- Determine change category (feat/fix/refactor/docs/chore/test/style)

**If no changes after safety scans:** Report "Nothing to ship after safety filtering" and exit cleanly.

---

## STAGE 3: Generate Commit Message

Analyze the diff content and generate a semantic commit message:

**Format:** `<type>(<scope>): <description>`

| Type | When to Use |
|------|-------------|
| `feat` | New feature, new functionality |
| `fix` | Bug fix, error correction |
| `refactor` | Code restructure, no behavior change |
| `docs` | Documentation only |
| `chore` | Maintenance, dependencies, config |
| `test` | Adding or fixing tests |
| `style` | Formatting, whitespace, linting |
| `perf` | Performance improvement |

**Rules:**
- Max 72 characters
- Imperative mood ("Add feature" not "Added feature")
- No period at end
- Be specific about WHAT changed
- Add scope if changes are focused (e.g., `fix(auth)`, `feat(api)`)

**Examples:**
- `feat(reddit): Add comprehensive monitoring keywords`
- `fix(schema): Resolve Supabase permission error`
- `chore(deps): Update dependencies and lock file`
- `refactor(api): Extract client into separate module`

---

## STAGE 4: Stage & Commit

```bash
# Stage all changes (safety-scanned files already excluded)
git add -A

# Commit with generated message
git commit -m "$(cat <<'EOF'
<generated-message>

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

**If pre-commit hooks fail → AUTO-FIX:**

1. **ESLint/Biome errors:**
   ```bash
   $LINTER_FIX
   git add -A
   git commit -m "<message>"
   ```

2. **Prettier errors:**
   ```bash
   npx prettier --write .
   git add -A
   git commit -m "<message>"
   ```

3. **TypeScript errors:**
   - Read the error output
   - Fix the type issue in code
   - Re-stage and commit

4. **Other hook failures:**
   - If fixable: fix and retry
   - If not fixable after 3 attempts: `git commit --no-verify` with warning log

---

## STAGE 5: Branch Management

```bash
CURRENT_BRANCH=$(git branch --show-current)
```

**If on main/master → Create feature branch:**
```bash
# Generate branch name from commit message
BRANCH_NAME="feature/$(echo '<commit-message>' | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | cut -c1-50)"
git checkout -b "$BRANCH_NAME"
```

**If already on feature branch:** Continue on current branch.

---

## STAGE 6: Push to Remote

```bash
git push -u origin HEAD
```

**If push rejected (branch diverged) → AUTO-FIX:**
```bash
git fetch origin
git rebase origin/main

# If rebase conflicts:
# 1. For lock files - regenerate:
if git diff --name-only --diff-filter=U | grep -qE "package-lock\.json|pnpm-lock\.yaml|yarn\.lock|bun\.lockb"; then
  git checkout --theirs package-lock.json pnpm-lock.yaml yarn.lock bun.lockb 2>/dev/null || true
  $PM_INSTALL
  git add package-lock.json pnpm-lock.yaml yarn.lock bun.lockb 2>/dev/null || true
fi

# 2. For generated files (*.generated.ts, *.d.ts): prefer theirs
git checkout --theirs "*.generated.ts" "*.d.ts" 2>/dev/null || true

# 3. For source files: prefer ours (our changes)
# Manual review if complex conflicts

# 4. Continue rebase
git rebase --continue

# 5. Force push (safe on feature branch)
git push --force-with-lease origin HEAD
```

**If push fails (network) → Retry:**
- Wait 2 seconds, retry
- Wait 5 seconds, retry
- Wait 10 seconds, retry
- If still failing: log error and abort

---

## STAGE 7: Create Pull Request

```bash
# Check if PR already exists
EXISTING_PR=$(gh pr list --head "$(git branch --show-current)" --json number --jq '.[0].number' 2>/dev/null)

if [ -n "$EXISTING_PR" ]; then
  echo "PR #$EXISTING_PR already exists, using it"
  PR_NUMBER=$EXISTING_PR
else
  # Create new PR
  PR_URL=$(gh pr create \
    --title "<commit-message>" \
    --body "$(cat <<'EOF'
## Summary
<2-3 bullets describing the changes>

## Changes
<list key files changed>

## Test Plan
- [ ] CI passes
- [ ] Vercel preview works
- [ ] Changes verified

---
🤖 Shipped with [Claude Code](https://claude.com/claude-code)
EOF
)" 2>&1)

  PR_NUMBER=$(echo "$PR_URL" | grep -oE '[0-9]+$')
fi
```

**If PR creation fails → Diagnose:**
- Rate limited: Wait 60 seconds, retry
- Auth error: Log and abort (can't auto-fix)
- Network error: Retry with backoff

---

## STAGE 8: Monitor CI & Vercel Preview

### CI/CD Architecture Overview

**IMPORTANT:** The workflow has been restructured for safety and efficiency:

1. **CI Workflow** (`.github/workflows/ci.yml`)
   - Triggers on BOTH `pull_request` AND `push` to `main`
   - Runs: Lint, TypeCheck, Security Audit, Build, E2E Tests
   - Prevents broken code from reaching production

2. **Database Migrations** (`.github/workflows/deploy.yml`)
   - Runs after CI passes on push to main
   - Applies Supabase migrations automatically

3. **Vercel Deployment** (Automatic via GitHub Integration)
   - Handled by `vercel[bot]` (Vercel GitHub App)
   - Deploys preview on PR creation/update
   - Deploys production after merge to main
   - **Should be configured to "Wait for Checks"** in Vercel dashboard

**No manual Vercel CLI deployment** - The vercel[bot] handles everything.

### 8.1 Wait for CI Checks
```bash
MAX_WAIT=600  # 10 minutes
ELAPSED=0
INTERVAL=15

while [ $ELAPSED -lt $MAX_WAIT ]; do
  # Get all check statuses
  CI_STATUS=$(gh pr checks "$PR_NUMBER" --json state --jq '.[].state' 2>/dev/null | sort -u)

  if echo "$CI_STATUS" | grep -q "FAILURE"; then
    echo "❌ CI failed - attempting fix"
    break
  elif echo "$CI_STATUS" | grep -q "PENDING"; then
    echo "⏳ CI in progress..."
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  elif echo "$CI_STATUS" | grep -q "SUCCESS"; then
    echo "✅ CI passed!"
    break
  else
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  fi
done
```

### 8.2 Wait for Vercel Bot Deployment
```bash
# Vercel bot deploys automatically via GitHub integration
# Check for vercel[bot] deployment status
MAX_VERCEL_WAIT=300  # 5 minutes for Vercel
VERCEL_ELAPSED=0

while [ $VERCEL_ELAPSED -lt $MAX_VERCEL_WAIT ]; do
  VERCEL_CHECK=$(gh pr checks "$PR_NUMBER" --json name,status,conclusion --jq '.[] | select(.name | contains("vercel")) | {name, status, conclusion}' 2>/dev/null)

  if echo "$VERCEL_CHECK" | grep -q '"conclusion":"SUCCESS"'; then
    echo "✅ Vercel deployment complete"
    break
  elif echo "$VERCEL_CHECK" | grep -q '"status":"IN_PROGRESS"'; then
    echo "⏳ Vercel deployment in progress..."
    sleep $INTERVAL
    VERCEL_ELAPSED=$((VERCEL_ELAPSED + INTERVAL))
  else
    sleep $INTERVAL
    VERCEL_ELAPSED=$((VERCEL_ELAPSED + INTERVAL))
  fi
done
```

### 8.3 Get Vercel Preview URL
```bash
# Get preview URL from vercel[bot] comment
PREVIEW_URL=$(gh pr view "$PR_NUMBER" --json comments --jq '.comments[] | select(.author.login=="vercel") | .body' | grep -oE 'https://[a-zA-Z0-9-]+\.vercel\.app' | head -1)

# Or from deployment checks
if [ -z "$PREVIEW_URL" ]; then
  PREVIEW_URL=$(gh pr checks "$PR_NUMBER" --json name,targetUrl --jq '.[] | select(.name | contains("vercel")) | .targetUrl' 2>/dev/null | head -1)
fi
```

### 8.4 Verify Preview Deployment
```bash
# Check preview is accessible
if [ -n "$PREVIEW_URL" ]; then
  PREVIEW_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$PREVIEW_URL" 2>/dev/null || echo "000")
  if [ "$PREVIEW_STATUS" = "200" ]; then
    echo "✅ Preview deployment live: $PREVIEW_URL"
  else
    echo "⚠️ Preview returned HTTP $PREVIEW_STATUS"
  fi
else
  echo "⚠️ Vercel preview URL not found (check may still be pending)"
fi
```

---

## STAGE 9: Fix CI Failures (Auto-Remediation Loop)

**Max 3 fix attempts.** For each failure:

### 9.1 Get Failure Details
```bash
gh pr checks "$PR_NUMBER"
FAILED_RUN=$(gh run list --limit 5 --json databaseId,status,conclusion --jq '.[] | select(.conclusion=="failure") | .databaseId' | head -1)
gh run view "$FAILED_RUN" --log-failed 2>/dev/null | tail -100
```

### 9.2 Identify & Fix Issue

| Error Pattern | Auto-Fix Action |
|---------------|-----------------|
| `eslint` / `Linting` | `$LINTER_FIX && git add -A && git commit -m "fix: Lint errors" && git push` |
| `biome` | `npx @biomejs/biome check --write . && git add -A && git commit -m "fix: Biome errors" && git push` |
| `prettier` / `Formatting` | `npx prettier --write . && git add -A && git commit -m "fix: Formatting" && git push` |
| `tsc` / `TypeScript` / `Type error` | Read error, fix type in code, commit, push |
| `test` / `jest` / `vitest` / `FAIL` | Read failing test, fix code or test, commit, push |
| `build` / `Build failed` | Read error, fix build issue, commit, push |
| `npm audit` / `vulnerability` | `$PM_INSTALL audit fix && git add -A && git commit -m "fix: Security vulnerabilities" && git push` |
| `npm ERR!` / `dependency` | `rm -rf node_modules *.lock* && $PM_INSTALL && git add -A && git commit -m "fix: Regenerate dependencies" && git push` |
| `ENOSPC` / `disk space` | Cannot auto-fix, log warning |
| `rate limit` | Wait 60 seconds, retry |
| `timeout` | Retry CI: `gh run rerun "$FAILED_RUN"` |

### 9.3 After Fix
- Push changes
- Return to STAGE 8 (monitor CI again)
- Decrement attempt counter

### 9.4 If 3 Attempts Failed
- Log detailed error information
- Do NOT merge broken code
- Report: "CI unfixable after 3 attempts: <specific error>"
- Keep PR open for manual review
- Exit with error status

---

## STAGE 10: Merge PR

Once CI passes:

```bash
# Check for required reviews
REVIEW_STATUS=$(gh pr view "$PR_NUMBER" --json reviewDecision --jq '.reviewDecision')

if [ "$REVIEW_STATUS" = "REVIEW_REQUIRED" ]; then
  echo "⚠️ PR requires review approval - cannot auto-merge"
  echo "PR ready for review: $(gh pr view $PR_NUMBER --json url --jq '.url')"
  exit 0  # Exit successfully, but don't merge
fi

# Merge with squash
gh pr merge "$PR_NUMBER" --squash --delete-branch
```

**If merge fails:**
- **Merge conflict:**
  ```bash
  git fetch origin main
  git rebase origin/main
  # Resolve conflicts (prefer incoming for lock files)
  git push --force-with-lease
  # Retry merge
  ```
- **Branch protection:** Log requirements and exit (can't bypass)
- **Network error:** Retry with backoff

---

## STAGE 11: Verify Production Deployment

After merge to main, verify production deployment:

### Overview

After PR merge to main:
1. **CI runs automatically** on push to main (quality gate)
2. **Database Migrations** workflow applies any new migrations
3. **Vercel bot deploys** to production (if CI passes)

**IMPORTANT:** If "Wait for Checks" is configured in Vercel dashboard, production deployment will wait for CI to pass. This prevents broken code from reaching production.

### 11.1 Wait for CI on Main Branch

After merge, CI runs again on main branch:
```bash
echo "⏳ Waiting for CI to run on main branch..."

MAX_WAIT=600  # 10 minutes
ELAPSED=0
INTERVAL=15

while [ $ELAPSED -lt $MAX_WAIT ]; do
  # Get latest workflow run on main
  MAIN_CI_STATUS=$(gh run list --branch main --limit 1 --json status,conclusion --jq '.[0] | {status, conclusion}' 2>/dev/null)

  if echo "$MAIN_CI_STATUS" | grep -q '"conclusion":"success"'; then
    echo "✅ CI passed on main"
    break
  elif echo "$MAIN_CI_STATUS" | grep -q '"conclusion":"failure"'; then
    echo "❌ CI failed on main - production deployment blocked"
    PROD_VERIFIED="ci-failed"
    break
  elif echo "$MAIN_CI_STATUS" | grep -q '"status":"in_progress"'; then
    echo "⏳ CI running on main..."
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  else
    sleep $INTERVAL
    ELAPSED=$((ELAPSED + INTERVAL))
  fi
done
```

### 11.2 Wait for Vercel Production Deployment

```bash
# Vercel bot deploys to production after CI passes
echo "⏳ Waiting for Vercel production deployment..."

MAX_VERCEL_WAIT=300  # 5 minutes
VERCEL_ELAPSED=0

while [ $VERCEL_ELAPSED -lt $MAX_VERCEL_WAIT ]; do
  # Check for vercel[bot] deployment on main branch
  VERCEL_DEPLOY=$(gh api repos/:owner/:repo/deployments --jq '.[] | select(.ref=="main" and .creator.login=="vercel[bot]") | {id, state, environment}' 2>/dev/null | head -1)

  if echo "$VERCEL_DEPLOY" | grep -q '"state":"success"'; then
    echo "✅ Vercel production deployment successful"
    break
  elif echo "$VERCEL_DEPLOY" | grep -q '"state":"in_progress"'; then
    echo "⏳ Vercel deploying to production..."
    sleep $INTERVAL
    VERCEL_ELAPSED=$((VERCEL_ELAPSED + INTERVAL))
  else
    sleep $INTERVAL
    VERCEL_ELAPSED=$((VERCEL_ELAPSED + INTERVAL))
  fi
done
```

### 11.3 Get Production Domain & Verify
```bash
# Get production domain from repository homepage or package.json
PROD_DOMAIN=$(gh api repos/:owner/:repo --jq '.homepage' 2>/dev/null || echo "")

if [ -z "$PROD_DOMAIN" ]; then
  # Fallback: read from package.json if exists
  if [ -f "package.json" ]; then
    PROD_DOMAIN=$(jq -r '.homepage // empty' package.json 2>/dev/null)
  fi
fi

# Verify production is live
if [ -n "$PROD_DOMAIN" ]; then
  # Wait a moment for DNS/CDN to propagate
  sleep 5

  PROD_HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$PROD_DOMAIN" 2>/dev/null || echo "000")
  if [ "$PROD_HTTP_STATUS" = "200" ]; then
    echo "✅ Production live: $PROD_DOMAIN (HTTP 200)"
    PROD_VERIFIED="yes"
  else
    echo "⚠️ Production returned HTTP $PROD_HTTP_STATUS"
    echo "   (Deployment may still be propagating - check manually)"
    PROD_VERIFIED="warning"
  fi
else
  echo "ℹ️ Production domain not configured - skipping verification"
  PROD_VERIFIED="skipped"
fi
```

### 11.4 Check Database Migrations

```bash
# Verify database migrations workflow completed (if it ran)
echo "Checking database migrations status..."

MIGRATION_RUN=$(gh run list --workflow=deploy.yml --branch main --limit 1 --json status,conclusion --jq '.[0]' 2>/dev/null)

if [ -n "$MIGRATION_RUN" ]; then
  if echo "$MIGRATION_RUN" | grep -q '"conclusion":"success"'; then
    echo "✅ Database migrations applied"
  elif echo "$MIGRATION_RUN" | grep -q '"conclusion":"skipped"'; then
    echo "ℹ️ No new migrations to apply"
  elif echo "$MIGRATION_RUN" | grep -q '"conclusion":"failure"'; then
    echo "⚠️ Database migrations failed - check workflow logs"
    PROD_VERIFIED="migration-failed"
  fi
else
  echo "ℹ️ No migration workflow found"
fi
```

---

## STAGE 12: Cleanup & Final Verification

**MANDATORY:** Ensure ZERO stale work remains.

### 12.1 Switch Back to Main
```bash
git checkout main
git pull origin main
```

### 12.2 Delete Local Feature Branch
```bash
# Delete the feature branch locally
git branch -d "$BRANCH_NAME" 2>/dev/null || git branch -D "$BRANCH_NAME" 2>/dev/null || true
echo "🗑️ Deleted local branch: $BRANCH_NAME"
```

### 12.3 Verify Remote Branch Deleted
```bash
# Verify the remote branch was deleted (gh pr merge --delete-branch should have done this)
REMOTE_BRANCH_EXISTS=$(git ls-remote --heads origin "$BRANCH_NAME" 2>/dev/null)
if [ -n "$REMOTE_BRANCH_EXISTS" ]; then
  echo "⚠️ Remote branch still exists - deleting..."
  git push origin --delete "$BRANCH_NAME"
  echo "✅ Deleted remote branch: $BRANCH_NAME"
fi
```

### 12.4 Verify PR Closed
```bash
# Verify PR is merged and closed
PR_STATE=$(gh pr view "$PR_NUMBER" --json state --jq '.state')
if [ "$PR_STATE" != "MERGED" ]; then
  echo "⚠️ Warning: PR #$PR_NUMBER state is $PR_STATE (expected MERGED)"
fi
```

### 12.5 Clean Up ALL Stale Local Branches
```bash
# Remove any other merged branches
git fetch --prune
STALE_BRANCHES=$(git branch --merged main | grep -v "main\|master\|\*" | xargs)
if [ -n "$STALE_BRANCHES" ]; then
  echo "🧹 Cleaning up additional stale branches..."
  echo "$STALE_BRANCHES" | xargs git branch -d
  echo "✅ Cleaned up: $STALE_BRANCHES"
fi
```

### 12.6 Verify Clean Repository State
```bash
# Final verification - no uncommitted work, no untracked files (except excluded ones)
UNCOMMITTED=$(git status --porcelain 2>/dev/null)
if [ -n "$UNCOMMITTED" ]; then
  # Filter out .gitignored files
  UNCOMMITTED_REAL=$(echo "$UNCOMMITTED" | grep -v "^??" || true)
  if [ -n "$UNCOMMITTED_REAL" ]; then
    echo "⚠️ Warning: Uncommitted changes remain:"
    echo "$UNCOMMITTED_REAL"
  fi
fi

# Check for any remaining feature/fix branches
REMAINING_BRANCHES=$(git branch | grep -E "feature/|fix/|refactor/|experiment/" | wc -l)
if [ "$REMAINING_BRANCHES" -gt 0 ]; then
  echo "ℹ️ Note: $REMAINING_BRANCHES unshipped branches remain - run /gh-ship from those branches to ship them"
fi
```

### 12.7 Restore Stashed Changes
```bash
# Restore stashed changes if any
if [ "$ORIGINAL_STASH" = "yes" ]; then
  git stash pop
  echo "📦 Restored stashed changes"
fi
```

### 12.8 Calculate Final Metrics
```bash
# Get final commit SHA
FINAL_SHA=$(git rev-parse --short HEAD)

# Calculate total time
END_TIME=$(date +%s)
TOTAL_TIME=$((END_TIME - START_TIME))
MINUTES=$((TOTAL_TIME / 60))
SECONDS=$((TOTAL_TIME % 60))
```

---

## ROLLBACK PROCEDURE

If catastrophic failure at any stage:

```bash
echo "🔄 Rolling back to original state..."

# Restore original branch
git checkout "$ORIGINAL_BRANCH" 2>/dev/null || git checkout -

# Reset to original commit
git reset --hard "$ORIGINAL_COMMIT"

# Delete any created feature branch (both local and remote)
if [ -n "$BRANCH_NAME" ] && [ "$BRANCH_NAME" != "$ORIGINAL_BRANCH" ]; then
  git branch -D "$BRANCH_NAME" 2>/dev/null || true
  git push origin --delete "$BRANCH_NAME" 2>/dev/null || true
fi

# Close PR if created
if [ -n "$PR_NUMBER" ]; then
  gh pr close "$PR_NUMBER" --comment "Rolled back due to error" 2>/dev/null || true
fi

# Restore stash if we stashed anything
if [ "$ORIGINAL_STASH" = "yes" ]; then
  git stash pop
fi

echo "✅ Rolled back to: $ORIGINAL_COMMIT on $ORIGINAL_BRANCH"
```

---

## NETWORK RESILIENCE

For ANY network operation, use this retry pattern:

```bash
retry_with_backoff() {
  local max_attempts=3
  local delay=2
  local attempt=1

  while [ $attempt -le $max_attempts ]; do
    if "$@"; then
      return 0
    fi
    echo "Attempt $attempt failed, retrying in ${delay}s..."
    sleep $delay
    delay=$((delay * 2))
    attempt=$((attempt + 1))
  done

  return 1
}

# Usage: retry_with_backoff git push origin HEAD
```

---

## OUTPUT FORMAT - FINAL REPORT

**On Success:**

```
═══════════════════════════════════════════════════════════════
🎉 SHIPPED SUCCESSFULLY!
═══════════════════════════════════════════════════════════════

📋 Summary
   Commit:      def456g
   PR:          #42 (merged & closed)
   Branch:      main
   Total Time:  2m 34s

🔗 Deployments
   Preview:     https://rowan-abc123.vercel.app    ✅ HTTP 200
   Production:  https://rowanapp.com               ✅ HTTP 200

🔧 Auto-Fixes Applied
   • ESLint: 3 files fixed
   • Prettier: 1 file formatted
   • Secrets: 0 detected
   • Large files: 0 excluded

🔒 Security Scan
   • Secrets detection: ✅ passed
   • .env files: ✅ none staged
   • Credentials: ✅ none found

🧹 Cleanup Status
   • Feature branch: ✅ deleted (local & remote)
   • PR state: ✅ merged & closed
   • Stale branches: ✅ 2 cleaned up
   • Repository: ✅ clean (no uncommitted work)

📦 Package Manager: pnpm
🔍 Linter: biome

═══════════════════════════════════════════════════════════════
```

**On Partial Success (merged but deployment unverified):**

```
═══════════════════════════════════════════════════════════════
⚠️ SHIPPED WITH WARNINGS
═══════════════════════════════════════════════════════════════

📋 Summary
   Commit:      def456g
   PR:          #42 (merged & closed)
   Branch:      main
   Total Time:  3m 12s

🔗 Deployments
   Preview:     https://rowan-abc123.vercel.app    ✅ HTTP 200
   Production:  https://rowanapp.com               ⚠️ HTTP 503

⚠️ Warnings
   • Production deployment may still be in progress
   • Verify manually: https://rowanapp.com

🧹 Cleanup Status
   • Feature branch: ✅ deleted
   • PR state: ✅ merged & closed
   • Repository: ✅ clean

═══════════════════════════════════════════════════════════════
```

**On Failure:**

```
═══════════════════════════════════════════════════════════════
❌ SHIP FAILED - ROLLED BACK
═══════════════════════════════════════════════════════════════

📋 Summary
   Failed at:   Stage 9 (CI Fix)
   Reason:      TypeScript error unfixable after 3 attempts
   Time spent:  4m 18s

❌ Error Details
   src/api.ts:42 - Type 'string' is not assignable to type 'number'
   src/api.ts:67 - Property 'foo' does not exist on type 'Bar'

🔗 PR Status
   https://github.com/VetSecItPro/rowan-app/pull/42
   State: OPEN (needs manual review)

🔄 Rollback Complete
   Local state restored to: abc123f on main
   Feature branch: deleted (local & remote)
   PR: left open for manual review
   Stashed changes: restored
   Repository: clean

═══════════════════════════════════════════════════════════════
```

---

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### GH-Ship-Specific Cleanup

This skill already has comprehensive cleanup in Stage 12. No additional cleanup required.
Stage 12 covers: branch deletion, PR verification, stash restoration, stale branch pruning, and clean state verification.

---

## IMPORTANT REMINDERS

- This skill runs FULLY AUTONOMOUSLY
- Generate GOOD commit messages by reading the actual diff
- FIX everything that's fixable, don't just report
- NEVER ship broken code - if CI can't pass after 3 attempts, STOP
- ALWAYS rollback cleanly on catastrophic failure
- VERIFY both preview and production deployments
- **ALWAYS cleanup - delete branches, close PRs, verify clean state**
- **NEVER leave uncommitted work or stale branches**
- The user trusts you to ship their code safely, correctly, and COMPLETELY
- Check CLAUDE.md for directory exclusions before starting
- A professional workflow means ZERO artifacts left behind

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
