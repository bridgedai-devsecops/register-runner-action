# BridgedAI Register Runner Action

Register the CI runner executing a workflow job with BridgedAI.

## Usage

```yaml
- uses: bridgedai/register-runner-action@v1
  with:
    organization-id: org_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    mode: observe  # observe | production | mock
```

### Production mode (direct API registration)

```yaml
- uses: bridgedai/auth-action@v1
  id: auth
  with:
    organization-id: ${{ vars.BRIDGED_ORG_ID }}

- uses: bridgedai/register-runner-action@v1
  with:
    organization-id: ${{ vars.BRIDGED_ORG_ID }}
    access-token: ${{ steps.auth.outputs.access-token }}
    mode: production
    build-id: ${{ steps.upload.outputs.build-id }}
    fail-on-error: 'false'
```

## Modes

| Mode | Behavior |
|------|----------|
| `observe` (default) | Writes `runner.used` to `.bridgedai/signals.jsonl`; ingest via `upload-evidence-action` |
| `production` | Calls `POST /v1/runners/register` plus local signal |
| `mock` | Deterministic outputs, no network |

## Security

- Does not collect secrets or print sensitive environment variables
- Supports `dry-run: true` in production mode
- Supports `fail-on-error` for fail-open (default) vs fail-closed

## Correlation

Works with:

- `collect-context-action` (also emits `runner.used` — safe when dedupe is enabled; see below)
- `upload-evidence-action`
- `release-gate-action`
- `deployment-guard-action`

### Deduplication with collect-context-action

Both actions emit `runner.used` with the same canonical idempotency key:

`github-actions:{repository}:{workflowRunId}:{runAttempt}:{jobName}:runner.used`

The backend dedupes on ingest and on `POST /v1/runners/register`. Using both actions in one job is supported.

### When to use which

- **collect-context-action** — full workflow context bundle (recommended when you already collect context).
- **register-runner-action** — runner-only registration or direct API registration in production mode.

## API endpoint

`POST /v1/runners/register` on `https://api.bridgedai.io`

Required permission: `evidence:upload` or `projects:update`.
