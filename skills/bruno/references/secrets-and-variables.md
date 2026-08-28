# Secrets and Variables Reference

> Priorities 4 & 5: Variable system overview, secret management, and decision rule.

## Variable Types and Scope

Bruno v4 supports the following variable types:

| Type | Scope | Description |
|---|---|---|
| Global Environment | All collections | Available across all collections in a workspace |
| Environment | Collection-specific | Tied to a specific environment |
| Collection | Collection-wide | Available within a collection |
| Folder | Folder-scoped | Available within a folder |
| Request | Request-scoped | Available only in a specific request |
| Runtime | Execution-time | Set during script execution |
| Prompt | UI-only | Prompted at runtime in the GUI |
| Process Environment | System | `process.env.VAR` — OS-level environment variables |

### Precedence (high to low)

```
Runtime > Request > Folder > Environment > Collection > Global
```

- `{{?Prompt String}}` and `{{process.env.VAR}}` use special syntax and do **not** participate in normal precedence.

## v4 Typed Variables

Bruno v4 introduces typed variables:

| Type | Description |
|---|---|
| `string` | String value (default) |
| `number` | Numeric value |
| `boolean` | Boolean value |
| `object` | JSON object |

### YAML Format

For non-string values, use `type` and `data` fields:

```yaml
variables:
  timeout:
    type: number
    data: 30000
  debug:
    type: boolean
    data: true
  config:
    type: object
    data:
      maxRetries: 3
      baseUrl: https://api.example.com
```

### Setting Typed Variables in Scripts

```javascript
bru.setEnvVar("timeout", 30000);    // infers type: number
bru.setEnvVar("debug", true);       // infers type: boolean
bru.setEnvVar("config", { maxRetries: 3 });  // infers type: object
```

`bru.setEnvVar(key, value)` infers the type from the value.

## Interpolation

### Syntax

```yaml
{{variableName}}
```

### Recursive Resolution

Variables can reference other variables:

```yaml
base_url: https://api.example.com
endpoint: {{base_url}}/users
```

Bruno resolves `{{endpoint}}` by first resolving `{{base_url}}`, then combining.

### Object Dot Notation

```yaml
config:
  type: object
  data:
    baseUrl: https://api.example.com
    timeout: 30000

# In request:
url: {{config.baseUrl}}
```

### Array Indexing

```yaml
urls:
  type: object
  data:
    - https://api.example.com
    - https://api2.example.com

# In request:
url: {{urls[0]}}
```

## When to Use a Variable vs a Secret

### Use a Variable when:

- The value is **not sensitive** — base URLs, non-secret IDs, feature flags, environment names.
- The value needs to be **shared** across team members.
- The value is **committed to version control**.

### Use a Secret when:

- The value is **sensitive** — API keys, passwords, tokens, database connection strings with credentials.
- The value **must not be committed** or exported.
- The value is **environment-specific** and must be stored securely.

### Decision Rule

> **If the value would be a security risk if leaked, use a secret. Otherwise, use a variable.**

## Secret Management Options

### 1. Secret Variables

Mark a variable as secret in the environment UI. Stored encrypted locally, not written to the environment file, and excluded on export.

In YAML format, the environment file lists only the secret variable name with `secret: true` and an empty value:

```yaml
variables:
  api_key:
    secret: true
    value: ""
  base_url:
    secret: false
    value: https://api.example.com
```

### 2. `.env` File

Place `.env` at the collection root; reference via `{{process.env.VAR_NAME}}`.

```bash
# .env
API_KEY=sk-1234567890abcdef
BASE_URL=https://api.example.com
```

```yaml
# In request YAML
http:
  url: {{process.env.BASE_URL}}/users
  headers:
    - name: Authorization
      value: "Bearer {{process.env.API_KEY}}"
```

**Rules:**
- Always add `.env` to `.gitignore`.
- Quote values containing `#`, `\n`, `"`, or `\`.
- Use bracket notation for names with dots: `{{process.env['example.test']}}`.

### 3. External Secret Managers

Mentioned for awareness. Full provider setup is out of scope:

- **HashiCorp Vault**
- **AWS Secrets Manager**
- **Azure Key Vault**
- **Google Cloud Secret Manager**

These are typically integrated via `process.env` or custom scripts.

## Secret Protection

### Storage

- Secrets are stored encrypted using **OS-level encryption** when available (e.g., macOS Keychain, Windows Credential Manager).
- Falls back to **AES256** encryption.

### Commit Rules

- **Never commit secret values** — rely on secret variables, `.env` + `.gitignore`, or external managers.
- Commit only the variable names, not the values.

## Secret Masking

Sensitive headers and values are automatically masked in reports:

### Headers that are always masked

- `authorization`
- `x-api-key`
- `cookie`
- `set-cookie`
- `x-auth-token`
- `client-secret`

### What gets masked

- Secret environment variables
- External secrets
- `.env` values
- Sensitive headers

### Applies to

- HTML reports
- JSON reports
- JUnit reports

## Complete Secret Example

### Environment YAML with secrets

```yaml
# environments/prod.yml
variables:
  base_url:
    secret: false
    value: https://api.example.com
  api_key:
    secret: true
    value: ""
  db_password:
    secret: true
    value: ""
```

### Request using secrets

```yaml
info:
  name: Get Users
  type: http

http:
  method: GET
  url: {{base_url}}/users
  headers:
    - name: Authorization
      value: "Bearer {{api_key}}"
  auth:
    type: bearer
    token: "{{api_key}}"

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
```

### Environment YAML with `.env` reference

```yaml
# environments/local.yml
variables:
  base_url:
    secret: false
    value: http://localhost:3000
```

And in `.env`:

```bash
# .env
API_KEY=sk-test-key
DB_PASSWORD=secret123
```

Request uses `{{process.env.API_KEY}}` to reference these.

## Variable Precedence Example

Given:
- Collection: `base_url: https://api.example.com`
- Environment: `base_url: https://api-staging.example.com`
- Request: `base_url: https://api-dev.example.com`

The request will use `https://api-dev.example.com` (Request > Environment > Collection).