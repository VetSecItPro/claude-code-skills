# /sec-weekly-scan — Weekly Security Audit Across All Repos

**Scope:** Global (rowan, kaulby, clarus, styrby, steelmotion)
**Model:** Sonnet 4.5 (strategic analysis + auto-fix)
**Schedule:** Saturday 12:30 AM CST (automated via GitHub Actions)
**Runtime:** 1-2 hours (parallel processing)

---

## Purpose

Comprehensive weekly security audit across all SaaS products and company website:
- Automated vulnerability scanning (GitHub Actions runs Saturday 12:30 AM CST)
- Strategic auto-fixing with code impact analysis
- Parallel processing for efficiency
- PR creation with CI validation
- Auto-merge on green CI (NEVER push directly to main)
- Master audit report for Sunday morning review

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

Key phases for this skill:

```
🚀 Phase 1: PARALLEL DISCOVERY (2-3 min)
   ├─ Spawning 4 agents for parallel scanning
   ├─ Rowan: Scanning dependencies, code, licenses, types...
   ├─ Kaulby: Scanning dependencies, code, licenses, types...
   ├─ Clarus: Scanning dependencies, code, licenses, types...
   └─ Styrby: Scanning dependencies, code, licenses, types...
   ✅ Discovery complete - X issues found across 4 repos

📊 Phase 2: STRATEGIC ANALYSIS (2-3 min)
   ├─ Analyzing 23 findings for auto-fix feasibility
   ├─ Dependency impact analysis (reverse deps, breaking changes)
   ├─ Code vulnerability context analysis
   └─ ✅ 18 auto-fixable, 5 need manual review

🔧 Phase 3: PARALLEL AUTO-FIX (20-30 min)
   ├─ Agent 1 (Rowan): Fixing 6 issues...
   │  ├─ Fixed dependency CVE (lodash)
   │  ├─ Updated packages (validated build)
   │  └─ ✅ 5/6 fixed (1 needs review)
   ├─ Agent 2 (Kaulby): Fixing 3 issues...
   └─ [Similar for Clarus, Styrby]

📝 Phase 4: PR VALIDATION & MERGE (5 min per repo)
   ├─ Rowan: PR created → CI running → ✅ Merged
   ├─ Kaulby: PR created → CI running → ✅ Merged
   └─ [Results for each repo]

📄 Phase 5: MASTER AUDIT REPORT
   └─ Generated: ~/Downloads/security-audit-2026-02-08.md
```

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Per-repo scan agents return < 500 tokens each (full findings written to each repo's .security-reports/)
- State file tracks which repos are scanned, which have PRs created
- Resume from checkpoint if context resets — skip already-scanned repos
- Max 2 parallel repo scans (to limit context accumulation)
- Orchestrator messages stay lean: "Repo 2/4 Kaulby: 3 findings (1 critical), PR created"

---

## Usage

```bash
# Full scan + fix (typical Sunday morning workflow)
/sec-weekly-scan

# Fix only (use findings from Saturday's GitHub Actions scan)
/sec-weekly-scan --fix-only

# Single repo testing
/sec-weekly-scan --repo rowan
/sec-weekly-scan --repo kaulby
/sec-weekly-scan --repo clarus
/sec-weekly-scan --repo styrby
/sec-weekly-scan --repo steelmotion

# Include Steel Motion (first Saturday of month)
/sec-weekly-scan --include-steelmotion

# Dry run (report only, no fixes)
/sec-weekly-scan --dry-run

# Skip specific repos
/sec-weekly-scan --skip rowan,kaulby
```

---

## Architecture: Hybrid Approach

### Saturday 12:30 AM CST - GitHub Actions (Automated)
- Scans all repos in parallel
- Generates findings
- Creates GitHub Issues for critical items
- Stores artifacts

### Sunday Morning - Skill (Strategic Fixing)
- Reads findings from GitHub Issues or runs fresh scan
- Analyzes each issue strategically
- Auto-fixes safe items with validation
- Creates PRs, validates CI, auto-merges
- Generates master audit report

---

## Project Registry

```typescript
const PROJECTS = {
  rowan: {
    path: '/Users/airborneshellback/vibecode-projects/rowan-app',
    repo: 'VetSecItPro/rowan-app',
    schedule: 'weekly',
    priority: 'high'
  },
  kaulby: {
    path: '/Users/airborneshellback/vibecode-projects/kaulby-app',
    repo: 'VetSecItPro/kaulby-app',
    schedule: 'weekly',
    priority: 'high'
  },
  clarus: {
    path: '/Users/airborneshellback/vibecode-projects/clarus-app',
    repo: 'VetSecItPro/clarus-app',
    schedule: 'weekly',
    priority: 'high'
  },
  styrby: {
    path: '/Users/airborneshellback/vibecode-projects/styrby-app',
    repo: 'VetSecItPro/styrby-app',
    schedule: 'weekly',
    priority: 'high'
  },
  steelmotion: {
    path: '/Users/airborneshellback/vibecode-projects/steelmotion',
    repo: 'VetSecItPro/steelmotion',
    schedule: 'monthly', // First Saturday only
    priority: 'medium'
  }
};
```

---

## Execution Flow

### PHASE 1: PARALLEL DISCOVERY

**For each repo, spawn agent to:**
1. Clone/update repo
2. Install dependencies (`pnpm install`)
3. Run security scans in parallel:
   - `pnpm audit --json` (dependency vulnerabilities)
   - `semgrep scan` (code vulnerabilities - OWASP, Next.js, React, TypeScript)
   - `npx license-checker` (license compliance)
   - `pnpm type-check` (TypeScript errors)
   - `pnpm build --dry-run` (build validation)
4. Return findings with metadata

**Agent instruction:** "Return ONLY a structured summary under 500 tokens. Full scan results go to disk in the repo's .security-reports/ directory."

**Status updates:** Show progress for each repo as agents work

---

### PHASE 2: STRATEGIC ANALYSIS

**For EACH finding, analyze:**

```typescript
type Finding = {
  id: string;
  repo: string;
  severity: 'CRITICAL' | 'HIGH' | 'MODERATE' | 'LOW';
  type: 'dependency' | 'code' | 'license' | 'type-error' | 'build';
  file?: string;
  line?: number;
  description: string;
  autoFixable: boolean;
  breakingRisk: 'HIGH' | 'MEDIUM' | 'LOW';
  strategy: 'AUTO_FIX' | 'MANUAL_REVIEW' | 'DEFER';
  reasoning: string;
}

// Strategic decision making
async function analyzeFixability(finding: Finding): Promise<Strategy> {
  if (finding.type === 'dependency') {
    // Check reverse dependencies
    const dependents = await analyzeReverseDeps(finding.package);

    // Check changelog for breaking changes
    const changelog = await fetchChangelog(finding.package, finding.currentVersion, finding.targetVersion);
    const hasBreaking = detectBreakingChanges(changelog);

    // Check version bump type
    const versionBump = getVersionBumpType(finding.currentVersion, finding.targetVersion);

    if (versionBump === 'patch' && !hasBreaking) {
      return {
        strategy: 'AUTO_FIX',
        reasoning: 'Patch version, no breaking changes, low risk',
        breakingRisk: 'LOW'
      };
    }

    if (versionBump === 'minor' && dependents.length === 0) {
      return {
        strategy: 'AUTO_FIX',
        reasoning: 'Minor version, no reverse dependencies',
        breakingRisk: 'MEDIUM'
      };
    }

    return {
      strategy: 'MANUAL_REVIEW',
      reasoning: `Major version bump affects ${dependents.length} packages`,
      breakingRisk: 'HIGH'
    };
  }

  if (finding.type === 'code') {
    // Understand code context
    const context = await analyzeCodeContext(finding.file, finding.line);
    const affectedCode = await findRelatedCode(finding.file);

    // Check if fix is straightforward
    if (canFixSafely(context, finding.vulnerability)) {
      return {
        strategy: 'AUTO_FIX',
        reasoning: 'Straightforward fix, no side effects',
        breakingRisk: 'LOW'
      };
    }

    return {
      strategy: 'MANUAL_REVIEW',
      reasoning: 'Complex code flow, requires human judgment',
      breakingRisk: 'MEDIUM'
    };
  }

  // Continue for other types...
}
```

**Output:** Categorized findings (auto-fixable vs manual review)

---

### PHASE 3: PARALLEL AUTO-FIX

**Spawn fix agents (max 2 parallel, one per repo) to fix:**

**Agent instruction:** "Return ONLY a structured summary under 500 tokens. Full scan results go to disk in the repo's .security-reports/ directory."

```typescript
async function fixRepo(repo: string, findings: Finding[]) {
  const branch = `fix/security-scan-${new Date().toISOString().split('T')[0]}`;

  // Create branch
  await git.createBranch(branch);

  // Process each auto-fixable finding
  for (const finding of findings.filter(f => f.strategy === 'AUTO_FIX')) {
    try {
      // Apply fix
      await applyFix(finding);

      // Validate immediately
      const typeCheckPassed = await runTypeCheck();
      if (!typeCheckPassed) {
        await rollbackFix(finding);
        markAsManualReview(finding, 'Type check failed after fix');
        continue;
      }

      const buildPassed = await runBuild();
      if (!buildPassed) {
        await rollbackFix(finding);
        markAsManualReview(finding, 'Build failed after fix');
        continue;
      }

      // Fix successful
      console.log(`✅ Fixed: ${finding.description}`);

    } catch (error) {
      await rollbackFix(finding);
      markAsManualReview(finding, `Error during fix: ${error.message}`);
    }
  }

  // Commit all successful fixes
  await git.commit(`chore(security): weekly security fixes

Auto-fixed issues:
${findings.filter(f => f.fixed).map(f => `- ${f.description}`).join('\n')}

Issues requiring manual review:
${findings.filter(f => !f.fixed && f.strategy !== 'DEFER').map(f => `- ${f.description}`).join('\n')}

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
  `);

  // Push branch
  await git.push(branch);

  return findings;
}
```

**Critical Rules:**
1. **Never fix and push without validation** - Always run type-check + build after each fix
2. **Rollback on failure** - If validation fails, revert the specific fix
3. **Document everything** - Track what was fixed and what needs review
4. **Strategic thinking** - Understand code before changing it
5. **No breaking changes** - If unsure, mark for manual review

---

### PHASE 4: PR VALIDATION & MERGE

**For each repo (sequential to avoid rate limits):**

```typescript
async function createAndMergePR(repo: string, findings: Finding[]) {
  // Create PR
  const pr = await github.createPR({
    repo: repo.repo,
    title: `chore(security): weekly security fixes ${date}`,
    body: generatePRDescription(findings),
    head: branch,
    base: 'main',
    labels: ['security', 'automated']
  });

  console.log(`📝 ${repo.name}: PR #${pr.number} created`);
  console.log(`⏳ ${repo.name}: Waiting for CI checks...`);

  // Wait for CI (max 10 minutes)
  const ciStatus = await waitForCI(pr, {
    timeout: 10 * 60 * 1000,
    checkInterval: 10 * 1000
  });

  if (ciStatus === 'success') {
    // Auto-merge if CI passes
    await github.mergePR(pr);
    console.log(`✅ ${repo.name}: Merged to main`);
    return { status: 'merged', pr };
  } else {
    console.log(`❌ ${repo.name}: CI failed - PR left open for review`);
    return { status: 'ci-failed', pr };
  }
}
```

**Rules:**
- ✅ ALWAYS create PRs (NEVER push directly to main)
- ✅ ALWAYS wait for CI to complete
- ✅ Auto-merge ONLY if CI passes
- ⚠️ Leave PR open if CI fails (mark for manual review)

---

### PHASE 5: MASTER AUDIT REPORT

**Generate comprehensive report:**

```markdown
# Weekly Security Audit - [DATE]

**Execution Time:** [START] - [END] ([DURATION])
**Status:** X/4 repos clean, Y need review

---

## Executive Summary

| Repo | Issues Found | Auto-Fixed | Merged | Status |
|------|--------------|------------|--------|--------|
| Rowan | X | Y | ✅/⚠️ | ... |
| Kaulby | X | Y | ✅/⚠️ | ... |
| Clarus | X | Y | ✅/⚠️ | ... |
| Styrby | X | Y | ✅/⚠️ | ... |
| **Total** | **X** | **Y** | **Z/4** | **W pending** |

---

## Issues Requiring Manual Review

[Detailed list with reasoning, suggested fixes, PR links]

---

## Auto-Fixed Issues (Merged)

[Detailed list by repo with validation results]

---

## Detailed Findings by Repo

[Expandable sections for each repo]

---

## Next Steps

### Immediate (Monday Morning)
[Critical items to address]

### This Week
[Important but not urgent]

### Deferred (Low Priority)
[Can be addressed gradually]

---

## Pull Requests Created

[List with links and status]

---

**Next scan:** [DATE]
```

**Save to:** `~/Downloads/security-audit-YYYY-MM-DD.md`

---

## Auto-Fix Examples

### Example 1: Dependency CVE (Safe)

```typescript
// Finding: lodash 4.17.20 has prototype pollution vulnerability
// Target: 4.17.21 (patch version)

// Analysis:
const analysis = {
  currentVersion: '4.17.20',
  targetVersion: '4.17.21',
  versionBump: 'patch',
  breakingChanges: false,
  reverseDeps: [], // No packages depend on lodash in our code
  strategy: 'AUTO_FIX',
  reasoning: 'Patch version, no breaking changes, low risk'
};

// Execute:
await updatePackageJson('lodash', '4.17.21');
await runPnpmInstall();
const typeCheckPassed = await runTypeCheck(); // ✅
const buildPassed = await runBuild(); // ✅
// Success - keep fix
```

### Example 2: Major Version Bump (Manual Review)

```typescript
// Finding: React 18.2.0 → 19.0.0

// Analysis:
const analysis = {
  currentVersion: '18.2.0',
  targetVersion: '19.0.0',
  versionBump: 'major',
  breakingChanges: true,
  changelog: [
    'useEffect cleanup behavior changed',
    'New JSX transform required',
    'Automatic batching changes'
  ],
  affectedFiles: 15, // 15 components use useEffect
  strategy: 'MANUAL_REVIEW',
  reasoning: 'Major version with breaking changes in useEffect, affects 15 components'
};

// Do NOT auto-fix - mark for manual review
```

### Example 3: Code Vulnerability (Context-Dependent)

```typescript
// Finding: Open redirect in auth callback

// Analysis:
const codeContext = await analyzeCodeContext('app/auth-callback/page.tsx', 45);
// Result: Complex OAuth flow with multiple redirect scenarios

const analysis = {
  type: 'open-redirect',
  file: 'app/auth-callback/page.tsx',
  line: 45,
  code: 'window.location.href = params.redirect',
  complexity: 'HIGH',
  affectedRoutes: ['/auth-callback', '/oauth/callback'],
  strategy: 'MANUAL_REVIEW',
  reasoning: 'Complex OAuth flow requires understanding of redirect logic',
  suggestedFix: `
    const isInternal = redirect.startsWith('/') && !redirect.startsWith('//');
    window.location.href = isInternal ? redirect : '/dashboard';
  `
};

// Do NOT auto-fix - provide suggested fix in report
```

---

## Error Handling

### Build Failure After Fix

```typescript
// If build fails after applying a fix:
1. Immediately rollback that specific fix
2. Mark finding as MANUAL_REVIEW
3. Document: "Build failed after fix - requires manual intervention"
4. Continue with other fixes
5. Include in audit report with details
```

### CI Failure on PR

```typescript
// If CI fails on PR:
1. Do NOT merge PR
2. Leave PR open for manual review
3. Add comment to PR with CI failure details
4. Mark repo as "needs review" in audit report
5. Continue with other repos
```

### Network/API Failures

```typescript
// If GitHub API fails:
1. Retry up to 3 times with exponential backoff
2. If still fails, save findings locally
3. Document in audit report
4. Provide manual instructions to create PR
```

---

## Output Files

```
~/Downloads/
├── security-audit-2026-02-08.md          # Master audit report
├── rowan-findings-2026-02-08.json        # Detailed findings (machine-readable)
├── kaulby-findings-2026-02-08.json
├── clarus-findings-2026-02-08.json
└── styrby-findings-2026-02-08.json
```

---

## Integration with GitHub Actions

### GitHub Actions Workflow (Saturday 12:30 AM CST)

The skill can read findings from the automated GitHub Actions scan:

```bash
# If GitHub Actions ran earlier, use those findings:
/sec-weekly-scan --fix-only

# Skill will:
1. Fetch findings from GitHub Actions artifacts
2. Skip re-scanning (save time)
3. Jump straight to strategic analysis + fixing
```

### Or run complete scan locally:

```bash
# Full local scan + fix:
/sec-weekly-scan

# Skill will:
1. Run complete scan locally
2. Strategic analysis
3. Auto-fix
4. Create PRs
```

---

## Schedule Configuration

### Weekly (Rowan, Kaulby, Clarus, Styrby)
- **Time:** Saturday 12:30 AM CST
- **Frequency:** Every Saturday
- **Cron:** `30 6 * * 6` (UTC)

### Monthly (Steel Motion)
- **Time:** First Saturday 12:30 AM CST
- **Frequency:** Monthly
- **Cron:** `30 6 1-7 * 6` (First Saturday of month)

---

## Safety Guarantees

1. ✅ **Never push directly to main** - Always create PRs
2. ✅ **Never merge without CI** - Wait for GitHub Actions validation
3. ✅ **Never fix without validation** - Type-check + build after every fix
4. ✅ **Rollback on failure** - Revert fixes that break builds
5. ✅ **Document everything** - Comprehensive audit trail
6. ✅ **Strategic thinking** - Analyze code impact before changes
7. ✅ **Continue on errors** - One repo failure doesn't block others

---

## Implementation

When this skill is invoked, execute the following workflow:

1. **Parse arguments** - Determine which repos to scan, mode (full/fix-only/dry-run)
2. **PHASE 1: PARALLEL DISCOVERY** - Spawn agents for each repo (max 2 parallel)
3. **PHASE 2: STRATEGIC ANALYSIS** - Categorize findings
4. **PHASE 3: PARALLEL AUTO-FIX** - Fix safe items with validation
5. **PHASE 4: PR VALIDATION & MERGE** - Create PRs, wait for CI, auto-merge if green
6. **PHASE 5: MASTER AUDIT REPORT** - Generate comprehensive report

**CRITICAL:** Follow status update protocol throughout execution. Never go silent during long-running operations.

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
