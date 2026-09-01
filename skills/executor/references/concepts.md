# Executor Concepts

## Integration

An integration is something Executor connects to, defined by a source that describes the available tools:

- An **MCP server**
- An **OpenAPI** specification
- A **GraphQL** endpoint

The integration describes the catalog of tools. On its own, it isn't a live, authenticated connection. To actually call tools, you create a connection.

## Connection

A connection is a configured instance of an integration. A single integration can have many connections: the same API authenticated for two different accounts, for example.

A connection doesn't have to be authenticated. Public APIs and public MCP servers can be connected with no credentials. Executor runs tool calls in a sandbox, so the agent can never access the credentials.

## Policy

A policy controls what an agent can do with each tool:

- **Allow** — the tool runs without interruption.
- **Require approval** — the call pauses until a human approves it.
- **Block** — the tool can't be called.

Policies start from a sensible default derived from the integration's spec. For example, read-only `GET` operations on an OpenAPI spec are allowed by default, while writes can be set to require approval. You can tune the policy for any tool at any time.

## MCP Proxy

Executor sits between your agents and your tools as a single MCP endpoint. Your agents connect to Executor; Executor connects out to your integrations and re-exposes them as one catalog. Every tool call passes through the proxy, which is where auth and policy live.

Key properties:

- **One endpoint, every agent.** Point Claude Code, Cursor, ChatGPT, or any MCP-capable agent at the same Executor endpoint instead of configuring each tool in each client.
- **Credentials stay out of the agent.** A connection's credentials are stored by Executor and attached to the upstream call. The agent runs in a sandbox and never sees them.
- **Per-tool policies.** Every call is governed by a policy: allow, require approval, or block.
- **Mixed integration types.** Upstream MCP servers, OpenAPI specs, and GraphQL endpoints all show up in the same catalog.
