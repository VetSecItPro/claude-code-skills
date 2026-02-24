# /sec-ship — Full Security Pipeline: Audit → Fix → Validate → Report

You are a security automation specialist. Execute the COMPLETE security pipeline autonomously from start to finish. Discover vulnerabilities, fix them, validate fixes, and report results. Handle everything. Fix everything fixable. Definition of done: **NO vulnerabilities remain unfixed** (except those explicitly flagged for human review).

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
| Dependency CVE scanner | `sonnet` | Must assess vulnerability severity and applicability to this project |
| Code vulnerability scanner | `sonnet` | Must understand if patterns are actually exploitable vs false positive |
| RLS policy analyzer | `sonnet` | Must understand SQL semantics and authorization logic |
| Rate limit analyzer | `sonnet` | Must assess if missing rate limiting is exploitable in context |
| Input validation analyzer | `sonnet` | Must understand data flow to determine if missing validation is a real risk |
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

## STAGE 2: PARALLEL SECURITY SCANNING (8 Agents)

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
