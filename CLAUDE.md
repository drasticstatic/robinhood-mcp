# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## ⚠ FIRST: sync this clone before you touch anything

```sh
git pull --rebase --autostash
```

Run this at the **start of every session**, before reading deeply or editing. Several agents and
Christopher push to these repos — including Cosmos agents that run unattended while nobody is at the
machine — so a clone can be behind by the time you open it.

**`--autostash` is what makes this safe on a dirty tree.** It stashes uncommitted changes, rebases
onto the remote, then reapplies them. Your in-progress work survives. Without it, `git pull --rebase`
refuses to run and you are tempted into something worse.

Why it matters more than it sounds:

- A stale clone **does not fail early.** It fails at push time, after the work is done, as a
  non-fast-forward rejection — the most expensive moment to discover it.
- The tempting fix at that point is `git push --force`, which discards whatever someone else pushed
  in the meantime. Syncing first removes the temptation.
- If a rebase does conflict, stop and resolve it deliberately. A conflict is information: someone
  else changed the same lines, and you want to know that *before* building on top of them.

**Fresh clone?** Also run `sh scripts/install-hooks.sh` — git hooks are not version-controlled, so
the commit-attribution hook stays inert until this clone is pointed at `.githooks/`. Details:
[`scripts/README.md`](./scripts/README.md).

---

## Project Overview

**robinhood-mcp** is a read-only MCP server that wraps the `robin_stocks` Python library to provide research tools for Robinhood portfolio data. This is strictly a research/educational tool - no trading functionality is exposed.

## Architecture

```
src/robinhood_mcp/
├── __init__.py      # Package version
├── auth.py          # Authentication with TOTP support
├── tools.py         # 12 read-only tool implementations
└── server.py        # FastMCP server with tool registration
```

### Key Design Decisions

1. **Read-Only Only**: We explicitly do NOT expose any trading functions:
   - No `order_buy_*`, `order_sell_*`
   - No `cancel_*_order`
   - No account modification functions

2. **FastMCP**: Uses FastMCP for simpler decorator-based tool registration

3. **Lazy Authentication**: Login happens on first tool call, not server startup

4. **Error Handling**: All robin_stocks calls are wrapped with `_safe_call()` for consistent error handling

## Common Commands

```bash
# Install dependencies
pip install -e ".[dev]"

# Run linting
ruff check .
ruff format --check .

# Run tests
pytest -v

# Run with coverage
pytest --cov=src --cov-report=html

# Run the server
robinhood-mcp
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ROBINHOOD_USERNAME` | Yes | Robinhood account email |
| `ROBINHOOD_PASSWORD` | Yes | Robinhood account password |
| `ROBINHOOD_TOTP_SECRET` | No | Base32 TOTP secret for 2FA |

## Testing

Tests use mocked robin_stocks responses. To run with real credentials (careful!):

```bash
# Set env vars first
pytest tests/ -v
```

## Adding New Tools

1. Add the implementation to `tools.py` with proper type hints
2. Register in `server.py` with `@mcp.tool()` decorator
3. Add tests in `tests/test_tools.py`
4. Update `server.json` tool list
5. Update README.md tool table

## Publishing

This project uses GitHub Actions for CI/CD:

1. Push to main triggers CI (lint + test)
2. Create a version tag (e.g., `v0.1.0`) to trigger release
3. Release workflow publishes to PyPI via Trusted Publishing

## Safety Reminders

- **Never** expose trading functions
- **Never** log credentials
- **Always** validate user input (symbols, etc.)
- Keep session tokens secure (stored in `~/.tokens/`)


---

## Before Cloning or Installing Any External Repo / Package

Before running `git clone`, `npm install`, `pip install`, or adding any external dependency:
1. **Review `package.json` scripts** — flag any `postinstall`, `preinstall`, or `prepare` hooks that execute shell commands
2. **Scan for credential harvesting** — look for patterns accessing `~/.ssh`, `~/.aws`, `.env`, `process.env`, or system credential paths in unexpected files
3. **Verify provenance** — check GitHub repo age, star/fork count, recent commit activity, and maintainer identity
4. **Check for typosquatting** — verify package names exactly match the intended library (e.g. `lodash` not `1odash`)
5. **Audit unexpected network calls** — flag external HTTP requests in scripts, entrypoints, or install hooks
6. **When in doubt, ask Christopher before proceeding** with any install or clone

---

## Canonical References

When skills, specs, or task files exist for a topic — follow the logic there, not here. This file holds identity, pointers, and short rules only.

- **AGENTS.md** — root-level config for all AI agents (Claude Code, Cursor, Copilot)
- **AGENTS.override.md** — temporary task-specific overrides; delete when done (template: `~/code/my-template/AGENTS.override.md`)
- **Skills:** `.claude/skills/` — full procedure lives in the skill file; CLAUDE.md holds triggers only
- **Tasks:** `PENDING-TASKS.md` or `tasks.md` if present — active/completed task tracking
- **Agent handoffs:** `AGENT-SYNC/` (hub: `~/code/trading-assistant/`) — see `AGENT_SYNC.md` for current state
- **Memory:** `~/.claude/projects/.../memory/MEMORY.md` — auto-loaded; detail in topic files
## Commit attribution (enforced by hook)

Every commit must carry two git trailers:

```
Co-Authored-By: <Agent> · <Engine> · <Provider> [<Model>]
<Platform>-Session: <full session URL>
```

Four fields, model in **square brackets**, separator is U+00B7 MIDDLE DOT ( · ). The session
trailer is a **separate** line — folding it onto the `Co-Authored-By:` line breaks git trailer
parsing. Use the full session URL, never a truncated prefix. Key varies by platform:
`Claude-Session:` for Claude Code CLI, `Cosmos-Session:` for Cosmos.

`.githooks/commit-msg` rejects non-conforming commits. **Activate it once per clone:**

```sh
sh scripts/install-hooks.sh
```

Human-only commits: `git commit --no-verify`. Canonical convention and rationale:
[`my-template/AGENT-SYNC/README.md`](https://github.com/drasticstatic/my-template/blob/main/AGENT-SYNC/README.md)
