# BUILD REPORT — trialledger

Clean-room rebuild of `com.dropwatchhq/trial-overdue`
v1.0.0 (remote-only MCP server at
`https://overdue.dropwatchhq.com/mcp`), built 2026-10-28
as a forwarding client. Package: `@reneturcios/trialledger`
0.1.0, registry name `io.github.CloudCorpRecords/trialledger`.

## Surface built

Full captured surface: **1 of 1 tools built**.

| Tool | Required fields as served |
|------|---------------------------|
| `overdue_trials` | NO `required` key in the capture — sponsor, ticker and minDays are all optional; served verbatim, none invented |

The served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
key, so none is served. The tool literal was generated
programmatically from
`harness/fixtures/trialoverdue-tools-original.json` by the script
at `/tmp/b48-trialledger-scratch/gen.py` (parse + re-serialise +
round-trip assertion; the emitted `TOOL_DEFS` block was verified to
byte-match the block in `src/forward.ts`) — never retyped — and the
registration-integrity test suite plus the parity harness confirmed
the served literal deep-equals the fixture's 1 tool object. The tool
is registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the single tool's name,
description, the absence of a `required` key, and the property
shapes. The spec's note that the capture carries no `title` key
matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://overdue.dropwatchhq.com/mcp`.
- Env override `TRIALOVERDUE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`TRIALOVERDUE_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `trialledger error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/trialledger`.
- Package entries: **128**.
- sha256: `e02850a6d3ab2358ffa259617349a31b9dfafb69458ac5682a5ff9e5e8a3df5b`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Clinical trials overdue on required results reporting, by sponsor or ticker, from public records." —
  **97 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/trialledger:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/trialledger"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/trialledger`
  and its smoke test asserts the server name `trialledger`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **53 tests, 53 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 1
   tool, including the no-title and no-required-key assertions,
   session establishment and reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation — unknown tool → McpError MethodNotFound and
   non-object args → InvalidParams, both before any network hop —
   and CLI behaviour: `--version` prints 0.1.0, the
   `overdue_trials` subcommand forwards to a stubbed local upstream
   and prints its result, and an unknown command prints Usage to
   stderr with exit 2).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/trialoverdue-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/trialledger/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/trialledger/parity-out` → **verdict PASS, parity 100.0**
   (1/1 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
