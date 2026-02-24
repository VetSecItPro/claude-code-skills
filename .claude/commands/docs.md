# /docs — Documentation Generation, Maintenance & WHY Documentation

**Comprehensive documentation audit, generation, and maintenance. Focus on the WHY, not just the WHAT.**

**FIRE AND FORGET** — Execute the entire pipeline without waiting for user input. Status updates every 5 minutes. Generate documentation that explains intent, not just implementation.

---

## STATUS UPDATES

This skill follows the **[Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md)**.

See standard for complete format. Skill-specific status examples are provided throughout the execution phases below.

---

## CONTEXT MANAGEMENT

This skill follows the **[Context Management Protocol](~/.claude/standards/CONTEXT_MANAGEMENT.md)**.

Key rules for this skill:
- Documentation audit agents return < 500 tokens each (full coverage analysis written to .docs-reports/)
- State file `.docs-reports/state-YYYYMMDD-HHMMSS.json` tracks which doc domains are complete
- Resume from checkpoint if context resets — skip completed doc audits
- Max 2 parallel agents (pair independent doc types: e.g., README + API docs)
- Orchestrator messages stay lean: "Agent 3/5 complete: Component docs 45% coverage, 12 undocumented"

---

## AGENT ORCHESTRATION

This skill follows the **[Agent Orchestration Protocol](~/.claude/standards/AGENT_ORCHESTRATION.md)**.

The orchestrator coordinates agents but NEVER scans or generates documentation directly. All heavy work is delegated to focused agents that write results to disk.

### Model Selection for This Skill

| Agent Type | Model | Why |
|-----------|-------|-----|
| File inventory / listing | `haiku` | Counting files, listing routes, checking file existence — no judgment |
| Link validator | `haiku` | Checking if URLs resolve — no judgment |
| README analyzer | `sonnet` | Must assess completeness, detect stale sections, understand project context |
| API documentation generator | `sonnet` | Must understand route logic, request/response types, error handling to document accurately |
| Component documentation generator | `sonnet` | Must understand props, usage patterns, when to use vs alternatives |
| Architecture documentation generator | `opus` | Must synthesize system-wide understanding into accurate architecture docs — high-stakes, often shared externally |
| ADR generator | `opus` | Must infer architectural decisions from code patterns and explain the WHY — requires deep reasoning |
| Inline JSDoc generator | `sonnet` | Must understand function purpose, parameters, and side effects to write meaningful docs |
| Staleness detector | `sonnet` | Must compare doc claims against current code behavior |
| CHANGELOG generator | `sonnet` | Must parse git history and categorize changes meaningfully |
| Report synthesizer | `sonnet` | Must compile coverage stats and recommendations |

### Agent Batching

- Inventory agents can run 2 in parallel (read-only)
- Documentation generators run SEQUENTIALLY (each may cross-reference others)
- Batch components into groups of 10-15 per generator agent
- API routes batched by directory (auth/, users/, etc.)
- JSDoc generation batched 5-10 files per agent

---

## Philosophy: Document the WHY

> "The code shows you WHAT it does. Documentation should explain WHY it does it."

**Bad documentation:**
```typescript
// Increment counter by 1
counter++;
```

**Good documentation:**
```typescript
// Track consecutive failed login attempts for rate limiting.
// After 5 failures, we lock the account for 15 minutes to prevent brute force.
counter++;
```

This skill generates documentation that captures **intent, decisions, trade-offs, and context** — things that can't be inferred from code.

---

## Execution Rules (CRITICAL)

- **NO permission requests** — just execute
- **NO "should I proceed?" questions** — just do it
- **NO waiting for user confirmation** — work continuously
- **Status updates every 5 minutes** — output progress without waiting for response
- **Focus on WHY** — every generated doc must explain reasoning, not just describe code
- **Self-healing** — if generation fails, try alternative approach

---

## Usage

```bash
/docs                         # Full documentation audit + generation
/docs --audit-only            # Just report gaps, don't generate
/docs --readme                # Update README only
/docs --api                   # Generate API documentation only
/docs --components            # Generate component documentation only
/docs --changelog             # Generate/update CHANGELOG only
/docs --architecture          # Generate architecture docs only
/docs --decisions             # Generate ADRs (Architecture Decision Records)
/docs --file=<path>           # Document specific file
```

---

## Output Files (All Gitignored except final docs)

**Gitignored (working files):**
| File | Purpose |
|------|---------|
| `.docs-reports/docs-YYYY-MM-DD-HHMMSS.md` | Audit report |
| `.docs-audit.json` | Machine-readable findings |
| `.docs-reports/coverage.json` | Documentation coverage |

**Generated documentation (gitignored — review before committing anything):**
| File | Purpose |
|------|---------|
| `.docs-reports/documents/README.md` | Project overview (draft — copy to root README.md if approved) |
| `.docs-reports/documents/CHANGELOG.md` | Version history (draft — copy to root if approved) |
| `.docs-reports/documents/api/` | API documentation |
| `.docs-reports/documents/architecture/` | Architecture docs |
| `.docs-reports/documents/components/` | Component documentation |
| `.docs-reports/documents/decisions/` | ADRs |

**NOTE:** All generated docs go to `.docs-reports/documents/` first. Review and manually copy to the appropriate location if you want to commit them. Never auto-commit generated documentation.

---

## Report Persistence (CRITICAL — Living Document)

The markdown report file is a **LIVING DOCUMENT**. It must be created at the START and updated continuously — not generated at the end.

### Living Document Protocol

1. **Phase 0**: Create the full report skeleton at `.docs-reports/docs-YYYYMMDD-HHMMSS.md` with `⏳ Pending` placeholders for all sections (coverage analysis, missing docs, generated docs, validation)
2. **After each doc generated or gap found**: Fill in completed sections with real results immediately. Write to disk.
3. **If context dies**: User can open the `.md` and see exactly what docs were audited, what was generated, and what's pending

### Finding Statuses

| Status | Meaning |
|--------|---------|
| `FOUND` | Documentation gap identified |
| `🔧 WRITING` | Documentation being generated |
| `✅ FIXED` | Documentation written and committed |
| `⏸️ DEFERRED` | Needs human knowledge (tribal knowledge, business context). Reason documented. |
| `🚫 BLOCKED` | Unable to generate accurate docs without human input. Reason documented. |

### Deferred Items Table

Every gap that can't be auto-documented gets a row:

| # | ID | Type | Gap | Why Deferred | What Needs to Happen | Status |
|---|-----|------|-----|-------------|----------------------|--------|
| 1 | DOC-XXX | [API/arch/ADR] | [missing doc] | [needs tribal knowledge / business context / human review] | [specific action] | ⏳ PENDING |

When working on deferred items, update Status in real-time on disk:
`⏳ PENDING` → `🔧 IN PROGRESS` → `✅ FIXED` or `🚫 BLOCKED`

### Rules

1. **Write report skeleton at Phase 0** — file exists before any scanning
2. **Update after each doc generated or gap found** — status changes in real-time
3. **Write to disk after every status change** — if session dies, report shows where things stand
4. **SITREP section** — for every DEFERRED or BLOCKED item, document why it couldn't be auto-generated
5. **NEVER delete findings** — only update their status

---

## Architecture

```
═══════════════════════════════════════════════════════════════════════════════
                              /docs PIPELINE
═══════════════════════════════════════════════════════════════════════════════

PHASE 0: SETUP
├── Scan codebase structure
├── Identify existing documentation
├── Analyze git history for context
└── Create output directories

PHASE 1: AUDIT (5 parallel agents)
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   README    │ │     API     │ │ COMPONENTS  │ │   INLINE    │ │ ARCHITECTURE│
│  Agent 1    │ │  Agent 2    │ │  Agent 3    │ │  Agent 4    │ │  Agent 5    │
└──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
       │               │               │               │               │
       └───────────────┴───────────────┴───────┬───────┴───────────────┘
                                               │
                                        [MERGE FINDINGS]
                                        [STATUS UPDATE]

PHASE 2: GENERATE
├── README updates
├── API documentation
├── Component documentation
├── CHANGELOG from commits
├── Architecture diagrams
├── ADRs from code patterns
└── Inline JSDoc comments

PHASE 3: VALIDATE
├── Check all links work
├── Verify examples compile
├── Ensure no stale docs
└── Cross-reference accuracy

PHASE 4: REPORT
├── Coverage statistics
├── Generated files list
├── Remaining gaps
├── Recommendations
└── Save reports

═══════════════════════════════════════════════════════════════════════════════
```

---

## PHASE 0: SETUP

### 0.1 Scan Codebase Structure

```bash
# Get project structure
PROJECT_NAME=$(jq -r '.name' package.json)
PROJECT_DESC=$(jq -r '.description' package.json)

# Count files by type
API_ROUTES=$(find app/api -name "route.ts" 2>/dev/null | wc -l)
PAGES=$(find app -name "page.tsx" 2>/dev/null | wc -l)
COMPONENTS=$(find components -name "*.tsx" 2>/dev/null | wc -l)
HOOKS=$(find hooks lib/hooks -name "*.ts" 2>/dev/null | wc -l)
UTILS=$(find lib -name "*.ts" 2>/dev/null | wc -l)

echo "📊 Codebase: $API_ROUTES routes, $PAGES pages, $COMPONENTS components"
```

### 0.2 Analyze Git History

```bash
# Get commit history for CHANGELOG
git log --oneline --since="6 months ago" > .docs-reports/git-history.txt

# Get contributors
git shortlog -sne --since="1 year ago" > .docs-reports/contributors.txt

# Get file change frequency (identifies important files)
git log --pretty=format: --name-only --since="3 months ago" | sort | uniq -c | sort -rn | head -50 > .docs-reports/hot-files.txt
```

### 0.3 Identify Existing Documentation

```bash
# Check what docs exist
[ -f "README.md" ] && README_EXISTS=true || README_EXISTS=false
[ -f "CHANGELOG.md" ] && CHANGELOG_EXISTS=true || CHANGELOG_EXISTS=false
[ -f "CONTRIBUTING.md" ] && CONTRIBUTING_EXISTS=true || CONTRIBUTING_EXISTS=false
[ -d "docs" ] && DOCS_DIR_EXISTS=true || DOCS_DIR_EXISTS=false

# Check for inline documentation
JSDOC_COUNT=$(grep -r "@param\|@returns\|@example" --include="*.ts" --include="*.tsx" . 2>/dev/null | wc -l)
```

---

## PHASE 1: AUDIT (5 Parallel Agents)

### AGENT 1: README Audit

> **Return ONLY a structured summary under 500 tokens. Full analysis goes to disk.**

**Check README.md for:**

| Section | Required | Status |
|---------|----------|--------|
| Project name & description | ✅ | |
| Installation instructions | ✅ | |
| Quick start / Usage | ✅ | |
| Environment variables | ✅ | |
| Tech stack | ✅ | |
| Project structure | ⚠️ | |
| Contributing guidelines | ⚠️ | |
| License | ✅ | |
| Links (docs, demo, issues) | ⚠️ | |

**Check for staleness:**
- Last README update vs last code change
- Referenced files that no longer exist
- Commands that no longer work
- Screenshots that are outdated

**Findings format:**
```javascript
{
  "id": "DOC-README-001",
  "severity": "high",
  "issue": "Missing environment variables section",
  "recommendation": "Add .env.example documentation with all required variables"
}
```

---

### AGENT 2: API Documentation Audit

> **Return ONLY a structured summary under 500 tokens. Full analysis goes to disk.**

**For each API route (`app/api/**/route.ts`):**

| Check | What to Document |
|-------|------------------|
| Endpoint | Full URL path |
| Methods | GET, POST, PUT, DELETE |
| Authentication | Required? What type? |
| Request body | Schema with types |
| Response | Schema with types |
| Error codes | 400, 401, 403, 404, 500 |
| Rate limiting | Limits and windows |
| Examples | curl/fetch examples |
| **WHY** | Purpose and use cases |

**Missing docs detection:**
```bash
# Find routes without documentation
for route in $(find app/api -name "route.ts"); do
  # Check if corresponding doc exists
  doc_path="docs/api/$(dirname $route | sed 's|app/api/||').md"
  if [ ! -f "$doc_path" ]; then
    echo "Missing: $route"
  fi
done
```

**Generate from TypeScript:**
```typescript
// Extract types from route
export async function POST(req: NextRequest) {
  const body: CreateUserRequest = await req.json()
  // ...
  return Response.json(user satisfies UserResponse)
}

// Generate documentation:
// POST /api/users
// Request: CreateUserRequest { name: string, email: string }
// Response: UserResponse { id: string, name: string, email: string }
```

---

### AGENT 3: Component Documentation Audit

> **Return ONLY a structured summary under 500 tokens. Full analysis goes to disk.**

**For each component:**

| Check | What to Document |
|-------|------------------|
| Props | All props with types and defaults |
| Usage | How to use the component |
| Examples | Code snippets |
| Variants | Different states/modes |
| Accessibility | ARIA, keyboard nav |
| **WHY** | When to use vs alternatives |

**Detect undocumented components:**
```bash
# Components without JSDoc
for comp in $(find components -name "*.tsx"); do
  if ! grep -q "@description\|@example\|Props" "$comp"; then
    echo "Undocumented: $comp"
  fi
done
```

**Generate from props:**
```typescript
interface ButtonProps {
  /** The visual style variant */
  variant?: 'primary' | 'secondary' | 'ghost'
  /** Button size */
  size?: 'sm' | 'md' | 'lg'
  /** Disable the button */
  disabled?: boolean
  /** Click handler */
  onClick?: () => void
  /** Button content */
  children: React.ReactNode
}

// Generate:
// ## Button
// A clickable button component with multiple variants.
//
// ### Props
// | Prop | Type | Default | Description |
// |------|------|---------|-------------|
// | variant | 'primary' \| 'secondary' \| 'ghost' | 'primary' | The visual style |
// ...
```

---

### AGENT 4: Inline Documentation Audit

> **Return ONLY a structured summary under 500 tokens. Full analysis goes to disk.**

**Check for missing JSDoc on:**

| Export Type | Required Docs |
|-------------|---------------|
| Functions | @description, @param, @returns, @example |
| Classes | @description, constructor params |
| Types/Interfaces | @description for complex types |
| Constants | @description if not self-explanatory |

**Focus on the WHY:**

```typescript
// ❌ Bad - describes what (obvious from code)
/** Returns the user by ID */
function getUserById(id: string) { ... }

// ✅ Good - explains why and context
/**
 * Fetches user from cache first, falling back to database.
 * Cache is used because user data is accessed ~100x per page load
 * and rarely changes (updated via webhook on profile save).
 *
 * @param id - User ID from Supabase auth
 * @returns User object or null if not found
 * @throws {AuthError} If user ID is from a deleted account
 *
 * @example
 * const user = await getUserById(session.user.id)
 * if (!user) redirect('/onboarding')
 */
function getUserById(id: string) { ... }
```

**Scan for missing docs:**
```bash
# Exported functions without JSDoc
grep -rn "^export function\|^export async function\|^export const" --include="*.ts" . | \
  while read line; do
    file=$(echo $line | cut -d: -f1)
    linenum=$(echo $line | cut -d: -f2)
    # Check if previous lines have JSDoc
    prevline=$((linenum - 1))
    if ! sed -n "${prevline}p" "$file" | grep -q "\*/"; then
      echo "Missing JSDoc: $line"
    fi
  done
```

---

### AGENT 5: Architecture Documentation Audit

> **Return ONLY a structured summary under 500 tokens. Full analysis goes to disk.**

**Check for:**

| Document | Purpose |
|----------|---------|
| System overview | High-level architecture diagram |
| Data flow | How data moves through the system |
| Database schema | Tables, relationships, RLS |
| Authentication flow | Login, session, refresh |
| Deployment | How the app is deployed |
| ADRs | Why key decisions were made |

**Generate architecture from code:**
```bash
# Import graph (who imports who)
npx madge --image .docs-reports/dependency-graph.svg .

# Database schema from types
cat types/database.types.ts | extract_schema > docs/architecture/database.md
```

**ADR (Architecture Decision Record) detection:**
- Look for patterns that indicate decisions:
  - `// We chose X over Y because...`
  - `// TODO: consider switching to...`
  - Unusual patterns that need explanation
  - CLAUDE.md rules that aren't obvious

---

### Status Update: Every 5 Minutes

```
═══════════════════════════════════════════════════════════════
⏳ DOCUMENTATION AUDIT IN PROGRESS — 5:00 elapsed
═══════════════════════════════════════════════════════════════
Agent 1 (README):      ████████████ 100% — 3 sections missing
Agent 2 (API):         ████████░░░░ 75% — 20/26 routes checked
Agent 3 (Components):  ██████░░░░░░ 60% — 45/78 components checked
Agent 4 (Inline):      ████░░░░░░░░ 40% — Scanning exports...
Agent 5 (Architecture): ████████████ 100% — No existing docs

Documentation coverage: 34%
Gaps found: 45
═══════════════════════════════════════════════════════════════
```

---

## PHASE 2: GENERATE

### 2.1 README Generation/Update

**If README doesn't exist, generate:**
```markdown
# {Project Name}

{Description from package.json}

## Why This Exists

{Inferred from CLAUDE.md and codebase analysis}

## Quick Start

```bash
# Install dependencies
{package manager} install

# Set up environment
cp .env.example .env.local
# Edit .env.local with your values

# Run development server
{package manager} run dev
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
{Generated from .env.example}

## Tech Stack

{Detected from package.json}

## Project Structure

```
{Generated from file tree}
```

## Documentation

- [API Documentation](docs/api/)
- [Component Documentation](docs/components/)
- [Architecture](docs/architecture/)

## Contributing

{Link to CONTRIBUTING.md or brief guidelines}

## License

{From package.json}
```

**If README exists, patch missing sections.**

---

### 2.2 API Documentation Generation

**For each route without docs:**

```markdown
# POST /api/users

Create a new user account.

## Why This Endpoint Exists

This endpoint handles user registration. We separate registration from OAuth
because some users prefer email/password, and we need to support both flows
while maintaining a single user record.

## Authentication

None required (public endpoint for registration).

## Request

```typescript
interface CreateUserRequest {
  email: string      // Must be unique, validated format
  password: string   // Min 8 chars, requires number and special char
  name?: string      // Display name (optional at registration)
}
```

## Response

### Success (201 Created)

```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": null,
  "created_at": "2024-01-01T00:00:00Z"
}
```

### Errors

| Code | Reason |
|------|--------|
| 400 | Invalid email format or weak password |
| 409 | Email already registered |
| 429 | Rate limited (max 5 registrations/hour/IP) |

## Example

```bash
curl -X POST https://api.example.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "SecurePass123!"}'
```

## Related

- [POST /api/auth/login](./auth/login.md) - Login after registration
- [POST /api/auth/verify](./auth/verify.md) - Email verification
```

---

### 2.3 Component Documentation Generation

**For each component without docs:**

```markdown
# Button

A versatile button component supporting multiple variants and sizes.

## Why Use This Component

Use `<Button>` instead of native `<button>` because:
- Consistent styling across the app
- Built-in loading state handling
- Automatic focus ring for accessibility
- Proper disabled state styling

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| variant | `'primary' \| 'secondary' \| 'ghost' \| 'destructive'` | `'primary'` | Visual style |
| size | `'sm' \| 'md' \| 'lg'` | `'md'` | Button size |
| disabled | `boolean` | `false` | Disable interactions |
| loading | `boolean` | `false` | Show loading spinner |
| children | `ReactNode` | required | Button content |

## Usage

```tsx
import { Button } from '@/components/ui/button'

// Primary action
<Button onClick={handleSave}>Save Changes</Button>

// Destructive action with confirmation
<Button variant="destructive" onClick={handleDelete}>
  Delete Account
</Button>

// Loading state
<Button loading={isSubmitting}>
  {isSubmitting ? 'Saving...' : 'Save'}
</Button>
```

## Variants

### Primary
Use for main actions. One primary button per view.

### Secondary
Use for alternative actions alongside primary.

### Ghost
Use for tertiary actions or in toolbars.

### Destructive
Use for dangerous actions (delete, remove). Always confirm first.

## Accessibility

- Uses native `<button>` element
- Focus ring visible on keyboard navigation
- `aria-disabled` applied when disabled
- Loading state announces to screen readers
```

---

### 2.4 CHANGELOG Generation

**Parse git history with conventional commits:**

```bash
# Get commits since last tag
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
git log $LAST_TAG..HEAD --pretty=format:"%s|%h|%an|%ad" --date=short
```

**Generate CHANGELOG.md:**

```markdown
# Changelog

All notable changes to this project are documented here.

## [Unreleased]

### Added
- feat(auth): Add Google OAuth login (#123)
- feat(api): Add bulk import endpoint (#125)

### Changed
- refactor(db): Move to Clarus schema (#120)

### Fixed
- fix(ui): Button alignment on mobile (#124)
- fix(api): Rate limiting not resetting (#126)

### Security
- security: Update lodash to fix prototype pollution (#127)

## [1.2.0] - 2024-02-01

### Added
- feat(export): PDF export with custom styling
- feat(chat): Streaming AI responses

### Changed
- perf(bundle): Reduce bundle size by 45kB

...
```

---

### 2.5 Architecture Documentation Generation

**Generate from codebase analysis:**

```markdown
# Architecture Overview

## System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        Client                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  React  │  │  SWR    │  │ Supabase│  │  Zustand│        │
│  │   UI    │  │ Caching │  │  Auth   │  │  State  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
└───────┼────────────┼────────────┼────────────┼──────────────┘
        │            │            │            │
        └────────────┴─────┬──────┴────────────┘
                           │
┌──────────────────────────┼──────────────────────────────────┐
│                    Next.js API                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  Auth   │  │ Process │  │  Chat   │  │ Export  │        │
│  │ Routes  │  │ Content │  │ Routes  │  │ Routes  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
└───────┼────────────┼────────────┼────────────┼──────────────┘
        │            │            │            │
┌───────┼────────────┼────────────┼────────────┼──────────────┐
│       │       External Services │            │               │
│  ┌────┴────┐  ┌────┴────┐  ┌───┴────┐  ┌───┴────┐         │
│  │Supabase │  │OpenRouter│  │Firecrawl│  │ Polar  │         │
│  │   DB    │  │   AI    │  │ Scrape  │  │Payment │         │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## Why This Architecture

### Next.js App Router
We chose Next.js 15 with App Router because:
- Server Components reduce client bundle size
- API routes colocated with pages
- Built-in caching and ISR
- Vercel deployment is seamless

### Supabase over Firebase
We chose Supabase because:
- PostgreSQL gives us relational data + RLS
- Row Level Security handles auth at DB level
- Open source, can self-host if needed
- Better pricing for our usage pattern

### OpenRouter over direct OpenAI
We use OpenRouter because:
- Single API for multiple models (Gemini, Claude, GPT)
- Easy model switching without code changes
- Built-in fallbacks if one provider is down

## Data Flow

### Content Analysis Flow
1. User submits URL
2. Firecrawl scrapes content (or Supadata for YouTube)
3. Content stored in `clarus.content` table
4. OpenRouter generates 6 analysis sections
5. Results cached in `clarus.summaries`
6. User sees analysis in UI

### Authentication Flow
1. User clicks "Sign in with Google"
2. Supabase handles OAuth redirect
3. Callback creates/updates user in `clarus.users`
4. Session stored in httpOnly cookie
5. API routes check session via `getUser()`
```

---

### 2.6 ADR (Architecture Decision Record) Generation

**Detect decisions from code patterns:**

```markdown
# ADR-001: Use Clarus Schema Instead of Public

## Status
Accepted

## Context
This Supabase project is shared with other applications (Rowan, Steel Motion).
We needed to isolate Clarus tables to prevent accidental cross-contamination.

## Decision
All Clarus tables live in the `clarus` schema, not `public`.
The database `search_path` is configured to use `clarus` by default.

## Consequences

### Positive
- Complete isolation from other apps
- Can't accidentally query wrong tables
- Clear ownership of tables

### Negative
- Need to remember schema when writing raw SQL
- Supabase Studio shows all schemas (visual clutter)

## Code Reference
- `CLAUDE.md` line 45-80: Schema isolation rules
- `scripts/100-create-clarus-schema.sql`: Schema setup
```

---

### 2.7 Inline JSDoc Generation

**Add JSDoc to undocumented exports:**

```typescript
// BEFORE
export function processContent(options: ProcessContentOptions): Promise<ProcessResult> {
  // ...
}

// AFTER
/**
 * Process and analyze content from a URL.
 *
 * This is the core analysis pipeline. It:
 * 1. Validates the URL and checks tier limits
 * 2. Scrapes content via Firecrawl (articles) or Supadata (YouTube)
 * 3. Generates 6 analysis sections via OpenRouter
 * 4. Caches results to avoid re-processing
 *
 * We process in chunks because:
 * - Long content exceeds model context limits
 * - Chunking allows parallel processing
 * - Partial results can be shown during processing
 *
 * @param options - Processing configuration
 * @param options.contentId - UUID of the content record
 * @param options.userId - User requesting analysis (for tier limits)
 * @param options.language - Output language (default: 'en')
 * @param options.forceRegenerate - Skip cache and regenerate
 *
 * @returns Processing result with generated sections
 *
 * @throws {ProcessContentError} If content can't be processed
 * @throws {TierLimitError} If user has exceeded their quota
 *
 * @example
 * const result = await processContent({
 *   contentId: 'abc-123',
 *   userId: session.user.id,
 *   language: 'en'
 * })
 */
export function processContent(options: ProcessContentOptions): Promise<ProcessResult> {
  // ...
}
```

---

## PHASE 3: VALIDATE

### 3.1 Link Validation

```bash
# Find all markdown links
grep -roh "\[.*\](.*)" docs/ README.md | while read link; do
  url=$(echo $link | sed 's/.*(\(.*\))/\1/')
  if [[ $url == http* ]]; then
    # External link - check if reachable
    curl -s -o /dev/null -w "%{http_code}" "$url" | grep -q "200\|301\|302" || echo "Broken: $url"
  else
    # Internal link - check if file exists
    [ -f "$url" ] || echo "Broken: $url"
  fi
done
```

### 3.2 Example Validation

```bash
# Extract code examples and try to compile
grep -Pzo '```typescript\n\K[^`]+' docs/**/*.md | while read example; do
  echo "$example" > /tmp/example.ts
  npx tsc --noEmit /tmp/example.ts 2>&1 || echo "Invalid example"
done
```

### 3.3 Staleness Detection

```bash
# Find docs older than their source files
for doc in docs/**/*.md; do
  source=$(echo $doc | sed 's|docs/||' | sed 's|\.md$|.ts|')
  if [ -f "$source" ]; then
    if [ "$source" -nt "$doc" ]; then
      echo "Stale: $doc (source updated since doc)"
    fi
  fi
done
```

---

## PHASE 4: REPORT

### Documentation Coverage

```
═══════════════════════════════════════════════════════════════
📊 DOCUMENTATION COVERAGE
═══════════════════════════════════════════════════════════════

                        BEFORE          AFTER           DELTA
─────────────────────────────────────────────────────────────
README Completeness     45%             95%             +50% ✅
API Routes Documented   8/26 (31%)      26/26 (100%)    +18 ✅
Components Documented   12/78 (15%)     78/78 (100%)    +66 ✅
Functions with JSDoc    23/145 (16%)    120/145 (83%)   +97 ✅
Architecture Docs       0               5 files         +5 ✅
ADRs                    0               3 records       +3 ✅
─────────────────────────────────────────────────────────────

Overall Coverage: 34% → 89% (+55%)
═══════════════════════════════════════════════════════════════
```

### Generated Files

```
═══════════════════════════════════════════════════════════════
📁 GENERATED DOCUMENTATION
═══════════════════════════════════════════════════════════════

README.md                           (updated)
CHANGELOG.md                        (generated)
docs/
├── api/
│   ├── auth/
│   │   ├── login.md
│   │   ├── logout.md
│   │   └── verify.md
│   ├── content/
│   │   ├── process.md
│   │   └── [id].md
│   └── ...
├── components/
│   ├── button.md
│   ├── card.md
│   └── ...
├── architecture/
│   ├── overview.md
│   ├── data-flow.md
│   └── database.md
└── decisions/
    ├── ADR-001-clarus-schema.md
    ├── ADR-002-openrouter.md
    └── ADR-003-tier-limits.md

Total: 45 files generated/updated
═══════════════════════════════════════════════════════════════
```

### Final Summary

```
═══════════════════════════════════════════════════════════════════════════════
🎉 /docs COMPLETE
═══════════════════════════════════════════════════════════════════════════════

⏱️ Total Duration: 12 minutes 18 seconds

📊 RESULTS SUMMARY
─────────────────────────────────────────────────────────────────────────────

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Documentation Coverage | 34% | 89% | +55% ✅ |
| README Sections | 4/9 | 9/9 | +5 ✅ |
| API Docs | 8 | 26 | +18 ✅ |
| Component Docs | 12 | 78 | +66 ✅ |
| JSDoc Comments | 23 | 120 | +97 ✅ |
| Architecture Docs | 0 | 5 | +5 ✅ |
| ADRs | 0 | 3 | +3 ✅ |

📁 FILES
   • Generated: 45 documentation files
   • Updated: README.md, CHANGELOG.md
   • Report: .docs-reports/docs-2026-02-06-001500.md

✅ VALIDATION
   • All links valid
   • All examples compile
   • No stale documentation detected

💡 RECOMMENDATIONS
   • Consider adding screenshots to component docs
   • Add sequence diagrams for complex flows
   • Schedule quarterly doc review

═══════════════════════════════════════════════════════════════════════════════
```

---

## Rollback Procedure

```bash
echo "🔄 Rolling back documentation changes..."
git checkout -- README.md CHANGELOG.md docs/
echo "✅ Documentation reverted"
```

---

## Important Notes

- **Focus on WHY** — every generated doc explains reasoning, not just description
- **Validates everything** — links, examples, freshness
- **Learns from codebase** — matches existing patterns and conventions
- **Commits real docs** — working files gitignored, final docs committed
- Run monthly to keep docs fresh
- Run before onboarding new team members

## CLEANUP PROTOCOL

> Reference: [Resource Cleanup Protocol](~/.claude/standards/CLEANUP_PROTOCOL.md)

### Docs-Specific Cleanup

Cleanup actions:
1. **Temp file:** Delete `/tmp/example.ts` if created during validation
2. **Orphan file:** Delete `.docs-audit.json` from project root after data is captured in `.docs-reports/`
3. **Gitignore enforcement:** Ensure `.docs-reports/` is in `.gitignore`

<!-- Claude Code Skill by Steel Motion LLC — https://steelmotion.dev -->
<!-- Part of the Claude Code Skills Collection -->
<!-- Powered by Claude models: Haiku (fast extraction), Sonnet (balanced reasoning), Opus (deep analysis) -->
<!-- License: MIT -->
