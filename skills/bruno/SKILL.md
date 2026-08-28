---
name: bruno
description: Work with the Bruno API client and OpenCollection YAML collections. Use when creating or editing Bruno collections, OpenCollection YAML files, Bruno CLI commands, API request tests, secrets, variables, or GitHub Actions e2e API test workflows.
---

# Bruno Skill

## Quick Start

Minimal OpenCollection YAML collection tree:

```
my-collection/
├── opencollection.yml
├── environments/
│   └── local.yml
└── requests/
    └── get-users.yml
```

**opencollection.yml**
```yaml
name: My Collection
```

**environments/local.yml**
```yaml
variables:
  base_url:
    value: http://localhost:3000
```

**requests/get-users.yml**
```yaml
info:
  name: Get Users
  type: http

http:
  method: GET
  url: {{base_url}}/users
  headers:
    - name: Accept
      value: application/json

runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: 200
      description: "Status should be 200"
```

Run: `bru run --env local`

## Workflows

1. **Create a new OpenCollection YAML collection** — New collection → choose YAML → add requests as `.yml` files under `requests/`.
2. **Validate OpenCollection YAML syntax and structure** — Run `yamllint`, then `prettier --check`, optionally validate against JSON Schema `https://schema.opencollection.com/opencollection/v1.0.0.json`, then open in Bruno.
3. **Run a collection for a specific environment** — `bru run --env <env>` or `bru run --env-var key=value` for overrides.
4. **Wire Bruno CLI into GitHub Actions** — Use `usebruno/bruno-cli-action@v1` with `command: run --env prod`; auto-injects JUnit reporter; outputs `exit-code`, `passed`, `failed`, `total`, `duration-ms`.
5. **Create workspace/collection/folder/request/test in Bruno v3+** — Workspace: open folder with `workspace.yml`. Collection: `opencollection.yml` or `bruno.json`. Folder: `folder.yml`. Request: `.yml`. Test: `runtime.assertions` (declarative) or `runtime.scripts` with `type: tests` (Chai JS).
6. **Decide whether a value should be a variable or a secret** — Variable = non-sensitive, committable. Secret = sensitive, never committed. Use `secret: true` in YAML, `.env` + `.gitignore`, or external managers.
7. **Protect secrets when sharing or committing** — Encrypted storage (OS-level or AES256). Masked in reports. Never commit values; commit only names.

## CLI vs curl

Use `bru` CLI **only** for running Bruno collections (especially OpenCollection YAML) against an environment. For one-off HTTP calls, use `curl`, `wget`, or similar.

| Use case | Tool |
|---|---|
| Run a Bruno collection | `bru run` |
| Run a single API call | `curl` / `wget` |

## References

- [OpenCollection YAML](references/opencollection-yaml.md) — Request YAML schema, auth/body/script types, validation methods, protocol examples.
- [Bruno CLI](references/bru-cli.md) — Install, run commands, reporters, GitHub Actions workflow.
- [Bruno Basics](references/bruno-basics.md) — Workspace/collection/folder/request/test basics and v3+ requirements.
- [Secrets and Variables](references/secrets-and-variables.md) — Variable scope/precedence, typed variables, secret options, masking, decision rule.