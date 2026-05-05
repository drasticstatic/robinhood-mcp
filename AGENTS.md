# AGENTS.md
> AI Agent Configuration — robinhood-mcp
> Read by: Claude Code, Cursor, GitHub Copilot, and other AI coding assistants.
> See `CLAUDE.md` for Claude Code–specific rules.

---

## Project Overview

**robinhood-mcp** is a read-only MCP server wrapping the `robin_stocks` Python library. Provides 12 research tools for Robinhood portfolio data. Strictly educational/research — no trading functionality exposed.

**Upstream:** Forked — tracked for voluntary comparison.
**Visibility:** PRIVATE (working copy)

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| MCP server | Python (FastMCP) |
| Robinhood API | `robin_stocks` library |
| Auth | TOTP 2FA support |
| Package manager | pip |
| Testing | pytest |

---

## Common Commands

```bash
# Install with dev dependencies
pip install -e ".[dev]"

# Run linting
ruff check .
ruff format --check .

# Run tests
pytest -v

# Run with coverage
pytest --cov=src --cov-report=html

# Start the server
robinhood-mcp
```

---

## Coding Standards

- Type hints on all functions
- All robin_stocks calls wrapped with `_safe_call()` for consistent error handling
- Lazy authentication — login happens on first tool call, not server startup
- Tests use mocked responses — never use real credentials in tests
- `ruff` for linting and formatting

---

## Agent Boundaries

**Do:**
- Keep this strictly read-only — research and portfolio visibility only
- Use `_safe_call()` wrapper on all external API calls
- Add tests alongside any new tool

**Don't:**
- Expose any `order_buy_*`, `order_sell_*`, or `cancel_*_order` functions
- Log or print Robinhood credentials
- Store session tokens outside `~/.tokens/`

---

## Security Rules

- `ROBINHOOD_USERNAME`, `ROBINHOOD_PASSWORD`, `ROBINHOOD_TOTP_SECRET` — environment variables only, never committed
- Session tokens stored in `~/.tokens/` — gitignored
- **Never expose trading functions** — read-only is a hard constraint, not a preference

---

## Canonical References

- `CLAUDE.md` — Project overview, architecture, and session rules
- `AGENTS.md` (this file) — Universal AI agent config
- `README.md` — Tool list, setup, and publishing workflow
