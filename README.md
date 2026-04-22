# cc-project-scaffold

A Claude skill that scaffolds a new Claude Code project end-to-end. Turns a one-line description into a ready-to-build project folder in about 10 minutes.

## What it does

Given a short project description:

1. Detects the right **archetype** (standalone tool, full-stack product, automation agent, hybrid web + AI, mobile app).
2. Asks 2-3 disambiguating questions only if the description is ambiguous.
3. Previews the full scaffold in chat (folder tree + key file contents + deploy target).
4. After your confirmation, writes all files including the project's `CLAUDE.md`, `prompts/PROMPT-01-INIT-PROJECT.md`, dependency files, `.env` template, and `.gitignore`.
5. Hands off to Claude Code with the exact command to run.

Also handles follow-up prompts: say "generate next prompt" inside an existing project and the skill produces `PROMPT-0N+1` (or `PROMPT-0NB` for hotfixes) based on current state.

## The 5 archetypes

| Archetype | For | Stack | Deploy |
|-----------|-----|-------|--------|
| **A. Standalone tool** | Local utility, single file | HTML or Python/Node script | None or GitHub Pages |
| **B. Full-stack product** | Multi-user web app with auth | Next.js 15 + Prisma + NextAuth + Tailwind | Vercel |
| **C. Automation agent** | Headless agent (reads/processes/writes) | Python 3-layer (directives + execution + Claude orchestrator) | Render worker or local cron |
| **D. Hybrid web + AI** | AI-powered dashboard with streaming | FastAPI + vanilla JS + SSE + dual-model routing | Render web service |
| **E. Mobile app** | Native iOS / Android | Expo (React Native) + TypeScript | EAS Build |

Each archetype ships with its own opinionated stack and file templates. Defaults are deliberate choices (Vercel for Next.js, Render for FastAPI to avoid serverless timeouts, etc.), not neutral "industry standard" suggestions.

## The prompt-chain convention

All Claude Code prompts for a scaffolded project live in `./prompts/` with strict naming:

```
prompts/
├── PROMPT-01-INIT-PROJECT.md
├── PROMPT-02-AUTH.md
├── PROMPT-02B-HOTFIX.md        (letter suffix for hotfixes on the same prompt)
├── PROMPT-03-PAYMENTS.md
└── ...
```

Rules:
- Prefix `PROMPT-`
- Two-digit sequential number
- Letter suffix (B, C, D) for hotfixes on an existing prompt
- UPPERCASE hyphen-separated descriptive name
- `.md` extension

This convention turns every project into a readable build history. When you come back to a project months later, reading `prompts/` in order tells you exactly what happened.

## Installation

Drop the folder into `~/.claude/skills/cc-project-scaffold/`:

```bash
git clone https://github.com/<your-user>/cc-project-scaffold ~/.claude/skills/cc-project-scaffold
```

The skill is available in any Claude session (Cowork, Claude Code, Claude.ai with your configuration, etc.) after a restart.

## Trigger phrases

- "Start a new Claude Code project"
- "Scaffold a project for X"
- "I want to build Y"
- "Create a new app for Z"
- "Set up a new build"
- "Generate next prompt" (for existing projects)

## Design principles

- **Opinionated, not neutral.** The skill bakes in specific choices (stacks, deploy targets, folder conventions). Override any of them, but the defaults are the defaults for a reason.
- **Hybrid flow.** Preview in chat first. Never write files without explicit confirmation.
- **Prompt-chain first.** `prompts/PROMPT-01-INIT-PROJECT.md` is always generated. Subsequent prompts follow the convention.
- **Integrates with `founder-discipline`.** If you flag a new project as business-facing, the skill pre-creates a `kill-criteria.md` scaffold and hands off to the discipline plugin to fill it in.
- **Archetype detection is the core.** Picking the right archetype is 80% of the skill's value. The scaffolding comes after.

## Customization

To adapt the skill to your own preferences:

- **Change the default stacks**: edit each `references/archetype-<letter>-*.md` file.
- **Change the hosting recommendations**: edit `references/hosting-map.md`.
- **Change the prompt template**: edit `references/prompt-01-template.md`.
- **Change the CLAUDE.md format**: edit `templates/CLAUDE.md.template`.

Everything is markdown. No code to rewrite.

## License

MIT. Contributions welcome.

## Related skills

- `founder-discipline` — failure-pattern guard, kill-criteria enforcement, weighted business-fit scoring. Designed to pair with this skill when scaffolding business-facing projects.
