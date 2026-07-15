# Repository instructions for GitHub Copilot

Reusable standards for **C#/.NET back ends and Angular front ends**. This repository contains no application code — it ships configuration: `.claude/` for Claude Code and `.github/` for GitHub Copilot (custom agents in `agents/`, plus a `skills/` tree mirrored from `.claude/skills/`). Follow these instructions for all C# and Angular work.

## C# coding standards (always)

- Target the latest C# version (currently **C# 14**). File-scoped namespaces; single-line `using` directives; honor `.editorconfig`. Prefer pattern matching and switch expressions; use `nameof(...)` instead of string literals for member names. Newline before every opening `{`; a method's final `return` on its own line.
- PascalCase for types, methods, and public members; camelCase for private fields and locals; prefix interfaces with `I` (e.g. `IUserService`).
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
| `angular-developer` | general Angular (components, signals, forms, DI, routing, SSR, testing) |
| `ngrx-signal-store` | NgRx Signal Store state management (source of truth for state) |

## Detailed rules — `.claude/rules/`

These are **not auto-loaded for Copilot** — always read the matching rules file before editing matching files:

| Rule file | Applies to |
| --- | --- |
| `.claude/rules/csharp.md` | all `*.cs` |
| `.claude/rules/aspnet-rest-apis.md` | REST / ASP.NET Core APIs |
| `.claude/rules/azure-functions-csharp.md` | Azure Functions (isolated worker), `host.json`, `local.settings.json` |
| `.claude/rules/blazor.md` | `*.razor`, `*.razor.cs`, `*.razor.css` |
| `.claude/rules/csharp-mcp-server.md` | MCP servers in C# |

## Custom agents — `.github/agents/`

Intended flow: plan with **Planner Expert** → hand off to the recommended implementation agent → that agent invokes the matching code-reviewer subagent before finishing.

- **Planner Expert** — researches and outlines a plan, then routes via handoff buttons (VS Code).
- **C# Expert** — general C#/.NET implementation.
- **C# MCP Server Expert** — Model Context Protocol servers in C#.
- **C#/.NET Janitor** — cleanup, modernization, tech-debt remediation.
- **C# Code Reviewer** / **Angular Code Reviewer** — read-only reviewers reporting findings by severity with a verdict; invoked as subagents by the implementation agents after code changes, or run standalone from the agents dropdown.

When the microsoft-learn or angular-cli MCP servers are available, use them to ground version-specific .NET/Azure and Angular answers instead of relying on memory.

## Commands

```bash
dotnet build     # compile
dotnet test      # run xUnit tests
dotnet format    # apply .editorconfig formatting
```
