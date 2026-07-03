# Playwright Docker MCP — Testing Plan

## Goal
Use a **Docker-based Playwright MCP server** to test [claude-architect-certification.fly.dev](https://claude-architect-certification.fly.dev/) and build reusable knowledge for future Docker MCP projects.

---

## 1. Project Structure (to be created)

```
PlaywrightDockerMCP/
├── Dockerfile                    # Playwright + MCP server in a container
├── docker-compose.yml            # Orchestrate the container
├── mcp-server/                   # MCP server implementation
│   ├── package.json
│   └── index.ts                  # Exposes Playwright actions as MCP tools
├── tests/                        # Test specs
│   ├── smoke.test.ts             # Quick smoke tests
│   └── full-suite.test.ts        # Full page coverage
├── kilo.json                     # Project-level Kilo config with MCP entry
├── README.md                     # (already exists)
└── plan.md                       # This file
```

---

## 2. Docker Setup

### 2.1 Dockerfile
- **Base image**: `mcr.microsoft.com/playwright:v1.52.0` (official Playwright Docker image with all browsers pre-installed)
- Install Node.js (if not already in the base image)
- Copy the MCP server code
- Expose the MCP server port (e.g., 3100)

### 2.2 docker-compose.yml
- Service: `playwright-mcp`
- Build context: `.`
- Port mapping: `3100:3100`
- Environment variables (if needed for headless mode or remote URLs)

---

## 3. MCP Server Implementation

The MCP server will expose tools that Kilo (or any MCP client) can call:

| Tool Name              | Description                                      |
|------------------------|--------------------------------------------------|
| `browser_navigate`     | Navigate to a URL                                |
| `browser_click`        | Click an element by selector                     |
| `browser_type`         | Type text into an input                          |
| `browser_screenshot`   | Take a full-page screenshot                      |
| `browser_get_text`     | Extract text content from a selector             |
| `browser_evaluate`     | Run arbitrary JS in the page context             |
| `browser_wait`         | Wait for a selector or condition                 |
| `browser_close`        | Close the browser                                |

Protocol: MCP over stdio or HTTP (stdio is simpler for Docker integration).

---

## 4. Kilo MCP Configuration (`kilo.json`)

```json
{
  "mcp": {
    "servers": {
      "playwright": {
        "command": "docker",
        "args": [
          "compose", "-f", "/path/to/docker-compose.yml",
          "run", "--rm", "playwright-mcp"
        ]
      }
    }
  }
}
```

This tells Kilo to launch the Playwright MCP container when needed, pipe stdio to it, and expose the tools listed above.

---

## 5. Test Plan for claude-architect-certification.fly.dev

### 5.1 Smoke Tests
1. **Page loads** — Navigate to the site, confirm page title/heading exists
2. **SSL/TLS** — Confirm HTTPS is working (no cert errors)
3. **Console errors** — Open page and collect console logs; verify zero errors

### 5.2 Functional Tests
1. **Navigation** — Click through all main navigation links, verify each page loads
2. **Forms/Inputs** — If the site has forms, fill and submit them
3. **Responsive** — Resize viewport to mobile/tablet/desktop widths, take screenshots
4. **Assets** — Confirm CSS, JS, and images load without 404s

### 5.3 Visual Regression (future)
- Capture baseline screenshots per page
- Compare against future runs to detect unintended visual changes

### 5.4 Performance (future)
- Collect Lighthouse metrics (via Playwright's `page.metrics()`)
- Measure Time to Interactive, First Contentful Paint

---

## 6. Running Tests

```bash
# Option A: Run via Kilo with the MCP tool
# (After configuring kilo.json, ask Kilo to run tests)

# Option B: Direct Docker execution
docker compose run --rm playwright-mcp npx playwright test

# Option C: Build and run manually
docker build -t playwright-mcp .
docker run -it --rm playwright-mcp
```

---

## 7. Learning Roadmap — Docker MCP

| Phase | Topic                                       | What to Learn                                     |
|-------|---------------------------------------------|---------------------------------------------------|
| 1     | MCP Basics                                  | Protocol overview — tools, resources, prompts     |
| 2     | Dockerizing an MCP server                   | Writing Dockerfiles for Node.js MCP servers       |
| 3     | Playwright in Docker                        | Headless Chrome, system deps, `--no-sandbox`      |
| 4     | stdio MCP transport                         | How Kilo ↔ container stdio piping works           |
| 5     | HTTP MCP transport                          | Alternative to stdio for remote containers        |
| 6     | Multiple MCP servers                        | Running several containers side-by-side            |
| 7     | MCP security & permissions                  | Scoping tool access, sandboxing                   |
| 8     | Custom MCP tools                            | Building your own tools beyond Playwright         |

---

## 8. Success Criteria

- [ ] Docker container builds successfully
- [ ] MCP server starts and registers tools with Kilo
- [ ] Kilo can call `browser_navigate` and get back page content
- [ ] Smoke tests pass on claude-architect-certification.fly.dev
- [ ] All test results are logged and reproducible
