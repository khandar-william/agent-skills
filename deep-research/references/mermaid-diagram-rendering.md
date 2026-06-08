# Mermaid Diagram Rendering Workflow

## Context
When a research report benefits from a visual mindmap or diagram, render it as both an interactive HTML file and a PNG screenshot.

## Steps

### 1. Create the Mermaid source
Write the `.mmd` file or embed the mermaid code block in an HTML file with the Mermaid JS CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
<script>
    mermaid.initialize({
        startOnLoad: true,
        theme: 'dark',
        themeVariables: {
            primaryColor: '#1f6feb',
            primaryTextColor: '#c9d1d9',
            primaryBorderColor: '#30363d',
            lineColor: '#58a6ff',
            secondaryColor: '#21262d',
            tertiaryColor: '#161b22',
        }
    });
</script>
```

### 2. Render to PNG via Playwright
Use the **cloakbrowser venv** playwright (not `uv run --with playwright` — that needs `playwright install` which is slow):

```python
~/.cloakbrowser-venv/bin/python3 << 'PYEOF'
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={'width': 1600, 'height': 1200})
    page.goto('file:///absolute/path/to/mindmap.html')
    page.wait_for_timeout(4000)  # Wait for mermaid JS to render
    page.screenshot(path='/absolute/path/to/mindmap.png', full_page=True)
    browser.close()
PYEOF
```

### 3. Deliver
- Save HTML to `~/research/` alongside the report
- Save PNG screenshot to same directory
- Include raw mermaid code block in the response so user can paste into GitHub/Obsidian/Notion/mermaid.live

## Pitfalls
- `npm install -g @mermaid-js/mermaid-cli` times out frequently — avoid
- `uv run --with playwright` needs `playwright install` to download browsers — use cloakbrowser venv instead
- Mermaid mindmap syntax uses indentation for nesting (not braces)
- SVG export from browser console works but produces very large files — PNG via screenshot is more practical
- Dark theme requires explicit `themeVariables` override
