# Executor CLI Reference

## Daemon & Service

### `executor install`

Install and start Executor as an OS-supervised background service.

```bash
executor install              # install with default port
executor install --port 4788  # install on a specific port
```

### `executor daemon run`

Run the local Executor daemon. Runs in the background by default.

### `executor daemon status`

Show daemon status.

```bash
executor daemon status
```

### `executor daemon stop`

Stop the local daemon.

```bash
executor daemon stop
```

### `executor daemon restart`

Restart the local daemon.

```bash
executor daemon restart
```

### `executor service install`

Alias for the OS-supervised background service install. Same as `executor install`.

### `executor service status`

Show the OS-supervised service status.

### `executor service restart`

Restart the OS-supervised background service.

### `executor service uninstall`

Stop and remove the OS-supervised background service.

### `executor web`

Open the Executor web UI at `http://127.0.0.1:4788`.

```bash
executor web                       # open the installed background service UI
executor web --foreground          # run a temporary foreground server
executor web --foreground --port 4789  # foreground on a custom port
```

## MCP Endpoint

### `executor mcp`

Start an MCP server over stdio. Used by `npx add-mcp "executor mcp"` for stdio-based agent connections.

```bash
executor mcp
```

## Tool Discovery & Description

### `executor tools integrations`

List configured integrations and tool counts.

```bash
executor tools integrations                         # list all
executor tools integrations --query "petstore"      # filter by name
executor tools integrations --limit 10              # limit results
```

### `executor tools search`

Search tools by natural-language query.

```bash
executor tools search "send email"                  # search by intent
executor tools search "create issue" --namespace github  # search within a namespace
```

### `executor tools describe`

Describe a tool's TypeScript and JSON schema.

```bash
executor tools describe github issues create
```

## Execution

### `executor call`

Invoke a tool path. Use `--help` to browse; pass a JSON object to invoke.

```bash
executor call --help                                # list top-level namespaces
executor call --help --match "issues"               # filter namespaces
executor call --help --limit 5                      # limit results
executor call github issues --help                  # drill into a namespace
executor call github issues create --help           # see a method's schema
executor call github issues create '{"owner":"octocat","repo":"Hello-World","title":"Hi"}'  # invoke
```

### `executor resume`

Resume a paused execution (e.g., approval or auth required).

```bash
executor resume --execution-id exec_123                           # resume (default)
executor resume --execution-id exec_123 --action accept           # approve
executor resume --execution-id exec_123 --action decline          # decline
executor resume --execution-id exec_123 --action accept --content '{"key":"value"}'  # approve with data
```
