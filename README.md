# Claude Code Skills Library

**21 production-tested Claude Code skills** built from shipping real products at [Steel Motion LLC](https://steelmotionllc.com). All free, all open source.

A complete SDLC automation framework: plan, build, design, test, secure, and ship. Each skill is a structured prompt with execution phases, tool permissions, and built-in standards.

> **Disclaimer:** These skills are provided as-is under the MIT License. While they have been tested in production environments, they execute automated actions on your codebase (file edits, git operations, dependency changes, and more). Always review skill behavior in a safe environment before running on production code. Steel Motion LLC assumes no responsibility for unintended changes, data loss, or other issues resulting from the use of these skills. Use at your own discretion.

---

## Quick Install

```bash
bash <(curl -s https://raw.githubusercontent.com/VetSecItPro/claude-code-skills/main/install.sh)
```

Or manually:

```bash
git clone https://github.com/VetSecItPro/claude-code-skills.git
cp -r claude-code-skills/.claude/commands/* ~/.claude/commands/
cp -r claude-code-skills/.claude/standards/* ~/.claude/standards/
```

After install, restart Claude Code. Skills appear as `/skill-name` commands.

> The installer is additive. If you already have skills in `~/.claude/commands/`, it merges without removing your existing ones.

---

## Pipeline Cheat Sheet

Skills are ordered by SDLC phase. Use them in sequence or pick what you need.

### Plan

| Skill | When to Use |
|-------|-------------|
| `/mdmp` | Planning complex decisions with structured analysis |

### Build

| Skill | When to Use |
|-------|-------------|
| `/dev` | Starting a dev server with auto-healing and port management |
| `/db` | Managing schemas, migrations, and seed data |
| `/deps` | Auditing packages for outdated or vulnerable dependencies |
| `/docs` | Generating decision-focused documentation |
| `/blog` | Publishing blog posts with SEO and auto-deploy |

### Design

| Skill | When to Use |
|-------|-------------|
| `/design` | Running a full UI/UX audit across all viewports |
| `/landing-page` | Building or optimizing landing pages for conversion |

### Quality

| Skill | When to Use |
|-------|-------------|
| `/smoketest` | Quick verification that the build is healthy |
| `/cleancode` | Cleaning up dead code and unused exports |
| `/perf` | Profiling performance and Core Web Vitals |
| `/a11y` | Auditing and fixing WCAG accessibility issues |
| `/qatest` | Pre-ship quality gate across all pages and APIs |
| `/test-ship` | Auditing test coverage and writing missing tests |

### Security

| Skill | When to Use |
|-------|-------------|
| `/sec-ship` | Running a full OWASP-aligned security audit |
| `/sec-weekly-scan` | Scheduled weekly security sweeps across repos |
| `/redteam` | Simulating attacks against your local environment |
| `/compliance` | Auditing GDPR/CCPA compliance and data flows |
| `/compliance-docs` | Generating compliance policies and control docs |

### Ship

| Skill | When to Use |
|-------|-------------|
| `/launch` | Running pre-launch checks before going live |
| `/gh-ship` | Shipping code from commit to merged PR in one command |

---

## 3 Shared Standards

Every skill follows the same protocols for consistent behavior:

| Protocol | Purpose |
|----------|---------|
| `STATUS_UPDATES.md` | Standardized progress reporting: phase tracking, completion %, structured output |
| `CONTEXT_MANAGEMENT.md` | How skills read and preserve context: memory, file discovery, state |
| `AGENT_ORCHESTRATION.md` | Multi-agent coordination: delegation, parallel execution, result aggregation |

---

## How It Works

Claude Code skills are Markdown files that define structured prompts, execution phases, and tool permissions. When placed in `~/.claude/commands/`, they become available as `/command-name` in any Claude Code session.

The standards protocols in `~/.claude/standards/` are automatically loaded into every session, ensuring consistent behavior across all skills.

### Repo Structure

```
claude-code-skills/
├── README.md
├── LICENSE                    # MIT
├── install.sh                 # One-command installer
└── .claude/
    ├── commands/              # 21 skills
    │   ├── mdmp.md            # Plan
    │   ├── dev.md             # Build
    │   ├── db.md
    │   ├── deps.md
    │   ├── docs.md
    │   ├── blog.md
    │   ├── design.md          # Design
    │   ├── landing-page.md
    │   ├── smoketest.md       # Quality
    │   ├── cleancode.md
    │   ├── perf.md
    │   ├── a11y.md
    │   ├── qatest.md
    │   ├── test-ship.md
    │   ├── sec-ship.md        # Security
    │   ├── sec-weekly-scan.md
    │   ├── redteam.md
    │   ├── compliance.md
    │   ├── compliance-docs.md
    │   ├── launch.md          # Ship
    │   └── gh-ship.md
    └── standards/             # 3 shared protocols
        ├── STATUS_UPDATES.md
        ├── CONTEXT_MANAGEMENT.md
        └── AGENT_ORCHESTRATION.md
```

---

## Built By

[Steel Motion LLC](https://steelmotionllc.com) | [All Skills with Details](https://steelmotionllc.com/portfolio/software/claude-code-skills)

## License

MIT. See [LICENSE](./LICENSE).
