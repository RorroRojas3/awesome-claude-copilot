# CLAUDE.md

Reusable project memory for **C#/.NET back ends and Angular front ends**. It loads automatically every session and governs how Claude Code works in this project. Drop it into a repository and follow it for all C# and Angular work.

## Repository layout (`.claude/`)

- `CLAUDE.md` — this file (always-on standards + delegation rules).
- `rules/` — detailed standards that **auto-apply by file type** (see [Detailed standards](#detailed-standards--clauderules)).
- `skills/` — invokable best-practice skills (`angular-developer`, `csharp-async`, `csharp-docs`, `csharp-xunit`, `ef-core`, `github-actions-efficiency`, `github-actions-hardening`, `github-actions-runtime-upgrade-conventions`, `microsoft-agent-framework`, `microsoft-docs`, `ngrx-signal-store`, `prd`).
- `agents/` — subagents (`angular-code-reviewer`, `csharp-code-reviewer`, `github-actions-reviewer`, `prd-generator`, `se-technical-writer`).
- `commands/` — slash commands (`/ngrx-signals-sync`).
- `settings.json`, `skills-lock.json`, and `.mcp.json` (repo root) — model, default reasoning effort (`"effortLevel": "xhigh"`), skill-pinning, and MCP server configuration.
- `CHANGELOG.md` (repo root) — running record of features and notable changes, maintained by the `se-technical-writer` subagent (see [Changelog & feature tracking](#changelog--feature-tracking)).

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

Apply these to all C# you write or review. The detailed source of truth is in `.claude/rules/` and the skills in `.claude/skills/`.

**Language & formatting**

- Target the latest C# language version (currently **C# 14**).
- File-scoped namespaces; single-line `using` directives; honor `.editorconfig`.
- Prefer pattern matching and switch expressions.
- Use `nameof(...)` instead of string literals for member names.
- Put a newline before the opening `{` of every block; keep a method's final `return` on its own line.
- Prefer **primary constructors**; capture each injected dependency into a `private readonly` `_camelCase` field (`private readonly IUserRepository _repository = repository;`) and use the field in method bodies.
- Prefer **collection expressions** (`[]`, `[1, 2, 3]`, `[.. items]`) over `new List<T>()`, `new T[] { }`, or `Array.Empty<T>()`.

**Naming**

- PascalCase for types, methods, and public members; `_camelCase` for private fields (e.g. `_userService`); camelCase for locals and parameters; prefix interfaces with `I` (e.g. `IUserService`).

**Nullable reference types**

- Declare variables non-nullable; validate `null` at entry points only.
- Use `is null` / `is not null` — **never** `== null` / `!= null`.
- Trust the null annotations; do not add redundant null checks the type system already rules out.

**Async** (see the `csharp-async` skill)

- Suffix async methods with `Async`; return `Task`, `Task<T>`, or `ValueTask<T>` (for hot paths).
- **Never** block with `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()`.
- No `async void` except event handlers; always `await` Task-returning calls.
- Use `ConfigureAwait(false)` in library code; flow a `CancellationToken` through long-running operations.
- Parallelize with `Task.WhenAll` / `Task.WhenAny`.

**Validation & error handling**

- `try`/`catch` around `await`s; never silently swallow exceptions.
- Validate with FluentValidation or DataAnnotations; centralize with global exception middleware.
- Return errors as Problem Details (RFC 9457).

**Logging & security**

- Inject `ILogger<T>` via the constructor; use structured logging (e.g. Serilog).
- **Never log PII or secrets.**
- Prefer `DefaultAzureCredential` + Azure Key Vault / Managed Identity over secrets in code or config.

**Documentation** (see the `csharp-docs` skill)

- XML doc comments on all public APIs: `<summary>` starts with a present-tense, third-person verb; document `<param>`, `<returns>`, and `<exception>`; use `<see langword>` for keywords, `<inheritdoc/>` for overrides, and `<example>` with `<code language="csharp">`.

**Testing** (see the `csharp-xunit` skill)

- xUnit; tests live in a `[ProjectName].Tests` project; name tests `MethodName_Scenario_ExpectedBehavior`.
- Follow Arrange-Act-Assert structure but do **not** write `// Arrange` / `// Act` / `// Assert` comments.
- Data-driven tests with `[Theory]` + `[InlineData]` / `[MemberData]`; isolate with Moq or NSubstitute; run with `dotnet test`.

**Review posture**

- Make only **high-confidence** suggestions. Comment on _why_ a non-obvious design decision was made, not just what it does.

## Angular / NgRx state (always)

- Standalone components, `ChangeDetectionStrategy.OnPush`, signals for state. Assume zoneless.
- Non-trivial state belongs in an **NgRx Signal Store** (`@ngrx/signals`) — invoke the `ngrx-signal-store` skill rather than hand-rolling a `BehaviorSubject` service.
- Keep `protectedState` on so only the store's own methods write state; use `patchState` with standalone updaters.
- Reach for `rxMethod` (not `signalMethod`) whenever requests can overlap — `switchMap` is what prevents a stale response overwriting a fresh one.
- One store per entity type; use `withEntities` for keyed collections.
- This is **not** classic NgRx: no actions, reducers, or effects unless the Events plugin is a deliberate choice.
- For Angular questions that are not about state, use the `angular-developer` skill and the `angular-cli` MCP server instead of relying on memory.

## Skills

Available via the Skill tool:

- `csharp-async` — async/await best practices.
- `csharp-docs` — XML documentation conventions.
- `csharp-xunit` — xUnit unit-testing patterns.
- `ef-core` — Entity Framework Core best practices (DbContext, queries, migrations).
- `microsoft-agent-framework` — Microsoft Agent Framework solutions; shared guidance plus a `references/dotnet.md` deep-dive. Ground advice in live docs — the framework is in public preview and changes quickly.
- `microsoft-docs` — querying official Microsoft documentation: Microsoft Learn MCP by default, with Context7 (and the Aspire MCP, when its CLI is present) for content outside learn.microsoft.com (VS Code, GitHub, Aspire).
- `github-actions-hardening` — security review of GitHub Actions workflows (script injection, privileged triggers, SHA pinning, least-privilege tokens).
- `github-actions-efficiency` — audit workflow efficiency to cut CI minutes and cost (caching, concurrency, trigger scoping).
- `github-actions-runtime-upgrade-conventions` — upgrade actions to supported runtimes and safe versions. These three are the lane skills preloaded by the `github-actions-reviewer` subagent, but each is also invokable on its own.
- `angular-developer` — the official Angular team agent skill (from <https://angular.dev/ai/agent-skills>, installed from `github.com/angular/skills`). General Angular implementation guidance with progressive disclosure: `SKILL.md` routes to `references/` files for components, signals (`linkedSignal`, `resource`, effects), forms (incl. Signal Forms), DI, routing/SSR, styling, and testing. For state management, `ngrx-signal-store` remains the source of truth.
- `ngrx-signal-store` — NgRx Signal Store patterns for Angular. Uses progressive disclosure: `SKILL.md` carries the decision rules, and `references/` files (entities, async/RxJS, custom features, testing, events, recipes, API) are read only when needed. Pinned to a specific `@ngrx/signals` version and refreshed with `/ngrx-signals-sync`.
- `prd` — writing Product Requirements Documents and breaking features into epics and user stories with acceptance criteria, priorities, estimates, and dependencies. Progressive disclosure: `SKILL.md` carries the workflow and quality bar; `references/` holds the canonical PRD template, the story-breakdown rules, and the GitHub-issue recipes. Preloaded by the `prd-generator` subagent; mirrored to the Copilot harness.

## Detailed standards — `.claude/rules/`

The full guidelines live in `.claude/rules/` and **load automatically when you edit a matching file** (path-scoped rules). You do not need to open them manually; if one is out of context, `Read` it directly.

| Area                              | Rule file                                 | Auto-applies to                                                    |
| --------------------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| General C#                        | `.claude/rules/csharp.md`                 | `**/*.cs`                                                          |
| REST / ASP.NET Core APIs          | `.claude/rules/aspnet-rest-apis.md`       | `**/*.cs`, `**/*.json`                                             |
| Azure Functions (isolated worker) | `.claude/rules/azure-functions-csharp.md` | `**/*.cs`, `**/host.json`, `**/local.settings.json`, `**/*.csproj` |
| Blazor WebAssembly (standalone)   | `.claude/rules/blazor-wasm.md`            | `**/*.razor`, `**/*.razor.cs`, `**/*.razor.css`                    |
| MCP servers in C#                 | `.claude/rules/csharp-mcp-server.md`      | `**/*.cs`, `**/*.csproj`                                           |
| Terraform                         | `.claude/rules/terraform.md`              | `**/*.tf`                                                          |

## MCP servers — see `@.mcp.json`

`.claude/settings.json` sets `enableAllProjectMcpServers: true`, so the servers configured in `@.mcp.json` are available. Use them when relevant:

> **Trust gate:** since Claude Code v2.1.196, a checked-in `.claude/settings.json` cannot approve its own repo's MCP servers while the folder is **untrusted** — the key is ignored and servers sit at "Pending approval" until the workspace trust dialog is accepted. To have these servers auto-approve in every repo (even before trusting), add a name-based list to your **user-level** `~/.claude/settings.json`: `"enabledMcpjsonServers": ["microsoft-learn", "terraform", "angular-cli", "context7"]`. If a server shows **Rejected**, a stale per-project choice is cached — run `claude mcp reset-project-choices` in that repo.

- **`microsoft-learn`** — Ground .NET/Azure answers in official Microsoft Learn docs. Before answering a version-specific .NET or Azure question, query it (`microsoft_docs_search` → `microsoft_code_sample_search` → `microsoft_docs_fetch`) instead of relying on memory.
- **`angular-cli`** — Ground Angular answers in the installed Angular version rather than memory (`list_projects` → `get_best_practices` → `search_documentation` → `find_examples`). Use it for components, zoneless, routing, and CLI work; for state management, the `ngrx-signal-store` skill is the source of truth.
- **`terraform`** — infrastructure-as-code.
- **`context7`** — docs lookup for content that lives outside learn.microsoft.com (VS Code, GitHub, Aspire); used by the `microsoft-docs` skill. Resolve the library ID first, then query.

## Delegation rules

Every subagent is pinned to **extra-high reasoning effort** (`effort: xhigh` in its frontmatter), and `settings.json` sets the same session-wide default (`"effortLevel": "xhigh"`), so agents stay at extra-high even if the session level is lowered.

- **When the user asks to write a PRD, spec a feature, define requirements, or break a feature into epics/user stories**, delegate to the `prd-generator` subagent (runs on Sonnet at `xhigh` effort, with the `prd` skill preloaded) — do not write PRDs inline. Its report always starts with a `PRD-STATUS:` line. If it starts `PRD-STATUS: NEEDS-INPUT`, show its questions to the user verbatim (do not answer them yourself) and re-invoke the agent with the answers. It only creates GitHub issues when re-invoked with a statement that the user explicitly approved issue creation for the PRD path. `docs/prd/` is owned by `prd-generator`; a PRD is a pre-implementation artifact — writing one gets no `se-technical-writer` delegation and no changelog entry. Implementation plans for a feature that has a PRD should reference its story IDs (`US-xxx`).
- **After implementing or modifying C# code**, delegate a quality review to the `csharp-code-reviewer` subagent (runs on Sonnet at `xhigh` effort, with the C# and `ef-core` skills preloaded). It reports findings; it does not edit files.
- **After implementing or modifying Angular code**, delegate a quality review to the `angular-code-reviewer` subagent (runs on Sonnet at `xhigh` effort, with the `angular-developer` and `ngrx-signal-store` skills preloaded). It reports findings; it does not edit files.
- **After writing or modifying GitHub Actions workflows** (`.github/workflows/*.yml` or composite actions), delegate a review to the `github-actions-reviewer` subagent (runs on Opus at `xhigh` effort, with the three `github-actions-*` skills preloaded). It reports findings; it does not edit files.
- **After a feature is implemented and the relevant reviewer verdict passes**, ALWAYS delegate to the `se-technical-writer` subagent (runs on Sonnet at `xhigh` effort) to author or update Markdown docs under `docs/` (create the folder if it does not exist) **and** add the entry to the root `CHANGELOG.md` under `[Unreleased]`. Also delegate to it whenever implementation details need documenting on their own.

## Changelog & feature tracking

The root `CHANGELOG.md` follows the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format and is the running record of features and notable changes:

- Every PR adds or updates **one entry** under `## [Unreleased]`, in the matching subsection (`### Added`, `### Changed`, `### Fixed`, `### Removed`, `### Deprecated`, `### Security`). Concise, reader-facing phrasing — not a commit list.
- The `se-technical-writer` subagent owns the changelog and writes the entry as part of its post-implementation delegation. Routine cleanups with no behavior change still get a one-line entry, even when they need no docs.
- On release, the `[Unreleased]` section is renamed to the version and date, and a fresh `[Unreleased]` section is started.

## Common commands

```bash
dotnet build     # compile
dotnet test      # run xUnit tests
dotnet format    # apply .editorconfig formatting
```

## Maintenance

- `/ngrx-signals-sync` — check the official NgRx Signals docs for changes and refresh the `ngrx-signal-store` skill if they drifted. Cheap and silent when nothing changed; when something did, it edits the skill and leaves the diff in the working tree for review rather than committing.
