# Browserless × LlamaIndex (Python)

Runnable Python examples for the Browserless MCP server, using `llama-index-tools-mcp`.

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e .
pip freeze > requirements.txt
export BROWSERLESS_TOKEN=<your-token>
export ANTHROPIC_API_KEY=<your-key>
```

Get a Browserless token at [account.browserless.io](https://account.browserless.io).

## Notebooks

| Notebook | What it shows | Tier |
|---|---|---|
| `quickstart.ipynb` | Connect, list tools, call `browserless_smartscraper` directly, run a stateless `FunctionAgent` | 1 (stateless) |
| `research_agent.ipynb` | Multi-step search → scrape → summarize using stateless tools | 1 (stateless) |
| `browser_agent.ipynb` | Multi-turn browser agent: navigate → snapshot → click → re-snapshot → extract | 2 (stateful) |

### Note on multi-turn state

The default LlamaIndex MCP pattern (`McpToolSpec.to_tool_list_async()` + `tool.acall()`) opens a **fresh** MCP session per call — `BasicMCPClient.call_tool()` runs `_run_session()` internally for each invocation. Stateless tools don't care, but multi-turn `browserless_agent` flows lose browser state between calls.

To preserve state, wrap the multi-turn calls in `client._run_session()` and call `session.call_tool(...)` directly:

```python
async with client._run_session() as session:
    await session.call_tool("browserless_agent", arguments={"method": "goto", ...})
    await session.call_tool("browserless_agent", arguments={"method": "snapshot"})
    # ...
```

`_run_session()` is currently underscore-prefixed but is just an `@asynccontextmanager` method. `browser_agent.ipynb` shows this pattern in full. Issue tracking a public `client.session()` API: see the cookbook root README.

Also: pass `timeout=120` to `BasicMCPClient` — the default 30s is too short for the first call's MCP handshake + browser warm-up.

## Run

```bash
jupyter nbconvert --to notebook --execute quickstart.ipynb --output quickstart.executed.ipynb
jupyter nbconvert --to notebook --execute research_agent.ipynb --output research_agent.executed.ipynb
jupyter nbconvert --to notebook --execute browser_agent.ipynb --output browser_agent.executed.ipynb
```
