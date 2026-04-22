# Archetype D: Hybrid Web + AI

Web interface (usually dashboard-style) whose primary value is LLM interaction. SSE streaming, dual-model routing, small SaaS feel. Not a full product (archetype B), not headless (archetype C).

## Stack (locked defaults)

- **Backend**: FastAPI (Python 3.11+)
- **Frontend**: Vanilla HTML + JS + Tailwind (via CDN). No React. Keep it light.
- **Streaming**: Server-Sent Events (SSE)
- **LLM routing**: dual-model — Sonnet 4.6 for fast / cheap, Opus 4.6 for hard / expensive
- **Deploy**: Render (web service, because Vercel serverless kills long LLM streams)

Why vanilla JS instead of Next.js: this archetype is for interactive AI dashboards where the LLM latency dominates. Next.js adds complexity (hydration, SSR, deploy constraints) with no payoff. Vanilla + SSE is faster to ship, easier to debug, and the frontend stays tiny.

## Files to generate

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .env.template
├── requirements.txt
├── execution/
│   ├── __init__.py
│   ├── main.py                    ← FastAPI app entry
│   ├── engines/
│   │   ├── __init__.py
│   │   ├── router.py              ← dual-model routing logic
│   │   └── streaming.py           ← SSE helpers
│   └── prompts/
│       └── system_prompt.md       ← LLM system prompt
├── frontend/
│   ├── index.html
│   ├── app.js                     ← main client logic
│   └── styles.css                 ← minimal custom styles beyond Tailwind
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

Note: `prompts/` at project root (Claude Code prompts — PROMPT-01 etc) is SEPARATE from `execution/prompts/` (LLM system prompts for the runtime). Different concerns, do not merge.

## requirements.txt

```
fastapi>=0.115.0
uvicorn[standard]>=0.32.0
anthropic>=0.40.0
python-dotenv>=1.0
pydantic>=2.0
sse-starlette>=2.0
```

## execution/main.py

```python
"""<Project Name> — FastAPI entry point."""
import os
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
from fastapi.responses import FileResponse
from dotenv import load_dotenv
from sse_starlette.sse import EventSourceResponse

from engines.router import route_request
from engines.streaming import stream_response

load_dotenv()

app = FastAPI(title="<Project Name>")

# Serve frontend
app.mount("/static", StaticFiles(directory="../frontend"), name="static")

@app.get("/")
async def root():
    return FileResponse("../frontend/index.html")

@app.post("/api/chat")
async def chat(request: Request):
    body = await request.json()
    user_message = body.get("message", "")
    model_choice = route_request(user_message)  # Opus or Sonnet
    return EventSourceResponse(stream_response(user_message, model_choice))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=int(os.getenv("PORT", "8000")))
```

## execution/engines/router.py skeleton

```python
"""Dual-model routing. Decides Sonnet vs Opus per request."""

def route_request(message: str) -> str:
    """Return 'claude-sonnet-4-6' or 'claude-opus-4-6' based on request complexity."""
    # Simple heuristics to start. Refine later.
    if len(message) > 2000 or any(tag in message.lower() for tag in ["complex", "detailed", "analyze"]):
        return "claude-opus-4-6"
    return "claude-sonnet-4-6"
```

## execution/engines/streaming.py skeleton

```python
"""SSE streaming from Anthropic API."""
import os
from anthropic import Anthropic

client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

async def stream_response(user_message: str, model: str):
    """Stream tokens from the API as SSE events."""
    with client.messages.stream(
        model=model,
        max_tokens=4096,
        messages=[{"role": "user", "content": user_message}],
    ) as stream:
        for text in stream.text_stream:
            yield {"event": "token", "data": text}
        yield {"event": "done", "data": ""}
```

## frontend/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>[Project Name]</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="/static/styles.css" />
</head>
<body class="bg-gray-50 min-h-screen">
  <main class="max-w-3xl mx-auto p-6">
    <h1 class="text-3xl font-bold mb-4">[Project Name]</h1>
    <div id="output" class="bg-white rounded p-4 min-h-[200px] mb-4 whitespace-pre-wrap"></div>
    <form id="form" class="flex gap-2">
      <input
        id="input"
        type="text"
        placeholder="Ask anything..."
        class="flex-1 border rounded px-3 py-2"
      />
      <button type="submit" class="bg-black text-white px-4 py-2 rounded">Send</button>
    </form>
  </main>
  <script src="/static/app.js"></script>
</body>
</html>
```

## frontend/app.js

```javascript
const form = document.getElementById("form");
const input = document.getElementById("input");
const output = document.getElementById("output");

form.addEventListener("submit", async (e) => {
  e.preventDefault();
  const message = input.value.trim();
  if (!message) return;
  output.textContent = "";
  input.value = "";

  const response = await fetch("/api/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    const chunk = decoder.decode(value);
    const lines = chunk.split("\n");
    for (const line of lines) {
      if (line.startsWith("data: ")) {
        output.textContent += line.slice(6);
      }
    }
  }
});
```

## .env.template

```
ANTHROPIC_API_KEY=
PORT=8000
```

## CLAUDE.md section additions for Archetype D

In the project CLAUDE.md template, include:

- Tech stack: "FastAPI + vanilla JS + SSE streaming + dual-model routing (Opus + Sonnet)"
- Folder structure (show the tree above)
- Deploy: Render (web service)
- Required env vars: ANTHROPIC_API_KEY, PORT
- "First commands": `pip install -r requirements.txt`, `python execution/main.py`

## Deploy hint

```
1. Connect GitHub repo in Render dashboard
2. Create new Web Service (not Background Worker — this one has a UI)
3. Build command: pip install -r requirements.txt
4. Start command: cd execution && python main.py
5. Set env vars from .env.template
```

## Why not Next.js + API routes?

You might be tempted. Resist it. The failure mode is specifically: Vercel serverless functions have a 10-60s timeout. LLM streams for complex prompts exceed that. Render's web service has no such limit. Next.js is the right tool for archetype B. It's the wrong tool for archetype D.
