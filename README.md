# apt-bundle-action

[![CI](https://github.com/apt-bundle/apt-bundle-action/actions/workflows/test.yml/badge.svg)](https://github.com/apt-bundle/apt-bundle-action/actions/workflows/test.yml)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A GitHub Action for declarative APT package management using [apt-bundle](https://github.com/apt-bundle/apt-bundle).

Define your system dependencies in an `Aptfile` and install them with a single action. Features built-in caching for faster CI runs.

## Quick Start

1. Create an `Aptfile` in your repository:

```
apt curl
apt jq
apt vim
```

2. Add the action to your workflow:

```yaml
- uses: apt-bundle/apt-bundle-action@v1
```

That's it! Your packages will be installed with automatic caching.

## Usage

### Basic Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: apt-bundle/apt-bundle-action@v1
```

### Custom Aptfile Location

```yaml
- uses: apt-bundle/apt-bundle-action@v1
  with:
    file: './config/Aptfile'
```

### Aptfile.lock (reproducible installs)

If an `Aptfile.lock` file exists in the same directory as your Aptfile, the action uses it automatically: it runs `apt-bundle install --locked`, installing only the pinned versions from the lock file. If there is no lock file, it installs from the Aptfile as usual.

To generate or update the lock file locally, run:

```bash
apt-bundle lock
```

Commit both `Aptfile` and `Aptfile.lock` for reproducible CI installs. The cache key is based on the lock file when present, so cache invalidates when the lock changes.

### Disable Caching

```yaml
- uses: apt-bundle/apt-bundle-action@v1
  with:
    cache: 'false'
```

### Mode

Control what the action does with a single `mode` option:

| Value | Behavior |
|-------|----------|
| `install` | Run `apt-get update`, then install packages (from Aptfile.lock if present, else Aptfile) (default) |
| `install-no-update` | Install without `apt-get update` (from Aptfile.lock if present, else Aptfile) |
| `check` | Only verify that packages are installed; do not install anything |

```yaml
# Verify only (e.g. in a job that assumes deps are already present)
- uses: apt-bundle/apt-bundle-action@v1
  with:
    mode: 'check'

# Install without updating package indexes (faster when index is fresh)
- uses: apt-bundle/apt-bundle-action@v1
  with:
    mode: 'install-no-update'
```

### Pin apt-bundle Version

```yaml
- uses: apt-bundle/apt-bundle-action@v1
  with:
    version: 'v0.1.0'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `file` | Path to Aptfile | No | `Aptfile` |
| `mode` | `install`, `install-no-update`, or `check` | No | `install` |
| `version` | apt-bundle version to use | No | `latest` |
| `cache` | Enable caching of apt packages | No | `true` |
| `cache-key-prefix` | Custom prefix for cache key | No | `apt-bundle` |

## Outputs

| Output | Description |
|--------|-------------|
| `cache-hit` | Whether the cache was restored (`true` or `false`) |

## Aptfile Syntax

The `Aptfile` supports several directives:

### Basic Packages

```
apt curl
apt git
apt build-essential
```

### Version Pinning

```
apt "nginx=1.18.0-0ubuntu1"
```

### PPAs (Personal Package Archives)

```
ppa ppa:ondrej/php
apt php8.2
```

### Custom Repositories

```
key https://download.docker.com/linux/ubuntu/gpg
deb "[arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
apt docker-ce
```

For complete syntax documentation, see the [apt-bundle documentation](https://apt-bundle.org).

## Caching

This action caches downloaded `.deb` packages in `/var/cache/apt/archives`. The cache key is based on:

- The runner OS
- When `Aptfile.lock` exists next to your Aptfile: the lock file contents (reproducible installs)
- Otherwise: your Aptfile contents

On cache hit, `apt-get install` reuses cached packages instead of re-downloading, significantly speeding up subsequent runs.

### Cache Key Format

```
{cache-key-prefix}-{runner.os}-{sha256 of Aptfile or Aptfile.lock}
```

When `Aptfile.lock` is present, the hash is of the lock file so cache invalidates when pinned versions change.

### Custom Cache Key Prefix

Use a custom prefix to isolate caches between workflows:

```yaml
- uses: apt-bundle/apt-bundle-action@v1
  with:
    cache-key-prefix: 'my-project-apt'
```

## Examples

### Full Workflow Example

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: apt-bundle/apt-bundle-action@v1

      - name: Build
        run: make build
```

### Matrix Testing with apt-bundle

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-22.04, ubuntu-24.04]
    steps:
      - uses: actions/checkout@v4

      - uses: apt-bundle/apt-bundle-action@v1

      - name: Run tests
        run: make test
```

## Requirements

- Runs on Ubuntu runners only (`ubuntu-latest`, `ubuntu-22.04`, `ubuntu-24.04`)
- Requires `sudo` access (available by default on GitHub-hosted runners)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [apt-bundle](https://github.com/apt-bundle/apt-bundle) - The CLI tool this action wraps
- [apt-bundle.org](https://apt-bundle.org) - Official documentation
