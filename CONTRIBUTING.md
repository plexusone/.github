# Contributing to plexusone projects

Thank you for your interest in contributing!

## Definition of Done

All PRs must satisfy the [Definition of Done](CLAUDE.md#definition-of-done) before merge. This includes passing tests, linting, documentation updates, RMI trailers, and changelog entries.

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Create a feature branch
4. Make your changes
5. Run tests and linting
6. Submit a pull request

## Work Tracking (VisionStudio)

Many plexusone repos register initiatives and roadmap items (RMIs) in
[visionstudio](https://github.com/ProductBuildersHQ/visionstudio). Before starting
work on a roadmap item:

```bash
visionstudio work ready               # Find claimable RMIs
visionstudio work claim <RMI-ID>      # Claim before starting
visionstudio work complete <RMI-ID>   # When done
```

Commits implementing roadmap items must carry the `Refs: RMI-<REPOSLUG>-<NNN>`
trailer (a trailer, not the subject line).

## Development Requirements

- Go 1.26 or later
- golangci-lint

## Code Style

- Run `gofmt` for formatting
- Run `golangci-lint run` before committing
- Follow [Effective Go](https://go.dev/doc/effective_go) guidelines

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add new feature
fix: resolve bug
docs: update documentation
test: add tests
refactor: restructure code
chore: maintenance tasks
```

## Pull Request Process

1. Ensure tests pass
2. Ensure linting passes
3. Update documentation if needed
4. Add changelog entry if applicable
5. Include RMI trailer if implementing a roadmap item
6. Request review from maintainers

## Questions?

Open an issue for questions or discussions.
