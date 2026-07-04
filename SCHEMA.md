# Entry schema

Hints are Markdown so humans can read them and AIs can parse them, but each
entry follows a fixed shape so tooling can later compile them into package data.

## Directory layout

```
hints/<language>/<framework>/sources-and-sinks.md   # the core list
hints/<language>/<framework>/notes.md               # optional terse review notes
```

`<language>` and `<framework>` are lowercase, no spaces (`csharp`, `aspnet`,
`core`, `python`, `django`, `javascript`, `express`). Use `core` as the framework
slug for language/standard-library hints that aren't framework-specific.

## Source / sink table format

Each `sources-and-sinks.md` contains one or both of these tables. Columns are
fixed; keep cells terse.

**Sources** — where untrusted or attacker-influenced data enters:

| api | kind | notes |
|-----|------|-------|
| `HttpRequest.Query` | http-input | Query string values; fully attacker-controlled. |

**Sinks** — where tainted data causes harm if it reaches unsanitized:

| api | vuln | safe-alternative |
|-----|------|------------------|
| `SqlCommand(text)` | sql-injection | Parameterized queries / `SqlParameter`. |

### Controlled vocabularies

- **source `kind`**: `http-input`, `file`, `env`, `cli-arg`, `network`,
  `deserialization`, `ipc`, `db-read`.
- **sink `vuln`**: `sql-injection`, `command-injection`, `path-traversal`,
  `xss`, `xxe`, `ssrf`, `insecure-deserialization`, `open-redirect`,
  `ldap-injection`, `log-injection`, `template-injection`, `code-injection`.

If you need a term that isn't listed, add it in the same PR and note it here —
the vocabulary grows by contribution.

## Rules for entries

- **Concise.** An entry is a lookup row, not an essay. One sentence per cell.
- **Defensive framing.** Describe the risk and the safe alternative. No payloads.
- **Cite when non-obvious.** Link official docs or an advisory in `notes.md`, not
  inline in the table.
- **One concept per row.** Split overloads only when they differ in risk.
