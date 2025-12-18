# Lint Action

A GitHub composite action that performs linting and validation on Ansible projects using `yamllint` and `ansible-lint`.

## Description

This action helps maintain code quality in Ansible projects by running two linting tools:

- **yamllint**: Validates YAML syntax and formatting
- **ansible-lint**: Checks Ansible playbooks, roles, and collections against best practices run-ansible-lint[https://github.com/marketplace/actions/run-ansible-lint]

## Features

- 🐍 Configurable Python version
- 📦 Automatic dependency installation with pip caching
- 🧹 Optional yamllint execution
- ✅ Optional ansible-lint execution
- 🔧 Customizable ansible-lint version

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `python-version` | Python version to use for the environment | No | `3.11` |
| `ansible_lint_version` | Version of ansible-lint to use (e.g., v25.5.0) | No | `v25.5.0` |
| `run_yamllint` | Enable or disable yamllint execution | No | `true` |
| `run_ansible_lint` | Enable or disable ansible-lint execution | No | `true` |

## Prerequisites

Your repository must contain:

- `requirements.txt`: Python dependencies (should include `ansible-core`, `yamllint`, etc.)
- `requirements.yml`: Ansible Galaxy dependencies (collections and roles)

## Usage

### Basic Usage

Add this action to your workflow:

```yaml
name: Lint
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Ansible Lint
        uses: ignaciost/gha-ignaciostactions/ansible-lint@main
```

### Custom Python Version

```yaml
- name: Run Ansible Lint
  uses: ignaciost/gha-ignaciostactions/ansible-lint@main
  with:
    python-version: '3.12'
```

### Specific ansible-lint Version

```yaml
- name: Run Ansible Lint
  uses: ignaciost/gha-ignaciostactions/ansible-lint@main
  with:
    ansible_lint_version: 'v24.2.0'
```

### Run Only yamllint

```yaml
- name: Run Only YAML Lint
  uses: ignaciost/gha-ignaciostactions/ansible-lint@main
  with:
    run_ansible_lint: 'false'
```

### Run Only ansible-lint

```yaml
- name: Run Only Ansible Lint
  uses: ignaciost/gha-ignaciostactions/ansible-lint@main
  with:
    run_yamllint: 'false'
```

### Complete Example with All Options

```yaml
name: Comprehensive Lint
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Lint Ansible Project
        uses: ignaciost/gha-ignaciostactions/ansible-lint@main
        with:
          python-version: '3.11'
          ansible_lint_version: 'v25.5.0'
          run_yamllint: 'true'
          run_ansible_lint: 'true'
```

## How It Works

1. **Sets up Python**: Configures the specified Python version with pip caching enabled for faster subsequent runs
2. **Installs dependencies**: Updates pip and installs all packages from `requirements.txt`
3. **Runs yamllint** (if enabled): Validates YAML syntax across all `.yml` and `.yaml` files with GitHub-formatted output
4. **Runs ansible-lint** (if enabled): Validates Ansible content against best practices, processing Galaxy dependencies from `requirements.yml`

## Output

- **yamllint**: Reports YAML syntax errors and style violations with file locations
- **ansible-lint**: Reports Ansible-specific issues, deprecated features, and best practice violations

Both tools integrate with GitHub's annotation system, displaying issues directly in the pull request interface.

## Example requirements.txt

```txt
ansible-core>=2.15.0
yamllint>=1.35.0
```

## Example requirements.yml

```yaml
---
collections:
  - name: ansible.posix
    version: ">=1.5.0"
  - name: community.general
    version: ">=8.0.0"
```

## Troubleshooting

### yamllint fails
- Check your `.yamllint` configuration file if present
- Ensure all YAML files have proper syntax

### ansible-lint fails
- Verify `requirements.yml` is present and valid
- Check ansible-lint rules configuration in `.ansible-lint` if present
- Review the verbose output for specific rule violations

## License

This action is part of the gha-ignaciostactions collection.

## Contributing

Contributions are welcome! Please ensure any changes maintain backward compatibility.
