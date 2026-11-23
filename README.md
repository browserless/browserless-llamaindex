# Browserless × LlamaIndex

Runnable Python examples for the [Browserless](https://browserless.io) MCP server. Browserless runs real browsers in the cloud, and its multi-turn browser agent keeps one browser across calls, so an LLM can navigate, look, and act in a real loop over HTTP.

The hosted endpoint at `https://mcp.browserless.io/mcp` works with `llama-index-tools-mcp`; no partner package needed.

| | Single-call tools | Multi-turn browser |
|---|---|---|
| | `smartscraper`, `search`, `map`, `crawl`, `export`, `performance`, `function` | `agent`, `skill` |
| Use case | Scraping, research, data extraction | Multi-step browser flows with persistent browser state |

> **Multi-turn agent in LlamaIndex:** `browserless_agent` calls are one-shot by default. Pass `keepSessionAlive: true` on the first call and the returned `sessionId` on later calls; the LLM does this on its own from the tool description. See [`python/browser_agent.ipynb`](./python/browser_agent.ipynb).

## Quick connect

```python
import os
from llama_index.tools.mcp import BasicMCPClient, McpToolSpec


class BrowserlessToolSpec(McpToolSpec):
    # Pydantic rejects field names with a leading underscore, so skip
    # optional fields such as `_prompt` when building each tool's schema.
    def create_model_from_json_schema(self, schema, model_name="DynamicModel"):
        properties = {
            name: prop
            for name, prop in schema.get("properties", {}).items()
            if not name.startswith("_")
        }
        return super().create_model_from_json_schema(
            {**schema, "properties": properties}, model_name
        )


client = BasicMCPClient(
    "https://mcp.browserless.io/mcp",
    headers={"Authorization": f"Bearer {os.environ['BROWSERLESS_TOKEN']}"},
    timeout=120,
)
tools = await BrowserlessToolSpec(
    client=client,
    allowed_tools=["browserless_smartscraper", "browserless_search", "browserless_agent"],
).to_tool_list_async()
print([t.metadata.name for t in tools])
```

See [`python/`](./python/) for full notebooks: `quickstart.ipynb`, `research_agent.ipynb`, `browser_agent.ipynb`.

## TypeScript

`LlamaIndexTS` does not yet ship a first-party MCP client. Until it does, TypeScript users wanting Browserless via MCP should use [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) directly. We may revisit this if upstream lands a `@llamaindex/tools-mcp` adapter, or contribute one ourselves.

For a TypeScript Browserless example using a different framework, see [`browserless/browserless-langchain`](https://github.com/browserless/browserless-langchain) (which uses `@langchain/mcp-adapters`).

## Get a token

[account.browserless.io](https://account.browserless.io)

## Links

- Browserless MCP server setup: [docs.browserless.io/mcp/browserless-mcp-server/setup](https://docs.browserless.io/mcp/browserless-mcp-server/setup)
- Browser agent reference: [docs.browserless.io/mcp/browser-agent](https://docs.browserless.io/mcp/browser-agent)
- LlamaIndex MCP adapter: [`llama-index-tools-mcp`](https://github.com/run-llama/llama_index/tree/main/llama-index-integrations/tools/llama-index-tools-mcp)
- Sister cookbook (LangChain): [`browserless/browserless-langchain`](https://github.com/browserless/browserless-langchain)
