# PROMPT-01 Generation Rules

When generating `prompts/PROMPT-01-INIT-PROJECT.md`, use this template. Fill in `<placeholders>` from the user's brief + detected archetype.

## Template

```markdown
# PROMPT 01: Init Project

## Goal

Implement the first working version of **<project-name>**.

<One-paragraph project summary from the user's brief. Include what the user actually said, not a reformulation. Preserve intent.>

## Stack

<Pulled from the detected archetype's stack table. Example for archetype B:>

- Next.js 15 (App Router)
- TypeScript
- Prisma + SQLite (local) / Turso (prod)
- NextAuth v5
- Tailwind CSS
- Deploy: Vercel

## What to build in this pass

<3-5 specific, shippable milestones. NOT a full product. First pass only.>

Example for archetype B (a marketplace for freelancers):

1. Basic Next.js skeleton running with `npm run dev`
2. Prisma schema for `User` and `Listing` models + migration
3. Public homepage showing all listings (no auth required to view)
4. Minimal `/sign-in` page with magic-link email auth via NextAuth
5. Authenticated users can create a new listing via `/new`

Each milestone should be:
- Observable (you can see it working)
- Small (< 2 hours of Claude Code work)
- Not dependent on a future milestone

## Constraints

- Follow the architecture set in CLAUDE.md
- No premature optimization. Ship the happy path first.
- No tests in this pass (add in PROMPT-02 or later)
- No production deploy in this pass (local dev only)
- <Any specific constraints the user mentioned in their brief>

## Done when

- `npm run dev` (or equivalent) starts cleanly
- <Specific observable outcome 1>
- <Specific observable outcome 2>
- <Specific observable outcome 3>

## Out of scope for this pass

<List anything the user mentioned that you're explicitly deferring to PROMPT-02+>

Example:
- Stripe integration (PROMPT-02)
- Email notifications (PROMPT-03)
- Admin dashboard (PROMPT-04)

## Next steps after this pass

Once this pass is complete, run the next prompt via the `generate next prompt` trigger phrase in Cowork (or write PROMPT-02 manually). Typical PROMPT-02 candidates:

- Add auth flow end-to-end
- Add payment integration
- Add second database entity
- Add admin / moderation surface
```

## Rules for filling the template

### Goal section

- Copy the user's description verbatim when possible. Don't improve their prose.
- If their description is too short to be useful, ask one targeted question before generating PROMPT-01.

### Stack section

- Pull from the archetype's stack table. Don't improvise additions here.
- If the user asked for a specific stack override (e.g. "use Svelte instead of Next.js"), honor it and note the override.

### What to build section (the hardest part)

- Break the project into 3-5 milestones. Fewer means the prompt is too vague, more means you're planning PROMPT-02 already.
- Each milestone must be OBSERVABLE — you can verify it's done by running something, not by reading code.
- Each milestone should be 1-2 hours of work. If a single milestone looks like a day's work, split it.
- Do NOT include: tests, deploy, polish, accessibility, performance optimization. Those are PROMPT-02+.

### Constraints section

- Always include "Follow architecture in CLAUDE.md."
- Always include "No premature optimization."
- Add user-specific constraints from their brief (e.g., "Must work on mobile Safari," "Must support French input").

### Done when section

- Concrete, observable outcomes. "User can sign in" is good. "Good UX" is not.
- 2-4 items max.

### Out of scope section

- Explicitly name the features the user mentioned that you're deferring.
- This is the anti-scope-creep insurance. Without this section, PROMPT-01 tends to swell.

## Anti-patterns to avoid

- Do not generate a PROMPT-01 that tries to ship the whole product in one pass.
- Do not generate vague milestones. "Build the frontend" is not a milestone. "Users can view the listings index page and click into a detail page" is.
- Do not include optional nice-to-haves in PROMPT-01. They go in PROMPT-02+.
- Do not forget the "Done when" and "Out of scope" sections. They are load-bearing.
