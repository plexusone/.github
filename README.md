# .github

Shared GitHub Actions workflows and organization-wide configurations for [PlexusOne](https://github.com/plexusone).

## Reusable Workflows

### Go

#### go-ci.yaml

Cross-platform Go testing with matrix support.

```yaml
jobs:
  ci:
    uses: plexusone/.github/.github/workflows/go-ci.yaml@main
    with:
      modules: '["omnillm", "omnistorage"]'  # Multi-module support
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-versions` | `["1.26.x"]` | JSON array of Go versions |
| `platforms` | `["ubuntu-latest", "macos-latest", "windows-latest"]` | JSON array of platforms |
| `modules` | `["."]` | JSON array of module directories |
| `test-flags` | `-v -covermode=count` | Additional `go test` flags |

#### go-lint.yaml

Go linting with golangci-lint.

```yaml
jobs:
  lint:
    uses: plexusone/.github/.github/workflows/go-lint.yaml@main
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-version` | `1.26.x` | Go version |
| `golangci-lint-version` | `latest` | golangci-lint version |
| `modules` | `["."]` | JSON array of module directories |
| `timeout` | `3m` | Lint timeout |
| `args` | `--verbose` | Additional golangci-lint arguments |

#### go-sast-codeql.yaml

Security scanning with GitHub CodeQL.

```yaml
jobs:
  sast:
    uses: plexusone/.github/.github/workflows/go-sast-codeql.yaml@main
```

| Input | Default | Description |
|-------|---------|-------------|
| `go-version` | `1.26.x` | Go version |
| `queries` | `security-extended,security-and-quality` | CodeQL query suites |
| `modules` | `["."]` | JSON array of module directories |

### Swift

#### swift-ci.yaml

Swift package building and testing.

```yaml
jobs:
  ci:
    uses: plexusone/.github/.github/workflows/swift-ci.yaml@main
```

| Input | Default | Description |
|-------|---------|-------------|
| `xcode-version` | `latest-stable` | Xcode version |
| `macos-version` | `macos-14` | macOS runner version |
| `working-directory` | `.` | Directory containing Package.swift |
| `test-flags` | `` | Additional `swift test` flags |

## Usage

### Single Module (default)

```yaml
name: Go CI
on: [push, pull_request]

jobs:
  ci:
    uses: plexusone/.github/.github/workflows/go-ci.yaml@main
```

### Multi-Module Repository

```yaml
name: Go CI
on:
  push:
    paths:
      - 'module1/**'
      - 'module2/**'

jobs:
  ci:
    uses: plexusone/.github/.github/workflows/go-ci.yaml@main
    with:
      modules: '["module1", "module2"]'
```

### Custom Go Versions

```yaml
jobs:
  ci:
    uses: plexusone/.github/.github/workflows/go-ci.yaml@main
    with:
      go-versions: '["1.25.x", "1.26.x"]'
      platforms: '["ubuntu-latest"]'
```

## Versioning

Use `@main` for latest or pin to a specific tag:

```yaml
uses: plexusone/.github/.github/workflows/go-ci.yaml@v0.2.0
```
