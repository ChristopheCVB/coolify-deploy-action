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

### Deploy by Tag

```yaml
- name: Deploy by Tag
  uses: christophecvb/deploy-coolify-action@v1
  with:
    token: ${{ secrets.COOLIFY_API_TOKEN }}
    domain: 'your-coolify-domain.com'
    tag: 'v1.0.0'
```

### Deploy by Pull Request

```yaml
- name: Deploy by PR
  uses: christophecvb/deploy-coolify-action@v1
  with:
    token: ${{ secrets.COOLIFY_API_TOKEN }}
    domain: 'your-coolify-domain.com'
    pr: '123'
```

### Deploy Multiple Tags

```yaml
- name: Deploy Multiple Tags
  uses: christophecvb/deploy-coolify-action@v1
  with:
    token: ${{ secrets.COOLIFY_API_TOKEN }}
    domain: 'your-coolify-domain.com'
    tag: 'v1.0.0,v1.1.0'
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
    interval: 20
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | Coolify API token | ✅ Yes | - |
| `domain` | Coolify domain (without https://) | ✅ Yes | - |
| `app_uuid` | Resource UUID(s). Comma separated list is also accepted. | ❌ No* | - |
| `tag` | Tag name(s) to deploy. Comma separated list is also accepted | ❌ No* | - |
| `pr` | Pull Request Id for deploying specific PR builds. Cannot be used with tag parameter | ❌ No* | - |
| `force` | Force rebuild (without cache) | ❌ No | `false` |
| `waitForDeploy` | Wait for deployment to complete | ❌ No | `false` |
| `timeout` | Timeout in seconds for deployment waiting | ❌ No | `300` |
| `interval` | Interval in seconds for deployment waiting | ❌ No | `10` |

*At least one of `app_uuid`, `tag`, or `pr` must be provided. `tag` and `pr` cannot be used together.

## Features

- **Multiple Deployment Methods**: Deploy by UUID, tag, or pull request
- **Simple Deployment**: Just trigger a deployment without waiting
- **Deployment Waiting**: Optionally wait for deployment completion
- **Status Monitoring**: Monitors deployment status and provides feedback
- **Timeout Handling**: Configurable timeout for deployment waiting
- **Error Handling**: Proper error handling and exit codes
- **Detailed Logging**: Comprehensive logging for debugging
- **Parameter Validation**: Validates required parameters and prevents conflicts

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

## Example Workflows

### Release Deployment

```yaml
name: Deploy on Release

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
          tag: ${{ github.ref_name }}
          force: false
          waitForDeploy: true
          timeout: 600
          interval: 20
```

### Pull Request Deployment

```yaml
name: Deploy PR

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy PR
        uses: christophecvb/deploy-coolify-action@v1
        with:
          token: ${{ secrets.COOLIFY_API_TOKEN }}
          domain: ${{ vars.COOLIFY_DOMAIN }}
          pr: ${{ github.event.number }}
          force: true
          waitForDeploy: true
          timeout: 300
          interval: 10
```

### Manual Deployment

```yaml
name: Manual Deploy

on:
  workflow_dispatch:
    inputs:
      app_uuid:
        description: 'Application UUID'
        required: true
      force:
        description: 'Force rebuild'
        required: false
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Application
        uses: christophecvb/deploy-coolify-action@v1
        with:
          token: ${{ secrets.COOLIFY_API_TOKEN }}
          domain: ${{ vars.COOLIFY_DOMAIN }}
          app_uuid: ${{ github.event.inputs.app_uuid }}
          force: ${{ github.event.inputs.force }}
          waitForDeploy: true
          timeout: 600
          interval: 20
```

## Requirements

- `jq` must be available in the runner environment (included in Ubuntu runners)
- Valid Coolify API token with deployment permissions
- At least one of the following:
  - Valid application UUID, or
  - Valid tag name(s), or
  - Valid pull request ID 