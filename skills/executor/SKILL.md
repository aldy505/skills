---
name: executor
description: Use a self-hosted/local Executor instance as your MCP gateway. Use when discovering, calling, or managing tools through Executor; adding integrations; resuming paused executions; diagnosing the daemon; or connecting an agent to a local Executor endpoint.
---

# Executor

## When to use

- **Tool discovery** — list integrations, search tools by intent, inspect a tool's schema.
- **Tool calls** — invoke tools through Executor's MCP proxy, with credentials and policy enforced upstream.
- **Integration management** — add MCP servers, OpenAPI specs, or GraphQL endpoints as integrations.
- **Paused execution recovery** — resume calls paused by approval or auth requirements.
- **Daemon diagnostics** — check status, restart, or troubleshoot the local Executor service.
- **MCP connection** — configure an agent to talk to a local Executor endpoint.

**Not for Executor Cloud.** If the user wants the hosted service, point them to [executor.sh](https://executor.sh) instead.

## Quick mental model

Executor is a single MCP endpoint that sits between your agents and all your tool integrations. You configure integrations (MCP servers, OpenAPI specs, GraphQL endpoints) once, and every connected agent sees the same catalog with shared auth and per-tool policies.

```
Your agents ──MCP──▶ Executor ──MCP──▶ Upstream MCP servers
                         │
                         ├──HTTP──▶ OpenAPI APIs
                         │
                         └──HTTP──▶ GraphQL endpoints
```

- **Integration** — something Executor connects to (an MCP server, OpenAPI spec, or GraphQL endpoint). It describes the catalog of tools.
- **Connection** — a configured, optionally authenticated instance of an integration. One integration can have many connections. Credentials stay in Executor; the agent never sees them.
- **Policy** — per-tool control: allow, require approval, or block. Defaults are derived from the spec (e.g., OpenAPI GET requests are allowed by default).

## Self-hosted variants

| Variant | Best for | Default local URL |
|---------|----------|-------------------|
| **CLI** (`executor`) | Headless or server environments | `http://127.0.0.1:4788/mcp` |
| **Desktop app** | Regular desktop environment (Mac, Windows, Linux) | `http://127.0.0.1:4788/mcp` |
| **Docker** | Single-container self-hosted deploy with SQLite | `http://localhost:4788/mcp` |
| **Cloudflare Worker** | Single-tenant deploy with D1 storage and Cloudflare Access | `https://executor-cloudflare.<subdomain>.workers.dev/mcp` |

All variants expose the same functionality — pick the one that fits your infrastructure.

## Connect an agent

Add Executor to any MCP client (Claude Code, Cursor, OpenCode) with `npx add-mcp`. It detects the client and writes its config for you.

**HTTP** (streamable-HTTP endpoint):

```bash
npx add-mcp http://127.0.0.1:4788/mcp --transport http --name executor
```

**CLI (stdio)** (launches `executor mcp` directly, no URL needed):

```bash
npx add-mcp "executor mcp" --name executor
```

MCP clients usually load servers at startup — after adding Executor, restart the client or open a new chat before the tools appear.

## Discover tools

```bash
executor tools integrations          # list configured integrations and tool counts
executor tools search "send email"   # find tools by natural-language intent
executor tools describe <path>       # show a tool's TypeScript and JSON schema
executor call <namespace> --help     # browse a namespace (use --match and --limit to filter)
```

## Call tools

```bash
executor call <namespace> <subcommand> --help          # browse a method
executor call <namespace> <subcommand> '{"key":"val"}' # invoke a tool
```

If an execution pauses for auth or approval:

```bash
executor resume --execution-id <id>                  # resume with default action
executor resume --execution-id <id> --action accept  # approve explicitly
executor resume --execution-id <id> --action decline # decline
```

## Add an integration (example)

```bash
executor call executor openapi addSource '{
  "spec": "https://petstore3.swagger.io/api/v3/openapi.json",
  "namespace": "petstore",
  "baseUrl": "https://petstore3.swagger.io/api/v3"
}'
```

Verify it's live:

```bash
executor tools integrations
```

## Daemon management

```bash
executor install              # install and start the OS-supervised background service
executor daemon status        # show daemon status
executor daemon stop          # stop the local daemon
executor daemon restart       # restart the local daemon
executor web                  # open the web UI at http://127.0.0.1:4788
executor web --foreground     # run a temporary foreground server (no install needed)
```

## References

- [CLI Reference](references/cli-reference.md) — full command signatures and examples.
- [Concepts](references/concepts.md) — integrations, connections, policies, and the MCP proxy model.
