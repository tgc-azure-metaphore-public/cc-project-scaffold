# Generate Next Prompt Flow

Triggered when the user says "generate next prompt", "PROMPT-0N+1", "give me PROMPT-0X", or equivalent while inside an existing project that already has a `prompts/` folder.

## Step 1: Find the project

Determine which project the user is referring to. Check:

1. The current working directory in the conversation.
2. The project folder the user mentions by name.
3. The most recently active project in the session.

If ambiguous, ask.

## Step 2: Read existing project state

Use the Read tool to load:

- `<project>/CLAUDE.md` — current project state, stack, goals
- `<project>/prompts/` — list all existing PROMPT files to determine the next number

Also optionally read (if they exist):

- Recent commits (ask user for `git log --oneline -20`) — what has shipped
- Open TODOs in the code (grep for `TODO:` and `FIXME:`)

## Step 3: Determine the next prompt number

Scan `prompts/` for existing files matching `PROMPT-XX-*.md`:

- If highest is `PROMPT-03-AUTH.md`, next is `PROMPT-04`.
- If a hotfix is in progress (recent file is `PROMPT-03B-FIX.md`), the next full prompt is still `PROMPT-04`. A new hotfix to PROMPT-03 would be `PROMPT-03C`.

Ask the user: "Is this the next full step (PROMPT-0N+1), or a hotfix iteration on the previous prompt (PROMPT-0NB/C)?"

## Step 4: Gather scope

Ask the user:

- **What's the goal of this pass?** One sentence.
- **What's shipped since the last prompt?** (Confirms the baseline.)
- **What's blocking this pass?** (Surfaces dependencies.)
- **What's explicitly OUT of scope?** (Prevents swell.)

Do not guess these from CLAUDE.md alone. Ask.

## Step 5: Draft the prompt

Use the same structure as PROMPT-01 (see `prompt-01-template.md`), but with adjustments:

- **Goal**: specific to this pass, not the whole project
- **Stack**: reference CLAUDE.md, don't re-enumerate unless the stack changed
- **What to build**: 2-4 milestones (PROMPT-02+ can be smaller than PROMPT-01 since the foundation exists)
- **Constraints**: reference the architecture in CLAUDE.md + any new constraints the user mentioned
- **Done when**: concrete, observable outcomes
- **Out of scope**: explicit list
- **Prerequisites**: what must be true before running this prompt (e.g., "PROMPT-03 complete and auth works end-to-end")

## Step 6: Preview and confirm

Show the drafted prompt in chat. Ask: "Proceed with this prompt, adjust, or cancel?"

Apply any adjustments. Re-show if needed. Then save.

## Step 7: Save with correct naming

Use the Write tool to save at `<project>/prompts/PROMPT-XX-DESCRIPTIVE-NAME.md`.

Naming rules (enforced):
- Prefix `PROMPT-`
- Two-digit number, sequential
- Letter suffix (B, C, D) for hotfixes on the same topic
- Descriptive name: UPPERCASE, hyphen-separated, 1-4 words
- Extension `.md`

## Step 8: Handoff

Print:

```
PROMPT-<XX>-<NAME>.md ready at <path>.

Next step — paste it into Claude Code:

cd <project-path>
claude

Then paste the contents of prompts/PROMPT-<XX>-<NAME>.md.

Come back here when:
- You need the NEXT prompt (say "generate next prompt")
- Something went wrong and you need strategic help
- You hit a decision point not covered by the current prompt
```

## Anti-patterns to avoid

- Do not generate PROMPT-02+ as a copy-paste of PROMPT-01 with minor tweaks. Each prompt must be genuinely the next step.
- Do not skip reading the existing CLAUDE.md. The project's state dictates what's next.
- Do not try to plan all future prompts in one pass. Generate one. Let the user ship it. Come back for the next.
- Do not ignore hotfix numbering (letter suffixes). If the user is fixing a bug in work from PROMPT-05, the prompt is PROMPT-05B, not PROMPT-06.
- Do not bundle multiple concerns into one prompt. If the user says "add auth AND add payments," that's PROMPT-N (auth) + PROMPT-N+1 (payments). Tell them.
