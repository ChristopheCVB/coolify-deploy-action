# Deploy to Coolify Action

A reusable GitHub Action for deploying applications to Coolify with optional deployment waiting.

## Usage

### Basic Usage

```yaml
- name: Deploy to Coolify
  uses: christophecvb/deploy-coolify-action@v1
  with:
    token: ${{ secrets.COOLIFY_API_TOKEN }}
    domain: 'your-coolify-domain.com'
    app_uuid: 'your-app-uuid'
```

### Advanced Usage with Waiting

```yaml
- name: Deploy and Wait
  uses: christophecvb/deploy-coolify-action@v1
  with:
    token: ${{ secrets.COOLIFY_API_TOKEN }}
    domain: 'your-coolify-domain.com'
    app_uuid: 'your-app-uuid'
    force: true
    waitForDeploy: true
    timeout: 600
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | Coolify API token | ✅ Yes | - |
| `domain` | Coolify domain (without https://) | ✅ Yes | - |
| `app_uuid` | Application UUID to deploy | ✅ Yes | - |
| `force` | Force rebuild (without cache) | ❌ No | `false` |
| `waitForDeploy` | Wait for deployment to complete | ❌ No | `false` |
| `timeout` | Timeout in seconds for deployment waiting | ❌ No | `300` |

## Features

- **Simple Deployment**: Just trigger a deployment without waiting
- **Deployment Waiting**: Optionally wait for deployment completion
- **Status Monitoring**: Monitors deployment status and provides feedback
- **Timeout Handling**: Configurable timeout for deployment waiting
- **Error Handling**: Proper error handling and exit codes
- **Detailed Logging**: Comprehensive logging for debugging

## Deployment Status

When `waitForDeploy` is enabled, the action monitors these deployment statuses:

### Successful Statuses
- `successful` - Deployment completed successfully ✅
- `finished` - Deployment finished successfully ✅

### Failed Statuses
- `failed` - Deployment failed ❌
- `cancelled` - Deployment was cancelled ❌
- `skipped` - Deployment was skipped ❌

### In Progress Statuses
- `running` - Deployment is currently running ⏳
- `pending` - Deployment is pending ⏳
- `queued` - Deployment is queued ⏳
- `in_progress` - Deployment is in progress ⏳
- `processing` - Deployment is being processed ⏳

### Unknown Statuses
- Any other status will be treated as unknown and the action will continue waiting ⚠️

## Example Workflow

```yaml
name: Deploy

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Application
        uses: christophecvb/deploy-coolify-action@v1
        with:
          token: ${{ secrets.COOLIFY_API_TOKEN }}
          domain: ${{ vars.COOLIFY_DOMAIN }}
          app_uuid: ${{ vars.COOLIFY_APP_UUID }}
          force: false
          waitForDeploy: true
          timeout: 600
```

## Requirements

- `jq` must be available in the runner environment (included in Ubuntu runners)
- Valid Coolify API token with deployment permissions
- Valid application UUID 