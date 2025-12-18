# Conventional Commits Validator Action

A GitHub composite action that validates commit messages and PR titles follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

## Description

This action ensures consistent and meaningful commit messages across your projects by enforcing the Conventional Commits specification. It validates:

- **Commit messages**: All commits in a Pull Request
- **PR titles**: The title of the Pull Request itself

Using standardized commit messages enables:

- 🔄 Automated versioning (semantic versioning)
- 📝 Automatic CHANGELOG generation
- 🔍 Better searchability in git history
- 📊 Easier code review and understanding

## Conventional Commits Format

```markdown
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Examples

```markdown
feat: add user authentication
fix(api): resolve database connection timeout
docs: update README with installation steps
style(css): format styles with prettier
refactor(auth): simplify token validation logic
perf(database): optimize query execution
test(user): add unit tests for user service
build(deps): upgrade ansible-core to 2.16
ci: add workflow for automated testing
chore: update .gitignore
revert: revert "feat: add feature X"
```

### Breaking Changes

```markdown
feat!: remove deprecated API endpoint

BREAKING CHANGE: The /api/v1/old endpoint has been removed.
Use /api/v2/new instead.
```

## Features

- ✅ Validates commit messages and PR titles
- 🎯 Configurable commit types
- 📏 Customizable subject length limits
- 🔧 Optional scope requirement
- 📊 Detailed validation reports
- ⚙️ Warning mode (non-blocking validation)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `validate_commits` | Validate all commits in the PR | No | `true` |
| `validate_pr_title` | Validate PR title | No | `true` |
| `allowed_types` | Comma-separated list of allowed types | No | `feat,fix,docs,style,refactor,perf,test,build,ci,chore,revert` |
| `require_scope` | Require scope in commit messages | No | `false` |
| `min_subject_length` | Minimum subject length | No | `3` |
| `max_subject_length` | Maximum subject length | No | `100` |
| `fail_on_error` | Fail workflow on validation error | No | `true` |

## Standard Commit Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat: add login button` |
| `fix` | Bug fix | `fix: resolve null pointer exception` |
| `docs` | Documentation changes | `docs: update API documentation` |
| `style` | Code style changes (formatting) | `style: apply black formatter` |
| `refactor` | Code refactoring | `refactor: simplify auth logic` |
| `perf` | Performance improvements | `perf: optimize database queries` |
| `test` | Adding or updating tests | `test: add user service tests` |
| `build` | Build system or dependencies | `build: update Ansible version` |
| `ci` | CI/CD changes | `ci: add GitHub Actions workflow` |
| `chore` | Maintenance tasks | `chore: clean up old files` |
| `revert` | Revert previous commit | `revert: revert commit abc123` |

## Usage

### Basic Usage

Validate commits and PR title:

```yaml
name: Validate Commits
on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  conventional-commits:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required to get full commit history

      - name: Validate Conventional Commits
        uses: ignaciost/gha-ignaciostactions/conventional-commits@main
```

### Validate Only PR Title

```yaml
- name: Validate PR Title
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    validate_commits: 'false'
    validate_pr_title: 'true'
```

### Validate Only Commits

```yaml
- name: Validate Commits
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    validate_commits: 'true'
    validate_pr_title: 'false'
```

### Custom Allowed Types

For Ansible projects, you might want additional types:

```yaml
- name: Validate with Custom Types
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    allowed_types: 'feat,fix,docs,style,refactor,test,chore,role,playbook'
```

### Require Scope

Enforce that all commits include a scope:

```yaml
- name: Validate with Required Scope
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    require_scope: 'true'
```

Examples with required scope:

- ✅ `feat(auth): add OAuth support`
- ✅ `fix(database): resolve connection pool leak`
- ❌ `feat: add new feature` (missing scope)

### Custom Subject Length

```yaml
- name: Validate with Length Limits
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    min_subject_length: '10'
    max_subject_length: '72'
```

### Warning Mode (Non-Blocking)

Useful during initial adoption or for informational purposes:

```yaml
- name: Validate Commits (Warning Only)
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main
  with:
    fail_on_error: 'false'
```

### Complete Example

```yaml
name: PR Validation
on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

jobs:
  validate-conventional-commits:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: read
      contents: read
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Validate Conventional Commits
        uses: ignaciost/gha-ignaciostactions/conventional-commits@main
        with:
          validate_commits: 'true'
          validate_pr_title: 'true'
          allowed_types: 'feat,fix,docs,style,refactor,perf,test,build,ci,chore,revert'
          require_scope: 'false'
          min_subject_length: '3'
          max_subject_length: '100'
          fail_on_error: 'true'
```

## Validation Rules

The action enforces these rules by default:

1. **Type**: Must be one of the allowed types (lowercase)
2. **Scope**: Optional, but must be lowercase if present
3. **Subject**: 
   - Must be lowercase
   - Minimum 3 characters
   - Maximum 100 characters
   - Should not end with a period
4. **Header**: Maximum 150 characters total

## Integration with Semantic Release

This action pairs perfectly with semantic-release for automated versioning:

```yaml
- name: Validate Commits
  uses: ignaciost/gha-ignaciostactions/conventional-commits@main

- name: Semantic Release
  uses: cycjimmy/semantic-release-action@v4
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Local Development Setup

### Git Hook with Commitlint

Create `.commitlintrc.js`:

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert'
      ]
    ]
  }
};
```

Install Husky for commit hooks:

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional husky

# Setup husky
npx husky init
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
```

### VS Code Extension

Install the [Conventional Commits](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits) extension for VS Code to help write properly formatted commits.

## Common Validation Errors

### ❌ Missing Type

```markdown
add new feature
```

**Fix:**

```markdown
feat: add new feature
```

### ❌ Invalid Type

```markdown
added: new login button
```

**Fix:**

```markdown
feat: add new login button
```

### ❌ Uppercase Subject

```markdown
feat: Add new feature
```

**Fix:**

```markdown
feat: add new feature
```

### ❌ Subject Too Short

```markdown
feat: add
```

**Fix:**

```markdown
feat: add user authentication
```

### ❌ Missing Colon

```markdown
feat add new feature
```

**Fix:**

```markdown
feat: add new feature
```

### ❌ Period at End

```markdown
feat: add new feature.
```

**Fix:**

```markdown
feat: add new feature
```

## Benefits

### Automated Versioning

Commits determine version bumps:

- `feat:` → Minor version bump (0.1.0 → 0.2.0)
- `fix:` → Patch version bump (0.1.0 → 0.1.1)
- `BREAKING CHANGE:` → Major version bump (0.1.0 → 1.0.0)

### Automatic CHANGELOG

Generate changelogs grouped by type:

```markdown
# Changelog

## [1.2.0] - 2025-12-18

### Features
- add user authentication
- add password reset functionality

### Bug Fixes
- resolve database connection timeout
- fix null pointer in user service

### Documentation
- update README with new API endpoints
```

### Better Git History

```bash
# Find all features added
git log --oneline --grep="^feat"

# Find all bug fixes
git log --oneline --grep="^fix"

# Find breaking changes
git log --oneline --grep="BREAKING CHANGE"
```

## Adoption Strategy

### Phase 1: Warning Mode (Week 1-2)

Enable validation but don't fail builds:

```yaml
with:
  fail_on_error: 'false'
```

### Phase 2: PR Title Only (Week 3-4)

Enforce PR titles only:

```yaml
with:
  validate_commits: 'false'
  validate_pr_title: 'true'
  fail_on_error: 'true'
```

### Phase 3: Full Enforcement (Week 5+)

Enforce all commits and PR titles:

```yaml
with:
  validate_commits: 'true'
  validate_pr_title: 'true'
  fail_on_error: 'true'
```

## Troubleshooting

### "No commits found to validate"

Ensure you're using `fetch-depth: 0` in checkout action:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

### "Action runs but doesn't validate"

Make sure the workflow triggers on PR events:

```yaml
on:
  pull_request:
    types: [opened, edited, synchronize]
```

### "Scope validation too strict"

If scope requirement is too restrictive:

```yaml
with:
  require_scope: 'false'
```

## Related Resources

- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [Commitlint](https://commitlint.js.org/)
- [Semantic Versioning](https://semver.org/)
- [Semantic Release](https://semantic-release.gitbook.io/)

## Contributing

Have suggestions for improvement? Please open an issue or PR!

## License

See LICENSE file in the root of this repository.
