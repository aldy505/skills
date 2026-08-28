# OpenCollection YAML Reference

> Priority 1: The vendor-native, executable YAML format for Bruno.

## What is OpenCollection YAML?

OpenCollection YAML is Bruno's open specification (introduced in Bruno 3.0.0) for defining API collections as plain YAML files. Unlike Postman Collection, which is an import/export interchange format, OpenCollection YAML is the **native, executable** format used by Bruno's Collection Runner and `bru` CLI.

- File extension: `.yml`
- Root file: `opencollection.yml` (in the collection directory)
- Each request is a separate `.yml` file
- Each environment is a separate `.yml` file

### Contrast with legacy `.bru` format

Bruno supports two collection formats:
- **OpenCollection YAML**: `opencollection.yml` root, `.yml` request files, `.yml` environment files.
- **Legacy `.bru`**: `bruno.json` root, `.bru` request files, `.bru` environment files.

OpenCollection YAML is the recommended format for new collections.

## File Layout

```
my-collection/
├── opencollection.yml          # Collection root
├── environments/
│   ├── local.yml               # Environment definition
│   └── prod.yml
├── requests/
│   ├── get-users.yml           # Request file
│   └── create-post.yml
└── folders/
    └── auth/
        ├── folder.yml          # Folder config
        └── requests/
            └── login.yml
```

A legacy collection uses the same structure but with `bruno.json` and `.bru` files instead.

## Top-Level Structure: Request YAML

A request YAML file has these top-level sections:

```yaml
info:
  name: Get Users
  type: http          # or graphql, grpc, websocket, script
  seq: 1
  tags:
    - users
    - read

http:                 # Protocol-specific section (http, graphql, grpc, websocket)
  method: GET
  url: https://api.example.com/users
  headers: []
  params: []
  body:
    type: json
    data: ""

runtime:
  variables: {}
  scripts: []
  assertions: []
  actions: []

settings:
  encodeUrl: true
  timeout: 30000
  followRedirects: true
  maxRedirects: 10

docs: |
  Description of this request.
```

### Top-Level Structure: Folder YAML

```yaml
info:
  name: Auth
  type: folder
  seq: 1

items: []  # Array of request/folder references
```

## `info.type` Values

| Value | Protocol section | Description |
|---|---|---|
| `http` | `http:` | Standard HTTP/REST request |
| `graphql` | `graphql:` | GraphQL query or mutation |
| `grpc` | `grpc:` | gRPC unary/streaming call |
| `websocket` | `websocket:` | WebSocket connection/message |
| `folder` | (none) | Folder container |
| `script` | (none) | Script request |

## HTTP Requests

```yaml
info:
  name: Create Post
  type: http
  seq: 2

http:
  method: POST
  url: https://api.example.com/posts
  headers:
    - name: Content-Type
      value: application/json
    - name: Authorization
      value: "Bearer {{token}}"
  params:
    - name: page
      value: "1"
      type: query    # query or path
  body:
    type: json       # json, text, xml, sparql, form-urlencoded, multipart-form, file
    data: |
      {
        "title": "Hello World",
        "body": "This is a test post.",
        "userId": 1
      }
  auth:
    type: bearer
    token: "{{api_token}}"
```

### Body types

| `http.body.type` | Description |
|---|---|
| `json` | JSON body |
| `text` | Plain text |
| `xml` | XML body (also used for SOAP) |
| `sparql` | SPARQL query |
| `form-urlencoded` | URL-encoded form |
| `multipart-form` | Multipart form (supports files) |
| `file` | File upload |

### SSE (Server-Sent Events)

SSE is an HTTP request with header `Accept: text/event-stream`:

```yaml
http:
  method: GET
  url: https://api.example.com/events
  headers:
    - name: Accept
      value: text/event-stream
```

> **Note:** The Collection Runner and `bru run` skip SSE requests. Test them individually or remove the SSE header for CI runs.

### HTTP Params

Params have a `type` field:

```yaml
http:
  params:
    - name: userId
      value: "42"
      type: path    # path parameter
    - name: filter
      value: active
      type: query   # query parameter
```

## GraphQL Requests

```yaml
info:
  name: Get User
  type: graphql
  seq: 3

graphql:
  method: POST
  url: https://api.example.com/graphql
  headers:
    - name: Authorization
      value: "Bearer {{token}}"
  params: []
  body:
    query: |
      query GetUser($id: ID!) {
        user(id: $id) {
          name
          email
        }
      }
    variables: '{"id": "42"}'
  auth:
    type: bearer
    token: "{{api_token}}"
```

- `graphql.body.query` uses a literal block scalar.
- `graphql.body.variables` is a JSON string (not a nested object).

## gRPC Requests

```yaml
info:
  name: Get User
  type: grpc
  seq: 4

grpc:
  url: https://api.example.com:443
  method: user.UserService/GetUser
  methodType: unary          # unary, client-streaming, server-streaming, bidi-streaming
  protoFilePath: ./proto/user.proto
  metadata:
    - name: Authorization
      value: "Bearer {{token}}"
  message: |
    {
      "id": "42"
    }
  auth:
    type: bearer
    token: "{{api_token}}"
```

### Proto file resolution

- Request-level `protoFilePath` is the primary path.
- Collection-level `config.protobuf.protoFiles` and `config.protobuf.importPaths` provide fallback.
- Import resolution order: proto file directory → configured import paths → standard protobuf library paths.
- Reflection is supported in the Bruno UI.

## WebSocket Requests

```yaml
info:
  name: WebSocket Chat
  type: websocket
  seq: 5

websocket:
  url: wss://api.example.com/ws
  headers:
    - name: Authorization
      value: "Bearer {{token}}"
  message:
    type: json
    data: |
      {"action": "subscribe", "channel": "general"}
  auth:
    type: bearer
    token: "{{api_token}}"
```

- URL must be `ws://` or `wss://`.
- Message types: `text`, `json`, `xml`, `binary`.

## SOAP Requests

SOAP is an HTTP request with `http.body.type: xml` containing the SOAP envelope:

```yaml
info:
  name: SOAP GetWeather
  type: http
  seq: 6

http:
  method: POST
  url: https://api.example.com/weather
  headers:
    - name: Content-Type
      value: text/xml
  body:
    type: xml
    data: |
      <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
          <GetWeather xmlns="http://example.com/weather">
            <City>London</City>
          </GetWeather>
        </soap:Body>
      </soap:Envelope>
```

WSDL import is supported in the UI; manual SOAP authoring uses the XML body.

## Auth Types

| `auth.type` | Description |
|---|---|
| `none` | No authentication |
| `inherit` | Inherit from parent folder/collection |
| `basic` | Username + password |
| `bearer` | Bearer token |
| `apikey` | API key (header or query param) |
| `digest` | HTTP Digest Auth |
| `oauth1` | OAuth 1.0 |
| `oauth2` | OAuth 2.0 |
| `awsv4` | AWS Signature Version 4 |
| `ntlm` | NTLM authentication |
| `wsse` | WSSE authentication |

Use `auth: inherit` to inherit the parent's auth configuration.

### Auth example

```yaml
http:
  auth:
    type: basic
    username: admin
    password: "{{password}}"
```

## Runtime

```yaml
runtime:
  variables:
    userId: "42"

  scripts:
    - type: before-request
      code: |
        bru.setVar("timestamp", Date.now());
    - type: after-response
      code: |
        console.log(res.status);
    - type: tests
      code: |
        expect(res.status).to.equal(200);
    - type: hooks
      code: |
        bru.setVar("hookExecuted", true);

  assertions:
    - expression: res.status
      operator: eq
      value: 200
      disabled: false
      description: "Status should be 200"

  actions:
    - type: set-variable
      name: userId
      selector: "$.data.id"    # jsonq selector
```

### Script types

| `scripts[].type` | When it runs |
|---|---|
| `before-request` | Before the HTTP call |
| `after-response` | After receiving the response |
| `tests` | During test/assertion phase |
| `hooks` | Collection-level hooks |

## Settings

```yaml
settings:
  encodeUrl: true          # HTTP/GraphQL
  timeout: 30000           # HTTP/GraphQL (ms), WebSocket (ms)
  followRedirects: true    # HTTP/GraphQL
  maxRedirects: 10         # HTTP/GraphQL
  keepAliveInterval: 5000  # WebSocket only (ms)
```

Values can be booleans/numbers or the string `"inherit"` to inherit from the parent.

## Validation Methods

1. **yamllint** — YAML syntax validation.
2. **prettier --check** — Formatting validation.
3. **JSON Schema** — Validate against `https://schema.opencollection.com/opencollection/v1.0.0.json`.
4. **Bruno UI** — Open the collection in Bruno as the final validation.

## Complete Request Examples

### HTTP GET

```yaml
info:
  name: List Users
  type: http
  seq: 1

http:
  method: GET
  url: https://api.example.com/users
  headers:
    - name: Accept
      value: application/json
  params:
    - name: limit
      value: "10"
      type: query

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "OK"
    - expression: res.body.length
      operator: gt
      value: 0
      description: "Users returned"

settings:
  encodeUrl: true
  timeout: 10000
```

### HTTP POST JSON with Bearer Auth

```yaml
info:
  name: Create Post
  type: http
  seq: 2

http:
  method: POST
  url: https://api.example.com/posts
  headers:
    - name: Content-Type
      value: application/json
  body:
    type: json
    data: |
      {
        "title": "Hello World",
        "body": "This is a test post.",
        "userId": 1
      }
  auth:
    type: bearer
    token: "{{api_token}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 201
      description: "Created"
    - expression: res.body.title
      operator: eq
      value: "Hello World"
      description: "Title matches"
```

### HTTP form-urlencoded

```yaml
info:
  name: Login
  type: http
  seq: 3

http:
  method: POST
  url: https://api.example.com/auth/login
  headers:
    - name: Content-Type
      value: application/x-www-form-urlencoded
  body:
    type: form-urlencoded
    data: |
      username=admin
      password={{password}}

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "Login success"
```

### HTTP SSE

```yaml
info:
  name: Stream Events
  type: http
  seq: 4

http:
  method: GET
  url: https://api.example.com/events
  headers:
    - name: Accept
      value: text/event-stream
    - name: Authorization
      value: "Bearer {{token}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "SSE connection established"
```

> **Note:** Collection Runner and `bru run` skip SSE requests. Test individually.

### HTTP SOAP (XML body)

```yaml
info:
  name: SOAP GetWeather
  type: http
  seq: 5

http:
  method: POST
  url: https://api.example.com/weather
  headers:
    - name: Content-Type
      value: text/xml
  body:
    type: xml
    data: |
      <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
        <soap:Body>
          <GetWeather xmlns="http://example.com/weather">
            <City>London</City>
          </GetWeather>
        </soap:Body>
      </soap:Envelope>

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "SOAP response OK"
```

### GraphQL Query

```yaml
info:
  name: Get User
  type: graphql
  seq: 6

graphql:
  method: POST
  url: https://api.example.com/graphql
  headers:
    - name: Authorization
      value: "Bearer {{token}}"
  body:
    query: |
      query GetUser($id: ID!) {
        user(id: $id) {
          name
          email
          posts {
            id
            title
          }
        }
      }
    variables: '{"id": "42"}'
  auth:
    type: bearer
    token: "{{api_token}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "GraphQL OK"
    - expression: res.body.data.user.name
      operator: eq
      value: "John Doe"
      description: "User name matches"
```

### gRPC Unary

```yaml
info:
  name: Get User
  type: grpc
  seq: 7

grpc:
  url: https://api.example.com:443
  method: user.UserService/GetUser
  methodType: unary
  protoFilePath: ./proto/user.proto
  metadata:
    - name: Authorization
      value: "Bearer {{token}}"
  message: |
    {
      "id": "42"
    }
  auth:
    type: bearer
    token: "{{api_token}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 0
      description: "gRPC OK"
```

### WebSocket JSON Message

```yaml
info:
  name: WebSocket Chat
  type: websocket
  seq: 8

websocket:
  url: wss://api.example.com/ws
  headers:
    - name: Authorization
      value: "Bearer {{token}}"
  message:
    type: json
    data: |
      {"action": "subscribe", "channel": "general"}
  auth:
    type: bearer
    token: "{{api_token}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "WebSocket connected"
```

## Assertions Example

```yaml
runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "Status code is 200"
    - expression: res.body.id
      operator: defined
      description: "Response body has id field"
    - expression: res.body.items.length
      operator: gt
      value: 0
      description: "Has items"
    - expression: res.headers['content-type']
      operator: contains
      value: application/json
      description: "Content-Type is JSON"
    - expression: res.body.data
      operator: match
      value: "^[A-Z].*"
      description: "Data starts with uppercase"
```

### Assertion operators

| Operator | Description |
|---|---|
| `eq` | Equals |
| `neq` | Not equals |
| `gt` | Greater than |
| `gte` | Greater than or equal |
| `lt` | Less than |
| `lte` | Less than or equal |
| `contains` | String contains |
| `not_contains` | String does not contain |
| `starts_with` | String starts with |
| `ends_with` | String ends with |
| `match` | Regex match |
| `not_match` | Regex not match |
| `defined` | Value is defined (not null/undefined) |
| `undefined` | Value is undefined |
| `null` | Value is null |
| `not_null` | Value is not null |
| `empty` | Value is empty |
| `not_empty` | Value is not empty |
| `in` | Value in array |
| `not_in` | Value not in array |