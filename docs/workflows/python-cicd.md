# Python Application CI/CD Pipeline

A complete CI/CD pipeline for Python applications with automated deployment based on branch names.

## Location

`.github/workflows/ci-cd-pipeline.yml`

## Features

- **Linting**: Automated code quality checks with flake8
- **Testing**: Unit tests with pytest
- **Automated Deployment**: Branch-based deployment strategy
  - `dev`, `feature/*`, `bug/*`, `refactor/*` branches → Development environment
  - `test` branch → Test environment
  - `main` branch → Production environment
- **HTTP Health Checks**: Validates service is responding after deployment
- **Production Approval**: Manual approval required for production deployments
- **Slack Notifications**: Optional deployment status notifications
- **Concurrency Control**: Prevents conflicting deployments

## Quick Start

In your application repository, create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - dev
      - 'feature/**'
      - 'bug/**'
      - 'refactor/**'
      - test
      - main
  pull_request:
    branches:
      - dev
      - test
      - main

jobs:
  ci_cd:
    uses: rmskone-org/shared-workflows/.github/workflows/ci-cd-pipeline.yml@main
    with:
      app_name: 'your-app-name'        # Required: Used for systemctl service name
      app_port: '8000'                  # Optional: Default is 7868
      python_version: '3.12'            # Optional: Default is 3.12
    secrets: inherit
```

## Input Parameters

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `app_name` | Yes | - | Application service name (used in systemctl, inventory paths) |
| `app_port` | No | `7868` | Port for health checks |
| `python_version` | No | `3.12` | Python version to use |
| `environment_hostnames` | No | See below | Custom hostnames for each environment |
| `health_check_path` | No | `/health` | HTTP endpoint path for health checks |
| `slack_webhook_url` | No | - | Slack webhook URL for deployment notifications |

### Default Hostnames

- **Development**: `dev01`
- **Test**: `test01`
- **Production**: `prod01`

### Custom Hostnames Example

```yaml
with:
  environment_hostnames: 'dev=custom-dev01,test=custom-test01,prod=custom-prod01'
```

## Required Secrets

Configure these secrets in your repository or organization settings:

- `OPENAI_API_KEY`: OpenAI API key for deployment (if required by your app)

To use all secrets from calling repo, add `secrets: inherit` to your workflow.

## Self-Hosted Runners

Each environment requires a self-hosted runner with the appropriate label:

- **Development**: `[self-hosted, dev]`
- **Test**: `[self-hosted, test]`
- **Production**: `[self-hosted, prod]`

[Learn how to set up self-hosted runners →](https://docs.github.com/en/actions/hosting-your-own-runners)

## Repository Structure

Your application repository should have:

```
your-app/
├── .github/
│   └── workflows/
│       └── ci-cd.yml           # Calls the reusable workflow
├── ansible/
│   ├── inventory/
│   │   ├── development/
│   │   │   └── hosts.yml
│   │   ├── test/
│   │   │   └── hosts.yml
│   │   └── production/
│   │       └── hosts.yml
│   └── playbooks/
│       └── deploy.yml          # Deployment playbook
├── tests/                      # Your pytest tests
├── requirements.txt            # Python dependencies
├── requirements-dev.txt        # Development dependencies (optional)
└── (your application code)
```

## Pipeline Stages

### 1. Linting (runs on: ubuntu-latest)

- Installs flake8
- Runs syntax error checks (E9, F63, F7, F82)
- Runs code quality checks (complexity, line length)

### 2. Testing (runs on: ubuntu-latest)

- Creates Python virtual environment
- Installs dependencies from `requirements.txt` and `requirements-dev.txt`
- Runs pytest with verbose output
- Fails the pipeline if tests fail

### 3. Deployment (runs on: self-hosted runners)

**Environment Selection**:
- Branches matching `dev`, `feature/*`, `bug/*`, `refactor/*` → Development
- Branch `test` → Test
- Branch `main` → Production (requires approval)

**Deployment Steps**:
1. Checkout code
2. Parse environment hostnames
3. Install Ansible
4. Run Ansible deployment playbook
5. Verify service status with systemctl
6. Check recent logs with journalctl
7. Perform HTTP health check on configured port
8. Send Slack notification (if configured)

## Production Deployments

Production deployments require manual approval. Configure this in your repository:

1. Go to **Settings** → **Environments**
2. Create environment named `production`
3. Add **Required reviewers** (team members who must approve)
4. Optionally add **Wait timer** for delayed deployments
5. Save protection rules

The workflow will pause before deploying to production and wait for approval.

## Health Checks

The workflow performs two types of health checks:

1. **Service Status**: Checks if systemd service is active
2. **HTTP Check**: Validates the application responds on the configured port

### Custom Health Check Path

If your app uses a different health endpoint:

```yaml
with:
  health_check_path: '/api/v1/health'
```

## Slack Notifications

To receive deployment notifications in Slack:

1. Create a Slack Incoming Webhook
2. Add webhook URL as workflow input:

```yaml
with:
  slack_webhook_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
```

Notifications include:
- Deployment status (success/failure)
- Repository and branch
- Commit SHA
- Triggering actor

## Troubleshooting

### Deployment Fails: "ansible directory not found"

Ensure your repository has an `ansible/` directory with the required structure.

### Health Check Fails

- Verify your app exposes the health check endpoint (default: `/health`)
- Check the port matches your app configuration
- Review service logs: `journalctl -u your-app-name -n 50`

### Tests Fail to Install Dependencies

- Ensure `requirements.txt` exists
- Check all dependencies are available in PyPI
- Consider adding `requirements-dev.txt` for test-only dependencies

### Production Deployment Doesn't Trigger

- Verify you've created the `production` environment in repo settings
- Check that you're pushing to the `main` branch
- Ensure tests passed successfully

## Advanced Configuration

### Using Different Python Versions per Environment

You can't vary Python version per environment with this workflow, but you can:
1. Create separate workflow files for different configurations
2. Fork this workflow and customize per repository

### Custom Ansible Variables

Pass additional variables to your Ansible playbook by modifying the workflow in your repo or extending the reusable workflow.

### Rollback Strategy

This workflow doesn't include automated rollback. Consider:
1. Using Ansible's built-in rollback features
2. Keeping previous deployment artifacts
3. Implementing blue-green deployments in your Ansible playbooks

## Examples

### Minimal Configuration

```yaml
jobs:
  ci_cd:
    uses: rmskone-org/shared-workflows/.github/workflows/ci-cd-pipeline.yml@main
    with:
      app_name: 'api-service'
    secrets: inherit
```

### Full Configuration

```yaml
jobs:
  ci_cd:
    uses: rmskone-org/shared-workflows/.github/workflows/ci-cd-pipeline.yml@main
    with:
      app_name: 'api-service'
      app_port: '8080'
      python_version: '3.11'
      environment_hostnames: 'dev=dev-api01,test=test-api01,prod=prod-api01'
      health_check_path: '/api/health'
      slack_webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    secrets: inherit
```

## Related Documentation

- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Using Environments for Deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Ansible Deployment Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
