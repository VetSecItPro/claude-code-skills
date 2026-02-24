# Status Update Protocol for Claude Skills

**Version:** 1.0
**Applies to:** All skills in `~/.claude/commands/`

---

## Purpose

Skills should communicate progress in real-time, never leaving users wondering what's happening. This protocol defines standards for status updates across all skills.

## Core Principles

1. **Frequent Updates** - Report progress every 10-30 seconds
2. **Clear Phases** - Mark beginning and end of each phase
3. **Visual Indicators** - Use emojis for quick scanning
4. **Actionable Info** - Tell user what's happening and why
5. **Never Silent** - Long-running tasks need progress updates
6. **Consistent Format** - Use standard patterns across all skills

---

## Standard Format

### Phase Updates

```markdown
🚀 [PHASE] Starting...
   ├─ [STEP] Action being performed
   ├─ [STEP] Action being performed
   └─ ✅ [PHASE] Complete
```

### Results Summary

```markdown
📊 [RESULTS] Summary of findings
   • Key metric 1
   • Key metric 2
   • Key metric 3
```

### Warnings

```markdown
⚠️ [WARNING] Issue detected
   → Suggested action
```

### Errors

```markdown
❌ [ERROR] Problem encountered
   → How to fix
```

### Information

```markdown
ℹ️ [INFO] Additional context
   → Helpful detail
```

---

## Emoji Reference

Use these emojis consistently across all skills:

| Emoji | Meaning | Usage |
|-------|---------|-------|
| 🚀 | Starting | Phase initiation |
| ✅ | Success | Phase completion, task done |
| 📊 | Results | Summary, metrics, findings |
| 🔍 | Scanning | Detection, search, analysis |
| 🔒 | Security | Security checks, auth, RLS |
| 🎨 | Design | UI/UX, styling, theming |
| 📁 | Files | File operations, directory creation |
| 🔨 | Building | Implementation, construction |
| ⚡ | Performance | Speed, optimization |
| 📅 | Time | Dates, scheduling, temporal |
| 💾 | Data | Export, storage, database |
| 🚨 | Alerts | Alert system, notifications |
| 📋 | Tasks | Task creation, management |
| 🤖 | Agents | Agent deployment, automation |
| 🔄 | Progress | Long-running operation |
| ❌ | Error | Failure, problem |
| ⚠️ | Warning | Non-critical issue |
| ℹ️ | Info | Context, explanation |
| 🔧 | Fix | Repair, correction |
| 📈 | Metrics | Analytics, tracking |
| 🧪 | Testing | Test execution, validation |
| 🛡️ | Protection | Hardening, safeguards |
| 🧹 | Cleanup | Code cleanup, removal |
| 📝 | Documentation | Docs, comments, guides |
| 🎯 | Target | Goals, objectives |
| 💡 | Insight | Discovery, realization |

---

## Required Updates by Phase

### 1. Initialization

```markdown
🚀 [Skill Name] Started
   Project: [project-name]
   Mode: [mode-name]
   Path: [working-directory]
```

### 2. During Work

**Every major step:**
```markdown
🔍 [Action]...
   ├─ [Sub-action 1]
   ├─ [Sub-action 2]
   └─ ✅ [Action] complete
```

**Progress for long tasks:**
```markdown
🔄 [Long-running task]... (this may take 2-3 minutes)
   [30s] Step 1
   [60s] Step 2
   [90s] Step 3
   └─ ✅ Task complete
```

### 3. Results/Findings

```markdown
📊 [Category] Analysis
   • Finding 1: [result]
   • Finding 2: [result]
   • Finding 3: [result]

Score: [X/Y] ([percentage]%)
```

### 4. Task Management

```markdown
📋 Creating tasks...
   ├─ Task 1: [name]
   ├─ Task 2: [name]
   └─ ✅ Created [X] tasks

🤖 Deploying agents...
   ├─ Agent 1: [purpose]
   └─ ✅ Agents deployed

✅ Task Complete: [task-name]
   [What was accomplished]
```

### 5. Completion

```markdown
✅ [Skill Name] Complete
   Duration: [X] minutes
   [Key metrics]

📈 Summary:
   • [Achievement 1]
   • [Achievement 2]
   • [Achievement 3]

📄 Reports:
   • [Report path 1]
   • [Report path 2]
```

---

## Error Handling

### Format

```markdown
❌ Error: [Brief description]
   File: [file-path]
   Error: [error-message]

   ℹ️ This is likely because:
   • Reason 1
   • Reason 2

   🔧 Suggested fix:
   1. Step 1
   2. Step 2
   3. Re-run: [command]
```

### Example

```markdown
❌ Error: Database migration failed
   File: supabase/migrations/20241201_create_users.sql
   Error: relation "users" already exists

   ℹ️ This is likely because:
   • Migration was already applied manually
   • Previous migration run was interrupted

   🔧 Suggested fix:
   1. Check existing tables: supabase db inspect
   2. Drop table if needed: DROP TABLE users CASCADE;
   3. Re-run migration: /db migrate
```

---

## Warning Handling

### Format

```markdown
⚠️ Warning: [Issue description]
   • Impact: [what this means]
   • Severity: [low/medium/high]

   ℹ️ Recommendation:
   → [What user should do]
```

### Example

```markdown
⚠️ Warning: No test coverage detected
   • Impact: Cannot validate code correctness
   • Severity: HIGH

   ℹ️ Recommendation:
   → Run /test-ship to add comprehensive tests
```

---

## Long-Running Operations

For any operation taking >30 seconds, provide periodic updates:

```markdown
🔄 [Operation name]... (estimated time: [X] minutes)
   [timestamp] [Status update]
   [timestamp] [Status update]
   └─ ✅ Complete
```

**Time intervals:**
- 0-60 seconds: Update every 15 seconds
- 1-5 minutes: Update every 30 seconds
- 5+ minutes: Update every 60 seconds

**Example:**
```markdown
🔄 Running comprehensive test suite... (estimated time: 3 minutes)
   [15s] Setting up test environment
   [30s] Running unit tests (25/100)
   [60s] Running unit tests (50/100)
   [90s] Running unit tests (75/100)
   [120s] Running integration tests (10/20)
   [150s] Running integration tests (20/20)
   [180s] Generating coverage report
   └─ ✅ All tests passed (100% coverage)
```

---

## Multi-Project Operations

When working across multiple projects:

```markdown
🚀 Multi-Project Operation: [skill-name]
   Projects: [project1, project2, project3]

📁 Project 1: [name]
   ├─ [Operation]
   └─ ✅ Complete

📁 Project 2: [name]
   ├─ [Operation]
   └─ ✅ Complete

📁 Project 3: [name]
   ├─ [Operation]
   └─ ✅ Complete

✅ All Projects Complete
   Success: [X/Y]
   Failures: [Y-X]
```

---

## Agent Deployment

When deploying sub-agents:

```markdown
🤖 Deploying [agent-type] agent...
   Purpose: [what the agent will do]
   Estimated time: [X] minutes

🔄 Agent working... (background)
   You can continue working while agent runs.

✅ Agent Complete: [agent-name]
   Duration: [X] minutes
   Output: [summary of what was done]
```

---

## Skill-Specific Customization

Each skill should adapt these standards to its domain:

### /admin
- Focus on: dashboard features, security, metrics
- Key phases: audit, security, implementation, deployment

### /sec-ship
- Focus on: vulnerabilities, fixes, validation
- Key phases: scan, fix, test, deploy

### /test-ship
- Focus on: coverage, test types, pass/fail
- Key phases: analyze, write, run, report

### /db
- Focus on: migrations, schema, data integrity
- Key phases: inspect, plan, migrate, verify

### /deps
- Focus on: outdated packages, security, updates
- Key phases: scan, plan, update, test

### /compliance
- Focus on: policies, regulations, violations
- Key phases: scan, generate, validate, document

### /perf
- Focus on: bottlenecks, optimizations, benchmarks
- Key phases: profile, optimize, benchmark, report

### /cleancode
- Focus on: debt, unused code, complexity
- Key phases: scan, clean, refactor, verify

### /docs
- Focus on: coverage, quality, examples
- Key phases: scan, generate, enhance, publish

### /design
- Focus on: UI/UX, responsiveness, accessibility
- Key phases: audit, design, implement, test

### /a11y
- Focus on: WCAG compliance, violations, fixes
- Key phases: audit, fix, validate, report

### /smoketest
- Focus on: critical paths, quick validation
- Key phases: plan, execute, report

---

## Progress Bars (Optional)

For visual progress tracking:

```markdown
Progress: [████████░░] 80% (4/5 tasks complete)
```

**Character guide:**
- █ = completed
- ░ = remaining
- Total: 10 characters for 10% increments

---

## Best Practices

1. **Start immediately** - First status update within 5 seconds
2. **Be specific** - "Creating user_roles table" not "Working on database"
3. **Include context** - Why are we doing this?
4. **Show progress** - Use tree structure (├─, └─) for nested operations
5. **Summarize** - End with clear summary of what was accomplished
6. **Link artifacts** - Show paths to generated reports/files
7. **Be concise** - One line per update, no walls of text
8. **Use consistent spacing** - 3 spaces for indentation
9. **Group related items** - Use bullet points for lists
10. **Time estimates** - Provide realistic estimates for long tasks

---

## Anti-Patterns (Avoid These)

❌ **Going silent**
```
Running tests...
[5 minutes of silence]
Tests complete.
```

✅ **Provide updates**
```
🧪 Running test suite...
   [30s] Unit tests: 50/100
   [60s] Unit tests: 100/100 ✅
   [90s] Integration tests: 10/20
   [120s] Integration tests: 20/20 ✅
   └─ ✅ All tests passed
```

❌ **Vague messages**
```
Processing...
Working...
Almost done...
```

✅ **Specific actions**
```
🔍 Scanning codebase for vulnerabilities...
🔒 Checking authentication implementations...
📊 Analyzing security configurations...
```

❌ **No context**
```
Error occurred
```

✅ **Full context**
```
❌ Error: Migration failed
   File: create_users.sql
   Reason: Table already exists
   Fix: Check database state with 'supabase db inspect'
```

---

## Testing Your Status Updates

When implementing status updates in a skill:

1. **Run the skill** - Execute with typical usage
2. **Time the silence** - Ensure no gap >30 seconds without update
3. **Check clarity** - Can user understand what's happening?
4. **Verify emojis** - Are they rendering correctly?
5. **Test errors** - Trigger errors, verify helpful messages
6. **Long operations** - Ensure progress updates appear
7. **Multi-step** - Verify all phases are marked

---

## Implementation Checklist

For each skill, ensure:

- [ ] Initialization message within 5 seconds
- [ ] Phase markers for each major step
- [ ] Progress updates for long operations (>30s)
- [ ] Tree structure for nested operations
- [ ] Summary at completion
- [ ] Error messages with context and fixes
- [ ] Warnings with severity and recommendations
- [ ] Report/artifact paths at end
- [ ] Consistent emoji usage
- [ ] No silent periods >30 seconds

---

**Reference this standard in your skill:**

```markdown
## STATUS UPDATES

This skill follows the [Status Update Protocol](~/.claude/standards/STATUS_UPDATES.md).

[Skill-specific examples here]
```

---

**Last Updated:** 2026-02-08
**Maintained by:** Claude Code Skills Team
