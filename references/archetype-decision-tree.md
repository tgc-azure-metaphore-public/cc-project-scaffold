# Archetype Decision Tree

Map the user's project description to one of 5 archetypes. Use signal phrases to score confidence. If ambiguous, ask the smallest set of disambiguating questions needed.

## Signal phrase matrix

### Archetype A: Standalone tool

**Signals**: "just for me", "local", "widget", "small script", "quick utility", "animation", "one-off", "a visualization", "a calculator", "tiny tool", "no backend", "static".

**Typical**: A Python script that does one thing, a single HTML file with an animation, a local CLI helper, a personal dashboard with no users but the builder.

### Archetype B: Full-stack product

**Signals**: "web app", "users will sign up", "public", "multi-user", "auth", "accounts", "marketplace", "directory", "community", "users can", "SaaS (with accounts)", "social", "profile pages", "Tinder for X", "Airbnb for X", "a platform where users".

**Typical**: A product where external users create accounts, interact with each other or with content, and the backend has persistent per-user state.

### Archetype C: Automation agent

**Signals**: "agent", "reads my inbox", "scrapes X", "processes Y automatically", "runs in the background", "cron", "every day at", "when X happens, do Y", "assistant that handles", "automation", "scraper", "bot that monitors", "headless".

**Typical**: A script (orchestrated by Claude or scheduled) that reads an input source, makes decisions, writes outputs. Runs on a schedule or trigger. No interactive UI.

### Archetype D: Hybrid web + AI

**Signals**: "dashboard with AI", "small SaaS with LLM", "interactive AI tool", "streaming chat interface", "LLM-powered dashboard", "an interactive tool where the user types and AI responds", "analysis UI with AI", "dual-model", "Opus + Sonnet".

**Typical**: A web interface (usually dashboard-style) that makes LLM API calls. Different from B because the value IS the AI interaction, not a product with incidental AI. Different from A because it's deployed and usable over the web.

### Archetype E: Mobile app

**Signals**: "mobile", "iOS", "Android", "native", "Expo", "React Native", "app for the phone", "I need to distribute through an app store", "TestFlight".

**Typical**: An app users install on their phone from an app store (or TestFlight). Not a responsive web app — a real native build.

## Decision flow

1. Parse the user's brief.
2. Score each archetype by counting signal matches.
3. If one archetype scores clearly higher (2+ unique signals and no strong match elsewhere), proceed with that archetype and confirm with the user before scaffolding.
4. If two archetypes tie or both look plausible, use the disambiguating questions below.
5. If the description is too vague to score anything (zero signals, user gave 5 words), ask the user to say more about what the project actually does.
6. If the description is clear but NO archetype matches well (every A-E scores 0-1 signals after a careful read), the project is genuinely novel. Do NOT force-fit. Switch to the custom fallback flow in `archetype-custom-fallback.md`. Examples that correctly trigger the fallback:
   - Browser extension
   - Desktop app (Electron, Tauri)
   - Game (Godot, Unity, p5.js)
   - CLI tool distributed via npm / brew / pip
   - Library (not a product; an importable module)
   - IoT / hardware controller
   - Discord / Slack bot (closer to C but with interactive UI layer)
   - Deno / Bun / Cloudflare Workers runtime specifically required
   - A hybrid that genuinely spans two archetypes and shouldn't be forced into one

## Disambiguating questions

Ask the FEWEST questions needed to resolve the ambiguity. Do not batch-ask all four if two resolve it.

### Q1: "Who uses it? Just you, or external users?"

- "Just me" → leans A or C
- "Me plus a few friends / family" → could be B (lightweight) or A
- "The public / customers / strangers" → leans B, D, or E

### Q2: "Does it have a UI, or does it run headless / in the background?"

- "Headless / runs automatically" → C
- "UI on my phone" → E
- "Web UI" → B, D, or A (single-HTML)

### Q3: "Will it call an LLM API (Anthropic, OpenAI) directly, or is it static / CRUD?"

- "Yes, LLM is core" → C or D (depending on headless vs UI)
- "No, static or CRUD" → A, B, or E
- "Yes, but just as a feature" → B (with AI as a feature)

### Q4: "Is it a one-time script, or something you'll deploy and run continuously?"

- "One-time / manual" → A or C-local
- "Deployed to cloud / runs 24/7" → B, C-Render, D

## Common pairs to disambiguate

- **A vs C**: both can be scripts. A is one-off or local utility, C runs on schedule or trigger. Ask Q4.
- **B vs D**: both have a web UI. B is user-facing product with auth + persistence; D is an AI tool (usually for the builder or a small audience). Ask Q1 + Q3.
- **C vs D**: both LLM-heavy. C is headless / agentic; D has a UI. Ask Q2.

## Never

- Never guess silently. Always state the detected archetype back to the user and ask them to confirm before scaffolding.
- Never default to Archetype B just because it's the "biggest" or most fleshed-out template. Pick the archetype that fits, not the one that's easiest to generate.
