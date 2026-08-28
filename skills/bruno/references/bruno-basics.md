# Bruno Basics Reference

> Priority 3: Workspace, collection, folder, request, and test fundamentals.

**Bruno v3+ is required** for workspaces and OpenCollection YAML. v3 differences are noted parenthetically; v1 and v2 are ignored.

## Workspace

Available in Bruno 3.0.0+.

### Types

- **Default Workspace**: Auto-created on first launch. Cannot be closed or deleted.
- **Custom Workspaces**: User-created workspaces for organizing collections.

### Layout

```
workspace/
├── workspace.yml
├── collections/
│   └── my-api/
│       ├── opencollection.yml
│       └── ...
└── environments/
    └── local.yml
```

### Limits

- Free/OSS supports two workspaces open at a time.
- Pro/Ultimate supports unlimited workspaces.

### Opening a Workspace

Select a folder containing `workspace.yml` in Bruno. Bruno will scan and load all collections within.

## Collection

### Root File

- **OpenCollection YAML**: `opencollection.yml` (recommended for new collections)
- **Legacy `.bru`**: `bruno.json` (legacy format)

Bruno scans for either root file to identify collections.

### Creating a Collection

In Bruno: File → New Collection. Choose:
- **YAML** (recommended) — generates `opencollection.yml` and `.yml` request files.
- **BRU** (legacy) — generates `bruno.json` and `.bru` request files.

### Default Save Location

`~/Documents/bruno`

### Auto-generated `.gitignore`

Bruno auto-generates `.gitignore` to exclude:
- Temp files
- Environment files (unless explicitly committed)
- Generated metadata

### Opening Collections

Bruno scans for `opencollection.yml` or `bruno.json` in the selected directory.

## Folder

- Created under a collection.
- Config file: `folder.yml` in YAML format (or `.bru` in legacy).
- Ignore folders via the UI; updates the ignore list in `opencollection.yml` or `bruno.json`.

### Folder YAML

```yaml
info:
  name: Auth
  type: folder
  seq: 1
```

## Request

### Saving Formats

- **YAML**: `.yml` file in YAML format (recommended)
- **Legacy**: `.bru` file

### Creating Requests

- Within a collection (click `+` in the sidebar)
- Without a collection (unsaved, v3.1.0+)
- Via inline `+` tab (v3.1.0+)
- Import from cURL: `File → Import → cURL`

### Supported Protocols

- HTTP (REST)
- GraphQL
- gRPC
- WebSocket
- SOAP (via HTTP with XML body)
- cURL import

### Request YAML Structure

```yaml
info:
  name: Get Users
  type: http       # http, graphql, grpc, websocket, script
  seq: 1
  tags:
    - users

http:
  method: GET
  url: https://api.example.com/users
  headers:
    - name: Accept
      value: application/json
  params: []
  body:
    type: json
    data: ""
  auth:
    type: bearer
    token: "{{token}}"

runtime:
  variables: {}
  scripts: []
  assertions: []
  actions: []

settings:
  encodeUrl: true
  timeout: 30000
```

## Test

Bruno supports two test styles:

### 1. Assertions (Declarative)

Declarative assertions in the YAML:

```yaml
runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "Status should be 200"
```

See [opencollection-yaml.md](opencollection-yaml.md#assertions-example) for full operator list.

### 2. JavaScript Tests (Chai)

Using `test()` and `expect()` from Chai:

```yaml
runtime:
  scripts:
    - type: tests
      code: |
        expect(res.status).to.equal(200);
        expect(res.body).to.have.property('id');
        expect(res.body.data).to.be.an('array').with.length.above(0);
```

### Where Tests Live

Tests live in `runtime.scripts` with `type: tests` in YAML, or in `runtime.assertions` for declarative assertions.

### Test Execution

- **Collection Runner** (GUI): Runs all requests, executes tests.
- **CLI**: `bru run` executes requests and runs tests/assertions.

## Collection Runner

- Runs from the Bruno app UI (HTTP only; gRPC/WebSocket excluded).
- Data-driven testing with CSV is **Pro/Ultimate-only** in the GUI runner.
- CLI supports data-driven testing regardless of license.

## Quick Reference: Creating a Workspace/Collection/Folder/Request

### 1. Create a Workspace

```
File → New Workspace → Select a folder
```

### 2. Create a Collection

```
File → New Collection → Choose YAML or BRU format
```

### 3. Create a Folder

```
Right-click collection → New Folder
```

### 4. Create a Request

```
Click + in sidebar → Select protocol (HTTP, GraphQL, etc.)
```

### 5. Add a Test

```
Open request → Switch to Tests tab → Add assertion or write JavaScript test
```