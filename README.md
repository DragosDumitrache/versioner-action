# Versioner Action

A GitHub Action that calculates semantic versions from git tags and branches using the [versioner script](https://github.com/DragosDumitrache/versioner).

## Features

- ✅ **Proper semver**: `v1.2.3` on main, `v1.2.3-dev.feature.abc123` on branches
- ✅ **Configuration via version.json**: No action inputs needed for most use cases
- ✅ **Pre-release detection**: Automatic identification of development builds
- ✅ **Branch aware**: Different behavior for main vs feature branches
- ✅ **Script version pinning**: Use specific versions of the versioning logic

## Quick Start

### 1. Create version.json in your repository

```json
{
  "default_branch": "main",
  "major": "1",
  "minor": "0",
  "tag_prefix": "v",
  "include_v_prefix": true
}
```

### 2. Use in your workflow

```yaml
name: Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # ⚠️ Required for version calculation

      - name: Calculate Version
        id: version
        uses: DragosDumitrache/versioner-action@v3.0.0

      - name: Build with version
        run: |
          echo "Building version: ${{ steps.version.outputs.version }}"
          echo "Is pre-release: ${{ steps.version.outputs.is-prerelease }}"
```

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `version` | Full calculated version | `v1.2.3` or `v1.2.3-dev.feature.abc123` |
| `clean-version` | Version without v prefix | `1.2.3` |
| `is-prerelease` | Whether this is a pre-release | `true`/`false` |

## Configuration

All configuration is done via `version.json` in your repository root:

| Field | Description | Example |
|-------|-------------|---------|
| `default_branch` | Main branch name | `"main"` or `"master"` |
| `major` | Minimum major version | `"1"` |
| `minor` | Minimum minor version | `"2"` |
| `tag_prefix` | Prefix for git tags | `"v"` or `""` |
| `include_v_prefix` | Include v in output version | `true` or `false` |

## Examples

### Basic Usage

```yaml
- name: Calculate Version
  id: version
  uses: DragosDumitrache/versioner-action@v3.0.0

- name: Build Docker Image
  run: |
    docker build -t myapp:${{ steps.version.outputs.version }} .
```

### Pin Script Version

```yaml
- name: Calculate Version
  id: version
  uses: DragosDumitrache/versioner-action@v3.0.0
```

### Conditional Deployment

```yaml
- name: Calculate Version
  id: version
  uses: DragosDumitrache/versioner-action@v3.0.0

- name: Deploy to Production
  if: steps.version.outputs.is-prerelease == 'false'
  run: |
    echo "Deploying ${{ steps.version.outputs.version }} to production"

- name: Deploy to Staging  
  if: steps.version.outputs.is-prerelease == 'true'
  run: |
    echo "Deploying ${{ steps.version.outputs.version }} to staging"
```

### Publishing Packages

```yaml
- name: Calculate Version
  id: version
  uses: DragosDumitrache/versioner-action@v3.0.0

- name: Update package.json
  run: |
    npm version ${{ steps.version.outputs.clean-version }} --no-git-tag-version

- name: Publish to NPM
  if: steps.version.outputs.is-prerelease == 'false'
  run: npm publish

- name: Publish Pre-release
  if: steps.version.outputs.is-prerelease == 'true'
  run: npm publish --tag beta
```

### Matrix Builds with Versioning

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0

  - name: Calculate Version
    id: version
    uses: DragosDumitrache/versioner-action@v3.0.0

  - name: Build for ${{ matrix.os }}
    run: |
      echo "Building ${{ steps.version.outputs.version }} on ${{ matrix.os }}"
```

## Version Behavior

### Main Branch
- Clean versions: `v1.2.3`, `v2.0.0`
- Based on latest tag + commits since tag
- Respects version constraints from `version.json`

### Feature Branches
- Pre-release versions: `v1.2.3-dev.feature-auth.abc123`
- Shows target version when merged to main
- Never deployed to production (use `is-prerelease` check)

### Version Constraints
```json
// If latest tag is v0.8.5 but your version.json has:
{
  "major": "1",
  "minor": "2"
}
// Output will be: v1.2.0 (enforces minimums)

// If latest tag is v2.5.0 with same version.json:
// Output will be: v2.5.1 (respects existing higher version)
```

## Requirements

⚠️ **Important**: You must checkout with full git history:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # This is required!
```

## Error Handling

The action will fail with clear error messages if:
- Repository is not checked out with full history
- `version.json` is malformed
- Git repository is not properly initialized

## Related

- 📦 **[Versioner Script](https://github.com/DragosDumitrache/versioner)** - The core versioning logic
- 📖 **[Semantic Versioning](https://semver.org/)** - Version format specification
- 🏃 **[GitHub Actions](https://docs.github.com/en/actions)** - CI/CD platform

## Contributing

Issues and PRs welcome! This action is a thin wrapper around the [versioner script](https://github.com/DragosDumitrache/versioner), so core logic improvements should be made there.

## License

MIT
