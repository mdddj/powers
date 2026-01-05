---
name: "webapp-testing"
displayName: "Web Application Testing"
description: "Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs."
keywords: ["playwright", "testing", "web", "browser", "automation", "e2e", "end-to-end", "screenshot", "ui testing", "frontend testing", "测试", "浏览器", "自动化", "端到端测试", "前端测试"]
---

# Web Application Testing

To test local web applications, write native Python Playwright scripts.

## Prerequisites

Ensure you have Playwright installed:

```bash
pip install playwright
playwright install chromium
```

## Helper Scripts Available

- `scripts/with_server.py` - Manages server lifecycle (supports multiple servers)

**Always run scripts with `--help` first** to see usage.

## Decision Tree: Choosing Your Approach

```
User task → Is it static HTML?
    ├─ Yes → Read HTML file directly to identify selectors
    │         ├─ Success → Write Playwright script using selectors
    │         └─ Fails/Incomplete → Treat as dynamic (below)
    │
    └─ No (dynamic webapp) → Is the server already running?
        ├─ No → Run: python scripts/with_server.py --help
        │        Then use the helper + write simplified Playwright script
        │
        └─ Yes → Reconnaissance-then-action:
            1. Navigate and wait for networkidle
            2. Take screenshot or inspect DOM
            3. Identify selectors from rendered state
            4. Execute actions with discovered selectors
```

# When to Load Steering Files

- Writing Playwright automation scripts → `playwright-patterns.md`
- Using the server helper script → `server-helper.md`
- Debugging or discovering elements → `element-discovery.md`
