# Python — Django sources and sinks

Framework-specific entry points and dangerous sinks for Django request handling.
For language/stdlib sinks that apply everywhere, see
[`../core/sources-and-sinks.md`](../core/sources-and-sinks.md).

## Sources — untrusted input entering a Django app

| api | kind | notes |
|-----|------|-------|
| `request.GET['...']` | http-input | Query-string values; attacker-controlled. |
| `request.POST['...']` | http-input | Form body fields. |
| `request.body` | http-input | Raw request body; size-limit and content-type check. |
| `request.FILES['...']` | file | Uploaded files; never trust `.name` as a path. |
| `request.headers` / `request.META` | http-input | Includes `HTTP_HOST`, `HTTP_REFERER` — forgeable. |
| `request.COOKIES['...']` | http-input | Client-supplied; verify integrity (signed cookies). |
| URL kwargs (view `**kwargs`) | http-input | Path segments; validate before use in paths/queries. |
| DRF `serializer.validated_data` / `request.data` | deserialization | Attacker-shaped; validation rules must actually constrain. |

## Sinks — where Django-tainted data causes harm

| api | vuln | safe-alternative |
|-----|------|------------------|
| `Model.objects.raw(sql)` with interpolation | sql-injection | Params: `raw(sql, [params])`, or the ORM. |
| `.extra(where=[input])` / `RawSQL(input)` | sql-injection | Avoid `extra`; use ORM filters or parameterized `RawSQL`. |
| `cursor.execute("..." % input)` | sql-injection | `cursor.execute(sql, [params])`. |
| `mark_safe(input)` / `format_html` with raw input | xss | Let templates auto-escape; only mark trusted, sanitized HTML safe. |
| `\|safe` template filter on user data | xss | Remove `\|safe`; rely on default escaping. |
| `HttpResponseRedirect(input)` | open-redirect | Validate against allow-list; `url_has_allowed_host_and_scheme`. |
| `render()` with user-controlled template name | template-injection | Fix template names; never derive from input. |
| `Template(input).render()` | template-injection | Never build templates from untrusted strings. |
| `os.path.join(MEDIA_ROOT, input)` | path-traversal | Validate filename; use storage APIs; reject `..`. |
| `serializers.deserialize`/`pickle` of input | insecure-deserialization | Never deserialize untrusted data (see core). |

## Cross-cutting notes

- **CSRF**: a state-changing view exempted via `@csrf_exempt` (or a DRF view
  without session auth protection) is a CSRF finding — flag every exemption.
- **Authorization**: a view mutating data without a permission check / `login_required`
  / DRF `permission_classes` is the most common real bug.
- **Config**: `DEBUG=True` in production, a hardcoded `SECRET_KEY`, or
  `ALLOWED_HOSTS=['*']` are findings.
- **Mass assignment**: `Model(**request.POST)` / over-broad serializer `fields`
  lets attackers set fields you didn't mean to expose.
