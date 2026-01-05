# Server Helper Script

The `with_server.py` script manages server lifecycle automatically.

## Usage

**Single server:**
```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**Multiple servers (e.g., backend + frontend):**
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

## Options

- `--server` - Server command (can be repeated for multiple servers)
- `--port` - Port for each server (must match --server count)
- `--timeout` - Timeout in seconds per server (default: 30)

## How It Works

1. Starts all specified servers
2. Waits for each server to be ready by polling the port
3. Runs your command after all servers are ready
4. Cleans up all servers when done

## Example Automation Script

When using `with_server.py`, your automation script only needs Playwright logic:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto('http://localhost:5173')  # Server already running and ready
    page.wait_for_load_state('networkidle')
    
    # Your automation logic here
    
    browser.close()
```
