# Hosting Map

Deploy target per archetype. These are the skill's opinionated defaults, not universal truths. Override on explicit user request.

| Archetype | Default target | Why |
|-----------|----------------|-----|
| A. Standalone tool | None (local) OR GitHub Pages | Single HTML + no backend = GitHub Pages is free + trivial. Python scripts stay local. |
| B. Full-stack product | **Vercel** | Native support for Next.js App Router, edge functions, Prisma, database connections. |
| C. Automation agent | **Render** (background worker) OR local cron | Render supports long-running background workers with scheduled jobs. Local cron works fine for personal agents that don't need cloud. |
| D. Hybrid web + AI | **Render** (web service) | FastAPI + SSE streaming work cleanly on Render. Vercel's serverless timeouts break long-running LLM streams. |
| E. Mobile app | **EAS Build** (Expo Application Services) | Native builds for iOS + Android, TestFlight distribution, OTA updates. |

## When to override the default

- User explicitly says "deploy to X": do what they said, note if X is a less-than-ideal fit but don't argue beyond one sentence.
- Project has specific Docker requirements: **Fly.io** — supports containerized deploys better than Render for unusual stacks.
- Project has high compute needs (ML inference, video processing): Fly.io or bare VPS (Hetzner).
- Project requires European-only data residency: Fly.io (EU regions) or Scaleway.

## What NOT to recommend

- Heroku — pricing model broken post-2022.
- AWS / GCP / Azure for a solo / small project — too much infra overhead, kills velocity.
- Netlify for Next.js App Router — works but Vercel is the reference implementation, fewer edge cases.

## Deploy hint format in CLAUDE.md

When generating the project's CLAUDE.md, include a "Deploy" section with this exact format:

```markdown
## Deploy

- **Target**: <Vercel | Render | Fly | GitHub Pages | Local only>
- **Reason**: <one line why this target fits>
- **Required env vars**: <list from .env.template>
- **Setup cheatsheet** (first deploy):
  1. <step>
  2. <step>
  3. <step>
```

Keep it to 3-5 setup steps. If the target is Render:

```
1. Connect GitHub repo in Render dashboard
2. Create new Web Service (or Background Worker for archetype C)
3. Set env vars from .env.template
4. Deploy. Render auto-detects Node / Python runtime.
```

If the target is Vercel:

```
1. `npm i -g vercel` then `vercel login`
2. From project root: `vercel --prod`
3. Set env vars in Vercel dashboard (or via `vercel env add`)
4. Add database connection (Vercel Postgres, Turso, or external)
```

If the target is Fly:

```
1. Install flyctl: `brew install flyctl`
2. `fly launch` from project root
3. Set secrets via `fly secrets set KEY=value`
4. `fly deploy`
```

If the target is EAS:

```
1. `npm install -g eas-cli`
2. `eas login`
3. `eas build:configure` then `eas build --platform ios` (or android)
4. TestFlight distribution via `eas submit`
```
