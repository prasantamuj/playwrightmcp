# Claude Code project memory

The project rulebook lives in AGENTS.md (symlinked to .github/copilot-instructions.md). It applies to every session and every subagent.

@AGENTS.md

## Claude Code agents
Three subagents live in `.claude/agents/`. Run them as a chain: Planner → Generator → Healer.
- `playwright-test-planner`: explores the app and writes `specs/<feature>.md`. Read-only browser.
- `playwright-test-generator`: turns one numbered scenario into a passing spec.
- `playwright-test-healer`: fixes a failing spec without weakening assertions and always writes a Healer Report.

They drive the browser through the `playwright-test` MCP server declared in `.mcp.json` (`npx playwright run-test-mcp-server`).
