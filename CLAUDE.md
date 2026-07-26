# CLAUDE.md — plexusone

Organization-wide guidelines for Claude Code across all plexusone repositories.

## Definition of Done

All PRs must satisfy before merge:

| Category | Criteria |
|----------|----------|
| **Code** | Implementation complete, follows existing patterns in the repo |
| **Tests** | Unit tests for new functions; integration tests if behavior crosses packages |
| **Lint** | `golangci-lint run` passes with no issues |
| **Docs** | README/MkDocs updated if user-visible behavior changes |
| **Changelog** | Entry added, or commit follows conventional commits for auto-generation |
| **RMI Trailer** | Commits carry `Refs: RMI-<REPOSLUG>-<NNN>` when implementing roadmap items |
| **Pre-push** | No local `replace` directives in go.mod, no references to untracked files |

## Standard Stack

- **Go version**: 1.26+
- **ORM**: Ent (`entgo.io/ent`) for MySQL-compatible databases
- **CLI**: Cobra (`github.com/spf13/cobra`)
- **MCP servers**: Official Go SDK (`github.com/modelcontextprotocol/go-sdk`)
- **Linting**: golangci-lint
- **Formatting**: gofmt
- **Commit style**: [Conventional Commits](https://www.conventionalcommits.org/)
- **Merge strategy**: Rebase-merge or merge commit only — squash merge is disabled to preserve conventional commit history

## PRISM Control Integration

Plexusone repos may be registered in [prism-control](https://github.com/ProductBuildersHQ/prism-control). When working on registered repos:

- Use `prismctl work ready --repo <repo>` to find claimable work
- Use `prismctl work claim <RMI-ID>` before starting work
- Carry `Refs: RMI-<REPOSLUG>-<NNN>` trailer on commits (trailer, not subject line)
- Use `prismctl work complete <RMI-ID>` when done

## Architecture Principles

- **Library-first**: Put as much code as possible in reusable packages (importable SDK), with thin CLI/MCP adapters over one shared service layer
- **Prefer unit tests over integration tests**
- **Specs before implementation**: New projects specced in `docs/specs/{PRD.md,TRD.md,PLAN.md,ROADMAP.md}`

## Error Handling (Go)

Follow the priority order in the global CLAUDE.md:
1. Panic (invariant violation)
2. t.Fatal (in tests)
3. Return error
4. Log via slog
5. Report to human

Never silently discard errors with `_`.

## Excluded from Auto-Read

Never read files that may contain secrets:
- `.envrc`, `.env`, `.env.*`
- `credentials.json`, `secrets.json`, `*.pem`, `*.key`
