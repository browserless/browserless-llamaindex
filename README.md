# Browserless × LlamaIndex

Runnable Python examples for the [Browserless](https://browserless.io) MCP server. Browserless is a **stateful** MCP server: a multi-turn browser agent works in a real loop over HTTP, not just batched single-call mode.

The hosted endpoint at `https://mcp.browserless.io/mcp` exposes 10 tools auto-importable via `llama-index-tools-mcp` — no partner package needed.

| | Stateless tools (8) | Stateful tools (2) |
|---|---|---|
| | `smartscraper`, `search`, `map`, `crawl`, `export`, `performance`, `function`, `download` | `agent`, `skill` |
| Use case | Single-shot scraping, research, data extraction | Multi-turn browser automation, multi-step browser flows with persistent browser state |

> **Multi-turn agent in LlamaIndex:** the default `McpToolSpec` flow opens a fresh MCP session per call. To preserve state across multi-turn `browserless_agent` calls, use `async with client._run_session() as session: ...` and call `session.call_tool(...)` directly — see [`python/README.md`](./python/README.md#note-on-multi-turn-state) and [`python/browser_agent.ipynb`](./python/browser_agent.ipynb).

## Quick connect

```python
import os
from llama_index.tools.mcp import BasicMCPClient, McpToolSpec

client = BasicMCPClient(
    "https://mcp.browserless.io/mcp",
    headers={"Authorization": f"Bearer {os.environ['BROWSERLESS_TOKEN']}"},
    timeout=120,
)
tools = await McpToolSpec(client=client).to_tool_list_async()
print([t.metadata.name for t in tools])  # 10 tools
```

See [`python/`](./python/) for full notebooks: `quickstart.ipynb`, `research_agent.ipynb`, `browser_agent.ipynb`.

## TypeScript

`LlamaIndexTS` does not yet ship a first-party MCP client. Until it does, TypeScript users wanting Browserless via MCP should use [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) directly. We may revisit this if upstream lands a `@llamaindex/tools-mcp` adapter, or contribute one ourselves.

For a TypeScript Browserless example using a different framework, see [`browserless/browserless-langchain`](https://github.com/browserless/browserless-langchain) (which uses `@langchain/mcp-adapters`).

## Get a token

[account.browserless.io](https://account.browserless.io)

## Links

- Hosted MCP server: [mcp.browserless.io](https://mcp.browserless.io)
- LlamaIndex MCP adapter: [`llama-index-tools-mcp`](https://github.com/run-llama/llama_index/tree/main/llama-index-integrations/tools/llama-index-tools-mcp)
- Sister cookbook (LangChain): [`browserless/browserless-langchain`](https://github.com/browserless/browserless-langchain)
