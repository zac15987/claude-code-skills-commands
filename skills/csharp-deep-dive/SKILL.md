---
name: csharp-deep-dive
description: >
  Deep C# code intelligence skill that uses LSP to trace call chains, type relationships,
  and symbol definitions in the user's codebase, and clones .NET reference source repos
  to understand framework internals when needed. Use this skill when: the user is stuck
  on a C# issue that won't resolve after initial attempts (e.g. "still not working",
  "still broken", "same issue", "not resolved"); the user needs to design complex C#
  architecture and needs to understand how framework classes actually work; the user mentions
  source code, internals, under the hood, or wants to look at the underlying implementation;
  or the bug involves deep framework behavior (UI rendering, DI, threading, collections,
  networking, serialization, EF queries, middleware pipeline, etc.). Trigger proactively
  when repeated fix attempts on .cs/.xaml/.csproj files fail — don't wait for the user
  to explicitly ask for deep analysis.
---

# C# Deep Dive

Two layers of investigation. Always start with Layer 1. Escalate to Layer 2 only when
the user's code alone cannot explain the behavior.

## Layer 1 — LSP Code Intelligence (always do this first)

Use the LSP tool to build a precise understanding of the user's code before making changes.
This replaces guesswork with facts: actual types, actual call sites, actual implementations.

### When to use each LSP operation

| Operation | Use when you need to... |
|-----------|------------------------|
| `hover` | Check the exact type or signature of a symbol. Fastest confirmation. |
| `goToDefinition` | Jump to where a class/method/property is declared. Follow usage → source. |
| `goToImplementation` | Find concrete classes behind an interface or abstract method. Essential for DI-heavy codebases. |
| `findReferences` | See everywhere a symbol is used. Answers "what breaks if I change this?". |
| `documentSymbol` | Get a file's full structure at a glance. Good for unfamiliar files. |
| `workspaceSymbol` | Search for a class or method by name across the entire solution. |
| `incomingCalls` | Trace callers — "who calls this method?". Build the call chain upward. |
| `outgoingCalls` | Trace callees — "what does this method call?". Build the call chain downward. |

### Investigation workflow

1. **Locate the symptom** — find the exact line/symbol where the problem manifests.
2. **`hover`** on the symbol to confirm its type and signature.
3. **`goToDefinition`** to read the actual implementation (not what you assume it is).
4. **`goToImplementation`** if the definition lands on an interface — find the real code.
5. **`incomingCalls` / `outgoingCalls`** to trace the call chain in both directions.
6. **`findReferences`** to understand the full impact of any change.

Goal: before proposing a fix, you should be able to explain the complete path from trigger
to symptom, backed by LSP evidence. If you can't, keep tracing.

### Common patterns

- **DI registration mismatch**: `hover` shows interface type, but `goToImplementation` reveals
  unexpected concrete class → check DI registration.
- **Event subscription leak**: `findReferences` on an event shows subscribe but no unsubscribe.
- **XAML binding failure**: property exists but DataContext type is wrong →
  `goToDefinition` on the ViewModel, `documentSymbol` to verify property names.
- **Wrong overload resolution**: `hover` reveals the compiler picked a different overload.

## Layer 2 — .NET Reference Source (escalate when needed)

When the problem is not in the user's code but in how .NET itself behaves, reading the
actual framework source code is the fastest path to a definitive answer.

### Step 1: Detect the user's target framework

Read the project's `.csproj` and look for `<TargetFramework>` or `<TargetFrameworks>`:

| Value pattern | Framework |
|---------------|-----------|
| `net48`, `net472`, `net461`, etc. | .NET Framework 4.x |
| `netcoreapp3.1` | .NET Core 3.1 |
| `net5.0` — `net9.0`+ | .NET 5 — 9+ (modern .NET) |

### Step 2: Identify which repo contains the relevant source

Examine the problem area — which namespace or library is involved? Use this table to
pick the correct repo. **Only clone what is needed for the current problem.**

#### .NET Framework 4.x

| Repo | Content |
|------|---------|
| `microsoft/referencesource` | Everything: WPF, WinForms, System.*, mscorlib, ASP.NET (classic) |

No version branches — clone without `--branch`.

#### .NET Core / .NET 5+ (modern .NET)

| Repo | Content | Clone when the problem involves... |
|------|---------|------------------------------------|
| `dotnet/runtime` | BCL, CoreCLR, CoreLib | System.Collections, System.Threading, System.IO, System.Net.Http, System.Text.Json, System.Linq, Dependency Injection (`Microsoft.Extensions.DependencyInjection`), Logging (`Microsoft.Extensions.Logging`), Configuration, and most `System.*` namespaces |
| `dotnet/wpf` | WPF | Layout, rendering, binding, templating, DependencyProperty, routed events, XAML parser |
| `dotnet/winforms` | WinForms | Controls, designers, Win32 interop for WinForms apps |
| `dotnet/aspnetcore` | ASP.NET Core | Kestrel, middleware pipeline, MVC, Razor, Blazor, SignalR, minimal APIs, authentication, authorization |
| `dotnet/efcore` | Entity Framework Core | DbContext, LINQ-to-SQL translation, migrations, change tracking, query pipeline |
| `dotnet/maui` | .NET MAUI | Cross-platform UI, handlers, platform-specific renderers |
| `dotnet/roslyn` | C# / VB compiler | Compiler behavior, analyzers, source generators, language features |

All repos above use `https://github.com/{name}.git` as the clone URL.

### Step 3: Ask the user

1. Ask if they already have a local clone of the relevant repo. If yes, ask the path.
2. If not, explain which repo you need and why, then ask where to clone it.
3. If the user doesn't specify a location, default to the system temp directory.

### Step 4: Clone with the correct branch

Always shallow-clone to minimize size and time:

```bash
git clone --depth 1 [--branch <branch>] <repo-url> <target-path>
```

**Branch mapping for modern .NET repos** (`dotnet/*`):

| TargetFramework | Branch |
|-----------------|--------|
| `netcoreapp3.1` | `release/3.1` |
| `net5.0` | `release/5.0` |
| `net6.0` | `release/6.0` |
| `net7.0` | `release/7.0` |
| `net8.0` | `release/8.0` |
| `net9.0` | `release/9.0` |
| `net10.0`+ | Check if `release/X.0` exists; fall back to `main` |

For `microsoft/referencesource`: no branch needed (omit `--branch`).

If the target branch does not exist in the repo, fall back to `main`.

### Step 5: Navigate the reference source

After cloning, use `Grep` and `Glob` to locate the relevant code:

- **Find a class**: `Grep` for `class ClassName` or `partial class ClassName`.
- **Trace internal logic**: Read the method, follow internal calls with `Grep`.
  Microsoft's source has extensive comments explaining design decisions — read them.
- **Focus narrowly**: Don't try to understand the whole framework.
  Zero in on the exact method or property that relates to the user's problem.

### Example: debugging an EF Core query issue in a .NET 8 project

1. **Layer 1**: LSP traces the user's LINQ query. The query itself looks correct.
2. Suspect EF Core's query translation is generating unexpected SQL.
3. Detect `<TargetFramework>net8.0</TargetFramework>`.
4. Clone: `git clone --depth 1 --branch release/8.0 https://github.com/dotnet/efcore.git <path>`
5. `Grep` for the LINQ method causing trouble (e.g., `GroupBy` translation).
6. Read the query pipeline source → find how EF translates that LINQ pattern to SQL.
7. Explain the root cause. Propose a workaround grounded in actual framework behavior.
