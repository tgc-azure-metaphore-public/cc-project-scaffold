# Archetype A: Standalone Tool

Single-file or tiny-script project. Runs locally or ships as a static page. No backend, no auth, no multi-user state.

## Decision sub-branch

Ask the user: "Is this a **single HTML file** (runs in a browser, pure frontend), or a **Python / Node script** (runs in a terminal)?"

- HTML → scaffold as HTML project
- Script → scaffold as script project

## Files to generate

### If HTML (single-file web tool)

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── index.html                 ← everything in one file (HTML + CSS + JS)
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

`index.html` skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>[Project Name]</title>
  <style>
    body { font-family: system-ui, sans-serif; margin: 2rem; max-width: 800px; }
    /* More styles inline as needed */
  </style>
</head>
<body>
  <h1>[Project Name]</h1>
  <p>[One-line description]</p>
  <main id="app"></main>
  <script>
    // Main logic here
  </script>
</body>
</html>
```

### If Script (Python / Node CLI or utility)

```
<project-name>/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .env.template              ← only if it needs secrets
├── requirements.txt           ← if Python
├── package.json               ← if Node
├── main.py  OR  main.js
├── prompts/
│   └── PROMPT-01-INIT-PROJECT.md
└── (optional) kill-criteria.md
```

Python `main.py` skeleton:

```python
"""[Project name] — [one-line description]"""

def main() -> None:
    # TODO: implement
    pass

if __name__ == "__main__":
    main()
```

`requirements.txt`:

```
# Add dependencies here. Keep minimal.
```

Node `main.js` skeleton:

```javascript
#!/usr/bin/env node
// [Project name] — [one-line description]

function main() {
  // TODO: implement
}

main();
```

`package.json`:

```json
{
  "name": "<project-name>",
  "version": "0.1.0",
  "description": "<one-line description>",
  "main": "main.js",
  "scripts": {
    "start": "node main.js"
  },
  "dependencies": {}
}
```

## CLAUDE.md section additions for Archetype A

In the project CLAUDE.md template, include:

- Tech stack: "Single HTML file, no backend" OR "Python 3.11+ script" OR "Node 20+ script"
- Deploy: None (local) OR GitHub Pages (for HTML)
- No folder structure section needed (it's flat)

## Deploy hint

- HTML: "Drop `index.html` in a GitHub repo, enable Pages, done."
- Script: "Runs locally. `python main.py` or `node main.js`."

## When in doubt, stay minimal

Archetype A is the archetype where "less is more" matters most. Do not add folders, modules, or config files that aren't needed. Resist the urge to generate a `src/` folder for a 50-line script.
