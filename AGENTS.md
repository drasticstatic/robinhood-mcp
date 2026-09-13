# AGENTS.md
> AI Agent Configuration — robinhood-mcp
> Read by: Claude Code, Cursor, GitHub Copilot, and other AI coding assistants.
> See `CLAUDE.md` for Claude Code–specific rules.

---

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

## Override System

Create `AGENTS.override.md` for temporary task-specific rules. Delete when done. Template: `~/code/my-template/AGENTS.override.md`

---

## Canonical References

- `CLAUDE.md` — Project overview, architecture, and session rules
- `AGENTS.md` (this file) — Universal AI agent config
- `README.md` — Tool list, setup, and publishing workflow

## Commit attribution (enforced by hook)

Every commit must carry two git trailers:

```
Co-Authored-By: <Agent> · <Engine> · <Provider> [<Model>]                  # direct
Co-Authored-By: <Agent> · <Engine> · <Gateway> · <Provider> [<Model>]      # proxied
<Platform>-Session: <full session URL>
```

Model in **square brackets**, separator is U+00B7 MIDDLE DOT ( · ). Add `<Gateway>` **only when
inference is proxied** — it names what *routed* the request (`NVIDIA NIM`, `OpenRouter`), never who
made the model (`Z.ai`, `Moonshot AI`, `MiniMaxAI`). The field order mirrors the `/model` selector
string, so `anthropic/nvidia_nim/z-ai/glm4.7` transcribes to `NVIDIA NIM · Z.ai [GLM-4.7]` —
read it left to right rather than memorising it. Local runtimes (`Ollama`, `llama.cpp`,
`LM Studio`) have no gateway: the weights ran on your machine, so the runtime is the Provider. The session
trailer is a **separate** line — folding it onto the `Co-Authored-By:` line breaks git trailer
parsing. Use the full session URL, never a truncated prefix. Key varies by platform:
`Claude-Session:` for Claude Code CLI, `Cosmos-Session:` for Cosmos.

`.githooks/commit-msg` rejects non-conforming commits. **Activate it once per clone:**

```sh
sh scripts/install-hooks.sh
```

Human-only commits: `git commit --no-verify`. **Canonical spec — single source of truth. Do not restate the field table locally; link it:**
[`my-template/AGENT-SYNC/README.md`](https://github.com/drasticstatic/my-template/blob/main/AGENT-SYNC/README.md)
