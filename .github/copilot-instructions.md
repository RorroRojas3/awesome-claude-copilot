# Repository instructions for GitHub Copilot

Reusable standards for **C#/.NET back ends and Angular front ends**. This repository contains no application code — it ships GitHub Copilot configuration under `.github/`: these repository instructions, path-scoped instructions (`instructions/`), custom agents (`agents/`), and agent skills (`skills/`). Follow these instructions for all C# and Angular work.

## Communication & comments (always)

**Responses**

- Lead with the answer or the change. No preamble, no filler, no restating the request.
- Do not re-summarize a plan or diff the user already saw; report only what changed or went wrong.
- Match length to substance — a one-line answer is a complete answer.

**Code comments**

- Comment only what code cannot say: why a decision was made, constraints, non-obvious invariants, workarounds with links.
- Never narrate what code does. No per-function comment quota. No change-narration comments ("added X", "now uses Y").
- XML doc comments on public APIs are API documentation, not comments — that standard stands.

## C# coding standards (always)

- Target the latest C# version (currently **C# 14**). File-scoped namespaces; single-line `using` directives; honor `.editorconfig`. Prefer pattern matching and switch expressions; use `nameof(...)` instead of string literals for member names. Newline before every opening `{`; a method's final `return` on its own line.
- Prefer **primary constructors**; capture each injected dependency into a `private readonly` `_camelCase` field (`private readonly IUserRepository _repository = repository;`) and use the field in method bodies.
- Prefer **collection expressions** (`[]`, `[1, 2, 3]`, `[.. items]`) over `new List<T>()`, `new T[] { }`, or `Array.Empty<T>()`.
- PascalCase for types, methods, and public members; `_camelCase` for private fields (e.g. `_userService`); camelCase for locals and parameters; prefix interfaces with `I` (e.g. `IUserService`).
- Declare variables non-nullable and validate `null` at entry points only. Use `is null` / `is not null` — never `== null` / `!= null`. Trust null annotations; no redundant checks.
- Async: suffix async methods with `Async`; return `Task`, `Task<T>`, or `ValueTask<T>`. Never block with `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()`. No `async void` except event handlers; always `await` Task-returning calls. `ConfigureAwait(false)` in library code; flow a `CancellationToken` through long-running operations; parallelize with `Task.WhenAll` / `Task.WhenAny`.
- Validation & errors: `try`/`catch` around `await`s; never silently swallow exceptions. Validate with FluentValidation or DataAnnotations; centralize with global exception middleware; return errors as Problem Details (RFC 9457).
- Logging & security: inject `ILogger<T>` via the constructor; use structured logging. **Never log PII or secrets.** Prefer `DefaultAzureCredential` + Azure Key Vault / Managed Identity over secrets in code or config.
- Documentation: XML doc comments on all public APIs — `<summary>` starts with a present-tense, third-person verb; document `<param>`, `<returns>`, `<exception>`; `<see langword>` for keywords, `<inheritdoc/>` for overrides.
- Testing: xUnit in a `[ProjectName].Tests` project; name tests `MethodName_Scenario_ExpectedBehavior`; Arrange-Act-Assert structure but **no** `// Arrange` / `// Act` / `// Assert` comments; `[Theory]` + `[InlineData]` / `[MemberData]` for data-driven tests; isolate with Moq or NSubstitute; run with `dotnet test`.

## Angular / NgRx standards (always)

- Standalone components, `ChangeDetectionStrategy.OnPush`, signals for state. Assume zoneless.
- Non-trivial state belongs in an **NgRx Signal Store** (`@ngrx/signals`), not a hand-rolled `BehaviorSubject` service. Keep `protectedState` on; write state only via `patchState` with standalone updaters.
- Use `rxMethod` (not `signalMethod`) whenever requests can overlap — `switchMap` prevents a stale response overwriting a fresh one. One store per entity type; `withEntities` for keyed collections.
- This is **not** classic NgRx: no actions, reducers, or effects unless the Events plugin is a deliberate choice.

## Skills — `.github/skills/`

Before working in an area, read that skill's `SKILL.md`, then only the reference files it points to:

| Skill | Use for |
| --- | --- |
| `csharp-async` | async/await, cancellation, concurrency |
| `csharp-docs` | XML documentation on public APIs |
| `csharp-xunit` | xUnit unit testing |
| `ef-core` | Entity Framework Core (DbContext, queries, migrations) |
| `microsoft-agent-framework` | Microsoft Agent Framework solutions |
| `microsoft-docs` | querying official Microsoft documentation (Learn MCP first; Context7 for docs outside learn.microsoft.com) |
| `github-actions-hardening` | security review of workflows (script injection, privileged triggers, SHA pinning, least-privilege tokens) |
| `github-actions-efficiency` | workflow efficiency audits (caching, concurrency, trigger scoping, CI cost) |
| `github-actions-runtime-upgrade-conventions` | upgrading actions to supported runtimes and safe versions |
| `angular-developer` | general Angular (components, signals, forms, DI, routing, SSR, testing) |
| `ngrx-signal-store` | NgRx Signal Store state management (source of truth for state) |
| `prd` | writing PRDs and breaking features into epics/user stories with acceptance criteria, priorities, estimates, and dependencies |

## Path-scoped instructions — `.github/instructions/`

These apply automatically (via `applyTo` globs) when working on matching files; when reviewing or planning, read the matching file explicitly:

| Instructions file | Applies to |
| --- | --- |
| `csharp.instructions.md` | all `*.cs` |
| `aspnet-rest-apis.instructions.md` | REST / ASP.NET Core APIs (`*.cs`, `*.json`) |
| `azure-functions-csharp.instructions.md` | Azure Functions (isolated worker), `host.json`, `local.settings.json` |
| `blazor-wasm.instructions.md` | standalone Blazor WebAssembly (`*.razor`, `*.razor.cs`, `*.razor.css`) |
| `csharp-mcp-server.instructions.md` | MCP servers in C# (`*.cs`, `*.csproj`) |
| `terraform.instructions.md` | `*.tf` |

## Custom agents — `.github/agents/`

Intended flow: (optional) spec with **PRD Generator** → plan with **Planner Expert** → hand off to the recommended implementation agent → that agent invokes the matching code-reviewer subagent → once the verdict passes, it invokes the **SE Technical Writer** subagent to document the feature under `docs/` and add the `CHANGELOG.md` entry before finishing. Any agent that modified GitHub Actions workflows also invokes the **GitHub Actions Reviewer** on those files.

- **PRD Generator** — interactive PRD chat mode (VS Code): asks clarifying questions, analyzes the codebase, and writes a PRD under `docs/prd/` per the `prd` skill — epics and user stories with acceptance criteria, priorities, estimates, and dependencies. Creates GitHub issues from the stories only after explicit approval, then hands off to the Planner Expert. A PRD is a pre-implementation artifact — it gets no changelog entry.
- **Planner Expert** — researches and outlines a plan, then routes via handoff buttons (VS Code). When `docs/prd/` holds a PRD for the feature, it plans against the PRD's story IDs.
- **C# Expert** — general C#/.NET implementation.
- **C# MCP Server Expert** — Model Context Protocol servers in C#.
- **C#/.NET Janitor** — cleanup, modernization, tech-debt remediation.
- **Angular Expert** — Angular implementation (components, signals, forms, routing, SSR, Signal Store).
- **Full-Stack Expert** — orchestrates features spanning both stacks: fixes the API contract, delegates to C# Expert and Angular Expert in parallel, verifies the integrated seam, ensures both sides end with a passing review verdict, and invokes the SE Technical Writer once for the whole feature (sub-experts skip their own documentation step).
- **C# Code Reviewer** / **Angular Code Reviewer** — read-only reviewers reporting findings by severity with a verdict; invoked as subagents by the implementation agents after code changes, or run standalone from the agents dropdown.
- **GitHub Actions Reviewer** — read-only reviewer for workflow files (`.github/workflows/*.yml`) and composite actions; covers security hardening, CI efficiency, and runtime/version currency using the three `github-actions-*` skills. Use it after writing or modifying any workflow.
- **SE Technical Writer** — creates or updates developer documentation as Markdown under `docs/` (guides, tutorials, ADRs, reference docs) after a feature is implemented or when implementation details need documenting. Also owns the root `CHANGELOG.md` and records each feature or notable change under `[Unreleased]`.

When the microsoft-learn or angular-cli MCP servers are available, use them to ground version-specific .NET/Azure and Angular answers instead of relying on memory. When the context7 MCP server is available, use it for docs that live outside learn.microsoft.com (VS Code, GitHub, Aspire) — see the `microsoft-docs` skill. Repo-level MCP configuration for VS Code lives in `.vscode/mcp.json`.

## Changelog & feature tracking

The root `CHANGELOG.md` follows the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format and is the running record of features and notable changes:

- Every PR adds or updates **one entry** under `## [Unreleased]`, in the matching subsection (`### Added`, `### Changed`, `### Fixed`, `### Removed`, `### Deprecated`, `### Security`). Concise, reader-facing phrasing — not a commit list.
- The **SE Technical Writer** owns the changelog and writes the entry as part of its post-implementation invocation. Exception: the C#/.NET Janitor appends its own one-line entry for routine cleanups with no behavior change.
- On release, the `[Unreleased]` section is renamed to the version and date, and a fresh `[Unreleased]` section is started.

## Commands

```bash
dotnet build     # compile
dotnet test      # run xUnit tests
dotnet format    # apply .editorconfig formatting
```
