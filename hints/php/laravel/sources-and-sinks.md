# PHP — Laravel sources and sinks

Framework-specific entry points and dangerous sinks for Laravel request handling.
For language/stdlib sinks that apply everywhere, see
[`../core/sources-and-sinks.md`](../core/sources-and-sinks.md).

## Sources — untrusted input entering a Laravel app

| api | kind | notes |
|-----|------|-------|
| `$request->input('...')` / `request('...')` | http-input | Query + body; attacker-controlled. |
| `$request->query('...')` | http-input | Query-string values. |
| `$request->all()` / `$request->only([...])` | http-input | Full input array — mass-assignment risk downstream. |
| route params (`{id}`, `$request->route('id')`) | http-input | Validate before use in paths/queries. |
| `$request->header('...')` / `$request->cookie('...')` | http-input | Forgeable / client-supplied. |
| `$request->file('...')` | file | Uploads; never use client filename as a path. |

## Sinks — where Laravel-tainted data causes harm

| api | vuln | safe-alternative |
|-----|------|------------------|
| `DB::raw($input)` / `->whereRaw("... ".$input)` | sql-injection | Query builder bindings or `whereRaw('col = ?', [$v])`. |
| `DB::select("... ".$input)` | sql-injection | Bound params: `DB::select($sql, [$v])`. |
| Blade `{!! $input !!}` | xss | `{{ $input }}` (auto-escaped); sanitize any raw HTML. |
| `redirect($input)` / `redirect()->to($input)` | open-redirect | Allow-list; use named routes, not raw input. |
| `Storage::get($input)` / `File::get($input)` | path-traversal | Validate filename; use storage disks; reject `..`. |
| `Model::create($request->all())` | insecure-deserialization (mass-assignment) | Define `$fillable`; pass `$request->validated()` / `only([...])`. |
| `view($input)` with input template name | template-injection | Fix view names; never derive from input. |
| `eval` / `unserialize` of input | code-injection / insecure-deserialization | See core. |

## Cross-cutting notes

- **Authorization**: a controller mutating state without a policy/`Gate`/
  `authorize()` or auth middleware is the most common real bug.
- **CSRF**: routes added to `VerifyCsrfToken::$except` drop CSRF protection — flag
  each exemption.
- **Mass assignment**: `$guarded = []` (guard nothing) plus `create($request->all())`
  lets attackers set any column, including `is_admin`.
- **Config**: `APP_DEBUG=true` in production leaks stack traces / env; committed
  `.env` leaks secrets.
- **Validation**: prefer `$request->validate([...])` / FormRequests so input is
  constrained before it reaches a sink.
