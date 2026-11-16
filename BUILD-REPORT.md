# BUILD REPORT — radonscope

Clean-room rebuild of `com.lexingtonradonpros/booking` v1.0.0
(remote-only MCP server at `https://lexingtonradonpros.com/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/radonscope`
0.1.0, registry name `io.github.CloudCorpRecords/radonscope`.

## Surface built

Labelled subset: **2 of 3 captured tools built**, one omission (below).

| Tool | Required fields as served |
|------|---------------------------|
| `get_service_info` | *(none — no `required` key)* |
| `check_service_area` | `zip` |

**OMITTED TOOL — `request_service`:** the upstream's 3rd captured
tool. Its schema requires `name`, `phone`, `zip`, `need` and its
description states it books radon mitigation for a real person who is
then called back by a local pro (PII lead-routing). It is deliberately
omitted: it is not in the fixture, not in the registered tool literals,
not served in tools/list, and no CLI command or route reaches it.
Tests assert its absence at registry level (not in
`TOOL_DEFS`/`TOOL_NAMES`, `isKnownTool` false, not present in the
serialised literals), MCP level (not in the served tools/list surface,
and `handleCallTool` rejects it with MethodNotFound before any network
hop), and CLI level (invoking it prints usage, exits 2, and its name
does not appear in the usage text).

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured (neither fixture
inputSchema carries a `$schema` marker; none was added). The tool
literals were generated programmatically from
`harness/fixtures/lexingtonradonpros-tools-original.json` by the script
at `/tmp/laneb3-radonscope-scratch/gen.py` (parse + re-serialise,
placeholder substitution into `src/forward.ts` from the template in the
same scratch dir) — never retyped — and a post-generation check (the
generator's own deep-equal assertion, the registration-integrity test
suite, plus the parity harness) confirmed the served literals
deep-equal the fixture's 2 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers),
not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, required fields (`get_service_info` carries no
`required` key; `check_service_area` requires `zip`) and property
shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://lexingtonradonpros.com/mcp`.
- Env override `LEXINGTONRADONPROS_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LEXINGTONRADONPROS_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `LEXINGTONRADONPROS_TOKEN` variables present.
- Error rendering `radonscope error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then node_modules populated with
  `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/radonscope`.
- Package entries: **128**.
- sha256: `a783e01b8095bc0130fbc43c3430ba72604b41a184a08c9eff12bab48bc885d0`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Radon mitigation in Lexington, KY: published
  service info, prices and service-area checks." — **90 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/radonscope:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/radonscope"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/radonscope`
  and its smoke test asserts the server name `radonscope`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **62 tests, 62 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 2
   tools; the omission suite asserts `request_service` absent at
   registry, MCP and CLI level; plus session handling/reuse, env
   override, the no-Authorization-header assertions under every
   environment, verbatim argument forwarding and result pass-through
   including upstream isError, SSE parsing, error rendering,
   MCP-layer validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/lexingtonradonpros-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/radonscope/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/radonscope/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
