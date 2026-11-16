# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Lexington Radon Pros booking MCP server. The two captured subset tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `get_service_info`, `check_service_area` — a labelled subset, 2 of the 3 captured tools. The third captured tool, `request_service`, is deliberately omitted because it books radon mitigation for a real person (name, phone, ZIP and need) who is then called back by a local pro; it is not registered, not served, and has no CLI route.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `radonscope error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LEXINGTONRADONPROS_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
