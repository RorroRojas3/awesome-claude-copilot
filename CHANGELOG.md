# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

One entry per feature/PR under `[Unreleased]`; the SE Technical Writer agent
maintains this file as part of every post-implementation invocation.

## [Unreleased]

### Added

- Root `CHANGELOG.md` and a repo-wide changelog convention: every PR records one entry under `[Unreleased]`, owned by the SE Technical Writer agent in both harnesses (`.claude/agents/se-technical-writer.md`, `.github/agents/se-technical-writer.agent.md`).
- Documentation step in every GitHub Copilot implementation agent: after the code-review verdict passes, the C# Expert, Angular Expert, C# MCP Server Expert, and C#/.NET Janitor invoke the SE Technical Writer for docs under `docs/` and the changelog entry; the Full-Stack Expert documents the whole feature once (its sub-experts skip their own step).
- GitHub Actions Reviewer wiring for implementation agents: any agent that modifies `.github/workflows/*.yml` or composite actions now invokes the GitHub Actions Reviewer on those files.
- Planner Expert handoff and routing for documentation-only work ("Document: SE Technical Writer") and an "Apply fixes" handoff on the GitHub Actions Reviewer for parity with the code reviewers.

### Changed

- Migrated all GitHub Copilot agents to `Claude Sonnet 5 (copilot)`; the SE Technical Writer runs on `Claude Haiku 4.5 (copilot)`.
- `CLAUDE.md` and `.github/copilot-instructions.md` now state the full flow — implement → reviewer loop → SE Technical Writer (docs + changelog) — and document the changelog convention.
- Synced the `ngrx-signal-store` skill with upstream NgRx docs: now tracks the new experimental `@ngrx/signals/resource` page (`extendResource` plus loading/error value extensions), documented in `references/async-and-rxjs.md` and `references/api-reference.md` in both harnesses; re-pinned at `@ngrx/signals` 21.1.1 (18 pages).
- Refreshed the always-on standards in both harnesses so generated code matches modern C# and current Blazor (see `docs/2026-08-standards-refresh.md`):
  - C# rules now mandate `_camelCase` private fields, primary constructors with `private readonly` field capture, and collection expressions — the previous rules were steering models toward pre-C#-12 style output. Applied identically in Claude Code (`.claude/rules/csharp.md`, `.claude/CLAUDE.md`, `.claude/agents/csharp-code-reviewer.md`) and GitHub Copilot (`.github/instructions/csharp.instructions.md`, `.github/copilot-instructions.md`, `.github/agents/csharp-code-reviewer.agent.md`).
  - Blazor rules rewritten from generic mixed-hosting Blazor to standalone Blazor WebAssembly on .NET 10 / C# 14, grounded in Microsoft Learn, including a "never suggest these outdated patterns" list; both files renamed to match the scope (`.claude/rules/blazor-wasm.md`, `.github/instructions/blazor-wasm.instructions.md`), with references updated across CLAUDE.md, copilot-instructions.md, both reviewer agents, and the README.
  - CLAUDE.md's MCP section now explains the Claude Code v2.1.196 workspace trust gate and recommends a user-level `enabledMcpjsonServers` allowlist (plus `claude mcp reset-project-choices` for stale rejections) so the repo's servers auto-load everywhere.
