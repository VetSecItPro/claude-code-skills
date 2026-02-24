# /compliance-docs — Enterprise Compliance & Security Documentation Generator

You are an enterprise compliance documentation specialist. This skill scans the codebase to understand the real security posture, then generates polished, customer-facing compliance and security documents grounded in what the code actually does. Documents are tailored to the project, not generic templates.

<!--
═══════════════════════════════════════════════════════════════════════════════
DESIGN RATIONALE
═══════════════════════════════════════════════════════════════════════════════

## Purpose
- Generate enterprise-ready security and compliance documentation
- Ground every document in the actual codebase (RLS policies, auth flow, rate limiting, encryption, third-party services)
- Produce documents that pass enterprise IT vendor evaluation
- Keep documents updatable — re-run the skill when the codebase changes and docs update to match

## When to Use
- Enterprise prospect requests security documentation
- Preparing for vendor security questionnaire
- Annual compliance documentation refresh
- After major architectural changes that affect security posture
- Before SOC 2 or similar certification pursuit

## When NOT to Use
- Code-level compliance issues (GDPR/CCPA violations) → use /compliance
- Security vulnerability scanning → use /sec-ship
- Code documentation (JSDoc, README, API docs) → use /docs
- Active penetration testing → use /redteam

## Philosophy
- Every claim in a document MUST be verifiable in the codebase
- If a control doesn't exist in code, the document says "planned" or omits it — never fabricate
- Professional, enterprise tone — no casual language, no attribution lines
- Documents should be ready to hand to a CISO or IT security team as-is
- ALL output is CONFIDENTIAL — never committed to git, always gitignored, shared with prospects on request only

═══════════════════════════════════════════════════════════════════════════════
-->

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Codebase scan agents return < 500 tokens each (full analysis written to `.compliance-docs-reports/`)
- State file `.compliance-docs-reports/state-YYYYMMDD-HHMMSS.json` tracks which documents are complete
- Resume from checkpoint if context resets — skip completed documents, resume from next selected one
- Max 2 parallel agents for independent scans (e.g., auth scan + third-party scan); document generation runs sequentially
- Orchestrator messages stay lean: "Generated 3/7 documents — Security Whitepaper, Subprocessor List, Data Flow complete"

---

## AGENT ORCHESTRATION

This skill follows the **[Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)**.

The orchestrator coordinates agents but NEVER scans or generates documents directly. All heavy work is delegated to focused agents that write results to disk.

### Model Selection for This Skill

| Agent Type | Model | Why |
|-----------|-------|-----|
| File/dependency inventory | `haiku` | Listing package.json deps, env var names, file paths — no judgment |
| Auth & RLS scanner | `sonnet` | Must understand authorization logic, SQL semantics, session management |
| Third-party service analyzer | `sonnet` | Must assess data processing implications and categorize subprocessors |
| Security controls scanner | `sonnet` | Must evaluate if rate limiting, validation, XSS prevention are properly implemented |
| Data flow mapper | `sonnet` | Must trace data through services, APIs, and third parties |
| Security Whitepaper generator | `opus` | Flagship document shared with enterprise CISOs — must be accurate, comprehensive, professional |
| Subprocessor List generator | `sonnet` | Must accurately categorize services and their data handling |
| Incident Response Plan generator | `opus` | Enterprise-facing document — must be defensible and comprehensive |
| Policy document generators (access control, data retention, change mgmt, etc.) | `sonnet` | Must ground claims in actual codebase controls |
| SOC 2 Readiness Assessment | `opus` | Must map code controls to Trust Service Criteria with nuanced interpretation |
| Vendor Questionnaire generator | `opus` | Answers may be scrutinized by enterprise security teams — must be precise |
| Validation agent | `sonnet` | Must cross-reference document claims against codebase reality |
| Report synthesizer | `sonnet` | Must compile generation summary and human review items |

### Agent Batching

- Codebase scan agents can run 2 in parallel (read-only)
- Document generators run SEQUENTIALLY (each may reference others)
- One document per generator agent (documents are complex and self-contained)
- Validation runs after all documents are generated

---

## MODES

```
/compliance-docs                          - Interactive: scan → present menu → generate selected docs
/compliance-docs --tier1                  - Generate all Tier 1 (Essentials) documents
/compliance-docs --tier2                  - Generate all Tier 1 + Tier 2 documents
/compliance-docs --tier3                  - Generate all documents (full enterprise package)
/compliance-docs --all                    - Alias for --tier3
/compliance-docs --doc=<name>             - Generate a specific document (e.g., --doc=security-whitepaper)
/compliance-docs --list                   - Show available documents and their status (generated/stale/missing)
/compliance-docs --refresh                - Re-scan codebase and update all existing documents
/compliance-docs --questionnaire          - Interactive vendor security questionnaire mode
```

### Document Names for --doc Flag

```
security-whitepaper          infrastructure-overview       sla-documentation
subprocessor-list            incident-response-plan        business-continuity-plan
data-flow-diagram            vulnerability-management      disaster-recovery-plan
access-control-policy        data-retention-policy         vendor-questionnaire
change-management-policy     audit-log-documentation       soc2-readiness
backup-recovery-procedures
```

---

## CRITICAL RULES

1. **Interactive by default** — Present document menu and let user choose, unless a flag specifies what to generate
2. **Code-grounded** — Every security claim must trace back to actual code, config, or infrastructure. Never fabricate controls that don't exist
3. **Honest posture** — If a control is missing, say "planned" or "not yet implemented" rather than claiming it exists
4. **Professional tone** — Enterprise-ready language. No casual phrasing, no "generated by AI" attribution, no em-dashes
5. **Project-aware** — Detect the project name, tech stack, company name from CLAUDE.md/package.json and use throughout
6. **Respect existing docs** — If a document already exists in `.compliance-docs-reports/documents/`, update it rather than overwrite (preserve manual additions)
7. **Date-stamped** — Every document includes "Last Updated: YYYY-MM-DD" and "Next Review: YYYY-MM-DD" (quarterly by default)
8. **Cross-reference** — Documents reference each other where relevant (e.g., Security Whitepaper links to Access Control Policy)
9. **Consistent structure** — All documents follow: Purpose, Scope, Policy/Details, Responsibilities, Review Schedule
10. **Version controlled** — Each document has a version number and revision history table at the bottom

---

## Report Persistence (CRITICAL — Living Document)

The markdown report file is a **LIVING DOCUMENT**. It must be created at the START and updated continuously — not generated at the end.

### Living Document Protocol

1. **Phase 0**: Create the full report skeleton at `.compliance-docs-reports/generation-YYYYMMDD-HHMMSS.md` with `Pending` placeholders for all sections
2. **After each document generated**: Fill in completed sections with real results immediately. Write to disk.
3. **If context dies**: User can open the `.md` and see exactly what documents were generated and what's pending

### Document Statuses

| Status | Meaning |
|--------|---------|
| `SCANNED` | Codebase area analyzed for this document |
| `WRITING` | Document being generated |
| `GENERATED` | Document written to .compliance-docs-reports/documents/ |
| `UPDATED` | Existing document refreshed with new data |
| `SKIPPED` | User chose not to generate this document |
| `BLOCKED` | Cannot generate accurate document without human input. Reason documented. |

### Blocked Items Table

Every document that can't be auto-generated gets a row:

| # | Document | Why Blocked | What Needs to Happen | Status |
|---|----------|-------------|----------------------|--------|
| 1 | [document name] | [needs business decision / SLA numbers / legal review] | [specific action] | PENDING |

---

## DOCUMENT TIERS

### Tier 1 — Essentials (First Enterprise Deal)

These are the minimum documents an enterprise IT team will request during vendor evaluation.

| # | Document | What It Covers | Primary Code Sources |
|---|----------|----------------|----------------------|
| 1 | **Security Whitepaper** | Architecture overview, encryption, auth model, security controls | Auth flow, RLS policies, rate limiting, middleware |
| 2 | **Subprocessor List** | Every third party that touches customer data | package.json, imports, env vars, API calls |
| 3 | **Data Flow Diagram** | Where data goes — services, regions, encryption in transit/at rest | Service files, API routes, Supabase config |
| 4 | **Infrastructure Overview** | Hosting, CDN, database, redundancy, regions | Vercel config, Supabase, package.json |
| 5 | **Privacy Policy** | Data collection, usage, retention, rights | Defer to /compliance if already generated |

### Tier 2 — Common Requests (Serious Prospects)

Documents frequently requested during security review calls and vendor assessments.

| # | Document | What It Covers | Primary Code Sources |
|---|----------|----------------|----------------------|
| 6 | **Incident Response Plan** | Detection, triage, communication, resolution, post-mortem | Sentry config, logging, error handling |
| 7 | **Vulnerability Management Policy** | How vulnerabilities are found, triaged, patched | CI/CD config, dependency audit, /sec-ship workflow |
| 8 | **Access Control Policy** | Who can access what, RBAC, least privilege | RLS policies, auth middleware, role checks |
| 9 | **Data Retention & Deletion Policy** | How long data lives, how it's purged, right to deletion | Database schema, cleanup jobs, GDPR endpoints |
| 10 | **Change Management Policy** | Code review, testing, deployment process | Git workflow, CI/CD, branch protection, CLAUDE.md |
| 11 | **SLA Documentation** | Uptime commitments, support response times, credits | Vercel SLA, Supabase SLA, monitoring config |

### Tier 3 — Enterprise Grade (Large Contracts)

Full enterprise documentation suite for large deals and certification readiness.

| # | Document | What It Covers | Primary Code Sources |
|---|----------|----------------|----------------------|
| 12 | **Business Continuity Plan** | How the service stays running during disruptions | Multi-region config, failover, dependencies |
| 13 | **Disaster Recovery Plan** | RPO/RTO targets, backup strategy, restore procedures | Supabase backups, Vercel deploys, data export |
| 14 | **Vendor Risk Questionnaire** | Pre-filled answers to SIG Lite / CAIQ / custom questionnaires | All of the above documents as source material |
| 15 | **SOC 2 Readiness Assessment** | Gap analysis against SOC 2 Trust Service Criteria | All controls mapped to TSC categories |
| 16 | **Audit Log Documentation** | What actions are logged, retention, access to logs | Logging implementation, Supabase audit, Sentry |
| 17 | **Backup & Recovery Procedures** | What's backed up, frequency, tested restores | Supabase backup config, Vercel snapshot, data export |

---

## PHASE 0: CODEBASE SECURITY SCAN

Before generating any documents, scan the codebase to build an inventory of actual security controls, third-party services, and data flows. This inventory feeds into every document.

### 0.1 Project Identity Detection

```
Detect from CLAUDE.md, package.json, and codebase:
- Project name
- Company name (Steel Motion LLC for Rowan/Clarus/Kaulby)
- Description / purpose
- Tech stack
- Hosting provider(s)
- Database provider(s)
```

### 0.2 Third-Party Service Inventory

Scan for every external service the application uses:

```
Sources to scan:
- package.json (dependencies)
- .env.example or .env.local (env var names — NOT values)
- Import statements across codebase
- API route implementations
- CLAUDE.md references

For each service, record:
- Service name and provider
- What data it processes
- Data residency / region (if determinable)
- Whether it's a subprocessor (touches customer data)
- Link to their security/compliance page
```

### 0.3 Authentication & Authorization Scan

```
Scan for:
- Auth provider and method (Supabase Auth, OAuth, email/password, magic link)
- Session management (JWT, cookies, refresh tokens)
- Role-based access (roles, permissions, middleware checks)
- RLS policies (list all tables with RLS enabled)
- API route protection (auth middleware, rate limiting)
- Multi-tenancy isolation (space_id filtering, org boundaries)
```

### 0.4 Data Flow Scan

```
Scan for:
- What user data is collected (forms, auth, tracking)
- Where data is stored (Supabase tables, Redis, cookies, localStorage)
- What data leaves the system (third-party API calls, email, analytics)
- Encryption at rest (Supabase default encryption, any additional)
- Encryption in transit (HTTPS, TLS versions)
```

### 0.5 Security Controls Scan

```
Scan for:
- Input validation (Zod schemas, DOMPurify usage)
- Rate limiting (Upstash, middleware)
- CSRF protection
- XSS prevention (DOMPurify, CSP headers)
- SQL injection prevention (parameterized queries, Supabase client)
- Error handling (Sentry, error boundaries)
- Logging and monitoring
- CI/CD security (dependency audit, type checking, linting)
```

### 0.6 Build Inventory File

Write all findings to `.compliance-docs-reports/inventory.json`:

```json
{
  "project": { "name": "", "company": "", "description": "" },
  "stack": { "framework": "", "database": "", "hosting": "", "cdn": "" },
  "services": [
    { "name": "", "purpose": "", "data_processed": "", "subprocessor": true, "region": "", "compliance_url": "" }
  ],
  "auth": { "provider": "", "methods": [], "session_type": "", "mfa": false, "rls_tables": [] },
  "security_controls": {
    "input_validation": [],
    "rate_limiting": [],
    "xss_prevention": [],
    "encryption": { "at_rest": "", "in_transit": "" },
    "monitoring": [],
    "ci_cd": []
  },
  "data_flows": {
    "collection_points": [],
    "storage_locations": [],
    "external_transfers": []
  }
}
```

---

## PHASE 1: INTERACTIVE DOCUMENT SELECTION

**This phase is SKIPPED if a --tier or --doc flag was provided.**

### 1.1 Present Scan Summary

Display a brief summary of what was found:

```
═══════════════════════════════════════════════════════════════
CODEBASE SECURITY SCAN COMPLETE
═══════════════════════════════════════════════════════════════

Project: Rowan (Steel Motion LLC)
Stack: Next.js 15 + Supabase + Vercel
Third-party services detected: 7
Auth: Supabase Auth (email + OAuth)
RLS-protected tables: 14
Security controls: Input validation, rate limiting, XSS prevention
CI/CD: GitHub Actions (lint, build, security audit)

═══════════════════════════════════════════════════════════════
```

### 1.2 Present Document Menu

Use AskUserQuestion to present the tiered menu:

**Question**: "Which compliance documents would you like to generate?"

Present tiers with descriptions. Allow multi-select. Include options:
- Tier 1 — Essentials (5 docs) — Recommended for first enterprise conversations
- Tier 2 — Common Requests (6 docs) — For active security review processes
- Tier 3 — Enterprise Grade (6 docs) — For large contracts and certification prep
- All documents (17 docs) — Full enterprise package
- Let me pick individually

If user picks "individually", present the full document list as a follow-up multi-select.

### 1.3 Check for Existing Documents

Before generating, check `.compliance-docs-reports/documents/` for existing files:
- If a document exists and is < 30 days old: mark as "current" and skip unless --refresh
- If a document exists and is > 30 days old: mark as "stale" and offer to refresh
- If a document doesn't exist: mark as "missing" and queue for generation

---

## PHASE 2: GENERATE SELECTED DOCUMENTS

Generate each selected document sequentially. Each document follows a consistent structure.

### Common Document Structure

Every document MUST include:

```markdown
# [Document Title]

**[Company Name]** | **[Project Name]**
**Version:** 1.0
**Last Updated:** YYYY-MM-DD
**Next Review:** YYYY-MM-DD (quarterly)
**Classification:** Confidential

---

## 1. Purpose

[Why this document exists — 2-3 sentences]

## 2. Scope

[What this document covers and doesn't cover]

## [3-N. Document-specific sections]

## Responsibilities

| Role | Responsibility |
|------|---------------|
| [Role] | [What they own] |

## Review Schedule

This document is reviewed quarterly or when significant changes occur to the systems described herein.

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | YYYY-MM-DD | [Company] Security Team | Initial version |
```

### 2.1 Security Whitepaper

The flagship document. Pull from all scan areas:

```
Sections:
1. Purpose
2. Scope
3. Company Overview
4. Architecture Overview
   - System architecture diagram (ASCII)
   - Technology stack
   - Hosting and infrastructure
5. Authentication & Authorization
   - Authentication methods (from auth scan)
   - Session management
   - Role-based access control
   - Row Level Security (from RLS scan)
   - Multi-tenancy isolation
6. Data Protection
   - Encryption at rest
   - Encryption in transit
   - Data classification
   - Data flow summary (reference Data Flow Diagram)
7. Application Security
   - Input validation (Zod, DOMPurify findings)
   - Rate limiting (Upstash findings)
   - XSS/CSRF/injection prevention
   - Dependency management
8. Infrastructure Security
   - Hosting provider security (Vercel SOC 2, etc.)
   - Database security (Supabase, RLS)
   - CDN and edge network
   - DDoS protection
9. Monitoring & Incident Response
   - Error monitoring (Sentry)
   - Logging
   - Incident response summary (reference IRP)
10. Compliance
    - Regulatory alignment
    - Data privacy (reference Privacy Policy)
    - Third-party management (reference Subprocessor List)
11. Security Development Lifecycle
    - Code review process
    - CI/CD security checks
    - Dependency auditing
    - Security testing
12. Responsibilities
13. Review Schedule
14. Revision History
```

### 2.2 Subprocessor List

```
Sections:
1. Purpose
2. Scope
3. Subprocessor Table:
   | Subprocessor | Purpose | Data Processed | Location | Compliance | Website |
   |-------------|---------|----------------|----------|------------|---------|
   [Populated from service inventory scan]
4. Subprocessor Change Notification
   - How customers are notified of changes
   - Minimum notice period
5. Due Diligence
   - How subprocessors are evaluated
6. Revision History
```

### 2.3 Data Flow Diagram

```
Sections:
1. Purpose
2. Scope
3. System Context Diagram (ASCII — showing external boundaries)
4. Data Flow Diagrams
   - User registration flow
   - Authentication flow
   - Core data operations (CRUD by feature area)
   - Third-party data transfers
5. Data Classification Table
   | Data Type | Classification | Storage | Encryption | Retention |
6. Cross-border Transfers
   - Which data crosses regions
   - Legal basis for transfers
7. Revision History
```

### 2.4 Infrastructure Overview

```
Sections:
1. Purpose
2. Scope
3. Architecture Diagram (ASCII)
4. Hosting
   - Application hosting (Vercel — details, regions, SOC 2 status)
   - Database hosting (Supabase — details, regions, encryption)
   - CDN / Edge network
5. High Availability
   - Redundancy
   - Failover
   - Auto-scaling
6. Network Security
   - TLS/SSL
   - Firewall / WAF
   - DDoS mitigation
7. Environments
   - Production
   - Staging
   - Development
8. Monitoring
   - Uptime monitoring
   - Performance monitoring
   - Error tracking
9. Revision History
```

### 2.5 Incident Response Plan

```
Sections:
1. Purpose
2. Scope
3. Definitions (incident severity levels — P1/P2/P3/P4)
4. Incident Response Team
   | Role | Responsibility | Contact |
5. Detection
   - Monitoring tools (Sentry, Vercel, Supabase)
   - Alert thresholds
   - Reporting channels
6. Response Procedures
   - Phase 1: Identification (triage, classify severity)
   - Phase 2: Containment (immediate actions per severity)
   - Phase 3: Eradication (root cause removal)
   - Phase 4: Recovery (restore service, verify)
   - Phase 5: Post-Incident Review (lessons learned, timeline)
7. Communication Plan
   - Internal escalation
   - Customer notification (timelines per severity)
   - Regulatory notification (GDPR 72-hour rule)
8. Severity Matrix
   | Severity | Description | Response Time | Update Frequency |
9. Post-Incident Review Template
10. Revision History
```

### 2.6 Vulnerability Management Policy

```
Sections:
1. Purpose
2. Scope
3. Vulnerability Sources
   - Dependency scanning (npm audit, CI pipeline)
   - SAST/DAST tools
   - Penetration testing schedule
   - Bug bounty / responsible disclosure
   - Third-party advisories
4. Severity Classification
   | Severity | CVSS Range | SLA to Patch |
   | Critical | 9.0-10.0 | 24 hours |
   | High | 7.0-8.9 | 7 days |
   | Medium | 4.0-6.9 | 30 days |
   | Low | 0.1-3.9 | 90 days |
5. Remediation Process
   - Triage
   - Assignment
   - Fix development
   - Testing
   - Deployment
   - Verification
6. Dependency Management
   - Automated scanning in CI (from CI config)
   - Update cadence
   - Lock file management
7. Responsible Disclosure Policy
8. Metrics and Reporting
9. Revision History
```

### 2.7 Access Control Policy

```
Sections:
1. Purpose
2. Scope
3. Access Control Model
   - RBAC overview
   - Role definitions (from auth scan)
   - Permission matrix
4. Authentication Requirements
   - Methods supported
   - Password policy
   - Session management
   - MFA status
5. Authorization
   - Row Level Security policies (list actual RLS policies found)
   - API route protection
   - Multi-tenancy isolation (space_id enforcement)
6. Administrative Access
   - Database access controls
   - Production environment access
   - Principle of least privilege
7. Access Reviews
   - Frequency
   - Process
8. Account Lifecycle
   - Provisioning
   - Modification
   - Deprovisioning / offboarding
9. Revision History
```

### 2.8 Data Retention & Deletion Policy

```
Sections:
1. Purpose
2. Scope
3. Retention Schedule
   | Data Category | Retention Period | Legal Basis | Deletion Method |
   [Populated from database schema analysis]
4. Deletion Procedures
   - User-initiated deletion (right to erasure)
   - Account deletion workflow
   - Data purge process
   - Backup data handling
5. Data Subject Rights
   - Right to access
   - Right to rectification
   - Right to erasure
   - Right to portability
   - How requests are processed
6. Exceptions
   - Legal holds
   - Regulatory requirements
7. Revision History
```

### 2.9 Change Management Policy

```
Sections:
1. Purpose
2. Scope
3. Change Categories
   - Standard (routine, pre-approved)
   - Normal (requires review)
   - Emergency (expedited process)
4. Change Process
   - Request (feature branch creation — from CLAUDE.md git workflow)
   - Review (PR review process)
   - Testing (CI pipeline — from GitHub Actions config)
   - Approval (merge requirements)
   - Deployment (Vercel deployment process)
   - Verification (post-deploy checks)
5. CI/CD Pipeline
   - Automated checks (lint, typecheck, build, security audit — from CI config)
   - Required checks before merge
   - Deployment strategy
6. Rollback Procedures
   - When to rollback
   - Rollback process (Vercel instant rollback)
   - Verification
7. Emergency Changes
   - Expedited approval process
   - Post-implementation review
8. Revision History
```

### 2.10 SLA Documentation

```
Sections:
1. Purpose
2. Scope
3. Service Level Objectives
   | Metric | Target | Measurement |
   | Availability | 99.9% | Monthly uptime |
   | Response Time | <500ms p95 | API response time |
   | Support Response | <24h | Business hours |
4. Hosting Provider SLAs
   - Vercel SLA summary and link
   - Supabase SLA summary and link
   - Upstash SLA summary and link
5. Maintenance Windows
   - Scheduled maintenance process
   - Notification timeline
6. Support Tiers
   | Priority | Description | Response Time | Resolution Time |
7. Escalation Path
8. Service Credits (if applicable)
9. Exclusions
10. Revision History
```

### 2.11 Business Continuity Plan

```
Sections:
1. Purpose
2. Scope
3. Business Impact Analysis
   | Function | RPO | RTO | Priority |
4. Continuity Strategies
   - Application layer (Vercel multi-region, edge network)
   - Database layer (Supabase redundancy, point-in-time recovery)
   - Third-party dependency failures (fallback strategies)
5. Communication Plan
   - Internal communication
   - Customer communication
   - Status page
6. Recovery Procedures
   - Scenario: Application hosting failure
   - Scenario: Database failure
   - Scenario: Third-party service outage
   - Scenario: Security breach
7. Testing
   - Test frequency
   - Test types (tabletop, simulation)
   - Last test date and results
8. Revision History
```

### 2.12 Disaster Recovery Plan

```
Sections:
1. Purpose
2. Scope
3. Recovery Objectives
   | System | RPO | RTO | Priority |
4. Backup Strategy
   - Database backups (Supabase PITR, daily snapshots)
   - Code repository (GitHub)
   - Configuration (environment variables, secrets)
   - Media/assets (if applicable)
5. Recovery Procedures
   - Database restoration steps
   - Application redeployment steps
   - DNS failover steps
   - Data verification steps
6. Recovery Team
   | Role | Responsibility | Contact |
7. Testing
   - Backup verification schedule
   - Recovery drill schedule
8. Revision History
```

### 2.13 Vendor Risk Questionnaire

```
This is a special document — it pre-fills common questionnaire formats.

Sections:
1. Purpose
2. How to Use This Document
3. Common Questions & Answers
   - Organized by category:
     - Company Information
     - Data Security
     - Access Control
     - Encryption
     - Incident Response
     - Business Continuity
     - Compliance
     - Third-Party Management
     - Infrastructure
     - Development Practices
4. SIG Lite Mapping (if applicable)
5. CAIQ Mapping (if applicable)
6. Revision History

Each answer references the source document for detailed information.
```

### 2.14 SOC 2 Readiness Assessment

```
Sections:
1. Purpose
2. Scope
3. Trust Service Criteria Assessment
   For each TSC category:
   - CC (Common Criteria / Security)
   - A (Availability)
   - PI (Processing Integrity)
   - C (Confidentiality)
   - P (Privacy)

   For each criterion:
   | Criterion | Description | Current State | Gap | Remediation |
   | Status: MET / PARTIAL / NOT MET / N/A |
4. Summary
   - Criteria Met: X/Y
   - Partial: X/Y
   - Gaps: X/Y
5. Remediation Roadmap
   | Priority | Gap | Effort | Timeline |
6. Recommendations
7. Revision History
```

### 2.15 Audit Log Documentation

```
Sections:
1. Purpose
2. Scope
3. Events Logged
   | Event Category | Events | Data Captured |
   | Authentication | Login, logout, failed attempt | User ID, IP, timestamp, result |
   | Data Access | Read, create, update, delete | User ID, resource, timestamp |
   | Admin Actions | Config changes, user management | Admin ID, action, timestamp |
   [Populated from actual logging implementation]
4. Log Storage
   - Where logs are stored
   - Retention period
   - Encryption
5. Access to Logs
   - Who can access
   - How access is granted
   - Audit of log access
6. Log Integrity
   - Tamper protection
   - Backup
7. Revision History
```

### 2.16 Backup & Recovery Procedures

```
Sections:
1. Purpose
2. Scope
3. Backup Schedule
   | System | Method | Frequency | Retention | Location |
   | Database | Supabase PITR | Continuous | 7 days | [region] |
   | Database | Daily snapshot | Daily | 30 days | [region] |
   | Code | GitHub | Every push | Indefinite | GitHub |
   | Config | Encrypted backup | Weekly | 90 days | [location] |
4. Recovery Procedures
   - Database point-in-time recovery (step by step)
   - Database snapshot restore (step by step)
   - Application rollback (Vercel)
   - Configuration restore
5. Verification
   - How backups are tested
   - Test frequency
   - Last successful test
6. Revision History
```

---

## PHASE 3: VALIDATE

After generating all selected documents:

### 3.1 Cross-Reference Check

```
For each document:
- Verify all claims trace back to scan inventory
- Check internal links between documents are valid
- Ensure consistent terminology across all documents
- Verify dates are current
- Check that company name and project name are consistent
```

### 3.2 Completeness Check

```
For each document:
- All required sections present
- No placeholder text remaining (no "[TBD]" or "[FILL IN]")
- Version and date populated
- Review schedule set
- Revision history initialized
```

### 3.3 Honesty Check

```
CRITICAL: Scan every generated document for claims about controls or capabilities
that were NOT found in the codebase scan. Flag any fabricated claims.

For each unverified claim:
- Mark as "Planned" or "Under Evaluation"
- Add a note in the report that this needs human verification
```

---

## PHASE 4: REPORT

### 4.1 Generation Summary

```
═══════════════════════════════════════════════════════════════════════════════
COMPLIANCE DOCUMENTATION GENERATION COMPLETE
═══════════════════════════════════════════════════════════════════════════════

Project: [Project Name] ([Company Name])
Date: YYYY-MM-DD
Duration: X minutes

DOCUMENTS GENERATED
─────────────────────────────────────────────────────────────────────────────

| # | Document | Status | Location |
|---|----------|--------|----------|
| 1 | Security Whitepaper | GENERATED | .compliance-docs-reports/documents/security-whitepaper.md |
| 2 | Subprocessor List | GENERATED | .compliance-docs-reports/documents/subprocessor-list.md |
| ... | ... | ... | ... |

SCAN RESULTS
─────────────────────────────────────────────────────────────────────────────

Third-party services detected: X
RLS-protected tables: X
Security controls verified: X
Data flows mapped: X

ITEMS NEEDING HUMAN REVIEW
─────────────────────────────────────────────────────────────────────────────

| # | Document | Section | Issue |
|---|----------|---------|-------|
| 1 | SLA Documentation | Service Level Objectives | Confirm uptime target |
| 2 | Incident Response Plan | Contact Information | Add real contact details |

RECOMMENDATIONS
─────────────────────────────────────────────────────────────────────────────

- Review all generated documents before sharing with customers
- Fill in contact information and real names in Incident Response Plan
- Confirm SLA targets with business stakeholders
- Schedule quarterly review of all compliance documents
- Consider SOC 2 Type II audit once all gaps are remediated

═══════════════════════════════════════════════════════════════════════════════
```

### 4.2 Output Files

**Gitignored (working files):**

| File | Purpose |
|------|---------|
| `.compliance-docs-reports/generation-YYYYMMDD-HHMMSS.md` | Generation report (living document) |
| `.compliance-docs-reports/state-YYYYMMDD-HHMMSS.json` | Checkpoint for resumption |
| `.compliance-docs-reports/inventory.json` | Detected services, controls, data flows |

**All output is gitignored.** Documents are confidential and shared with enterprise prospects on request — never committed to the repository.

| Directory | Purpose |
|-----------|---------|
| `.compliance-docs-reports/documents/` | All generated compliance documents (send to prospects as needed) |

---

## Execution Checklist

```
[ ] Phase 0: Codebase Security Scan
    [ ] Project identity detected
    [ ] Third-party services inventoried
    [ ] Auth & authorization scanned
    [ ] Data flows mapped
    [ ] Security controls cataloged
    [ ] Inventory file written

[ ] Phase 1: Document Selection
    [ ] Scan summary presented
    [ ] Document menu shown (or flag processed)
    [ ] Existing documents checked for staleness
    [ ] Generation queue finalized

[ ] Phase 2: Document Generation
    [ ] Each selected document generated
    [ ] All sections populated from scan data
    [ ] Cross-references between documents added
    [ ] Version and dates set

[ ] Phase 3: Validation
    [ ] Cross-reference check passed
    [ ] Completeness check passed
    [ ] Honesty check passed (no fabricated claims)

[ ] Phase 4: Report
    [ ] Generation summary output
    [ ] Human review items listed
    [ ] Recommendations provided
    [ ] Report saved to .compliance-docs-reports/
```

---

## Rollback Procedure

```bash
echo "Rolling back generated compliance documents..."
rm -rf .compliance-docs-reports/documents/
echo "Compliance documents removed"
```

---

## Important Notes

- **NEVER commit these documents** — all output stays in `.compliance-docs-reports/` (gitignored). These are confidential documents shared with enterprise prospects upon request, not public repo content
- **Run quarterly** to keep documents current with codebase changes
- **Always review** generated documents before sharing with customers — AI cannot know business decisions (SLA targets, support hours, contact info)
- **Legal review recommended** for Privacy Policy and Terms of Service before publication
- **Share as PDF** — when sending to prospects, export markdown to PDF for a professional presentation
- **Complements /compliance** — this skill generates documents, /compliance audits code for violations
- **Complements /sec-ship** — this skill documents security posture, /sec-ship finds and fixes vulnerabilities

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### Compliance-Docs-Specific Cleanup

Cleanup actions:
1. **Gitignore enforcement:** Ensure `.compliance-docs-reports/` is in `.gitignore` — generated documents are confidential
2. **No other resources created** — this skill generates documentation files only

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
