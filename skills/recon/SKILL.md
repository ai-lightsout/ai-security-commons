# Recon — orient in an unfamiliar codebase before security review

A generalized procedure for an AI agent to answer, cheaply and in order: _what
is this, what's the attack surface, and which hint files should I load?_ Run it
before deep review so you spend flagship tokens on the risky 5%, not on
rediscovering the stack.

## 1. Detect languages and build system

Look for signature files at the repo root and one level down — presence is the
signal, you don't need to read them yet:

| Signal file(s) | Language / stack | Build system |
|----------------|------------------|--------------|
| `*.csproj`, `*.sln`, `Directory.Build.props` | C# / .NET | MSBuild / `dotnet` |
| `package.json` | JavaScript / TypeScript | npm / yarn / pnpm |
| `pyproject.toml`, `requirements.txt`, `setup.py` | Python | pip / poetry / uv |
| `go.mod` | Go | go |
| `pom.xml`, `build.gradle` | Java / Kotlin | Maven / Gradle |
| `Cargo.toml` | Rust | cargo |
| `Gemfile` | Ruby | bundler |
| `composer.json` | PHP | composer |

Confirm with a language histogram (file-extension counts) — the build file names
the primary language; the histogram reveals secondary ones (a `package.json` next
to a `.csproj` means a JS front-end you must also review).

## 2. Identify frameworks (this selects which hint files to load)

Read the dependency manifest, not the whole tree:

- **.NET** — in `.csproj`/`packages`: `Microsoft.AspNetCore.*` → load
  `hints/csharp/aspnet/`; always also load `hints/csharp/core/`. `Microsoft.EntityFrameworkCore`
  → watch `FromSqlRaw`. WPF/WinForms → desktop, different surface.
- **JavaScript** — in `package.json` deps: `express`/`fastify`/`koa` → server
  request handling; `react`/`vue`/`angular` → client XSS surface; `next` → both.
- **Python** — `django`/`flask`/`fastapi` in the manifest select the framework
  hint set.

Map the detected frameworks to `hints/<language>/<framework>/` and load exactly
those files. Missing a framework folder? That's a **contribution opportunity** —
note it and consider a PR (see `CONTRIBUTING.md`).

## 3. Map the attack surface (entry points & trust boundaries)

Find where untrusted data enters — these are the roots of every taint path:

- **HTTP handlers**: controllers/routes/handlers (`[ApiController]`,
  `app.get(...)`, `@app.route`, etc.).
- **CLI**: `main`/entry points reading `args`/stdin.
- **Deserialization boundaries**: model binding, JSON/XML parse of external data.
- **File & network reads**: uploads, watched directories, sockets.
- **Message/queue consumers**: handlers for external events.

For each entry point, the review question is: _does tainted data from here reach
a sink from the loaded hint files without validation?_

## 4. Note the guardrails already present

Cheap context that changes severity: is there input validation middleware, an
ORM (parameterization by default), an auth layer, output encoding by default
(e.g. Razor)? Their _absence_ around an entry point raises priority.

## Output of recon

A short orientation note: primary/secondary languages, build system, frameworks,
the hint files loaded, the list of entry points, and the top 3 areas to review
first. Hand that to the deep-review pass so it starts already oriented.
