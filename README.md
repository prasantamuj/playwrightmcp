# pw-mcp-demo

Playwright + TypeScript test automation for [Sauce Demo](https://www.saucedemo.com), driven by Playwright's AI test agents (**Planner → Generator → Healer**) over the `playwright-test` MCP server.

The agents work in both **Claude Code** and **VS Code / GitHub Copilot**, and every agent follows the same project rulebook in [`AGENTS.md`](AGENTS.md).

## Prerequisites

- Node.js 20+
- npm

## Setup

```bash
npm install
npx playwright install chromium
```

## Running tests

| Command | What it does |
| --- | --- |
| `npm test` | Run the full suite |
| `npm run test:smoke` | Run only tests tagged `@smoke` |
| `npm run report` | Open the last HTML report |

The base URL (`https://www.saucedemo.com`) and the test-id attribute (`data-test-id`) are set in `playwright.config.ts`.

## Project structure

```
src/
  pages/        Page Object classes (one per page, extend BasePage)
  fixtures/     Custom fixtures; base.ts is the test entry point
  utils/        Pure helpers, no test logic
tests/          Spec files, mirroring the app's URL structure
  data/         JSON/CSV test data (e.g. users.json)
  seed.spec.ts  Seed test the agents use to bootstrap the page
specs/          Markdown test plans written by the Planner
.claude/agents/ Claude Code subagent definitions
.github/agents/ VS Code / Copilot agent definitions
AGENTS.md       Project rules for every AI agent (symlink to .github/copilot-instructions.md)
CLAUDE.md       Claude Code project memory (imports AGENTS.md)
```

## AI agent workflow

The three agents run as a chain:

1. **Planner** (`playwright-test-planner`): explores the app in a read-only browser and writes a numbered plan to `specs/<feature>.md`.
2. **Generator** (`playwright-test-generator`): turns one numbered scenario from a plan into a passing spec under `tests/`.
3. **Healer** (`playwright-test-healer`): fixes a failing spec without weakening its assertions, and always writes a Healer Report.

Example: [`specs/saucedemo-login.md`](specs/saucedemo-login.md) is a Planner-generated plan for the login flow.

### Claude Code

The MCP server is declared in `.mcp.json`. Ask Claude Code to run an agent by name, for example:

> Use the playwright-test-planner agent to plan the checkout flow.

### VS Code / GitHub Copilot

The MCP server is declared in `.vscode/mcp.json`. Pick the agent from the Copilot Chat agent picker.

To use the agents with the Copilot **coding agent** on GitHub, add this under **Settings → Copilot → Coding agent → MCP configuration**:

```json
{
  "mcpServers": {
    "playwright-test": {
      "type": "stdio",
      "command": "npx",
      "args": ["playwright", "run-test-mcp-server"],
      "tools": ["*"]
    }
  }
}
```

`.github/workflows/copilot-setup-steps.yml` prepares that environment.

> **Note:** `npx playwright init-agents` overwrites the custom agent files in `.github/agents/` with Playwright's generic templates. Don't re-run it unless you mean to reset them. If you do run it, restore the custom files with `git checkout -- .github/agents`.

## Coding conventions (summary)

The full rules are in [`AGENTS.md`](AGENTS.md). The key ones:

- Import `test` from `src/fixtures/base.ts`, never directly from `@playwright/test`.
- Locator priority: `getByRole` → `getByLabel` → `getByTestId` → `getByText` (static text only). CSS/XPath only with PR approval.
- Page objects extend `BasePage`, declare locators as `readonly`, and contain no `expect()` calls.
- Use web-first assertions only. No `waitForTimeout`, `waitForSelector` or `page.pause()`.
- Load test data from `tests/data/`, and tag tests `@smoke`, `@regression` or `@critical`.
- Never skip or comment out failing tests to make CI green.

## Credits

The agent setup is based on the [ShapeMyInterview Playwright MCP AI agents guide](https://www.shapemyinterview.com/resources/playwright-mcp-ai-agents-guide).
