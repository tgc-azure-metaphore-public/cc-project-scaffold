# Archetype X: Custom Fallback

When the user's project description doesn't fit any of the 5 core archetypes (A-E), do NOT force-fit. Design an adapted architecture on the fly.

This flow applies when:

- Every A-E archetype scores 0-1 signal matches after a careful read.
- The project is of a type the 5 archetypes don't cover (browser extension, desktop app, game, CLI tool, library, IoT, bot, niche runtime, spanning hybrid).
- The user explicitly says "none of your archetypes fit, I want something custom."

## Step 1: Acknowledge the mismatch out loud

Tell the user directly:

> "Your project doesn't fit the 5 core archetypes I have templates for. That's fine — I'll design a custom architecture for it. Give me 4-5 minutes to ask the right questions, then I'll propose a stack, folder structure, and first milestones."

Do not pretend the project fits an archetype it doesn't. Transparency is the whole point of this flow.

## Step 2: Ask the characterization questions

Ask these 4 questions in ONE batch (use AskUserQuestion if the surface supports it, otherwise a numbered list):

### Q1: Primary runtime

"Where does this code primarily run?"

- Browser (client-side, single page)
- Browser extension (Chrome, Firefox, etc.)
- Node.js / Bun / Deno server
- Python script or server
- Desktop (Electron, Tauri, native)
- Mobile (iOS / Android)
- Embedded / IoT
- Game engine (Unity, Godot, etc.)
- Other (ask user to specify)

### Q2: UI surface

"What's the user-facing surface?"

- Full web UI (pages, navigation, routes)
- Single-page UI (one screen, dashboard-style)
- Browser extension popup / side panel
- CLI (terminal, no graphical UI)
- No UI (library, daemon, API-only)
- Native desktop window
- Mobile app
- Game canvas
- Other

### Q3: Persistence

"Where does data live?"

- Nowhere (stateless, read-only, ephemeral)
- Local file(s) (JSON, SQLite, flat files)
- Local browser storage (localStorage, IndexedDB)
- Remote database (Postgres, Turso, MongoDB, etc.)
- Cloud storage (S3, R2, Drive, etc.)
- User's own system (reads/writes their filesystem)
- External API (the data is owned by a third-party service)

### Q4: Distribution

"How do end users get this?"

- GitHub (clone and run locally)
- npm / pip / brew / cargo / homebrew (package manager)
- App store (iOS App Store, Play Store, Chrome Web Store, Mac App Store)
- Direct download (.dmg, .exe, .AppImage)
- Deployed web URL (user just visits a site)
- SaaS product (user creates an account to use)
- Library embedded in someone else's project
- Other

## Step 3: Design the custom stack

Based on the 4 answers, propose a concrete stack. Write it out with rationale for each choice. Examples of what a proposal looks like:

### Example: Browser extension

**Runtime**: Browser extension (Chrome + Firefox via web-ext)
**UI**: Popup (250x400) + content script
**Persistence**: browser.storage.local
**Distribution**: Chrome Web Store + Firefox Add-ons

**Stack proposal**:

- Plain TypeScript (no framework — extensions are tiny)
- Vite for bundling
- `webextension-polyfill` for cross-browser compatibility
- Manifest V3

**Folder structure**:

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .env.template (if it calls external APIs)
├── package.json
├── vite.config.ts
├── tsconfig.json
├── manifest.json
├── src/
│   ├── popup/
│   │   ├── index.html
│   │   ├── popup.ts
│   │   └── popup.css
│   ├── content/
│   │   └── content-script.ts
│   └── background/
│       └── service-worker.ts
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

**Deploy**: Chrome Web Store via manual zip upload (free dev account $5 one-time). Firefox via AMO (free).

### Example: CLI tool distributed via npm

**Runtime**: Node.js
**UI**: CLI (commander-style)
**Persistence**: Local config file in `~/.<tool-name>/`
**Distribution**: npm package with `bin` entry

**Stack proposal**:

- TypeScript
- Commander.js for arg parsing
- chalk for terminal colors
- `zod` for config validation
- Packaged with `tsup` for single-file dist

**Folder structure**:

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── package.json
├── tsconfig.json
├── tsup.config.ts
├── src/
│   ├── index.ts          ← entry (hashbang)
│   ├── commands/
│   │   └── <command>.ts
│   └── lib/
│       ├── config.ts
│       └── utils.ts
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

**Deploy**: `npm publish` (needs an npm account). Users run `npx <project-name>` or `npm i -g <project-name>`.

### Example: Discord bot

**Runtime**: Node.js (long-running worker)
**UI**: Discord slash commands (server-side UI)
**Persistence**: Small Postgres or SQLite for command state
**Distribution**: Self-hosted or Render/Fly

**Stack proposal**:

- TypeScript
- discord.js
- Prisma + SQLite
- Deployed as a Render background worker

### Example: Desktop app (Tauri)

**Runtime**: Tauri (Rust backend + web frontend)
**UI**: Native window with HTML/JS inside
**Persistence**: Tauri's built-in fs + localStorage
**Distribution**: .dmg / .exe / .AppImage direct download

**Stack proposal**:

- Tauri + Rust backend
- React or vanilla JS frontend (user preference)
- Vite for frontend dev
- GitHub Releases for binary distribution

## Step 4: Propose and confirm

Present the stack proposal in chat as a short structured summary (not a wall of text). Include:

- One-sentence rationale for each major tech choice
- The full folder tree
- The deploy target and rationale
- Any alternatives the user should know about ("I went with X, but Y is also valid if you want Z")

Ask: "Proceed with this proposal, adjust, or cancel?"

Iterate if they want changes. Do not proceed to Step 5 until they explicitly approve the stack.

## Step 5: Scaffold using universal templates

The universal templates (`templates/CLAUDE.md.template`, `README.md.template`, `env.template`, `gitignore.template`) are archetype-agnostic and work for any stack. Fill them with the custom-stack details:

- **CLAUDE.md**: tech stack = the custom proposal; folder structure = the custom tree; deploy = the custom target
- **README.md**: same, adapted
- **.env.template**: slots for whatever secrets the custom stack needs
- **.gitignore**: base entries + any stack-specific additions (e.g., `dist/`, `.tauri/`, `extensions/build/`)
- **prompts/PROMPT-01-INIT-PROJECT.md**: built from the custom proposal, with 3-5 specific milestones

Use standard Write tool calls. No archetype-specific reference file to load (you're the reference for this one project).

## Step 6: Learning loop note

At the end of Step 5, append this note to the handoff message:

> "This project didn't match the 5 core archetypes, so I designed a custom one. If you build similar projects regularly (e.g., if you end up scaffolding 3+ browser extensions, 3+ CLI tools, 3+ bots, etc.), consider formalizing this as a new archetype reference file at `references/archetype-<letter>-<type>.md` so future scaffolds of this kind are instant. I won't do that automatically — it's your call based on pattern repetition."

## Anti-patterns to avoid

- Do not force a custom project into archetype A-E just to use an existing template. Users notice when the generated files don't match their actual project.
- Do not skip the 4 characterization questions. They prevent wrong assumptions.
- Do not improvise a stack outside the user's explicit runtime answer. If they said "browser extension," don't propose a Next.js app.
- Do not skip the proposal-confirm step. Scaffolding a custom stack without explicit approval leads to wasted cleanup.
- Do not promise to generate files for a stack you're not familiar with. If the user picks a runtime you genuinely don't know (e.g., Elixir/Phoenix for someone who's never used it), say so. Propose the closest thing you can reliably scaffold, or suggest they use that runtime's native CLI (e.g., `mix phx.new`) and scaffold only the CLAUDE.md + prompts/ on top.
