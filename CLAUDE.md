# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Perses also ships an `AGENTS.md` at the repo root written specifically for AI coding agents (architecture map,
engineering rules, validation commands, completion checklist). Read it — the guidance below is a Claude Code-focused
supplement, not a replacement.

## What this repo is

Perses is a CNCF dashboard/observability tool: a Go API server + CLI, CUE/Go Dashboard-as-Code SDKs, and a React (TS)
web UI. It combines multiple layers that must stay in sync when a resource shape changes: Go models → generated Go
clients → CUE schemas → Go SDK → UI/spec consumers.

## Commands

### Go / CUE (run from repo root)

```sh
make build          # build-ui + build-api + build-cli -> ./bin/perses, ./bin/percli
make build-api       # backend only (runs `generate` first: assets-compress + install-default-plugins + go generate)
make checkformat     # gofmt + cue fmt check
make checkstyle      # golangci-lint run
make checkunused     # go mod tidy, fails if go.mod/go.sum would change
make checklicense    # verify Apache license headers (fixlicense to add missing ones)
make cue-eval        # validate CUE schemas (cd cue && cue eval ./...)
make test            # go test ./... (runs `generate` first)
make integration-test # requires downloaded default plugins / CUE / Prometheus / DB services
go test ./path/to/package/...          # run a single package's tests
go test ./path/to/package/... -run TestName -v   # run a single test
make go-sdk-test     # cd go-sdk/test && go test ./...
```

Backend dev server: `bash scripts/api_backend_dev.sh` (quick setup), or `make build-api && ./bin/perses -config ./dev/config.yaml`, serving on :8080 (login `admin`/`password`).

### UI (run from `ui/`)

```sh
npm ci
npm run lint            # Oxlint across workspaces (turbo run lint)
npm run format:check    # Oxfmt check
npm run type-check      # tsc --noEmit across workspaces
npm run test            # turbo run test (vitest)
npm run test -w app -- <test-pattern>   # focused test run while iterating, single workspace
npm run start -w app    # dev server at http://localhost:3000 (needs backend running, see above)
npm run e2e              # Playwright end-to-end tests (app must be running)
npm run doctor           # full React Doctor scan (also runs in CI)
```

Run `npm run lint`, `npm run format:check`, and `npm run type-check` at the repo (`ui/`) level before finishing UI work, not just in the touched workspace.

### Docs

`make checkdocs` / `make fmt-docs` run mdox against Markdown outside `docs/` (which is owned by the separate
perses/website repo and rendered with MkDocs — do not mdox-format it).

## Architecture

- `cmd/perses/`, `cmd/percli/` — server and CLI entry points; keep thin, orchestration only.
- `internal/api/` — API implementation: routing (`route/`), persistence (`database/`, `archive/`), auth
  (`authorization/`), plugin loading (`plugin/`, `discovery/`), provisioning, and generated client endpoints
  (`internal/api/generate.go` drives `go generate`). Files with a generated-file notice must not be hand-edited —
  change the source and regenerate.
- `pkg/model/` — backend resource models and validation. Changing these can ripple into API compatibility, CUE
  schemas, the Go SDK, and the UI — check all four before considering a model change done.
- `pkg/client/` — generated Go API clients; never hand-edit.
- `go-sdk/` — public Go Dashboard-as-Code builders (`dashboard/`, `panel/`, `query/`, `variable/`, ...) and tests
  under `go-sdk/test/`.
- `cue/` — CUE schemas and utilities; files ending `_go_gen.cue` are generated from Go models via `make cue-gen`.
- `ui/app/` — the product React application (routes, admin, auth, projects). Organized as `views/` (feature/domain
  logic, mirrors routing), `components/` (reusable building blocks), `model/` (data models, fetch hooks, business
  logic), `context/` (app-wide state only).
- `ui/e2e/` — Playwright fixtures, page objects, critical-flow tests.
- `ui/internal-utils/` — private UI dev/test utilities.
- `internal/cli/cmd/plugin/generate/templates/` — templates used to scaffold new plugin modules; keep their
  Oxlint/Oxfmt config aligned with `ui/`.
- Reusable UI component libraries (`components`, `dashboards`, `explore`, `plugin-system`) now live in the separate
  **perses/shared** repo and are consumed from npm under `@perses-dev/*`; the canonical TS spec contracts live in
  **perses/spec**; official plugin implementations live in **perses/plugins**. Do not recreate these layers inside
  `ui/app` — if a change belongs there, it likely needs a companion PR in that other repo.
- The UI is an npm-workspaces + turborepo monorepo (`ui/` root plus `app`, `e2e`, `internal-utils` workspaces).
  `core` is the one library package that still lives in this repo (consumed by `app` and the shared packages).

## Engineering rules specific to this repo

- Prefer the smallest coherent change; don't mix feature work with unrelated migrations/cleanup.
- Treat auth, secret handling, and DB migrations as security-sensitive; preserve API and stored-resource
  compatibility unless a breaking change is explicit and intentional.
- New Go/CUE source files need the Apache license header (`make fixlicense` adds missing ones; `make checklicense`
  verifies).
- Commit/PR titles follow `[<CATALOG_ENTRY>] message` where entry is one of `FEATURE`, `ENHANCEMENT`, `BUGFIX`,
  `BREAKINGCHANGE`, `DOC`, `IGNORE` (drives changelog generation). PRs are squash-merged by default; multi-kind PRs
  with well-formed per-commit prefixes are merged instead of squashed. DCO signoff is required on all commits
  (`git commit -s`).
- Do not add `Co-Authored-By: Claude` (or any other AI attribution trailer) to commits in this repo — this overrides
  Claude Code's default attribution behavior. Commits should carry only the contributor's own DCO sign-off.
- Always ask for explicit confirmation before: pushing a branch/PR, writing/finalizing a commit message, or posting
  a comment on a GitHub issue or PR. Draft the content and show it to the user first; do not push or post until they
  approve.
- Do not raise lint warning ceilings or add broad suppressions; a needed Oxlint suppression must be narrow, name the
  rule, and explain why inline.
- TypeScript: prefer `unknown` + narrowing over `any`/non-null assertions; named exports; component prop types named
  `ComponentNameProps`; prefer optional/`undefined` over `null` unless `null` has explicit domain meaning; keep
  feature-specific code under `ui/app/src/views/<feature>` rather than importing it from other features.
- React: components at module scope, pure render, effects only to sync with external systems (with cleanup), derive
  values during render rather than mirroring them into state via an effect.
