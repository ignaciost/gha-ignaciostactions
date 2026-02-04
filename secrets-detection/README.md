# Secrets Detection Action

A GitHub composite action that scans your repository for hardcoded secrets, passwords, API keys, and credentials using industry-standard tools.

## Description

This action helps prevent accidental exposure of sensitive information by running two complementary security scanning tools:

- **detect-secrets**: A modular Python tool by Yelp that prevents new secrets from entering the codebase
- **gitleaks**: A SAST tool for detecting and preventing hardcoded secrets like passwords, API keys, and tokens

## Features

- 🔍 Dual-tool scanning for comprehensive coverage
- 🐍 Configurable Python version for detect-secrets
- 📋 Baseline support to track known/approved secrets
- 🔧 Customizable gitleaks version
- ⚙️ Independent tool execution (run one or both)
- 🚨 Configurable failure behavior
- 📁 Flexible scan path configuration

## Inputs

| Input | Description | Required | Default |
| ----- | ----------- | -------- | ------- |
| `python-version` | Python version for detect-secrets | No | `3.11` |
| `run_detect_secrets` | Enable or disable detect-secrets | No | `true` |
| `run_gitleaks` | Enable or disable gitleaks | No | `true` |
| `gitleaks_version` | Version of gitleaks to use | No | `v8.18.0` |
| `detect_secrets_baseline` | Path to baseline file | No | `.secrets.baseline` |
| `fail_on_secrets` | Fail workflow if secrets found | No | `true` |
| `scan_path` | Path to scan | No | `.` |

## Prerequisites

### Optional Configuration Files

- `.secrets.baseline`: Baseline file for detect-secrets (generated automatically if not present)
- `.gitleaks.toml`: Custom gitleaks configuration (optional)

## Usage

### Basic Usage

Scan your repository with both tools:

```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  secrets-detection:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for gitleaks to scan full history

      - name: Detect Secrets
        uses: ignaciost/gha-ignaciostactions/secrets-detection@main
```

### Run Only detect-secrets

```yaml
- name: Detect Secrets with detect-secrets
  uses: ignaciost/gha-ignaciostactions/secrets-detection@main
  with:
    run_gitleaks: 'false'
```

### Run Only Gitleaks

```yaml
- name: Detect Secrets with Gitleaks
  uses: ignaciost/gha-ignaciostactions/secrets-detection@main
  with:
    run_detect_secrets: 'false'
```

### Custom Baseline Path

```yaml
- name: Detect Secrets
  uses: ignaciost/gha-ignaciostactions/secrets-detection@main
  with:
    detect_secrets_baseline: 'config/.secrets.baseline'
```

### Scan Specific Directory

```yaml
- name: Scan Source Directory
  uses: ignaciost/gha-ignaciostactions/secrets-detection@main
  with:
    scan_path: 'src/'
```

### Warning Mode (Don't Fail on Detection)

Useful for initial setup or informational scans:

```yaml
- name: Detect Secrets (Warning Only)
  uses: ignaciost/gha-ignaciostactions/secrets-detection@main
  with:
    fail_on_secrets: 'false'
```

### Complete Example with All Options

```yaml
name: Comprehensive Security Scan
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  secrets-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for gitleaks

      - name: Detect Secrets and Credentials
        uses: ignaciost/gha-ignaciostactions/secrets-detection@main
        with:
          python-version: '3.11'
          run_detect_secrets: 'true'
          run_gitleaks: 'true'
          gitleaks_version: 'v8.18.0'
          detect_secrets_baseline: '.secrets.baseline'
          fail_on_secrets: 'true'
          scan_path: '.'
```

### Pre-commit Hook Integration

For local development, add to `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

## How It Works

### detect-secrets

1. **First Run**: Creates a baseline file containing all current "secrets"
2. **Subsequent Runs**: Compares new code against baseline
3. **Detection**: Flags any new potential secrets not in baseline
4. **Review**: Allows you to mark false positives in baseline

### gitleaks

1. **Scans**: Git history and current files
2. **Detection**: Uses regex patterns and entropy detection
3. **Configuration**: Supports custom rules via `.gitleaks.toml`
4. **Reporting**: Provides detailed findings with line numbers

## Common Patterns Detected

- 🔑 API keys and tokens
- 🔐 Passwords and credentials
- 🎫 OAuth tokens
- 📧 SMTP credentials
- 🗄️ Database connection strings
- 🌐 Private keys (RSA, SSH, etc.)
- 💳 Credit card numbers
- 📱 AWS/Azure/GCP credentials

## Handling False Positives

### False Positives in detect-secrets

Add to baseline or use inline comments:

```python
password = "example"  # pragma: allowlist secret
API_KEY = "not-a-real-key"  # nosec
```

### False Positives in gitleaks

Use `.gitleaks.toml` to configure allow lists:

```toml
[allowlist]
paths = [
    '''\.secrets.baseline''',
    '''test/fixtures/.*'''
]

regexes = [
    '''example\.com'''
]
```

## Best Practices

1. **Run in CI/CD**: Integrate into all pipelines
2. **Block PRs**: Set `fail_on_secrets: 'true'` for production branches
3. **Review Baseline**: Regularly audit `.secrets.baseline`
4. **Full History**: Use `fetch-depth: 0` for comprehensive scans
5. **Pre-commit**: Add local hooks for faster feedback
6. **Rotate Secrets**: If detected, rotate immediately
7. **Educate Team**: Train developers on secure coding practices

## Troubleshooting

### "Too many secrets detected"

- Review and update baseline: `detect-secrets audit .secrets.baseline`
- Add false positives to allowlist
- Use inline comments for known test values

### "Gitleaks found secrets in history"

- Secrets in git history require repository cleaning:

  ```bash
  git filter-branch --force --index-filter \
    "git rm --cached --ignore-unmatch PATH-TO-FILE" \
    --prune-empty --tag-name-filter cat -- --all
  ```

- Consider using BFG Repo-Cleaner for large histories

### Performance Issues

- Limit scan path: `scan_path: 'src/'`
- Use shallow clone when possible
- Cache Python dependencies

## Security Note

⚠️ **Important**: This action helps prevent accidental commits of secrets, but it's not foolproof:

- Always use proper secrets management (GitHub Secrets, Vault, etc.)
- Never commit real credentials, even temporarily
- Rotate any exposed secrets immediately
- Use environment variables for sensitive data
- Follow principle of least privilege

## Related Tools

- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [git-secrets](https://github.com/awslabs/git-secrets)

## Contributing

Found a pattern that should be detected? Please open an issue or PR!

## License

See LICENSE file in the root of this repository.
