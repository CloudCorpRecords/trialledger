# trialledger

Which clinical trials are past the deadline for reporting their results? trialledger is a local forwarding client for the hosted trial-overdue MCP service: it answers `tools/list` locally from the captured tool definition and forwards every call to the service, returning the answers exactly as the service sends them back.

The single tool reads the public ClinicalTrials.gov register. Trials covered by FDAAA 801 / 42 CFR Part 11 must report results within one year of primary completion; `overdue_trials` finds the ones that have missed that deadline, filterable by sponsor name or stock ticker and by a minimum number of days overdue. The results are public-record facts from the register — as the captured tool description itself states, they are NOT a finding of violation or legal/investment advice.

## The tool

- **`overdue_trials`** — Find clinical trials whose results are overdue under FDAAA 801 / 42 CFR Part 11. All parameters are optional: `sponsor` is a sponsor company name (partial match), `ticker` is a stock ticker, and `minDays` is the minimum days overdue.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/trialledger mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "trialledger": {
      "command": "npx",
      "args": ["-y", "@reneturcios/trialledger", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/trialledger
docker run --rm ghcr.io/cloudcorprecords/trialledger overdue_trials '{}'
```

## CLI

One binary serves both the command line and the MCP server. The tool is a subcommand taking a single JSON string with its arguments:

```bash
trialledger overdue_trials '{"sponsor":"Novartis"}'
trialledger overdue_trials '{"ticker":"BAYRY","minDays":90}'
trialledger overdue_trials '{}'
trialledger --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `TRIALOVERDUE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://overdue.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `trialledger error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the single tool, deep-equal, with no title invented and no `required` key added), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
