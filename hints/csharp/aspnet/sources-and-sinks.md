# C# — ASP.NET (Core & MVC) sources and sinks

Framework-specific entry points and dangerous sinks for ASP.NET request handling.
For language/stdlib sinks that apply everywhere in .NET, see
[`../core/sources-and-sinks.md`](../core/sources-and-sinks.md).

## Sources — untrusted input entering an ASP.NET app

| api | kind | notes |
|-----|------|-------|
| `HttpRequest.Query["..."]` / `[FromQuery]` | http-input | Query-string values; attacker-controlled. |
| `HttpRequest.Form["..."]` / `[FromForm]` | http-input | POST body form fields. |
| `HttpRequest.Headers["..."]` | http-input | Includes `Host`, `Referer`, `User-Agent` — all forgeable. |
| `HttpRequest.Cookies["..."]` | http-input | Client-supplied; never trust integrity without signing. |
| `[FromRoute]` / route template params | http-input | Path segments; validate before use in paths/queries. |
| `[FromBody]` model binding | deserialization | Bound DTOs are attacker-shaped; validate, don't assume ranges. |
| `HttpRequest.Body` / `ReadFormAsync` | http-input | Raw stream; size-limit and content-type check. |
| `IFormFile` (file upload) | file | Filename and content attacker-controlled; never use client filename as a path. |
| `RouteData.Values` | http-input | Same trust level as route params. |

## Sinks — where ASP.NET-tainted data causes harm

| api | vuln | safe-alternative |
|-----|------|------------------|
| `new SqlCommand(concatenatedText)` | sql-injection | Parameterized queries / `SqlParameter`; or an ORM with parameters. |
| `FromSqlRaw($"...{input}...")` (EF Core) | sql-injection | `FromSqlInterpolated` / parameter placeholders `{0}`. |
| `@Html.Raw(input)` / `HtmlString` | xss | Default Razor `@` encoding; sanitize HTML if it must render. |
| `IActionResult Redirect(input)` | open-redirect | `LocalRedirect`, or allow-list the target host. |
| `Path.Combine(root, input)` | path-traversal | Canonicalize and verify the result stays under `root`; strip `..`. |
| `Process.Start(..., input)` | command-injection | Pass args as an argument array, never a shell string; avoid shells. |
| `XmlReader` with `DtdProcessing.Parse` | xxe | `DtdProcessing.Prohibit` and null `XmlResolver`. |
| `BinaryFormatter` / `LosFormatter` on input | insecure-deserialization | Do not deserialize untrusted data; use `System.Text.Json`. |
| `HttpClient.GetAsync(input)` | ssrf | Allow-list destinations; block internal/link-local ranges. |
| `RedirectPermanent(input)` / `Response.Redirect(input)` | open-redirect | Validate against an allow-list of app URLs. |

## Cross-cutting notes

- **Antiforgery**: state-changing POST/PUT/DELETE without `[ValidateAntiForgeryToken]`
  (or the global filter) is a CSRF finding.
- **Authorization**: an action mutating data without `[Authorize]` (or explicit
  anonymous intent) is worth flagging — missing authz is the most common real bug.
- **Mass assignment**: binding directly to EF entities lets attackers set fields
  you didn't mean to expose; bind to a DTO and map explicitly.
