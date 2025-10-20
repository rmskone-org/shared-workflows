# Shared Workflows

Central repository for reusable GitHub Actions workflows and templates used across the organization.

## 🚀 Quick Start

### Using a Workflow Template

1. Navigate to your repository
2. Click **Actions** → **New workflow**
3. Look for organization templates in the suggestions
4. Click **Set up this workflow**
5. Customize and commit

### Using a Reusable Workflow

Reference workflows from this repository in your workflow files:

```yaml
jobs:
  my_job:
    uses: rmskone-org/shared-workflows/.github/workflows/WORKFLOW_NAME.yml@main
    with:
      # your inputs here
    secrets: inherit
```

## 📚 Available Workflows

### Reusable Workflows

| Workflow | Description | Documentation |
|----------|-------------|---------------|
| **Python CI/CD Pipeline** | Complete lint, test, and deployment pipeline for Python apps | [View Docs](docs/workflows/python-cicd.md) |

### Workflow Templates

| Template | Use Case | Auto-detected For |
|----------|----------|-------------------|
| **Python Application CI/CD** | Python apps with automated deployment | Projects with `requirements.txt`, `setup.py`, or `pyproject.toml` |

## 🏗️ Repository Structure

```
shared-workflows/
├── README.md                          # This file
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml         # Reusable Python CI/CD workflow
├── workflow-templates/                 # Templates that appear in Actions UI
│   ├── python-app-cicd.yml
│   └── python-app-cicd.properties.json
└── docs/
    └── workflows/                      # Detailed workflow documentation
        └── python-cicd.md
```

## 📖 Documentation

### For Workflow Users

- **[Python CI/CD Pipeline](docs/workflows/python-cicd.md)** - Complete guide for using the Python deployment workflow
  - Input parameters
  - Required secrets
  - Self-hosted runner setup
  - Repository structure requirements
  - Troubleshooting

### For Workflow Maintainers

- **Adding a New Workflow**: Create in `.github/workflows/` and document in `docs/workflows/`
- **Creating a Template**: Add to `workflow-templates/` with `.properties.json` metadata
- **Testing Changes**: Use a feature branch and reference it in test repos before merging

## 🔧 Common Setup Tasks

### Configure Production Environment

For workflows that deploy to production:

1. Go to your repo **Settings** → **Environments**
2. Create `production` environment
3. Add required reviewers
4. Configure deployment branch rules

### Add Organization Secrets

For secrets needed across multiple repositories:

1. Go to organization **Settings** → **Secrets and variables** → **Actions**
2. Add secrets (e.g., `OPENAI_API_KEY`)
3. Select which repositories can access them

### Set Up Self-Hosted Runners

For deployment workflows:

1. Go to organization **Settings** → **Actions** → **Runners**
2. Add runner with appropriate labels (`dev`, `test`, `prod`)
3. Configure runner on target environment servers

## 🤝 Contributing

### Adding a New Reusable Workflow

1. Create workflow in `.github/workflows/`
2. Use `workflow_call` trigger with clear inputs
3. Document in `docs/workflows/YOUR-WORKFLOW.md`
4. Update this README's table
5. Test with at least one repository
6. Submit PR for review

### Creating a Workflow Template

1. Create `workflow-templates/TEMPLATE-NAME.yml`
2. Create `workflow-templates/TEMPLATE-NAME.properties.json` with metadata:
   ```json
   {
     "name": "Display Name",
     "description": "Short description",
     "iconName": "icon-name",
     "categories": ["Language"],
     "filePatterns": ["pattern$"]
   }
   ```
3. Test that it appears in Actions UI
4. Submit PR for review

### Documentation Standards

- Each reusable workflow must have a dedicated doc in `docs/workflows/`
- Include: description, inputs, secrets, examples, troubleshooting
- Keep main README high-level with links
- Use tables for parameters and comparisons

## 🔍 Testing Workflows

### Test Reusable Workflow Changes

Create a test branch and reference it:

```yaml
uses: rmskone-org/shared-workflows/.github/workflows/ci-cd-pipeline.yml@feature/my-changes
```

### Validate Template Metadata

Templates appear in the Actions UI based on:
- Repository file patterns (defined in `.properties.json`)
- Template categories

Test by creating a new workflow in a matching repository.

## 📋 Support

### Getting Help

- **Documentation Issues**: Check `docs/workflows/` for detailed guides
- **Workflow Bugs**: Open an issue in this repository
- **Questions**: Contact DevOps team or open a discussion

### Reporting Issues

When reporting issues, include:
- Workflow name and version (branch/tag)
- Repository where issue occurred
- Error messages and logs
- Workflow run URL

## 📜 Best Practices

1. **Always use `@main` for stable workflows** - Don't pin to commits unless necessary
2. **Use `secrets: inherit`** - Simplifies secret management
3. **Test in dev environment first** - Use feature branches for deployment testing
4. **Document custom inputs** - Add comments in your workflow files
5. **Keep workflows DRY** - Use reusable workflows instead of copying code

## 🔒 Security

- **Never commit secrets** - Use GitHub Secrets or environment variables
- **Review workflow changes** - Malicious workflow changes can access secrets
- **Limit secret access** - Only grant secrets to repositories that need them
- **Use environment protection** - Require approval for production deployments

## 📌 Version History

- **v1.0** (2025-10-20): Initial Python CI/CD pipeline with deployment automation

---

**Maintained by**: DevOps Team
**Organization**: rmskone-org
