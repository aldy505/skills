# Bruno CLI Reference

> Priority 2: Running collections and wiring into CI/CD.

## Installation

### Node.js (recommended)

Requires Node.js 18+.

```bash
npm install -g @usebruno/cli
```

Or with pnpm/yarn:

```bash
pnpm add -g @usebruno/cli
yarn global add @usebruno/cli
```

Verify:

```bash
bru --version
```

### Docker

```bash
docker pull usebruno/cli:latest
```

### v3+ Safe Mode

Bruno CLI v3+ defaults to safe mode. For external npm packages or filesystem access, use `--sandbox=developer`.

## Core Commands

### Run entire collection

```bash
bru run
```

### Run with environment

```bash
bru run --env local
```

### Run with global env and workspace path

```bash
bru run --global-env prod --workspace-path /path/to/workspace
```

This is the scenario documented in Bruno's GitHub Actions guide.

### Run a single request

```bash
bru run requests/get-users.yml
```

### Run a folder

```bash
bru run folders/auth
```

### Run only tests

```bash
bru run --tests-only
```

### Tag filtering

```bash
bru run --tags users,auth
bru run --exclude-tags deprecated
```

### Stop on failure

```bash
bru run --bail
```

### Parallel execution

```bash
bru run --parallel
```

### Delay between requests

```bash
bru run --delay 1000
```

## Reports

### Reporter formats

```bash
bru run --reporter-json
bru run --reporter-junit
bru run --reporter-html
```

### Reporter masking options

```bash
bru run --reporter-skip-body
bru run --reporter-skip-request-body
bru run --reporter-skip-response-body
bru run --reporter-skip-all-headers
bru run --reporter-skip-headers "authorization,x-api-key"
```

## Environment Overrides

```bash
bru run --env-var key=value
bru run --global-env-var key=value
```

## Data-Driven Testing

```bash
bru run --csv-file-path data.csv
bru run --json-file-path data.json
```

> **Note:** In the GUI Collection Runner, CSV data-driven testing is a Pro/Ultimate-only feature. The CLI supports data-driven testing regardless of license.

## Scope Rule

Use `bru` CLI **only** for running Bruno collections against environments. For one-off HTTP calls, use `curl`, `wget`, or similar tools.

| Use case | Tool |
|---|---|
| Run a Bruno collection | `bru run` |
| Run a single API call | `curl` / `wget` |
| Test with environment variables | `bru run --env` |
| Quick API check | `curl` |

## GitHub Actions Integration

### Official Action

```yaml
uses: usebruno/bruno-cli-action@v1
```

### Key details

- `command` input is passed **without** the `bru` prefix (the action prepends `bru`).
- `working-directory` should be the collection root (folder containing `opencollection.yml` or `bruno.json`).
- `--reporter-junit` is auto-injected unless already present in the command.
- **Outputs**: `exit-code`, `passed`, `failed`, `total`, `duration-ms`.

### Complete Workflow Example

```yaml
name: E2E API Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  e2e-api:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Bruno CLI
        run: npm install -g @usebruno/cli

      - name: Run API Tests
        uses: usebruno/bruno-cli-action@v1
        with:
          command: run --env prod
          working-directory: ./collections/my-api

      - name: Upload HTML Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: bruno-report
          path: reports/

      - name: Upload JUnit Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: bruno-junit
          path: reports/junit.xml
```

### Workflow with HTML reporter explicitly

```yaml
      - name: Run API Tests with HTML Report
        uses: usebruno/bruno-cli-action@v1
        with:
          command: run --env prod --reporter-html
          working-directory: ./collections/my-api
```

## Complete Run Examples

### Run with env and bail

```bash
bru run --env prod --bail
```

### Run with tags and reporters

```bash
bru run --tags smoke --reporter-junit --reporter-html
```

### Run with data-driven testing

```bash
bru run --env local --csv-file-path test-data.csv --reporter-json
```

### Run with environment overrides

```bash
bru run --env dev --env-var BASE_URL=http://localhost:3000 --env-var API_KEY=test-key
```

### Run with sandbox mode

```bash
bru run --env dev --sandbox=developer
```