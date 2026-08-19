# SpeedOf.Me MCP Server

Official MCP server for [SpeedOf.Me](https://speedof.me). Lets an AI agent run a real internet
speed test and read the result: download, upload, latency and jitter, measured against 130+ global
edge servers.

[![npm version](https://img.shields.io/npm/v/@speedofme/mcp.svg)](https://www.npmjs.com/package/@speedofme/mcp)
[![MCP Registry](https://img.shields.io/badge/MCP%20Registry-me.speedof%2Fspeed--test-blue)](https://registry.modelcontextprotocol.io/v0/servers?search=me.speedof)
[![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org)

```
"How fast is my connection right now?"
→ 187.4 Mbps down / 41.2 Mbps up / 11 ms latency / 1.8 ms jitter (San Jose)
```

## Tools

| Tool | What it does |
|------|--------------|
| `run_speed_test` | Runs a live speed test and returns download, upload, latency and jitter |
| `get_test_history` | Returns previous results from local history |

| Resource | Contents |
|----------|----------|
| `speedofme://history` | Recent test results (JSON) |
| `speedofme://config` | Current configuration and status |

## Quick start

**1. Get a free API secret**

[Register](https://speedof.me/api/portal/register.php), then open
[Integrations](https://speedof.me/api/portal/settings.php?tab=integrations) and generate a secret.

**2. Add the server to your MCP client**

```json
{
  "mcpServers": {
    "speedofme": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@speedofme/mcp"],
      "env": {
        "SOM_API_SECRET": "SOM_SECRET_xxx"
      }
    }
  }
}
```

Claude Code, one command:

```bash
claude mcp add speedofme --env SOM_API_SECRET=SOM_SECRET_xxx -- npx -y @speedofme/mcp
```

VS Code (Copilot) uses `servers` instead of `mcpServers` in `.vscode/mcp.json`; the rest of the
block is identical. Cursor, Windsurf, Zed, Cline, Continue, JetBrains and Goose all take the JSON
above. Client-specific configs and agent-framework examples (LangChain, LlamaIndex, AutoGen,
CrewAI, Semantic Kernel) are in the
[integration docs](https://speedof.me/api/docs/#mcp-integrations).

**3. Restart the client and ask for a speed test.**

## Tool reference

### `run_speed_test`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `tests` | string[] | all | Which tests to run: `download`, `upload`, `latency` |
| `sustainTime` | number | 6 | Seconds to sustain each sample (1-8). Lower is faster, higher is more accurate |
| `testTimeout` | number | 300 | Wall-clock cap for the whole run, in seconds. 0 disables, min 30, max 1800 |

The tool replies with formatted text. The underlying result looks like this:

```json
{
  "testId": "abc123",
  "download": 150.5,
  "upload": 45.2,
  "latency": 12,
  "jitter": 2.3,
  "testServer": "San Jose",
  "timestamp": "2026-08-18T10:30:00Z"
}
```

Speeds are Mbps, latency and jitter are milliseconds.

### `get_test_history`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | number | 10 | Maximum number of results |
| `since` | string | none | ISO date string. Only results after this time |

History is stored locally in `~/.speedofme/history.json`, capped at 1 MB (roughly 2,600 results).

## Also a CLI

The same package runs standalone, no MCP client required:

```bash
SOM_API_SECRET=SOM_SECRET_xxx npx -y @speedofme/mcp --progress   # human readable
SOM_API_SECRET=SOM_SECRET_xxx npx -y @speedofme/mcp --json       # for scripts
npx -y @speedofme/mcp history --limit 5                          # past results
npx -y @speedofme/mcp --help
```

And as a Node SDK:

```javascript
const { SpeedTest } = require('@speedofme/mcp');

const test = new SpeedTest({ apiSecret: process.env.SOM_API_SECRET });
const result = await test.run();          // { download, upload, latency, jitter, ... }
```

Full CLI and SDK options are documented on
[npm](https://www.npmjs.com/package/@speedofme/mcp).

## How the measurement works

SpeedOf.Me has measured what we call Real-World Speed the same way since 2011. Rather than
blasting a fixed payload, the test downloads progressively larger samples, starting at 128 KB, until
one takes longer than the sustain time (6 seconds by default), and reports the rate of that final
sample. Upload works the same way in reverse, and latency and jitter come from 10 round-trip
samples. Tests run against the nearest of 130+ edge locations, so an agent gets the same number a
person would see in the browser.

Every test also lands in your [dashboard](https://speedof.me/api/portal/) with geographic, device
and platform breakdowns, and usage against your plan.

Details: [how it works](https://speedof.me/how-it-works).

## Requirements

- Node.js 18 or newer
- A SpeedOf.Me API account and secret (free tier available)

## About this repository

This repo is the home of the MCP server: documentation, the registry entry (`server.json`) and
the issue tracker. The server itself is distributed as the
[`@speedofme/mcp`](https://www.npmjs.com/package/@speedofme/mcp) npm package and its source is not
published here.

Found a bug or want a tool added? Open an issue, or email support@speedof.me.

## Links

- [npm package](https://www.npmjs.com/package/@speedofme/mcp)
- [Get started](https://speedof.me/api)
- [API portal](https://speedof.me/api/portal/)
- [API docs](https://speedof.me/api/docs/)
- [Status](https://speedofme.statuspage.io)

## License

The `@speedofme/mcp` package is proprietary software of Speed of Me, LLC. Unauthorized copying or
redistribution is prohibited. The documentation in this repository is provided for use with the
SpeedOf.Me API.
