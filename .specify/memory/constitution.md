<!--
Sync Impact Report
===================
Version change: N/A → 1.0.0 (initial adoption)

Modified principles: N/A (initial)

Added sections:
- Core Principles (5 principles)
- Technology Stack Constraints
- Development Workflow
- Governance

Removed sections: None

Templates requiring updates:
- .specify/templates/plan-template.md ✅ no changes needed (Constitution Check section is generic)
- .specify/templates/spec-template.md ✅ no changes needed (requirements align with principles)
- .specify/templates/tasks-template.md ✅ no changes needed (phase structure compatible)
- .specify/templates/checklist-template.md ✅ no changes needed (generic template)
- .specify/extensions/git/commands/*.md ✅ no outdated agent-specific references

Follow-up TODOs: None
-->

# Vibe Kanban Constitution

## Core Principles

### I. Type-Safe Boundaries

Shared types between Rust and TypeScript MUST be generated via ts-rs.
Manual edits to `shared/types.ts` or `shared/remote-types.ts` are
forbidden. Rust structs annotated with `#[derive(TS)]` are the single
source of truth for the cross-language interface.

- Run `pnpm run generate-types` (local) or
  `pnpm run remote:generate-types` (cloud) after any Rust type change.
- CI MUST verify generated types are up to date
  (`pnpm run generate-types:check`).
- Rationale: Drift between Rust and TypeScript types causes silent
  runtime failures that are expensive to diagnose.

### II. Monorepo Module Isolation

Each crate and package has clear ownership and boundaries. Backend
logic lives in Rust crates (`crates/`), frontend logic in TypeScript
packages (`packages/`), and shared types in the generated `shared/`
directory.

- A change that crosses a boundary (e.g., adding an API field) MUST
  update both sides in the same PR.
- Packages MUST NOT import directly from crate source; they consume
  generated types and HTTP APIs only.
- Crate-specific guidance lives in per-crate `AGENTS.md` files.
- Rationale: Isolated modules allow independent builds, tests, and
  reasoning about each layer.

### III. Quality Gates (NON-NEGOTIABLE)

Every change MUST pass all quality gates before merge:

- `pnpm run format` — Rust (`cargo fmt`) and TypeScript (Prettier).
- `pnpm run check` — TypeScript type checking across all packages and
  Rust workspace check.
- `pnpm run lint` — ESLint for TypeScript, `cargo clippy` with
  `-D warnings` for Rust.
- `cargo test --workspace` — all Rust unit tests.
- New runtime logic MUST include tests (Rust `#[cfg(test)]` or
  Vitest for TypeScript).

Rationale: Automated gates catch regressions cheaper than manual
review. Skipping a gate requires explicit, documented justification.

### IV. Discussion Before Contribution

External contributions MUST be discussed with the core team via
GitHub Discussions or Discord before opening a PR. Do not submit
unsolicited pull requests.

- Internal changes SHOULD reference an issue or discussion.
- Rationale: Alignment on approach before implementation prevents
  wasted effort and ensures compatibility with the existing roadmap.

### V. Secure by Default

Secrets MUST NOT appear in commits. Use `.env` for local overrides;
key environment variables (`FRONTEND_PORT`, `BACKEND_PORT`, `HOST`)
are managed by `scripts/setup-dev-environment.js`.

- Never commit `.env`, credentials, or API keys.
- Dev ports and asset paths are configuration, not secrets, and are
  managed centrally by dev scripts.
- Rationale: A single leaked secret can compromise the entire
  deployment pipeline and user trust.

## Technology Stack Constraints

The following stack is authoritative. Adding a new language, runtime,
or major dependency requires explicit team approval and documented
justification.

| Layer | Technology | Version Constraint |
|-------|------------|-------------------|
| Backend | Rust | Latest stable |
| Database | SQLite (local) / PostgreSQL (remote) | SQLx for access |
| Frontend | React + TypeScript | Vite, Tailwind CSS |
| Package manager | pnpm | >= 8 |
| Node.js | Node.js | >= 20 |
| Type bridge | ts-rs | Generated, never manual |
| Monorepo | pnpm workspaces + Cargo workspace | Single repo |

- Generated files (`shared/types.ts`, `shared/remote-types.ts`) MUST
  NOT be manually edited.
- Editing generated type sources requires modifying the corresponding
  Rust binary (`crates/server/src/bin/generate_types.rs` or
  `crates/remote/src/bin/remote-generate-types.rs`).

## Development Workflow

1. **Install dependencies**: `pnpm i`
2. **Run dev environment**: `pnpm run dev` (auto-assigns ports, starts
   both backend and frontend).
3. **Make changes**: Follow module isolation (Principle II). Update
   both sides of any cross-boundary change.
4. **Regenerate types** if Rust types changed:
   `pnpm run generate-types`.
5. **Format**: `pnpm run format` (MUST run before completing any task).
6. **Verify**: `pnpm run check && pnpm run lint && cargo test --workspace`.
7. **Commit**: Atomic commits with clear messages. Reference issues
   where applicable.
8. **PR**: Discuss first (Principle IV), then open PR with passing CI.

### Code Style

- **Rust**: `rustfmt` enforced; group imports by crate; `snake_case`
  modules, `PascalCase` types. Derive `Debug`/`Serialize`/`Deserialize`
  where useful.
- **TypeScript/React**: ESLint + Prettier (2 spaces, single quotes,
  80 cols). `PascalCase` components, `camelCase` vars/functions,
  `kebab-case` file names.

## Governance

This constitution is the authoritative source of project-level rules.
It supersedes ad-hoc conventions and informal agreements.

- **Amendments**: Any change to this constitution MUST be documented
  with a version bump, rationale, and updated Sync Impact Report.
  Amendments follow semantic versioning (see below).
- **Versioning**: MAJOR for principle removals or redefinitions; MINOR
  for new principles or materially expanded guidance; PATCH for
  clarifications and typo fixes.
- **Compliance**: All PRs and code reviews SHOULD verify adherence to
  these principles. Deviations MUST be justified in the PR description.
- **Review cadence**: The constitution SHOULD be reviewed when the
  tech stack, team structure, or project scope changes materially.

**Version**: 1.0.0 | **Ratified**: 2026-04-13 | **Last Amended**: 2026-04-13
