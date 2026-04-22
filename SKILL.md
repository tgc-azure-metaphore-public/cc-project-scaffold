---
name: cc-project-scaffold
description: Scaffolds a new Claude Code project end-to-end. Detects the right archetype from a short description (standalone tool, full-stack product, automation agent, hybrid web+AI, or mobile app), asks 2-3 disambiguating questions if needed, then previews and writes the full folder structure including CLAUDE.md, prompts/PROMPT-01-INIT-PROJECT.md, dependency files, .env template, and deploy hint. Also generates follow-up prompts (PROMPT-02+) on demand. Triggers on "start a new Claude Code project", "new project for", "scaffold a project", "set up a new build", "create a new app", "I want to build", "generate next prompt", "PROMPT-0N+1".
---

# CC Project Scaffold

Turn a one-line project description into a ready-to-build Claude Code project in 10 minutes. Detect archetype, preview scaffold, confirm, write files, hand off PROMPT-01.

## When to fire

Fire whenever the user:

1. Says they want to start, create, scaffold, or set up a new Claude Code project: "start a new Claude Code project", "I want to build X", "new project for Y", "scaffold me a Z".
2. Requests the next prompt in an existing project: "generate next prompt", "PROMPT-0N+1", "give me PROMPT-05".

Do NOT fire on general brainstorming, business idea validation (use `founder-discipline:business-fit-score` for that), or when the user is iterating inside an existing project without explicitly asking to scaffold.

## The two modes

### Mode 1: New project scaffold (default)

Triggered by "start a new project", "scaffold", etc. Goes through Steps 1-6 below.

### Mode 2: Generate next prompt

Triggered by "generate next prompt" or "PROMPT-0N+1". Skip to the flow in `references/generate-next-prompt.md`.

---

## Step 1: Understand the project

The user has given you a brief. Parse it. Decide if it's enough to pick an archetype with confidence, or if you need to ask disambiguating questions.

Use `references/archetype-decision-tree.md` to match the description against signal phrases. The 5 core archetypes:

| Archetype | One-line cue |
|-----------|--------------|
| A. Standalone tool | Just for the user, local, small utility, single-file |
| B. Full-stack product | Web app, users sign up, multi-user, public |
| C. Automation agent | Reads/scrapes/processes automatically, runs headless |
| D. Hybrid web + AI | Dashboard with AI calls, small SaaS, interactive LLM tool |
| E. Mobile app | iOS / Android / native / Expo |

Plus one fallback:

| Archetype | One-line cue |
|-----------|--------------|
| **X. Custom** | No clear match to A-E. Claude designs an adapted architecture on the fly. |

Decision logic:

- **Strong match** (one archetype with 2+ signal matches AND no competing archetype with equivalent or stronger matches): proceed to Step 2 and confirm archetype with user before scaffolding.
- **Ambiguous between core archetypes** (two or more A-E score similarly, or the top score is weak with 1 signal): ask 2-3 disambiguating questions from `archetype-decision-tree.md`. Do not ask all 4 default questions if fewer would resolve it. Converge fast.
- **No core archetype matches** (zero signal matches, or every archetype scores 0-1 signals after a careful read): do NOT force-fit. Switch to the custom fallback flow in `references/archetype-custom-fallback.md`. This is the correct response when the project is genuinely novel: a browser extension, a game, a CLI tool, a library, a desktop app, an IoT controller, a hybrid that spans two archetypes, etc.

## Step 2: Confirm archetype and collect project metadata

State the archetype you've detected and why. Then collect, in one batch:

- **Project name** (kebab-case, used for folder name)
- **One-line description** (goes into CLAUDE.md and README)
- **Target workspace location** (ASK the user for their parent folder for code projects; do not assume a default path)
- **Will this be a commercial / business-facing project?** (Yes → triggers `kill-criteria.md` scaffold, see Step 4)

Do not proceed without all four confirmed.

## Step 3: Preview the scaffold (hybrid mode, REQUIRED)

Before writing anything, produce an in-chat preview of the full scaffold:

1. **Folder tree** — show every file and subfolder that will be created.
2. **Key file contents** — show the full `CLAUDE.md`, `prompts/PROMPT-01-INIT-PROJECT.md`, and one stack-specific file (e.g. `package.json` for B, `requirements.txt` for C). Don't dump every file — just the ones that matter.
3. **Deploy target** — state the hosting recommendation based on archetype (see `references/hosting-map.md`).
4. **Confirmation question**: "Proceed, adjust, or cancel?"

Wait for the user to confirm before writing. If they say "adjust X," apply the change to the preview, re-show, ask again.

## Step 4: Write files

Once confirmed, use the Write tool (or Bash `mkdir`/`cat` when appropriate for directory creation) to generate:

### Universal files (always)

- `CLAUDE.md` at project root — follow `templates/CLAUDE.md.template`, substituting project name, description, stack, folder structure. The generated CLAUDE.md imports `@../CONTEXT.md` and `@../CLAUDE.md` from the workspace level.
- `README.md` — follow `templates/README.md.template`.
- `.env.template` — follow `templates/env.template`.
- `.gitignore` — follow `templates/gitignore.template` (stack-appropriate entries).
- `prompts/PROMPT-01-INIT-PROJECT.md` — follow `references/prompt-01-template.md`. Fill in goal, stack, milestones, constraints based on the user's brief + archetype.

### Archetype-specific files

Load the matching archetype reference and generate those files:

- Archetype A → `references/archetype-a-standalone.md`
- Archetype B → `references/archetype-b-fullstack.md`
- Archetype C → `references/archetype-c-automation-agent.md`
- Archetype D → `references/archetype-d-hybrid-web-ai.md`
- Archetype E → `references/archetype-e-mobile.md`
- Archetype X (Custom) → follow the proposal you designed in the custom fallback flow (see `references/archetype-custom-fallback.md`)

### Kill-criteria scaffold (conditional)

If the user answered YES to "commercial / business-facing" in Step 2, also generate:

- `kill-criteria.md` at project root — a blank version following the template that `founder-discipline:kill-criteria-enforce` expects. Do NOT try to fill it in yourself. Instead, prompt the user: "You flagged this as business-facing. Run `founder-discipline:kill-criteria-enforce` now to fill in validation signal, anti-signal, review dates, and accountability. Or say 'later' to proceed without it, but commitment actions will be blocked until you do."

### Prompts folder convention (enforced)

All generated Claude Code prompts live in `./prompts/` with strict naming:

```
prompts/
├── PROMPT-01-INIT-PROJECT.md    ← generated by this skill
├── PROMPT-02-DESCRIPTIVE-NAME.md ← generated later via "generate next prompt"
├── PROMPT-02B-HOTFIX.md          ← hotfix letter suffix
└── ...
```

- Prefix: `PROMPT-` (uppercase)
- Number: two digits, sequential (01-99)
- Letter suffix for hotfixes / iterations on the same prompt (`05B`, `05C`)
- Descriptive name: UPPERCASE, hyphen-separated, 1-4 words
- Extension: `.md`

Enforce this on scaffold AND on follow-up prompt generation.

## Step 5: Handoff

After all files are written, print a handoff block that the user can act on:

```
## Scaffold complete

<project-name>/
├── <files created>

Next step — hand off to Claude Code:

1. Open Claude Code in the project folder:
   cd <absolute-path-to-project>
   claude

2. Paste the contents of prompts/PROMPT-01-INIT-PROJECT.md.

3. Claude Code runs the first build pass. Come back here for strategic decisions or next-prompt generation.

Deploy target: <Vercel | Render | Fly | GitHub Pages | local only>
```

Do NOT auto-invoke Claude Code. The user runs the `claude` command themselves.

## Step 6: Stay available for follow-ups

After Step 5, the user may:

- Ask for the next prompt → load `references/generate-next-prompt.md` and follow that flow.
- Ask to adjust the scaffold → edit files in place.
- Move on — leave them to it.

## Formatting rules (inherit from workspace)

All generated files must follow these rules:

- **No em dashes anywhere.** Use hyphens or parentheses instead.
- **English only** for all file contents. This is non-negotiable.
- **No unnecessary bullet lists in prose.** Prefer short paragraphs. Lists for actual lists only.
- **Short, direct.** No hedging, no corporate filler.

The workspace-level CLAUDE.md enforces these too. The skill must inherit them in everything it generates.

## Anti-patterns to avoid

- Do not scaffold without the preview step. The hybrid flow is non-negotiable.
- Do not hardcode archetypes that don't match the user's description. If you're unsure, ask.
- Do not generate a project without a `CLAUDE.md` at its root. The skill's value IS the CLAUDE.md structure.
- Do not skip the `prompts/` folder or the PROMPT-01 file. These are the prompt-chain backbone.
- Do not auto-invoke Claude Code. Print the handoff, let the user run it.
- Do not generate files in the user's workspace without confirmation. Always preview first.
- Do not forget the `kill-criteria.md` scaffold when the project is business-facing. It's the bridge to `founder-discipline`.

## Integration with founder-discipline

This skill is designed to work alongside the `founder-discipline` plugin:

- If the project is commercial, `kill-criteria.md` is pre-created as a blank. `founder-discipline:kill-criteria-enforce` takes over to fill it in.
- `founder-discipline:failure-pattern-guard` may fire during the scaffolding conversation if the user's brief triggers a known pattern (e.g., "I'm going to build a cold outreach tool"). Let it surface — don't suppress it.
- `founder-discipline:business-fit-score` is NOT invoked by this skill. If the user wants to score the idea first, they run that skill separately.
