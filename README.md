# Claude Code Skills Library

**33 production-tested Claude Code skills** built from shipping real products at [Steel Motion LLC](https://steelmotionllc.com).

This repo contains the **free tier** — 10 skills + 3 shared standards protocols. The premium tier (12 additional skills for CI/CD, security, compliance, and planning) is available at [steelmotionllc.com/portfolio/software/claude-code-skills](https://steelmotionllc.com/portfolio/software/claude-code-skills).

---

## Quick Install

```bash
# Clone and copy to your Claude Code config
git clone https://github.com/VetSecItPro/claude-code-skills.git
cp -r claude-code-skills/.claude/commands/* ~/.claude/commands/
cp -r claude-code-skills/.claude/standards/* ~/.claude/standards/

# Or one-liner:
bash <(curl -s https://raw.githubusercontent.com/VetSecItPro/claude-code-skills/main/install.sh)
```

After install, restart Claude Code. Skills appear as `/skill-name` commands.

---

## Free Tier Skills (10)

| Skill | Category | Description |
|-------|----------|-------------|
| `/dev` | Development | Self-healing development server with auto port recovery |
| `/deps` | Development | Dependency health audit and supply chain security |
| `/docs` | Development | WHY-focused documentation generator |
| `/blog` | Development | Interactive content pipeline — draft to deploy |
| `/n8n-workflow` | Development | Build n8n automation workflows interactively |
| `/smoketest` | Quality | Quick build verification — lint, typecheck, build |
| `/cleancode` | Quality | Dead code removal and technical debt reduction |
| `/perf` | Quality | Lighthouse-powered performance audit |
| `/a11y` | Quality | WCAG accessibility audit with automated fixes |
| `/design` | Design | UI/UX audit across all viewports with trend research |

## Shared Standards (3)

Every skill follows these protocols for consistent behavior:

| Protocol | Purpose |
|----------|---------|
| `STATUS_UPDATES.md` | Standardized progress reporting — phase tracking, completion %, structured output |
| `CONTEXT_MANAGEMENT.md` | How skills read and preserve context — memory, file discovery, state |
| `AGENT_ORCHESTRATION.md` | Multi-agent coordination — delegation, parallel execution, result aggregation |

---

## Premium Tier (12 Skills)

Advanced skills for CI/CD, security, compliance, and strategic planning. **Free with email** at [steelmotionllc.com](https://steelmotionllc.com/portfolio/software/claude-code-skills).

| Skill | Category | Description |
|-------|----------|-------------|
| `/gh-ship` | Development | Full git pipeline — commit → PR → CI → merge → deploy |
| `/db` | Development | Database schema, migrations, and data management |
| `/qatest` | Quality | Autonomous QA/UAT — 13-phase testing with autofix |
| `/test-ship` | Quality | Test audit — find gaps, write tests, verify coverage |
| `/sec-ship` | Security | Full security pipeline — audit → fix → validate → report |
| `/sec-weekly-scan` | Security | Cross-repo weekly security audit |
| `/redteam` | Security | Active exploitation testing against localhost |
| `/compliance` | Security | GDPR/CCPA privacy compliance audit |
| `/compliance-docs` | Security | Enterprise compliance documentation generator |
| `/landing-page` | Design | Conversion-optimized landing page builder |
| `/launch` | Planning | Pre-launch validation pipeline |
| `/mdmp` | Planning | Military Decision-Making Process for software |

---

## Repo Structure

```
claude-code-skills/
├── README.md
├── LICENSE                    # MIT
├── install.sh                 # One-liner installer
├── PREMIUM.md                 # Premium tier info + link
└── .claude/
    ├── commands/              # 10 free-tier skills
    │   ├── dev.md
    │   ├── smoketest.md
    │   ├── cleancode.md
    │   ├── deps.md
    │   ├── a11y.md
    │   ├── perf.md
    │   ├── docs.md
    │   ├── design.md
    │   ├── blog.md
    │   └── n8n-workflow.md
    └── standards/             # 3 shared protocols
        ├── STATUS_UPDATES.md
        ├── CONTEXT_MANAGEMENT.md
        └── AGENT_ORCHESTRATION.md
```

---

## How It Works

Claude Code skills are Markdown files that define structured prompts, execution phases, and tool permissions. When placed in `~/.claude/commands/`, they become available as `/command-name` in any Claude Code session.

The standards protocols in `~/.claude/standards/` are automatically loaded into every session, ensuring consistent behavior across all skills — same status reporting format, same context management rules, same agent orchestration patterns.

---

## Built By

[Steel Motion LLC](https://steelmotionllc.com) — We build production software and the DevOps frameworks that ship it.

## License

MIT — see [LICENSE](./LICENSE).
