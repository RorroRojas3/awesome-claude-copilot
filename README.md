# awesome-claude-copilot

A curated, reusable collection of **AI coding-assistant configuration for C#/.NET**, packaged for **two assistants side by side**:

- **[Claude Code](https://code.claude.com)** — configured under [`.claude/`](.claude/)
- **[GitHub Copilot](https://github.com/features/copilot) (VS Code)** — configured under [`.github/`](.github/)

The two trees are **independent**. Each one implements the same C#/.NET best practices using its own platform's native mechanisms (memory, rules, skills, agents, MCP). Adopt either tree on its own, or use both. There is no application source code here — this repo is purely a portable set of assistant configs you drop into your own projects.

---

## Why this exists

C#/.NET guidance (async, nullable reference types, validation, logging/security, XML docs, testing, plus framework specifics for ASP.NET Core, Blazor, Azure Functions, and MCP servers) is encoded once as best-practice rules, then wired into each assistant so it is applied automatically — instead of being re-explained in every prompt. The result is consistent, opinionated, production-oriented C# across your team and tools.

---

## Repository structure

```
.
├── .claude/                      # Claude Code configuration
│   ├── CLAUDE.md                 # Always-loaded project memory: C# standards + delegation rules
│   ├── rules/                    # Path-scoped rules (auto-apply when matching files are edited)
│   │   ├── csharp.md                       → **/*.cs
│   │   ├── aspnet-rest-apis.md             → **/*.cs, **/*.json
│   │   ├── azure-functions-csharp.md       → **/*.cs, **/host.json, **/local.settings.json, **/*.csproj
│   │   ├── blazor.md                       → **/*.razor, **/*.razor.cs, **/*.razor.css
│   │   └── csharp-mcp-server.md            → **/*.cs, **/*.csproj
│   ├── skills/                   # Invokable skills (Skill tool)
│   │   ├── csharp-async/SKILL.md
│   │   ├── csharp-docs/SKILL.md
│   │   └── csharp-xunit/SKILL.md
│   ├── agents/                   # Subagents
│   │   ├── code-reviewer.agent.md          # Opus, read-only C#/.NET review
│   │   └── se-technical-writer.agent.md    # Sonnet, writes docs under docs/
│   └── settings.json             # Model + MCP defaults
│
├── .github/                      # GitHub Copilot (VS Code) configuration
│   ├── instructions/             # Custom instructions (applyTo globs, auto-apply in VS Code)
│   │   ├── csharp.instructions.md
│   │   ├── aspnet-rest-apis.instructions.md
│   │   ├── azure-functions-csharp.instructions.md
│   │   ├── blazor.instructions.md
│   │   └── csharp-mcp-server.instructions.md
│   ├── skills/                   # Mirror of the three skills
│   │   ├── csharp-async/SKILL.md
│   │   ├── csharp-docs/SKILL.md
│   │   └── csharp-xunit/SKILL.md
│   └── agents/                   # Custom Copilot agents / chat modes
│       ├── CSharpExpert.agent.md           # C# Expert
│       ├── csharp-mcp-expert.agent.md      # C# MCP Server Expert
│       ├── PlannerExpert.md                # Plan agent (hands off to the experts)
│       └── se-technical-writer.agent.md    # SE: Tech Writer
│
├── .mcp.json                     # MCP servers for Claude Code
└── .vscode/mcp.json              # MCP servers for VS Code / Copilot (mirror of .mcp.json)
```

---

## The two trees at a glance

| Concept | Claude Code (`.claude/`) | GitHub Copilot (`.github/`) |
| --- | --- | --- |
| Always-on guidance | `CLAUDE.md` (auto-loaded as memory) | `instructions/*.instructions.md` (auto-applied by VS Code) |
| File-scoped rules | `rules/*.md` with `paths:` frontmatter (load when matching files are edited) | `instructions/*.instructions.md` with `applyTo:` globs |
| Reusable knowledge | `skills/<name>/SKILL.md` (Skill tool) | `skills/<name>/SKILL.md` |
| Specialized agents | `agents/*.agent.md` subagents | `agents/*.agent.md` custom agents (with handoffs) |
| MCP servers | `.mcp.json` (`enableAllProjectMcpServers` in `settings.json`) | `.vscode/mcp.json` |

Both trees cover the **same C#/.NET domain**; only the mechanism differs.

---

## What's covered

**Cross-cutting C# standards** (latest C# / C# 14): file-scoped namespaces, pattern matching, `nameof`; PascalCase/camelCase and `I`-prefixed interfaces; nullable reference types with `is null` / `is not null`; async (`Async` suffix, no `.Result`/`.Wait()`/`async void`, `CancellationToken`, `ConfigureAwait(false)`); validation (FluentValidation/DataAnnotations) and Problem Details (RFC 9457); `ILogger<T>` structured logging, never logging PII/secrets, `DefaultAzureCredential` + Key Vault; XML doc comments; xUnit testing conventions.

**Framework-specific guidance:**

| Topic | Claude rule | Copilot instruction |
| --- | --- | --- |
| General C# | `csharp.md` | `csharp.instructions.md` |
| ASP.NET Core REST APIs | `aspnet-rest-apis.md` | `aspnet-rest-apis.instructions.md` |
| Azure Functions (isolated worker) | `azure-functions-csharp.md` | `azure-functions-csharp.instructions.md` |
| Blazor | `blazor.md` | `blazor.instructions.md` |
| MCP servers in C# | `csharp-mcp-server.md` | `csharp-mcp-server.instructions.md` |

**Skills** (shared by both trees): `csharp-async`, `csharp-docs`, `csharp-xunit`.

---

## MCP servers

Both `.mcp.json` (Claude Code) and `.vscode/mcp.json` (Copilot) configure the same servers:

| Server | Transport | Use |
| --- | --- | --- |
| `microsoft-learn` | HTTP (`https://learn.microsoft.com/api/mcp`) | Ground .NET/Azure answers in official Microsoft Learn docs |
| `terraform` | stdio (Docker: `hashicorp/terraform-mcp-server`) | Terraform / infrastructure-as-code |
| `angular-cli` | stdio (`npx @angular/cli mcp`) | Angular front-end work |

---

## Getting started

### Claude Code

1. Copy the [`.claude/`](.claude/) directory and [`.mcp.json`](.mcp.json) into the root of your C#/.NET repository.
2. Start Claude Code in that repo. It automatically loads `.claude/CLAUDE.md` (every session) and the matching `.claude/rules/*.md` whenever you edit a relevant file.
3. The `code-reviewer` (Opus) and `se-technical-writer` subagents and the three skills become available. Delegation is described in `CLAUDE.md`:
   - after changing C# → `code-reviewer` reviews it (read-only, reports findings),
   - new features / implementation notes → `se-technical-writer` writes Markdown under `docs/`.

> MCP: `.claude/settings.json` sets `enableAllProjectMcpServers: true`, so the servers in `.mcp.json` are enabled for the project.

### GitHub Copilot (VS Code)

1. Copy the [`.github/`](.github/) directory and [`.vscode/mcp.json`](.vscode/mcp.json) into the root of your repository.
2. In VS Code with GitHub Copilot, the `instructions/*.instructions.md` files apply automatically to files matching their `applyTo` globs.
3. Select a custom agent (C# Expert, C# MCP Server Expert, Plan, SE: Tech Writer) in Copilot Chat. The **Plan** agent can hand off to the implementation agents.

---

## Conventions for contributors

- **Keep the trees in sync where it makes sense.** The skills and the C#/.NET guidance exist in both `.claude/` and `.github/`; when you change guidance in one, mirror the intent in the other using that platform's mechanism (Claude `rules/` `paths:` ↔ Copilot `instructions/` `applyTo:`).
- **Rules:** a `.claude/rules/*.md` file with a `paths:` block list applies only to matching files; without `paths:` it loads at launch for every session.
- **Subagents:** `.claude/agents/*.agent.md` frontmatter uses comma-separated `tools`, a `model` (`opus`/`sonnet`/`haiku`/`inherit`), and optional `skills:`/`mcpServers:` lists. `.github/agents/` uses Copilot's agent schema (tool IDs, `handoffs`, etc.).
- **Skills:** one folder per skill containing `SKILL.md` with `name` and `description` frontmatter.
