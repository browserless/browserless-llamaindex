# Browserless × LlamaIndex (Python)

Runnable Python examples for the Browserless MCP server, using `llama-index-tools-mcp`.

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export BROWSERLESS_TOKEN=<your-token>
export ANTHROPIC_API_KEY=<your-key>
```

Get a Browserless token at [account.browserless.io](https://account.browserless.io).

## Notebooks

| Notebook | What it shows | Tier |
|---|---|---|
| `quickstart.ipynb` | Connect, load tools, call `browserless_smartscraper` directly, run a `FunctionAgent` | 1 (single-call) |
| `research_agent.ipynb` | Multi-step search → scrape → summarize | 1 (single-call) |
| `browser_agent.ipynb` | Multi-turn browser agent: navigate → snapshot → click → re-snapshot → extract | 2 (multi-turn) |

### Note on multi-turn state

`browserless_agent` calls are one-shot by default: the browser closes when the call finishes. Pass `keepSessionAlive: true` on the first call, then the returned `sessionId` on every later call, and send `close` when done. The `sessionId` identifies the browser on its own, so it works with the stock `McpToolSpec` tools even though `BasicMCPClient` opens a new MCP connection per call. `browser_agent.ipynb` shows this directly and through an LLM.

### Notes on loading tools

- Browserless tool schemas include an optional `_prompt` field, and Pydantic rejects field names that start with an underscore, so plain `McpToolSpec` fails to build them. Each notebook defines a small `BrowserlessToolSpec` subclass that skips those fields.
- Pass `timeout=120` to `BasicMCPClient`: the default 30 s is too short for the first call's MCP handshake plus browser warm-up.

## Run

```bash
jupyter nbconvert --to notebook --execute quickstart.ipynb --output quickstart.executed.ipynb
jupyter nbconvert --to notebook --execute research_agent.ipynb --output research_agent.executed.ipynb
jupyter nbconvert --to notebook --execute browser_agent.ipynb --output browser_agent.executed.ipynb
```
