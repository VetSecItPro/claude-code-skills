# /sec-ship — Full Security Pipeline: Audit → Fix → Validate → Report

You are a security automation specialist. Execute the COMPLETE security pipeline autonomously from start to finish. Discover vulnerabilities, fix them, validate fixes, and report results. Handle everything. Fix everything fixable. Definition of done: **NO vulnerabilities remain unfixed** (except those explicitly flagged for human review).

> **⚡ CONTEXT WARNING:** This skill is ~18K tokens. For best results, invoke `/sec-ship` at the start of a fresh conversation. If invoked mid-conversation, the orchestrator compensates by delegating ALL scanning to sub-agents (which start with clean context) and keeping its own footprint minimal — never read source files directly, only dispatch agents and collect their lean summaries.

<!--
═══════════════════════════════════════════════════════════════════════════════
DESIGN RATIONALE
═══════════════════════════════════════════════════════════════════════════════

## Purpose
- Find ALL security vulnerabilities automatically
- Fix everything that can be safely fixed
- Iterate until ZERO vulnerabilities remain
- Document everything in persistent .md file
- Mark tasks DONE (never delete)
- Write SITREP conclusion for historical perspective

## Coverage
- OWASP Top 10 (Web)
- OWASP API Security Top 10
- OWASP LLM Top 10 (AI Security)
- Mobile Security (OWASP MASVS)
- Supply Chain Security
- Configuration Security
- Business Logic Flaws

## Philosophy
- Comprehensive but not paranoid
- Secure but not broken
- Fix automatically when safe
- Flag for human when judgment needed
- Iterate until clean

═══════════════════════════════════════════════════════════════════════════════
-->

## CRITICAL RULES

1. **NEVER ask for permission** — fix automatically unless it requires human decision
2. **NEVER stop on fixable issues** — fix and continue
3. **ITERATE until clean** — re-scan after fixes, fix new findings, repeat until zero remain
4. **REPORT IS THE SINGLE SOURCE OF TRUTH** — written to disk after every status change
5. **UPDATE REPORT AS FIXES HAPPEN** — each finding's status changes in real-time in the markdown
6. **ALWAYS verify fixes** — don't assume, prove it works
7. **NEVER DELETE findings** — only update their status
8. **ATTEMPT FIX UP TO 3 TIMES** before marking BLOCKED — document each attempt
9. **SITREP ANNOTATES EVERYTHING** — what was fixed, what was deferred and WHY, what was blocked and WHY
10. **STATUS UPDATE every 5-10 minutes** — keep user informed
11. **PARALLEL when independent** — spawn agents for concurrent work

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

### Security Pipeline Status Flow

```markdown
🚀 Security Pipeline Started
   Mode: [full/audit/fix]
   Project: [name]

🔍 Phase 1: Vulnerability Discovery
   ├─ Scanning for OWASP Top 10 vulnerabilities
   ├─ Checking API security (OWASP API Top 10)
   ├─ Analyzing AI/LLM security (OWASP LLM Top 10)
   ├─ Inspecting configuration security
   ├─ Reviewing supply chain dependencies
   └─ ✅ Discovery complete ([X] findings)

📊 Findings Summary:
   • CRITICAL: [X] vulnerabilities
   • HIGH: [X] vulnerabilities
   • MEDIUM: [X] vulnerabilities
   • LOW: [X] vulnerabilities
   Total: [X] vulnerabilities

🔨 Phase 2: Automated Fixes
   ├─ [30s] Fixing SQL injection in /api/users
   ├─ [60s] Hardening authentication middleware
   ├─ [90s] Securing API rate limits
   └─ ✅ Fixes applied ([X/Y] successful)

🧪 Phase 3: Validation
   ├─ Re-scanning for regressions
   ├─ Running security tests
   ├─ Verifying build passes
   └─ ✅ Validation complete

📊 Fix Results:
   • ✅ FIXED: [X] vulnerabilities
   • 🚫 BLOCKED: [X] (require human review)
   • ⏸️ DEFERRED: [X] (architectural changes needed)
   • REMAINING: [X] vulnerabilities

📝 Phase 4: Report Generation
   ├─ Writing findings to .security-reports/
   ├─ Updating baseline.json
   └─ ✅ Report: .security-reports/sec-ship-2026-02-08.md

✅ Security Pipeline Complete
   Duration: [X] minutes
   Vulnerabilities Fixed: [X/Y]
   Security Score: [old]/10 → [new]/10
   Status: [COMPLETE/NEEDS_REVIEW]
```

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Sub-agents (scan agents, fix agents) return < 500 tokens each (full findings written to `.security-reports/`)
- State file `.security-reports/state-YYYYMMDD-HHMMSS.json` tracks which scan/fix phases are complete
- Resume from checkpoint if context resets — skip completed scans and fixes
- Fix agents run sequentially (they modify code); scan agents max 2 parallel
- Orchestrator messages stay lean: "Phase 2: 4/7 vulns fixed — 2 BLOCKED, 1 DEFERRED"

---

## AGENT ORCHESTRATION

This skill follows the **[Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)**.

The orchestrator coordinates agents but NEVER scans or fixes code directly. All heavy work is delegated to focused agents that write results to disk.

### Model Selection for This Skill

| Agent Type | Model | Why |
|-----------|-------|-----|
| File/route inventory | `haiku` | Listing API routes, counting files — no judgment |
| Dependency CVE scanner (Agent 1) | `sonnet` | Must assess vulnerability severity and applicability to this project |
| Secret detection (Agent 2) | `sonnet` | Must distinguish high-entropy strings from legitimate code |
| Code vulnerability scanner (Agents 3-6) | `sonnet` | Must understand if patterns are actually exploitable vs false positive |
| Config & headers scanner (Agent 7) | `sonnet` | Must assess security header completeness in context |
| AI/LLM security scanner (Agent 8) | `sonnet` | Must understand prompt injection vectors and AI output risks |
| Mobile security scanner (Agent 9) | `sonnet` | Must assess mobile-specific risks in context |
| Business logic scanner (Agent 10) | `sonnet` | Must understand workflow and state machine logic |
| Logic correctness scanner (Agent 11) | `opus` | Must trace multi-file logic paths and integration contracts — highest complexity |
| Cryptographic analyzer (Agent 12) | `sonnet` | Must identify weak crypto patterns and timing vulnerabilities |
| Infrastructure config scanner (Agent 13) | `sonnet` | Must assess CI/CD and deployment config risks |
| Data flow taint tracker (Agent 14) | `opus` | Must trace input across files/functions/DB — requires deep reasoning |
| Regression test generator (Agent 15) | `sonnet` | Must write correct, runnable test code |
| API drift detector (Agent 16) | `haiku` | Route inventory comparison — mostly mechanical |
| Build artifact scanner (Agent 17) | `sonnet` | Must distinguish real secrets from false positives in build output |
| Security fixer | `sonnet` | Must write correct security fixes that don't break functionality |
| Report synthesizer | `opus` | Security report may be shared with stakeholders — must be accurate and professional |

### Agent Batching

- Scanner agents process 15-25 files each (batch by directory: app/api/, lib/services/, components/)
- Fix agents handle up to 5 fixes each (one vulnerability category at a time)
- Fix agents run SEQUENTIALLY — never in parallel (they modify code)
- Scanner agents can run 2 at a time in parallel (they're read-only)

---

## Report Persistence (CRITICAL — Survives Compaction/Restart)

The markdown report file is the **living document**. It must be self-contained and updated continuously so that if the session is compacted, restarted, or resumed, the report captures all progress.

### Finding Statuses

| Status | Meaning |
|--------|---------|
| `FOUND` | Vulnerability discovered during scan, not yet addressed |
| `🔧 FIXING` | Currently being worked on |
| `✅ FIXED` | Fix applied, build verified, re-scan confirmed |
| `⏸️ DEFERRED` | Intentionally skipping — needs architectural change, business decision, or human review. Reason documented in SITREP. |
| `🚫 BLOCKED` | Attempted fix up to 3 times, all failed or broke the build. Each attempt documented in SITREP. |

### Rules

1. **Write report at Stage 0** — file exists before any scans, with header and empty sections
2. **Update after each agent completes** — findings added to task list as discovered
3. **Update after each fix** — status changes from FOUND → FIXING → FIXED/BLOCKED/DEFERRED
4. **Write to disk after every status change** — if session dies, report shows exactly where things stand
5. **Progress Log** — timestamped entries after each stage and each fix
6. **SITREP section** — for every DEFERRED or BLOCKED finding, document:
   - What was tried (all attempts)
   - Why it failed
   - What would be needed to fix it
   - Risk if left unfixed

### Resume Protocol

Before creating a new report, check for a recent incomplete one:

1. Find most recent `.security-reports/sec-ship-*.md`
2. If < 1 hour old AND Status is not `🟢 COMPLETE` → resume it
3. Read `## Progress Log` to find last completed step
4. Skip completed stages/agents, continue from next incomplete step
5. Do NOT re-scan completed agents or re-fix already-FIXED findings

---

## DEFINITION OF DONE

The audit is complete when:
- [ ] ALL CRITICAL findings are fixed (100%)
- [ ] ALL HIGH findings are fixed (100%)
- [ ] ALL MEDIUM findings are fixed (100%)
- [ ] ALL LOW findings are fixed OR explicitly accepted
- [ ] Re-scan shows ZERO new findings
- [ ] Build passes
- [ ] Tests pass
- [ ] All tasks marked ✅ DONE in MD file
- [ ] SITREP conclusion written

---

## OUTPUT STRUCTURE

```
.security-reports/
├── sec-ship-YYYYMMDD-HHMMSS.md    # Main report with task list & conclusion
├── baseline.json                   # Initial vulnerability count
├── history.json                    # All audits over time (append-only)
└── findings.json                   # Machine-readable current findings
```

---

## HUMAN DECISION TRIGGERS

Only pause for human input when:

| Situation | Action |
|-----------|--------|
| Fix would break core auth flow | Pause, explain, ask |
| Fix requires architectural change | Pause, explain options |
| Secret found in git history | Pause, recommend rotation |
| Business logic unclear | Flag in MD, continue |
| Fix fails after 3 alternative approaches | Flag in MD, continue |
| CSP would break critical features | Flag in MD, ask preference |

Everything else → fix autonomously.

---

## STAGE 0: INITIALIZATION

### 0.1 Pre-Flight Checks
```bash
mkdir -p .security-reports
TIMESTAMP=$(date +"%Y-%m-%d-%H%M%S")
AUDIT_FILE=".security-reports/sec-ship-${TIMESTAMP}.md"
```

### 0.2 Detect Project Context

**Detect frameworks, databases, package manager, and services:**
- Frameworks: Next.js, Vite, React Native, Expo, Capacitor
- Databases: Supabase, Prisma, Drizzle, MongoDB
- Services: Stripe, Resend, Auth0, Clerk, OpenAI, Anthropic, etc.
- Mobile: Capacitor, Expo, React Native

### 0.3 Check for Resumable Report

```bash
LATEST=$(ls -t .security-reports/sec-ship-*.md 2>/dev/null | head -1)
if [ -n "$LATEST" ]; then
  AGE=$(( $(date +%s) - $(stat -f %m "$LATEST") ))
  if [ "$AGE" -lt 3600 ] && ! grep -q "🟢 COMPLETE" "$LATEST"; then
    AUDIT_FILE="$LATEST"
    RESUMING=true
  fi
fi
```

If resuming: read the report, find last completed step in Progress Log, skip to next.

### 0.4 Initialize MD File (Skip if Resuming)

**Write this to disk IMMEDIATELY:**

```markdown
# Security Audit: [PROJECT_NAME]

**Created:** [YYYY-MM-DD HH:MM:SS]
**Status:** 🔴 IN PROGRESS — Stage 0: Initialization
**Iteration:** 1

## Metadata
- **Framework:** [detected]
- **Database:** [detected]
- **AI Services:** [detected]
- **Mobile:** [detected or N/A]

---

## Progress Log

| Time | Stage | Action | Result |
|------|-------|--------|--------|
| [HH:MM] | Stage 0 | Initialized | [framework] detected |

---

## Task List

| ID | Severity | Category | Finding | File:Line | Status |
|----|----------|----------|---------|-----------|--------|

---

## Fix Log

| ID | Finding | Action | Attempt | Build | Verified |
|----|---------|--------|---------|-------|----------|

---

## SITREP

_To be populated as fixes are applied_

### What Was Fixed
_Updated in real-time_

### What Was Deferred (and Why)
_Each deferred item with full explanation_

### What Was Blocked (and Why)
_Each blocked item with all attempts documented_
```

**IMPORTANT:** This file is written to disk NOW. Every subsequent stage updates this same file.

---

## STAGE 1: DISCOVERY & ATTACK SURFACE MAPPING

Map all entry points:
- API routes (REST, GraphQL, tRPC, Server Actions)
- Forms and user inputs
- File uploads
- WebSocket connections
- AI/LLM integrations
- OAuth callbacks
- Webhooks
- Deep links (mobile)

---

## STAGE 2: PARALLEL SECURITY SCANNING (17 Agents)

### Agent 1: Dependencies & Supply Chain

**Checks:**
| Check | What to Find | Finding ID |
|-------|--------------|------------|
| Known CVEs | `npm audit` vulnerabilities | SEC-DEP-001 |
| Outdated packages | Packages with security patches available | SEC-DEP-002 |
| Typosquatting | Similar names to popular packages | SEC-DEP-003 |
| License issues | GPL in proprietary code | SEC-DEP-004 |
| Unused dependencies | Attack surface reduction | SEC-DEP-005 |
| Lockfile integrity | Unexpected changes | SEC-DEP-006 |

**Auto-Fix:**
- `$PM audit fix` for non-breaking updates
- Update minor/patch versions with CVEs
- Remove unused dependencies

---

### Agent 2: Dynamic Secret Detection

**Entropy-Based Detection** (not just pattern matching):
- High-entropy strings > 20 chars
- Connection strings
- Private keys
- JWTs in code

**Service-Specific Patterns:**
```regex
sk_live_*, sk_test_*           # Stripe
AKIA[0-9A-Z]{16}               # AWS
ghp_*, github_pat_*            # GitHub
xox[baprs]-*                   # Slack
SG\.*                          # SendGrid
eyJ[a-zA-Z0-9_-]*\.eyJ*        # JWT
-----BEGIN .* PRIVATE KEY-----  # Private keys
postgres://, mongodb://         # Connection strings
```

**Git History Scan:**
```bash
git log -p --all -S "SECRET\|KEY\|TOKEN" --since="2020-01-01"
```

**Auto-Fix:**
- Add `.env*` to `.gitignore`
- Replace hardcoded secrets with `process.env.VAR_NAME`
- Create `.env.example` with placeholders
- Flag git history secrets for rotation (human action)

---

### Agent 3: Server-Side Security (OWASP Top 10)

**Injection Attacks:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **SQL Injection** | String concatenation in queries | SEC-INJ-001 |
| **NoSQL Injection** | `$where`, `$regex` with user input | SEC-INJ-002 |
| **Command Injection** | `exec()`, `spawn()`, `system()` with user input | SEC-INJ-003 |
| **LDAP Injection** | User input in LDAP queries | SEC-INJ-004 |
| **XPath Injection** | User input in XPath | SEC-INJ-005 |
| **Template Injection** | User input in template strings | SEC-INJ-006 |
| **Header Injection** | User input in HTTP headers | SEC-INJ-007 |
| **Log Injection** | User input in logs (CRLF) | SEC-INJ-008 |

**Path & File Attacks:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Path Traversal** | `../` in file paths | SEC-PATH-001 |
| **Arbitrary File Read** | `fs.readFile` with user input | SEC-PATH-002 |
| **Arbitrary File Write** | `fs.writeFile` with user input | SEC-PATH-003 |
| **Zip Slip** | Archive extraction without validation | SEC-PATH-004 |

**Server-Side Request Forgery (SSRF):**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Basic SSRF** | User-controlled URLs in fetch/axios | SEC-SSRF-001 |
| **DNS Rebinding** | No DNS pinning | SEC-SSRF-002 |
| **Cloud Metadata** | Access to 169.254.169.254 | SEC-SSRF-003 |
| **Internal Services** | Access to localhost, 10.*, 192.168.* | SEC-SSRF-004 |

**Deserialization:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Unsafe JSON** | `JSON.parse` on untrusted data without validation | SEC-DESER-001 |
| **Prototype Pollution** | Object.assign/spread with user input | SEC-DESER-002 |
| **YAML Parsing** | `yaml.load` without safe mode | SEC-DESER-003 |

**Auto-Fix:**
- Parameterized queries
- Input validation with Zod
- URL allowlisting for SSRF
- Safe deserialization patterns

---

### Agent 4: Authentication & Authorization

**Authentication Flaws:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Missing Auth** | API routes without auth middleware | SEC-AUTH-001 |
| **Broken Auth** | Auth bypass possible | SEC-AUTH-002 |
| **Weak Password Policy** | No minimum requirements | SEC-AUTH-003 |
| **No Brute Force Protection** | Missing rate limiting on auth | SEC-AUTH-004 |
| **Session Fixation** | Session not rotated after login | SEC-AUTH-005 |
| **Insecure Session** | Missing Secure/HttpOnly/SameSite | SEC-AUTH-006 |
| **JWT Vulnerabilities** | None/weak algorithm, no expiration | SEC-AUTH-007 |
| **MFA Bypass** | MFA can be skipped | SEC-AUTH-008 |

**Authorization Flaws:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **IDOR** | Accessing others' resources | SEC-AUTHZ-001 |
| **Missing Ownership Check** | No user_id verification | SEC-AUTHZ-002 |
| **Privilege Escalation** | User can become admin | SEC-AUTHZ-003 |
| **BOLA** | Broken Object Level Authorization | SEC-AUTHZ-004 |
| **BFLA** | Broken Function Level Authorization | SEC-AUTHZ-005 |
| **Mass Assignment** | User setting is_admin, role, tier | SEC-AUTHZ-006 |

**Database Security (Supabase):**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **RLS Disabled** | Tables without RLS | SEC-RLS-001 |
| **Missing Policies** | RLS enabled but no policies | SEC-RLS-002 |
| **Weak Policies** | `WITH CHECK(true)` | SEC-RLS-003 |
| **Service Role Exposure** | Service key in client code | SEC-RLS-004 |
| **Function Security** | Missing search_path | SEC-RLS-005 |

**Auto-Fix:**
- Add auth middleware
- Add ownership checks
- Enable RLS and add policies
- Add rate limiting
- Fix session configuration

---

### Agent 5: Client-Side Security

**XSS (Cross-Site Scripting):**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **DOM XSS** | dangerouslySetInnerHTML without sanitization | SEC-XSS-001 |
| **innerHTML** | Direct innerHTML assignment | SEC-XSS-002 |
| **eval()** | Any usage | SEC-XSS-003 |
| **Function()** | Dynamic function creation | SEC-XSS-004 |
| **document.write()** | Any usage | SEC-XSS-005 |
| **URL-based XSS** | location.hash/search in DOM | SEC-XSS-006 |
| **jQuery .html()** | With user input | SEC-XSS-007 |

**Other Client Attacks:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Open Redirect** | User-controlled redirect URLs | SEC-CLIENT-001 |
| **Clickjacking** | Missing X-Frame-Options | SEC-CLIENT-002 |
| **postMessage** | Missing origin validation | SEC-CLIENT-003 |
| **DOM Clobbering** | Named elements overwriting globals | SEC-CLIENT-004 |
| **Prototype Pollution** | Object merge without protection | SEC-CLIENT-005 |

**Storage Security:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **localStorage Secrets** | Tokens/PII in localStorage | SEC-STORAGE-001 |
| **Insecure Cookies** | Missing Secure/HttpOnly flags | SEC-STORAGE-002 |

**Auto-Fix:**
- Add DOMPurify
- Add origin checks to postMessage
- Move sensitive data from localStorage
- Fix cookie flags

---

### Agent 6: API Security

**API Vulnerabilities:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **No Rate Limiting** | Missing on all routes | SEC-API-001 |
| **No Input Validation** | Missing Zod/Yup schemas | SEC-API-002 |
| **Excessive Data Exposure** | Returning entire DB objects | SEC-API-003 |
| **No Pagination** | Unbounded queries | SEC-API-004 |
| **Improper Error Handling** | Stack traces in responses | SEC-API-005 |
| **CORS Misconfiguration** | `*` in production | SEC-API-006 |
| **HTTP Method Restriction** | Allowing unneeded methods | SEC-API-007 |

**GraphQL-Specific:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Introspection Enabled** | In production | SEC-GQL-001 |
| **No Query Depth Limit** | Nested query attacks | SEC-GQL-002 |
| **No Query Complexity Limit** | Resource exhaustion | SEC-GQL-003 |
| **Batching Attacks** | No batch limit | SEC-GQL-004 |

**WebSocket Security:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **No Auth on Connect** | Missing authentication | SEC-WS-001 |
| **No Message Validation** | Unvalidated messages | SEC-WS-002 |
| **No Rate Limiting** | Message flooding | SEC-WS-003 |
| **CSWSH** | Cross-Site WebSocket Hijacking | SEC-WS-004 |

**Auto-Fix:**
- Add rate limiting (Upstash)
- Add Zod validation
- Filter response fields
- Add pagination
- Wrap errors properly
- Fix CORS configuration

---

### Agent 7: Configuration & Headers

**Security Headers:**
| Header | Required Value | Finding ID |
|--------|----------------|------------|
| **Content-Security-Policy** | Strict, no unsafe-eval | SEC-HDR-001 |
| **Strict-Transport-Security** | max-age=31536000; includeSubDomains | SEC-HDR-002 |
| **X-Frame-Options** | DENY or SAMEORIGIN | SEC-HDR-003 |
| **X-Content-Type-Options** | nosniff | SEC-HDR-004 |
| **Referrer-Policy** | strict-origin-when-cross-origin | SEC-HDR-005 |
| **Permissions-Policy** | Restrictive | SEC-HDR-006 |
| **X-XSS-Protection** | 0 (rely on CSP instead) | SEC-HDR-007 |

**Environment & Config:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Debug Mode** | DEBUG=true in production | SEC-CFG-001 |
| **Source Maps** | Exposed in production | SEC-CFG-002 |
| **Console.logs** | In server-side code | SEC-CFG-003 |
| **NEXT_PUBLIC_ Secrets** | Sensitive vars exposed | SEC-CFG-004 |
| **Verbose Errors** | Stack traces in production | SEC-CFG-005 |
| **Default Credentials** | Unchanged defaults | SEC-CFG-006 |

**Auto-Fix:**
- Add security headers to next.config.js
- Remove console.logs from server code
- Add poweredByHeader: false
- Fix environment variable scoping

---

### Agent 8: AI/LLM Security (OWASP LLM Top 10)

**This agent is CRITICAL for apps using OpenAI, Anthropic, or any AI services.**

**LLM01: Prompt Injection:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Direct Prompt Injection** | User input directly in prompt without sanitization | SEC-LLM-001 |
| **System Prompt Leakage** | User can extract system prompt | SEC-LLM-002 |
| **Instruction Override** | User can override system instructions | SEC-LLM-003 |
| **Jailbreak Patterns** | Known jailbreak techniques not blocked | SEC-LLM-004 |

**Detection patterns:**
```javascript
// VULNERABLE: Direct concatenation
const response = await openai.chat.completions.create({
  messages: [
    { role: 'system', content: systemPrompt },
    { role: 'user', content: userInput }  // Direct user input!
  ]
})

// VULNERABLE: Template literal injection
const prompt = `Help the user with: ${userInput}`
```

**LLM02: Insecure Output Handling:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Unescaped AI Output** | AI response rendered as HTML | SEC-LLM-005 |
| **Code Execution** | AI output executed as code | SEC-LLM-006 |
| **SQL from AI** | AI-generated SQL executed directly | SEC-LLM-007 |
| **Command from AI** | AI output used in shell commands | SEC-LLM-008 |

**LLM03: Training Data Poisoning:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **User Data in Fine-tuning** | Unvalidated user data in training | SEC-LLM-009 |
| **RAG Poisoning** | Unvalidated documents in RAG | SEC-LLM-010 |

**LLM04: Model Denial of Service:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **No Token Limits** | No max_tokens parameter | SEC-LLM-011 |
| **No Request Limits** | Unlimited AI API calls | SEC-LLM-012 |
| **Large Context Attacks** | No input length validation | SEC-LLM-013 |
| **Recursive Prompts** | AI calling AI infinitely | SEC-LLM-014 |

**LLM05: Supply Chain Vulnerabilities:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Unverified Models** | Third-party models without verification | SEC-LLM-015 |
| **Malicious Plugins** | LangChain/AutoGPT plugins from untrusted sources | SEC-LLM-016 |

**LLM06: Sensitive Information Disclosure:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **PII in Prompts** | User PII sent to AI without filtering | SEC-LLM-017 |
| **Secrets in Context** | API keys/passwords in AI context | SEC-LLM-018 |
| **Training Data Extraction** | Model memorization attacks | SEC-LLM-019 |

**LLM07: Insecure Plugin Design:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **No Input Validation** | AI tool inputs not validated | SEC-LLM-020 |
| **Excessive Permissions** | Tools with unnecessary access | SEC-LLM-021 |
| **No Output Filtering** | Tool outputs not sanitized | SEC-LLM-022 |

**LLM08: Excessive Agency:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Auto-Execute Actions** | AI actions without user confirmation | SEC-LLM-023 |
| **Privileged Operations** | AI can delete/modify data directly | SEC-LLM-024 |
| **No Human in Loop** | Critical decisions without approval | SEC-LLM-025 |

**LLM09: Overreliance:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **No Fact Checking** | AI output used without verification | SEC-LLM-026 |
| **Hallucination Risk** | AI decisions affecting real data | SEC-LLM-027 |

**LLM10: Model Theft:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Exposed Model Endpoints** | No auth on model APIs | SEC-LLM-028 |
| **No Rate Limiting** | Model extraction via many queries | SEC-LLM-029 |

**Auto-Fix Templates:**

```typescript
// FIX: Prompt Injection Protection
import { sanitizeInput, validatePrompt } from '@/lib/ai-security'

// Security: Sanitize user input before including in prompt
const sanitizedInput = sanitizeInput(userInput, {
  maxLength: 2000,
  stripInstructions: true,  // Remove "ignore previous", "system:", etc.
  stripCodeBlocks: false,
  allowedCharsets: ['latin', 'common-punctuation']
})

// Security: Validate prompt structure
if (!validatePrompt(sanitizedInput)) {
  throw new Error('Invalid input detected')
}

const response = await openai.chat.completions.create({
  model: 'gpt-4',
  max_tokens: 1000,  // Security: Limit output tokens
  messages: [
    { role: 'system', content: systemPrompt },
    { role: 'user', content: sanitizedInput }
  ]
})

// Security: Sanitize AI output before rendering
const safeOutput = DOMPurify.sanitize(response.choices[0].message.content)
```

```typescript
// FIX: AI Rate Limiting & Cost Control
import { Ratelimit } from '@upstash/ratelimit'

// Security: Rate limit AI requests per user
const aiRatelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(20, '1 h'),  // 20 AI calls per hour
  prefix: 'ai-ratelimit'
})

// Security: Estimate token cost before request
const estimatedTokens = estimateTokens(sanitizedInput)
if (estimatedTokens > MAX_INPUT_TOKENS) {
  throw new Error('Input too long')
}

const { success, remaining } = await aiRatelimit.limit(userId)
if (!success) {
  return Response.json({ error: 'AI rate limit exceeded' }, { status: 429 })
}
```

```typescript
// FIX: AI Output Validation
// Security: Never execute AI-generated code directly
const aiResponse = response.choices[0].message.content

// Security: Validate AI output structure
const validated = aiOutputSchema.safeParse(JSON.parse(aiResponse))
if (!validated.success) {
  throw new Error('Invalid AI response structure')
}

// Security: Sanitize before database operations
const sanitizedData = sanitizeForDB(validated.data)
```

---

### Agent 9: Mobile Security (if detected)

**Mobile-Specific Checks:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Insecure Storage** | AsyncStorage for sensitive data | SEC-MOB-001 |
| **No Certificate Pinning** | Missing SSL pinning | SEC-MOB-002 |
| **Deep Link Injection** | Unvalidated deep link params | SEC-MOB-003 |
| **Biometric Bypass** | Weak biometric implementation | SEC-MOB-004 |
| **Screenshot Exposure** | No screenshot prevention | SEC-MOB-005 |
| **No Root Detection** | Missing jailbreak/root check | SEC-MOB-006 |
| **Clipboard Leakage** | Sensitive data in clipboard | SEC-MOB-007 |
| **Backup Exposure** | Sensitive data in backups | SEC-MOB-008 |

---

### Agent 10: Business Logic & Advanced

**Business Logic Flaws:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **Race Conditions** | Double-spend, double-submit | SEC-BIZ-001 |
| **Price Manipulation** | Client-side price calculations | SEC-BIZ-002 |
| **Workflow Bypass** | Skipping required steps | SEC-BIZ-003 |
| **Quota Bypass** | Circumventing limits | SEC-BIZ-004 |
| **Negative Values** | Negative quantities/amounts | SEC-BIZ-005 |
| **Time-Based Bypass** | Manipulating timestamps | SEC-BIZ-006 |

**Advanced Attacks:**
| Check | Pattern | Finding ID |
|-------|---------|------------|
| **ReDoS** | Catastrophic regex backtracking | SEC-ADV-001 |
| **HTTP Parameter Pollution** | Duplicate params | SEC-ADV-002 |
| **HTTP Request Smuggling** | Transfer-Encoding issues | SEC-ADV-003 |
| **Timing Attacks** | Non-constant-time comparison | SEC-ADV-004 |
| **Cache Poisoning** | Unkeyed inputs | SEC-ADV-005 |
| **Unicode Attacks** | Homoglyph confusion | SEC-ADV-006 |

---

### Agent 11: Logic Correctness, Integration Contracts & Admin Verification

**This agent checks whether code is CORRECT — not whether it's exploitable.**
- Agent 10 asks: "Can an attacker bypass payment?" (exploitability)
- Agent 11 asks: "Does the payment webhook correctly update the user's tier?" (correctness)

**Discovery-driven:** This agent first inventories the codebase (payment provider, AI service, email service, DB, scraping/ingestion, search, admin routes) using Stage 0 context, then applies the relevant checks below. Skip any category whose service is not detected.

**Key File Patterns to Discover:**
```
**/api/**/webhook*              # Payment/service webhooks
**/api/**/checkout*             # Checkout flows
lib/*tier*|lib/*plan*|lib/*sub* # Tier/plan resolution
lib/*usage*|lib/*quota*|lib/*limit*  # Usage tracking
lib/*process*|lib/*pipeline*    # Content/data pipelines
lib/*email*|lib/*mail*|lib/*send*   # Email integration
lib/*transcri*|lib/*audio*      # Transcription services
lib/*rss*|lib/*feed*|lib/*podcast*  # Feed/podcast ingestion
**/manage/**|**/admin/**        # Admin dashboard
**/api/admin/**                 # Admin API routes
lib/*auth*|middleware*           # Auth + route guards
**/api/cron*/**                 # Cron jobs
**/api/account/**               # Account management
```

#### 11A: Internal Logic Correctness

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Silent Tier Downgrade** | Tier resolution function falls back to lowest tier on lookup failure — user silently loses paid features without error | SEC-LOGIC-001 |
| **Time-Limited Pass Expiry Race** | Time-limited access (day pass, trial) expires during a long-running operation — starts as paid, finishes as free | SEC-LOGIC-002 |
| **Usage Counter Atomicity** | Check-then-increment patterns for usage/quota tracking (should be atomic SQL, transaction, or RPC) | SEC-LOGIC-003 |
| **Period Rollover Timezone** | Usage period calculation edge cases — midnight UTC vs user timezone causes double-count or missed reset | SEC-LOGIC-004 |
| **Subscription Status Mapping** | Payment statuses like `past_due`/`unpaid` mapped to `canceled` with no grace period — immediate downgrade | SEC-LOGIC-005 |
| **Cancellation End Date** | Subscription cancellation sets end date to `now()` instead of `currentPeriodEnd` — user loses remaining paid days | SEC-LOGIC-006 |
| **Immediate Tier Strip** | Cancellation removes paid tier immediately instead of honoring remaining billing period | SEC-LOGIC-007 |
| **Overlapping Entitlement Edge** | Multiple entitlements cleared on new subscription — if new subscription fails, user has no entitlement at all | SEC-LOGIC-008 |
| **Cache Cross-Contamination** | In-memory or client-side cache leaking one user's data (chat, ratings, preferences) to another user | SEC-LOGIC-009 |
| **Privileged Client Ownership** | Routes using admin/service DB client (RLS bypass) without verifying resource ownership before returning data | SEC-LOGIC-010 |
| **Error Classification** | Raw error messages (stack traces, DB errors, third-party errors) leaking to end users instead of friendly messages | SEC-LOGIC-011 |
| **Server-Side Limit Enforcement** | Feature limits (tags, bookmarks, items, collections) enforced only client-side — no server-side check on create/update | SEC-LOGIC-012 |

#### 11B: Database Integration Contracts (Supabase/Prisma/Drizzle)

**Apply checks matching detected DB client:**

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Silent Null Columns** | ORM/client returns `null` for non-existent or renamed columns in select — no error, just missing data | SEC-INTEG-001 |
| **RPC/Procedure Return Assumptions** | DB function results cast without validation — renamed column or changed return type silently zeros values | SEC-INTEG-002 |
| **Schema Drift** | Queries referencing wrong schema, stale table names, or columns that were renamed in a migration | SEC-INTEG-003 |
| **Upsert Conflict Columns** | Upsert with wrong conflict/unique columns → silent duplicates instead of updates | SEC-INTEG-004 |
| **Single Row Error Handling** | `.single()`/`.findUnique()` error — "no rows" vs "multiple rows" not distinguished, treated the same | SEC-INTEG-005 |
| **Nullable Column Access** | Nullable column results accessed without null checks → runtime `Cannot read property of null` | SEC-INTEG-006 |
| **Admin/Service Client in User Routes** | Privileged DB client (RLS bypass, service role) used in user-facing routes without ownership verification | SEC-INTEG-007 |

#### 11C: Payment Provider Contracts (Polar/Stripe/Lemon Squeezy/Paddle)

**Apply checks matching detected payment provider:**

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Webhook Idempotency** | No idempotency guard — duplicate webhook events re-run full handler (double tier change, double email) | SEC-INTEG-008 |
| **Empty/Missing Product Config** | Empty string or missing product/price ID env vars → tier resolver returns undefined → silent downgrade | SEC-INTEG-009 |
| **Checkout Metadata Trust** | User ID or metadata from checkout session — validated server-side or trusted from client payload? | SEC-INTEG-010 |
| **Customer Linking Race** | Customer ID linking race on simultaneous webhooks (checkout + subscription created at same time) | SEC-INTEG-011 |
| **Webhook Retry Behavior** | Returning 4xx when user not found — payment provider retries 4xx, creating retry storm noise (should return 200) | SEC-INTEG-012 |
| **Side-Effect Blocking Core Update** | Non-critical side effect (email, analytics, logging) failure blocks the critical update (tier change, access grant) | SEC-INTEG-013 |

#### 11D: AI/LLM Provider Contracts (OpenAI/Anthropic/OpenRouter/Gemini)

**Apply checks matching detected AI provider:**

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **No Circuit Breaker** | Every AI request waits for full timeout — no circuit breaker to fail fast during provider outage | SEC-INTEG-014 |
| **Retry Thundering Herd** | Retry without exponential backoff or jitter → all retries hit at same time during recovery | SEC-INTEG-015 |
| **No Token Pre-Validation** | No input size/token estimation before request → oversized inputs waste credits and hit 413/400 errors | SEC-INTEG-016 |
| **Invalid Model Fallback** | Model name configured externally (DB, env) — if invalid, does it crash or fall back gracefully? | SEC-INTEG-017 |
| **Response Parsing Resilience** | AI response parsing — handles partial JSON, markdown-wrapped JSON, empty responses, refusals? | SEC-INTEG-018 |
| **Prompt/Config Fetch Failure** | Prompt or AI config fetched from DB/remote — does pipeline crash or fall back to hardcoded default? | SEC-INTEG-019 |

#### 11E: Content Ingestion & External Service Contracts

**Apply checks matching detected scraping/transcription/ingestion services:**

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Timeout Mismatch** | External service timeout vs hosting platform function timeout — external finishes after host kills the function | SEC-INTEG-020 |
| **Duplicate Callback** | Duplicate webhook/callback from external service → duplicate processing, duplicate DB entries, double usage count | SEC-INTEG-021 |
| **Content Size Limit** | No content size limit before sending to AI — oversized content burns credits or hits API limits | SEC-INTEG-022 |
| **Backoff Strategy** | External service retry using linear or no backoff (should be exponential with jitter) — hammers API during outage | SEC-INTEG-023 |
| **Cascading Timeout** | Service A calls Service B — timeout of A shorter than B, or no coordination between them | SEC-INTEG-024 |
| **Failure Counter Logic** | Consecutive failure counter — does success reset it? Or does it accumulate forever / reset at wrong time? | SEC-INTEG-025 |
| **Permanent Disable on Transient Failure** | Auto-deactivation of recurring jobs (feeds, subscriptions) after transient outages — permanent disable from temporary issue | SEC-INTEG-026 |

#### 11F: Email & Search Service Contracts

**Apply checks matching detected email/search services:**

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Fire-and-Forget Correctness** | Email sent fire-and-forget — is this intentional, or does a downstream code path silently depend on delivery success? | SEC-INTEG-027 |
| **Missing Client Initialization** | Email/search client factory throws if API key missing — do ALL callers handle this? Unhandled → 500 on every request | SEC-INTEG-028 |
| **Sender Address Consistency** | Hardcoded `from`/sender address — consistent across all send calls, or some use different/invalid domains? | SEC-INTEG-029 |
| **Cache Cold Start Assumption** | Search/enrichment cache lost on serverless cold start — code assumes warm cache? First request returns different results? | SEC-INTEG-030 |
| **No Quota Tracking** | No API quota/usage tracking for external services — silent failure when quota exhausted mid-period | SEC-INTEG-031 |
| **Graceful Degradation** | External enrichment service unreachable — does the pipeline degrade gracefully (skip enrichment) or crash entirely? | SEC-INTEG-032 |

#### 11G: Cross-System Pipeline Coherence

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Feature Flag vs Limit Mismatch** | Boolean feature flags vs numeric limit caps — every client-side UI gate has a corresponding server-side enforcement check | SEC-LOGIC-013 |
| **Content Type Lifecycle Parity** | All content/entity types have complete lifecycle: create + process + cache + export + delete (no orphaned type missing a step) | SEC-LOGIC-014 |
| **Cron/Scheduled Job Auth** | Cron/scheduled routes validate a secret header or token — none are publicly accessible without authentication | SEC-LOGIC-015 |
| **Pipeline Coverage Gaps** | Processing pipeline (moderation, AI analysis, indexing) covers ALL content types, not just the first one built | SEC-LOGIC-016 |
| **Public Link Data Leakage** | Public/share links don't leak private data (chat history, user ratings, PII) to anonymous viewers | SEC-LOGIC-017 |
| **Account Deletion Cascade** | Account deletion cascades to ALL user data across ALL user-related tables — no orphaned rows left behind | SEC-LOGIC-018 |
| **Export-Deletion Parity** | Account data export scope matches deletion scope — everything that would be deleted is also included in export | SEC-LOGIC-019 |
| **Orphaned Child Records** | Child/junction table rows orphaned when parent record is deleted — missing cascade, trigger, or cleanup job | SEC-LOGIC-020 |

#### 11H: Admin Dashboard Verification (if admin routes detected)

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Multi-Layer Auth** | Every admin page and API route has layered auth: middleware + server-side check + role verification (not just one layer) | SEC-ADMIN-001 |
| **PII Over-Exposure** | Admin APIs return more PII than necessary for the admin task (full email, name, IP in list views) | SEC-ADMIN-002 |
| **No Audit Logging** | Admin actions (user ban, content moderation, config changes) not logged with admin ID, timestamp, and action details | SEC-ADMIN-003 |
| **Inconsistent Rate Limiting** | Rate limiting on some admin routes but missing on others — all admin APIs should be rate-limited | SEC-ADMIN-004 |
| **Cache Staleness Visibility** | Cached dashboard data with no "last updated" indicator — admin sees stale metrics without knowing they're stale | SEC-ADMIN-005 |
| **Hardcoded/Placeholder Metrics** | Dashboard metrics hardcoded to `0` or placeholder values — never wired to real data source | SEC-ADMIN-006 |
| **Return Shape Drift** | DB function/RPC return shape not validated — renamed column silently drops metric to zero with no error | SEC-ADMIN-007 |
| **Missing Error Boundaries** | Admin pages missing error boundaries — single API failure causes white screen instead of graceful degradation | SEC-ADMIN-008 |
| **Client-Side Direct DB Access** | Admin client components query DB directly instead of through admin API routes — bypasses server-side auth/logging | SEC-ADMIN-009 |
| **Legal/Compliance Hold** | Flagged/reported content deletion must be soft-delete only — hard delete violates legal hold requirements | SEC-ADMIN-010 |

**Auto-Fix Templates:**

```typescript
// FIX: Atomic Usage Increment (SEC-LOGIC-003)
// WRONG: check-then-increment (race condition)
const { count } = await db.from('usage').select('count').single()
if (count < limit) {
  await db.from('usage').update({ count: count + 1 })
}

// RIGHT: Atomic increment with limit check in SQL/RPC
const { error } = await db.rpc('increment_usage', {
  p_user_id: userId,
  p_type: usageType,
  p_limit: tierLimit
})
// RPC returns error if limit exceeded — no race window
```

```typescript
// FIX: Webhook Idempotency (SEC-INTEG-008)
// Store webhook event IDs, skip duplicates
const existing = await db
  .from('webhook_events')
  .select('id')
  .eq('event_id', event.id)
  .single()

if (existing.data) {
  return new Response('Already processed', { status: 200 })
}

// Process webhook, then record it
await processWebhook(event)
await db.from('webhook_events').insert({
  event_id: event.id,
  event_type: event.type,
  processed_at: new Date().toISOString()
})
```

```typescript
// FIX: Graceful External Service Degradation (SEC-INTEG-032)
let enrichmentData = null
try {
  enrichmentData = await externalService.enrich(query)
} catch (error) {
  console.warn(`${serviceName} unavailable, proceeding without enrichment:`, error.message)
  // Pipeline continues with null enrichment — degraded but functional
}
```

---

### Agent 12: Cryptographic & Randomness Analysis

**Checks for weak cryptographic practices across any codebase.**

**Discovery:** Scan for crypto usage patterns:
```
**/auth*|**/token*|**/session*     # Auth & token generation
**/hash*|**/encrypt*|**/crypto*    # Crypto utilities
**/password*|**/reset*|**/verify*  # Password handling
lib/**|utils/**|helpers/**          # Utility code
```

#### 12A: Hashing & Password Storage

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Weak Password Hashing** | MD5, SHA1, SHA256 used for password storage (should be bcrypt/scrypt/argon2) | SEC-CRYPTO-001 |
| **Low Cost Factor** | bcrypt rounds < 10 or scrypt N < 16384 — too fast to resist brute force | SEC-CRYPTO-002 |
| **Hardcoded Salt** | Static salt value in source code instead of per-password random salt | SEC-CRYPTO-003 |
| **Hash Without Salt** | `crypto.createHash()` for passwords without any salt | SEC-CRYPTO-004 |
| **Weak Token Hashing** | API tokens/reset tokens stored as plaintext or weak hash in DB | SEC-CRYPTO-005 |

#### 12B: Random Number Generation

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Math.random() for Security** | `Math.random()` used for tokens, session IDs, OTP codes, or any security-sensitive value | SEC-CRYPTO-006 |
| **Predictable ID Generation** | Sequential or timestamp-based IDs for sensitive resources (should be UUIDv4 or CSPRNG) | SEC-CRYPTO-007 |
| **Weak OTP/Code Generation** | Short OTP codes (<6 digits) or non-cryptographic random source | SEC-CRYPTO-008 |
| **Missing CSPRNG** | `crypto.randomBytes()` or `crypto.randomUUID()` not used where needed | SEC-CRYPTO-009 |

#### 12C: Comparison & Timing

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Non-Timing-Safe Comparison** | `===` or `==` used to compare secrets, tokens, API keys, HMAC signatures (should be `timingSafeEqual`) | SEC-CRYPTO-010 |
| **Token Comparison Leak** | Early return on first mismatched byte reveals token length/prefix | SEC-CRYPTO-011 |
| **Webhook Signature Check** | Webhook HMAC verification using string comparison instead of timing-safe | SEC-CRYPTO-012 |

#### 12D: Encryption & Keys

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Hardcoded Encryption Keys** | Encryption keys as string literals in source code | SEC-CRYPTO-013 |
| **Hardcoded IV/Nonce** | Static initialization vector — same IV reused across encryptions (breaks AES-CBC/GCM security) | SEC-CRYPTO-014 |
| **Weak Algorithm** | DES, 3DES, RC4, or ECB mode used for encryption | SEC-CRYPTO-015 |
| **Missing Key Rotation** | No mechanism to rotate encryption keys or API signing keys | SEC-CRYPTO-016 |
| **JWT Weak Algorithm** | JWT signed with HS256 using short secret, or `none` algorithm accepted | SEC-CRYPTO-017 |

**Auto-Fix:**
- Replace `Math.random()` with `crypto.randomUUID()` or `crypto.randomBytes()`
- Replace `===` with `crypto.timingSafeEqual()` for secret comparisons
- Upgrade password hashing to bcrypt with cost factor 12+
- Replace hardcoded IVs with `crypto.randomBytes(16)` per-encryption

---

### Agent 13: Infrastructure & Configuration Security

**Scans deployment configs, CI/CD pipelines, and build settings for security issues.**

**Discovery:** Detect infrastructure files:
```
vercel.json|netlify.toml|fly.toml   # Deployment config
next.config.*|vite.config.*|nuxt.config.*  # Framework config
.github/workflows/*.yml              # CI/CD pipelines
Dockerfile|docker-compose*           # Container config
.env.example|.env.*.example          # Env templates
supabase/config.toml                 # Supabase config
```

#### 13A: Deployment Configuration

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Permissive CORS in Config** | `allowedOrigins: ["*"]` or overly broad origins in deployment config | SEC-INFRA-001 |
| **Missing Security Headers** | Deployment config doesn't set HSTS, X-Frame-Options, CSP (relying on code only) | SEC-INFRA-002 |
| **Debug Mode in Production** | `NODE_ENV=development` or `debug: true` in production config | SEC-INFRA-003 |
| **Source Maps Enabled** | `productionBrowserSourceMaps: true` in Next.js config or equivalent | SEC-INFRA-004 |
| **Exposed Server Info** | `poweredByHeader: true` (default in Next.js) leaks framework version | SEC-INFRA-005 |
| **Permissive Rewrites/Redirects** | Wildcard rewrites that could proxy to internal services | SEC-INFRA-006 |

#### 13B: CI/CD Pipeline Security

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Secrets in Workflow Files** | Hardcoded secrets, tokens, or credentials in `.github/workflows/*.yml` | SEC-INFRA-007 |
| **Untrusted Input in Commands** | `${{ github.event.pull_request.title }}` used in `run:` (command injection via PR title) | SEC-INFRA-008 |
| **Overly Permissive Permissions** | `permissions: write-all` or `contents: write` without justification | SEC-INFRA-009 |
| **Missing Checkout Pin** | `uses: actions/checkout@v4` without SHA pin — vulnerable to supply chain attack | SEC-INFRA-010 |
| **Self-Hosted Runner Risk** | Workflows using `runs-on: self-hosted` without security controls | SEC-INFRA-011 |
| **No Dependency Review** | PRs merge without automated dependency vulnerability checks | SEC-INFRA-012 |

#### 13C: Container & Build Security

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Root Container** | `Dockerfile` runs as root (no `USER` directive) | SEC-INFRA-013 |
| **Secrets Copied to Image** | `COPY .env` or `COPY *.key` in Dockerfile — secrets baked into image | SEC-INFRA-014 |
| **No .dockerignore** | Missing `.dockerignore` — `.env`, `.git`, `node_modules` may be included in image | SEC-INFRA-015 |
| **Latest Tag** | `FROM node:latest` instead of pinned version — unpredictable builds | SEC-INFRA-016 |
| **Build Secrets Exposed** | `ARG` used for secrets (visible in image history) instead of `--mount=type=secret` | SEC-INFRA-017 |

#### 13D: Environment Variable Hygiene

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **NEXT_PUBLIC_ Secrets** | Sensitive values (API keys, DB URLs) exposed via `NEXT_PUBLIC_` prefix | SEC-INFRA-018 |
| **Missing .env.example** | No `.env.example` to document required variables — new devs may misconfigure | SEC-INFRA-019 |
| **Default/Placeholder Secrets** | `.env.example` contains real-looking values that could be used in production | SEC-INFRA-020 |
| **Env Validation Missing** | No runtime validation that required env vars are present (app crashes cryptically) | SEC-INFRA-021 |

**Auto-Fix:**
- Set `poweredByHeader: false` in Next.js config
- Disable `productionBrowserSourceMaps`
- Add SHA pins to GitHub Actions
- Add `USER node` to Dockerfiles
- Create `.dockerignore` excluding `.env`, `.git`, `node_modules`

---

### Agent 14: Data Flow & Taint Tracking

**Traces user input from entry point through transformations to dangerous sinks. Identifies taint chains where no sanitization exists in the full path.**

**This is the most complex agent.** It doesn't check patterns in isolation — it follows data across files and functions.

**Methodology:**
1. Identify all **sources** (user input entry points)
2. Identify all **sinks** (dangerous operations)
3. Trace the path from each source to each reachable sink
4. Check if sanitization/validation exists at any point in the path
5. Report paths where unsanitized user input reaches a dangerous sink

#### 14A: Source Identification

| Source | Pattern | How Input Enters |
|--------|---------|-----------------|
| Request body | `req.body`, `request.json()`, `await req.json()` | POST/PUT/PATCH body |
| URL params | `params.id`, `searchParams.get()`, `req.query` | URL path and query |
| Headers | `req.headers.get()`, `headers()` | HTTP headers |
| Cookies | `cookies().get()`, `req.cookies` | Cookie values |
| File uploads | `formData.get('file')`, `req.formData()` | Uploaded content |
| Database reads | `.select()`, `.findMany()`, user-generated content from DB | Stored user data |

#### 14B: Sink Identification

| Sink | Pattern | Risk |
|------|---------|------|
| **SQL** | String concatenation in `.from()`, `.rpc()`, raw SQL | SQL Injection |
| **HTML** | `dangerouslySetInnerHTML`, `innerHTML`, `document.write()` | XSS |
| **Redirect** | `redirect(userInput)`, `Response.redirect()`, `Location` header | Open Redirect |
| **Shell** | `exec()`, `spawn()`, `execSync()` with user input | Command Injection |
| **File system** | `readFile(userInput)`, `writeFile(userInput)` | Path Traversal |
| **Fetch/HTTP** | `fetch(userInput)`, `axios.get(userInput)` | SSRF |
| **Template** | Template literals with user input in prompts, emails | Template Injection |
| **Eval** | `eval()`, `Function()`, `vm.runInContext()` | Code Execution |
| **Log** | `console.log(userInput)`, `logger.info(userInput)` | Log Injection |

#### 14C: Taint Chain Analysis

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Unsanitized Body → SQL** | Request body reaches raw SQL query without parameterization | SEC-TAINT-001 |
| **Unsanitized Body → HTML** | Request body stored in DB, later rendered via `dangerouslySetInnerHTML` | SEC-TAINT-002 |
| **URL Param → Redirect** | URL parameter used in redirect without origin validation | SEC-TAINT-003 |
| **URL Param → File Path** | URL parameter used in file system operation without path sanitization | SEC-TAINT-004 |
| **URL Param → Fetch** | URL parameter used in `fetch()` without SSRF protection | SEC-TAINT-005 |
| **Body → Shell Command** | Request body reaches `exec()` or `spawn()` without escaping | SEC-TAINT-006 |
| **Stored Data → HTML** | User-generated content from DB rendered without sanitization | SEC-TAINT-007 |
| **Header → Log** | Request header written to log without CRLF stripping | SEC-TAINT-008 |
| **Cookie → SQL** | Cookie value used in database query without validation | SEC-TAINT-009 |
| **Upload → File System** | Uploaded filename used in file path without sanitization | SEC-TAINT-010 |
| **DB Config → Eval** | Database-stored configuration (model names, prompts) used in dynamic code execution | SEC-TAINT-011 |
| **Multi-Hop Taint** | Input passes through 3+ functions/files before reaching sink — each step looks safe individually | SEC-TAINT-012 |

**Key Instruction:** When tracing, follow the data through:
- Function calls (including async)
- Database storage and retrieval (stored taint)
- Template rendering
- API responses consumed by other endpoints
- Shared utility functions

---

### Agent 15: Security Regression Test Generation

**After vulnerabilities are fixed, generate permanent test files that catch regressions.**

**This agent runs AFTER the fix phase (Stage 4), not during scanning.** For each FIXED finding, create a test that would fail if the vulnerability is reintroduced.

**Discovery:** Detect test framework:
```bash
# Check for test runner
if grep -q "vitest" package.json; then TEST_RUNNER="vitest"
elif grep -q "jest" package.json; then TEST_RUNNER="jest"
elif grep -q "@playwright/test" package.json; then TEST_RUNNER="playwright"
else TEST_RUNNER="vitest"  # Default
fi
```

#### Test Generation Rules

| Finding Category | Test Type | What to Assert |
|-----------------|-----------|---------------|
| **SQL Injection** | Unit test | Parameterized query handles `' OR 1=1--` without error |
| **XSS** | Unit test | Output is sanitized (no `<script>` in rendered HTML) |
| **Auth Bypass** | Integration test | Protected route returns 401 without token |
| **IDOR** | Integration test | User A cannot read User B's resource |
| **Rate Limiting** | Integration test | 50 rapid requests → 429 after threshold |
| **RLS** | Database test | Query with User A's token returns only User A's data |
| **Mass Assignment** | Integration test | Extra fields (`is_admin`, `role`) are stripped |
| **SSRF** | Unit test | Private IPs rejected by URL validator |
| **Prompt Injection** | Unit test | System prompt not leaked when asked |
| **Missing Headers** | Integration test | Response includes CSP, HSTS, X-Frame-Options |
| **Crypto Weakness** | Unit test | Timing-safe comparison used for token verification |
| **Config Exposure** | Build test | Source maps disabled in production build |

**Output:** Write test files to `__tests__/security/` or `tests/security/`:
```
__tests__/security/
├── injection.test.ts       # SQLi, XSSi, Command injection regression
├── auth.test.ts            # Auth bypass, IDOR, privilege escalation
├── api.test.ts             # Rate limiting, mass assignment, CORS
├── crypto.test.ts          # Hashing, timing-safe, random generation
├── headers.test.ts         # Security headers presence
└── ai-security.test.ts     # Prompt injection, output validation
```

**Each test includes a comment linking it to the finding ID:** `// Regression test for SEC-INJ-001`

---

### Agent 16: API Specification Drift Detection

**Compares API documentation/specification against actual implementation to find shadow APIs and undocumented behavior.**

**Discovery:** Check for API specs:
```
openapi.json|openapi.yaml|swagger.json|swagger.yaml
docs/api/*|api-docs/*
README.md (API section)
```

#### 16A: Route Inventory vs Documentation

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Shadow APIs** | Routes that exist in code but not in documentation — undocumented endpoints | SEC-DRIFT-001 |
| **Ghost APIs** | Routes documented but not implemented — misleading docs | SEC-DRIFT-002 |
| **Method Mismatch** | Route supports GET in code but docs say POST only (or vice versa) | SEC-DRIFT-003 |
| **Auth Annotation Mismatch** | Route requires auth in code but docs don't mention it (or vice versa) | SEC-DRIFT-004 |

#### 16B: Parameter & Schema Drift

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Undocumented Parameters** | Route accepts parameters not listed in docs — hidden functionality | SEC-DRIFT-005 |
| **Missing Required Validation** | Docs say parameter is required but code has no validation for it | SEC-DRIFT-006 |
| **Response Schema Mismatch** | Actual response includes fields not in docs (PII leak) or missing documented fields | SEC-DRIFT-007 |
| **Deprecated But Active** | Endpoint marked deprecated in docs but still fully functional with no deprecation warning | SEC-DRIFT-008 |

#### 16C: Version & Compatibility

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Old Version Still Active** | `/api/v1/users` still works alongside `/api/v2/users` — old version may have weaker security | SEC-DRIFT-009 |
| **No Version Strategy** | API has no versioning — breaking changes affect all clients | SEC-DRIFT-010 |

**If no API specification exists:** Inventory all routes, document their methods, parameters, and auth requirements. Flag this as SEC-DRIFT-011 (No API Documentation). Generate a basic route inventory as a starting point.

---

### Agent 17: Build Artifact & Secrets in Output

**Scans build outputs, container images, CI logs, and deployment artifacts for leaked secrets.**

**Discovery:** Check for build artifacts:
```
.next/|dist/|build/|out/            # Build output directories
.github/workflows/*.yml              # CI pipeline logs
Dockerfile|docker-compose*           # Container definitions
.vercel/|.netlify/                   # Deployment cache
```

#### 17A: Build Output Scanning

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Secrets in Client Bundle** | API keys, service role keys, or private tokens in `.next/static/` JS files | SEC-BUILD-001 |
| **Source Maps in Build** | `.map` files present in build output (full source code exposure) | SEC-BUILD-002 |
| **Server Code in Client** | Server-only code or secrets leaked into client chunks via incorrect imports | SEC-BUILD-003 |
| **Debug Info in Build** | `console.log`, `debugger`, or verbose error messages in production build | SEC-BUILD-004 |
| **Environment Snapshot** | `.env` or environment variables captured in build output or build logs | SEC-BUILD-005 |

#### 17B: CI/CD Output Scanning

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Echo Secrets in CI** | `echo $SECRET` or `printenv` in workflow steps — secrets visible in CI logs | SEC-BUILD-006 |
| **Secrets in Artifact Upload** | CI uploads build artifacts that contain `.env` files or secret configs | SEC-BUILD-007 |
| **Cache Poisoning Risk** | CI cache includes sensitive files that could be read by other workflows | SEC-BUILD-008 |
| **No Secret Masking** | CI output doesn't mask secret values — visible in build logs | SEC-BUILD-009 |

#### 17C: Container Image Scanning

| Check | What to Find | Finding ID |
|-------|--------------|------------|
| **Env in Image** | `.env` files or secret configs baked into Docker image layers | SEC-BUILD-010 |
| **Git History in Image** | `.git/` directory included in container image — full repo history with secrets | SEC-BUILD-011 |
| **Node Modules in Image** | Full `node_modules/` in production image (increases attack surface) | SEC-BUILD-012 |
| **Multi-Stage Leak** | Secrets from build stage leaked into production stage via `COPY --from=builder` | SEC-BUILD-013 |

**Auto-Fix:**
- Add `.map` files to build exclusion
- Audit client bundle imports for server-only code
- Add `COPY .dockerignore` patterns for `.env`, `.git`, `node_modules`
- Replace `echo $SECRET` in CI with masked output
- Use multi-stage Docker builds with minimal final stage

---

## STAGE 3: ANALYSIS & PRIORITIZATION

### 3.1 Severity Classification

| Level | Criteria | Auto-Fix |
|-------|----------|----------|
| **CRITICAL** | Exploitable now, data breach risk | Yes, immediately |
| **HIGH** | Significant risk, some conditions needed | Yes, high priority |
| **MEDIUM** | Should fix, harder to exploit | Yes |
| **LOW** | Best practice, minor risk | Yes |

### 3.2 Build Fix Order

Respect dependencies:
```
RLS enable → RLS policies → Code ownership checks
Auth middleware → Protected routes
Input validation → Database queries
```

---

## STAGE 4: AUTO-FIX LOOP

### 4.1 For Each Finding (Living Report Update Cycle)

Fix in severity order: CRITICAL → HIGH → MEDIUM → LOW. For each finding:

```
1. Update finding row Status → 🔧 FIXING
2. Update report header Status → 🟡 FIXING — SEC-XXX [description]
3. Write report to disk

4. Check dependencies (are blockers fixed?)
5. Read file and context
6. ATTEMPT 1: Apply fix + add WHY comment
   - Verify: build + quick test
   - If passes → update row → ✅ FIXED, add to Fix Log, add to SITREP "What Was Fixed"
   - If fails → revert

7. ATTEMPT 2: Alternative approach
   - If passes → ✅ FIXED
   - If fails → revert

8. ATTEMPT 3: Different angle
   - If passes → ✅ FIXED
   - If fails → revert, decide:
     a. Needs human decision → ⏸️ DEFERRED
        Add to SITREP "What Was Deferred": reason + what was tried + risk if unfixed
     b. Fix impossible with current approach → 🚫 BLOCKED
        Add to SITREP "What Was Blocked": all 3 attempts + why each failed + what would be needed

9. Append to Progress Log: | [HH:MM] | Stage 4 | SEC-XXX | ✅/⏸️/🚫 |
10. Write report to disk — checkpoint after EVERY status change
11. Move to next finding
```

**The report file is written to disk after EVERY status change.** If the session dies mid-fix, the report shows exactly which findings are done and which remain.

### 4.2 Iterate Until Clean

After all fixes applied:
1. Re-run security scan
2. If new findings → add to task list with `FOUND` status, fix them
3. Repeat until ZERO `FOUND` findings remain
4. Update iteration count in report
5. Append to Progress Log: `| [HH:MM] | Stage 4 | Iteration [N] complete | [X] remaining |`

---

## STAGE 5: VALIDATION

### 5.1 Build & Test
```bash
npm run build  # Must pass
npm test       # Must pass
```

### 5.2 Re-Scan
Verify all findings are actually fixed.

### 5.3 Security Tests
Generate regression tests for critical fixes.

---

## STAGE 6: REPORT & SITREP

### 6.1 Main Report File

```markdown
# Security Audit: [PROJECT_NAME]

**Created:** [YYYY-MM-DD HH:MM:SS]
**Completed:** [YYYY-MM-DD HH:MM:SS]
**Status:** 🟢 COMPLETE

---

## Executive Summary

### Security Score

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| CRITICAL | 3 | 0 | -3 (100% fixed) |
| HIGH | 8 | 0 | -8 (100% fixed) |
| MEDIUM | 12 | 0 | -12 (100% fixed) |
| LOW | 5 | 0 | -5 (100% fixed) |
| **Score** | **F (32)** | **A+ (100)** | **+68 points** |

### OWASP Coverage

| Category | Findings | Fixed |
|----------|----------|-------|
| A01 Broken Access Control | 5 | 5 ✅ |
| A02 Cryptographic Failures | 2 | 2 ✅ |
| A03 Injection | 4 | 4 ✅ |
| A05 Security Misconfiguration | 6 | 6 ✅ |
| A06 Vulnerable Components | 3 | 3 ✅ |
| A07 Auth Failures | 4 | 4 ✅ |
| A09 Logging Failures | 2 | 2 ✅ |
| LLM01 Prompt Injection | 2 | 2 ✅ |
| LLM06 Data Disclosure | 1 | 1 ✅ |

---

## Task List

### CRITICAL

- [x] **SEC-INJ-001**: SQL injection in user search
  - File: `src/app/api/users/route.ts:45`
  - Fix: Parameterized query
  - Fixed at: 14:32:15
  - Verified: ✅ Build ✓ Test ✓

- [x] **SEC-AUTH-001**: Missing auth on admin routes
  - Files: `src/app/api/admin/*.ts` (5 files)
  - Fix: Added auth middleware
  - Fixed at: 14:35:22
  - Verified: ✅ Build ✓ Test ✓

### HIGH

- [x] **SEC-LLM-001**: Direct prompt injection vulnerability
  - File: `src/app/api/chat/route.ts:23`
  - Fix: Input sanitization + validation
  - Fixed at: 14:38:45
  - Verified: ✅ Build ✓ Test ✓

- [x] **SEC-RLS-001**: RLS disabled on transactions table
  - Fix: Enabled RLS + added policies
  - Fixed at: 14:42:10
  - Verified: ✅ Build ✓ Test ✓

### MEDIUM

- [x] **SEC-HDR-001**: Missing security headers
  - Fix: Added CSP, HSTS, X-Frame-Options, etc.
  - Fixed at: 14:45:30
  - Verified: ✅ Build ✓ Test ✓

- [x] **SEC-API-001**: No rate limiting on auth endpoints
  - Fix: Added Upstash rate limiting
  - Fixed at: 14:48:55
  - Verified: ✅ Build ✓ Test ✓

### LOW

- [x] **SEC-CFG-003**: Console.logs in server code
  - Fix: Removed 23 console.log statements
  - Fixed at: 14:52:00
  - Verified: ✅ Build ✓ Test ✓

---

## Iterations

| Iteration | Findings | Fixed | Remaining |
|-----------|----------|-------|-----------|
| 1 | 28 | 26 | 2 |
| 2 | 2 | 2 | 0 |
| **Final** | **0** | **-** | **0** ✅ |

---

## Files Modified

| File | Changes |
|------|---------|
| `src/app/api/users/route.ts` | Parameterized query |
| `src/app/api/chat/route.ts` | AI input sanitization |
| `src/middleware.ts` | Auth + rate limiting |
| `next.config.js` | Security headers |
| ... | ... |

**Total:** 34 files modified

---

## Files Created

| File | Purpose |
|------|---------|
| `src/lib/ai-security.ts` | AI input sanitization utilities |
| `src/lib/rate-limit.ts` | Rate limiting middleware |
| `supabase/migrations/20260205_security.sql` | RLS policies |

**Total:** 5 files created

---

## Security Tests Added

| Test | Coverage |
|------|----------|
| `tests/security/injection.test.ts` | SQL, NoSQL, Command injection |
| `tests/security/auth.test.ts` | Auth bypass, IDOR |
| `tests/security/ai.test.ts` | Prompt injection |

---

## Verification

| Check | Status |
|-------|--------|
| Build | ✅ Pass |
| Tests | ✅ 178/178 passing |
| Re-scan | ✅ 0 findings |
| Auth flows | ✅ Verified |
| AI endpoints | ✅ Verified |

---

## SITREP (Conclusion)

### Mission Status: 🟢 COMPLETE

**Duration:** 24 minutes 18 seconds
**Iterations:** 2

### What Was Accomplished

1. **Injection Vulnerabilities Eliminated (4)**
   - SQL injection in user search → parameterized
   - Command injection in export → input validation
   - Prompt injection in AI chat → sanitization layer
   - Log injection in audit → CRLF stripping

2. **Authentication & Authorization Hardened (9)**
   - 5 unprotected admin routes secured
   - RLS enabled on 3 tables with proper policies
   - Ownership checks added to 4 API routes
   - Session configuration fixed (HttpOnly, Secure, SameSite)

3. **AI Security Implemented (3)**
   - Input sanitization for all AI endpoints
   - Output validation and HTML escaping
   - Token limits and rate limiting
   - Created `src/lib/ai-security.ts` utility library

4. **API Security Enhanced (8)**
   - Rate limiting on all sensitive endpoints
   - Zod validation on all inputs
   - Response filtering (no full objects)
   - CORS properly configured
   - Error handling sanitized

5. **Security Headers Added (6)**
   - CSP (strict, no unsafe-eval)
   - HSTS (1 year, includeSubDomains)
   - X-Frame-Options (DENY)
   - X-Content-Type-Options (nosniff)
   - Referrer-Policy (strict-origin-when-cross-origin)
   - Permissions-Policy (restrictive)

### Security Score

| Before | After | Change |
|--------|-------|--------|
| F (32/100) | A+ (100/100) | **+68 points** |

### Vulnerabilities Fixed

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | 3 | 100% fixed ✅ |
| HIGH | 8 | 100% fixed ✅ |
| MEDIUM | 12 | 100% fixed ✅ |
| LOW | 5 | 100% fixed ✅ |
| **TOTAL** | **28** | **100% fixed** ✅ |

### Historical Context

This is security audit #3 for this project:
- Audit #1 (2025-12-01): Score F (28) → D (58)
- Audit #2 (2026-01-15): Score D (58) → B (82)
- **Audit #3 (today):** Score B (82) → A+ (100)

**Trend:** Security posture improving consistently

### Recommendations

1. **Maintain:** Run `/sec-ship` before each release
2. **Monitor:** Set up security monitoring (Sentry security alerts)
3. **Rotate:** Rotate the Stripe key found in git history
4. **Review:** Schedule quarterly security review

### Next Steps

1. Run `/gh-ship` to deploy security fixes
2. Rotate exposed API key (flagged)
3. Enable Vercel security monitoring

---

*Report generated by /sec-ship*
*Full history: .security-reports/history.json*
*No vulnerabilities remaining: ✅*
```

### 6.2 Update History

Append to `.security-reports/history.json`:

```json
{
  "audits": [
    {
      "id": 3,
      "timestamp": "2026-02-05T15:00:00Z",
      "duration_seconds": 1458,
      "iterations": 2,
      "findings": {
        "critical": { "found": 3, "fixed": 3 },
        "high": { "found": 8, "fixed": 8 },
        "medium": { "found": 12, "fixed": 12 },
        "low": { "found": 5, "fixed": 5 }
      },
      "score_before": 32,
      "score_after": 100,
      "owasp_coverage": ["A01", "A02", "A03", "A05", "A06", "A07", "A09", "LLM01", "LLM06"],
      "files_modified": 34,
      "files_created": 5,
      "tests_added": 3,
      "report_file": ".security-reports/sec-ship-20260205-150000.md"
    }
  ]
}
```

---

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### Sec-Ship-Specific Cleanup

Cleanup actions:
1. **Gitignore enforcement:** Ensure `.security-reports/` is in `.gitignore`
2. **Dependency disclosure:** If security fixes required installing packages (e.g., DOMPurify, helmet), document in SITREP as intentional additions
3. **No browser, server, or test data cleanup needed** — this skill modifies code only

---

## REMEMBER

- **Fix EVERYTHING** — Definition of done is ZERO vulnerabilities
- **Iterate until clean** — Re-scan, fix new findings, repeat
- **Report is the living task list** — Updated after every status change, written to disk constantly
- **Never delete findings** — Only update status: ✅ FIXED / ⏸️ DEFERRED / 🚫 BLOCKED
- **3 attempts before defeat** — Try 3 different approaches, document each, then mark BLOCKED
- **SITREP annotates everything** — What was fixed, what was deferred (WHY), what was blocked (WHY + all attempts)
- **AI security is critical** — Prompt injection is the new SQL injection
- **Test after fixes** — Verify nothing broke
- **Score and track** — Show improvement over time
- **Report survives session restart** — Resume from where it left off via Progress Log

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
