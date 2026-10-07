# AGENTS.md — plexusone

Organization-wide guidelines for coding agents across all plexusone repositories.

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

## CI Gotchas

- **`go.mod`'s `go` directive can't outrun CI's cached toolchain.** Every
  repo on the shared reusable `go-ci.yaml` workflow
  (`plexusone/.github/.github/workflows/go-ci.yaml`, called via
  `uses: plexusone/.github/.github/workflows/go-ci.yaml@main`) uses
  `actions/setup-go@v7` with `go-version: "1.26.x"` — it resolves that to
  whatever patch it last cached and pins `GOTOOLCHAIN=local`, so it does
  **not** auto-upgrade to a newer patch just because `go.mod` asks for
  one. Bumping `go.mod`'s `go` directive past CI's cached version (e.g.
  to pick up a `govulncheck`-found stdlib CVE fix) breaks every build on
  that workflow with `go: go.mod requires go >= X.Y.Z (running X.Y.W;
  GOTOOLCHAIN=local)`. Verify what CI actually resolves to before
  bumping — don't just bump-and-push and assume `GOTOOLCHAIN=auto`
  semantics apply. The durable fix is adding `check-latest: true` to the
  shared workflow's `actions/setup-go` step (not yet done, as of
  2026-08-17); until then, expect this to recur whenever a new Go patch
  ships with a fix you want.

## JSON & Naming Conventions

Rule zero: **external specs always win** — match the wire format of what you're
implementing (Anthropic API: snake_case; SARIF, OTLP/JSON: camelCase; MCP tool
params: snake_case; OTel semconv attributes: dot-namespaced snake_case).

For formats we own:

| Surface | Convention | Example |
|---------|-----------|---------|
| API/document JSON property names | camelCase | `ruleId`, `conformanceLevels` |
| API URL paths | kebab-case | `/style-profiles/{id}` |
| Custom telemetry event JSON | snake_case | `session_id`, `input_tokens` |
| OTel attributes/metrics | follow semconv (dot namespaces, snake_case words) | `gen_ai.usage.input_tokens` |
| YAML config keys | kebab-case | `severity-overrides` |

Notes:

- The telemetry snake_case convention matches the Anthropic API wire format
  our session/usage events mirror (`agentpair`, `omniagent`, `omnillm-evals`).
- OTel casing is never a choice: semconv defines attribute keys
  (dot-delimited namespaces, snake_case words within a segment); OTLP/JSON
  wire keys are camelCase but emitted by the SDK/exporter — never
  hand-authored.
- `schemakit lint`: pass `--property-case` matching the surface being
  linted — `camelCase` for API/document schemas (e.g. `api-style-spec`),
  `snake_case` for telemetry event schemas.

## VisionStudio Integration

<!-- Shared by the org guidelines (ProductBuildersHQ, grokify, plexusone): keep this section identical. -->

Initiative and work tracking lives in
[visionstudio](https://github.com/ProductBuildersHQ/visionstudio), a
DoltDB-backed app (CLI, daemon, web UI, MCP). Its build-progress artifact
types (Initiative, Phase, RMI) live in
[prism-build](https://github.com/ProductBuildersHQ/prism-build). Repos may
be registered in visionstudio. When working on registered repos:

- `visionstudio work ready` — list RMIs that are ready, unblocked, and
  unclaimed
- `visionstudio work claim <RMI-ID>` — claim before starting work (prints
  the git trailer to carry)
- Carry `Refs: RMI-<REPOSLUG>-<NNN>` trailer on commits (trailer, not
  subject line)
- `visionstudio work complete <RMI-ID>` — when done
- `work claim-phase` / `work complete-phase` do the same for a whole phase;
  `work status` lists active assignments and `work release` returns a claim
- Before picking an RMI number block, check
  `visionstudio rmi list --repo <repo>` — cross-repo initiatives may already
  hold IDs under a repo's slug

## Initiative Lifecycle, Quality & Efficiency (Operating Model)

The canonical operating model lives in the
[ProductBuildersHQ guidelines](https://github.com/ProductBuildersHQ/.github/blob/main/AGENTS.md)
(`~/go/src/github.com/ProductBuildersHQ/.github/AGENTS.md`, section
"Initiative Lifecycle, Quality & Efficiency") — read it when doing
initiative-tracked work. The habits that apply in every plexusone repo:

- Initiatives are tracked in the `visionstudio` binary (`INIT-<SLUG>-NNN`,
  workflow `pbhq-lite`); specs at
  `docs/specs/initiatives/{INIT-ID}/{PRD,TRD,PLAN,ROADMAP}.md`; commits
  carry `Refs: RMI-<REPOSLUG>-<NNN>` trailers.
- Transition initiatives promptly; `released` = acceptance testing passed —
  its timestamp anchors all quality measurement. Record releases in
  visionstudio (repo + version) at the same moment you update
  `CHANGELOG.json` and tag.
- Label defect issues **`bug`** (GitHub default); `enhancement` = demand
  signal, not a defect. `fix:` commits reference their issue (`Fixes #N`).
- Defects after the acceptance mark are **escaped defects**; external
  reporters make them customer-found (CFD). Tiering is automatic from
  labels, authors, and timestamps.
- Code review (CI reviewers, local agents) gates acceptance, not the local
  iteration loop, and is retained only while measured benefit justifies its
  cost.
- **We are early adopters.** Most plexusone repo history (some spanning
  ~10 years) predates RMI tracking — never infer a historical
  release-to-initiative match from date proximity. Going forward, every
  release record carries its initiative/RMI IDs; backfilling old history
  is a distinct AI-assisted activity requiring human confirmation per
  match (see the ProductBuildersHQ guidelines, section "AI-assisted
  historical backfill matching").

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
