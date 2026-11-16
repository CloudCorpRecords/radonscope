# radonscope

Checking whether a Lexington home sits inside a radon mitigation service area, without handing anyone your details. radonscope is a local forwarding client for the hosted Lexington Radon Pros MCP service: it answers `tools/list` on your own machine from the captured tool definitions and forwards every read to the service, returning the answers exactly as they come back.

Both tools are published reads. You can pull up the service's facts — towns served, typical prices, hours, response time, languages and phone — and ask whether a given US ZIP code falls inside the Lexington service area, and if so which served town is nearest. A ZIP code is all that ever leaves your machine; no name, no phone number, and nothing is booked or requested by any tool in this package.

## The tools

- **`get_service_info`** — Facts about Lexington Radon Pros, a free referral service for radon mitigation in Lexington, Kentucky: towns served, typical prices, hours, response time, languages and phone. Takes no arguments.
- **`check_service_area`** — Whether a US ZIP code is inside the Lexington service area, answered yes/no with the nearest served town. `zip` is required and must be five digits.

One upstream tool is deliberately not included: the service-request tool, which would book radon mitigation for a real person — name, phone, ZIP and need — to be called back by a local pro. It is not registered, not served over MCP, and has no CLI subcommand.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/radonscope mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "radonscope": {
      "command": "npx",
      "args": ["-y", "@reneturcios/radonscope", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/radonscope
docker run --rm ghcr.io/cloudcorprecords/radonscope check_service_area '{"zip":"40502"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
radonscope get_service_info '{}'
radonscope check_service_area '{"zip":"40502"}'
radonscope check_service_area '{"zip":"40202"}'
radonscope --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `LEXINGTONRADONPROS_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://lexingtonradonpros.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The Lexington Radon Pros registry record declares no headers on its remote, so radonscope sends no authentication header of any kind — the reads are keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `radonscope error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured subset, asserts the omitted service-request tool is absent at registry, MCP and CLI level, exercises the session handshake and its reuse, asserts that no `Authorization` header is sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
