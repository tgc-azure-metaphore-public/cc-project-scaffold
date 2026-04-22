# Archetype C: Automation Agent

Headless automation. Reads inputs (email, files, APIs, RSS, etc.), makes decisions, produces outputs. Runs on schedule or trigger, not interactively.

## The 3-layer architecture

This archetype uses a specific pattern refined across multiple real-world automation agents (inbox managers, invoice scrapers, data sync tools):

```
┌─────────────────────────────────────────┐
│  Layer 1: Directives (markdown SOPs)    │ ← What the agent should do, in prose
├─────────────────────────────────────────┤
│  Layer 2: Orchestrator (Claude)         │ ← Reads directives, decides, calls execution
├─────────────────────────────────────────┤
│  Layer 3: Execution (Python scripts)    │ ← Deterministic tools the agent invokes
└─────────────────────────────────────────┘
```

The agent is not a single program. It's Claude Code (or a CLI-invoked Claude) reading `directives/` and calling `execution/` scripts. The user runs it via a launch script (`run.sh` or `python run.py`) that bootstraps Claude with the directives folder as context.

## Stack (locked defaults)

- **Language**: Python 3.11+
- **Orchestrator**: Claude (Sonnet 4.6 for most tasks, Opus for complex reasoning)
- **Libraries**: stdlib + minimal third-party (`requests`, `pydantic`, domain-specific SDKs like `google-api-python-client`)
- **Config**: `.env` for secrets, `config.yaml` for behavior
- **Deploy**: Local cron OR Render background worker

## Files to generate

```
<project-name>/
├── CLAUDE.md                        ← operating instructions for Claude
├── AGENTS.md                        ← mirror of CLAUDE.md (compatibility)
├── GEMINI.md                        ← mirror of CLAUDE.md (compatibility)
├── README.md
├── .gitignore
├── .env.template
├── requirements.txt
├── config.yaml                      ← behavior config (thresholds, folder IDs, etc.)
├── directives/
│   ├── README.md                    ← explains the directive pattern
│   ├── 01-read-input.md             ← how to read the input source
│   ├── 02-classify.md               ← decision logic in prose
│   └── 03-write-output.md           ← output handling
├── execution/
│   ├── __init__.py
│   ├── read_inputs.py               ← reads input source (email/files/API)
│   ├── classify.py                  ← any deterministic classification
│   └── write_outputs.py             ← writes to output destination
├── run.py                           ← entry point
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

## CLAUDE.md body for Archetype C (critical — agent reads this)

```markdown
# <Project Name>

> Automation agent that [one-line description].

## Operating instructions

You are the orchestrator. Your job is to run the pipeline described in `directives/` by calling the tools in `execution/`.

## Pipeline

1. Read all files in `directives/` at session start. Each file describes one step of your job.
2. Call functions from `execution/*.py` to perform deterministic actions. Do not re-implement their logic in prose.
3. Produce outputs as specified in the final directive.

## Constraints

- Never modify the original input source. Only read.
- Every decision you make must be traceable to a directive. If a directive is missing, stop and report.
- Write a log entry per run: date, input count, decisions made, outputs written.

## Config

Behavior parameters (thresholds, folder IDs, account IDs) live in `config.yaml`. Read this file at startup.

## Folder structure

- `directives/` — markdown SOPs (your operating manual)
- `execution/` — Python utilities (your tools)
- `config.yaml` — runtime parameters
- `.env` — secrets (not committed)

## Workspace inheritance

This project inherits rules from `@../CLAUDE.md` and identity from `@../CONTEXT.md`. Follow those.
```

AGENTS.md and GEMINI.md are copies of CLAUDE.md (different AI code tools read different filenames). Generate all three identical.

## requirements.txt

```
anthropic>=0.40.0
pydantic>=2.0
python-dotenv>=1.0
requests>=2.31
pyyaml>=6.0
# Add domain-specific libs below (e.g., google-api-python-client for Gmail)
```

## .env.template

```
ANTHROPIC_API_KEY=
# Domain-specific credentials below
# GOOGLE_SERVICE_ACCOUNT_JSON_PATH=
# STRIPE_API_KEY=
```

## config.yaml skeleton

```yaml
# <Project Name> runtime configuration

# Input source
input:
  source: # e.g., gmail, local_folder, rss
  filter: # domain-specific filter

# Output destination
output:
  destination: # e.g., google_drive, local_folder, notion_db
  folder_id: # destination ID

# Behavior
behavior:
  dry_run: false
  log_level: info
```

## run.py skeleton

```python
"""<Project Name> — entry point."""
import os
from dotenv import load_dotenv
import yaml

load_dotenv()

def load_config() -> dict:
    with open("config.yaml") as f:
        return yaml.safe_load(f)

def main() -> None:
    config = load_config()
    # Bootstrap Claude with directives/ as context
    # Or invoke individual execution/*.py modules directly for testing
    print(f"Config loaded: {config}")
    # TODO: implement pipeline invocation

if __name__ == "__main__":
    main()
```

## directives/README.md skeleton

```markdown
# Directives

This folder is the agent's operating manual. Each file describes one step of the pipeline in plain English.

## Rules for directives

1. One concern per file. Do not combine "read input" and "classify" in one directive.
2. Numbered prefix (`01-`, `02-`, etc.) for execution order.
3. Reference specific functions in `../execution/` by name. Do not duplicate their logic.
4. Include examples of edge cases and how to handle them.

## Pipeline order

1. 01-read-input.md — how to read input source
2. 02-classify.md — decision logic
3. 03-write-output.md — output handling
```

## Deploy hint

Two options, ask the user:

**Local cron** (simpler, for personal agents):

```
1. crontab -e
2. Add: 0 */6 * * * cd /path/to/project && python run.py >> run.log 2>&1
3. First run: python run.py (test manually)
```

**Render background worker** (when cloud is needed):

```
1. Connect GitHub repo in Render dashboard
2. Create new Background Worker (not Web Service)
3. Build command: pip install -r requirements.txt
4. Start command: python run.py
5. Set env vars from .env.template
6. For scheduled: use Render Cron Jobs (separate service)
```

## Common add-ons

- **Google Workspace**: `google-api-python-client` + service account
- **Notion**: `notion-client`
- **Slack**: `slack-sdk`
- **OpenAI** (in addition to Anthropic): `openai`

Generate the base scaffold; let PROMPT-02+ add integrations.
