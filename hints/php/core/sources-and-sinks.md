# PHP — Core (language & standard library) sources and sinks

Framework-independent hints for any PHP code. Frameworks (Laravel, Symfony) wrap
these superglobals but the underlying sinks still apply.

## Sources — untrusted input entering a PHP script

| api | kind | notes |
|-----|------|-------|
| `$_GET` / `$_REQUEST` | http-input | Query params; `$_REQUEST` also mixes POST/cookies — avoid. |
| `$_POST` | http-input | Form body fields. |
| `$_COOKIE` | http-input | Client-supplied; verify integrity. |
| `$_SERVER['HTTP_*']` | http-input | Request headers (`HTTP_HOST`, `HTTP_REFERER`) — forgeable. |
| `$_FILES` | file | Uploads; never trust `name`/`type`; use the tmp path. |
| `php://input` / `file_get_contents('php://input')` | http-input | Raw request body. |
| `$argv` / `getenv` | cli-arg / env | CLI args and environment. |

## Sinks — dangerous language/stdlib operations

| api | vuln | safe-alternative |
|-----|------|------------------|
| `eval($s)` / `assert($string)` | code-injection | Never evaluate input; redesign. |
| `system` / `exec` / `shell_exec` / `passthru` / `` `$cmd` `` | command-injection | Avoid; if unavoidable, `escapeshellarg` each arg (still risky). |
| `include`/`require` with input | code-injection | Never include input-derived paths (LFI/RFI); allow-list. |
| `unserialize($input)` | insecure-deserialization | `json_decode`; if unavoidable, `['allowed_classes'=>false]`. |
| `mysqli_query("... ".$input)` / `$pdo->query(concat)` | sql-injection | Prepared statements with bound params (`PDO::prepare`). |
| `fopen`/`file_get_contents`/`unlink`($path) with input | path-traversal | `realpath` + verify under a root; reject `..`. |
| `echo $input` into HTML | xss | `htmlspecialchars($input, ENT_QUOTES)`. |
| `header("Location: ".$input)` | open-redirect | Allow-list targets; validate scheme/host. |
| `preg_replace('/.../e', ...)` | code-injection | Deprecated `/e`; use `preg_replace_callback`. |
| `extract($_GET)` / `extract($input)` | code-injection | Never extract untrusted arrays into scope. |
| `md5`/`sha1` for passwords | weak-crypto | `password_hash` (bcrypt/argon2). |
| `rand`/`mt_rand`/`uniqid` for tokens | weak-crypto | `random_bytes` / `random_int` (CSPRNG). |

## Notes

- **Secrets**: hardcoded credentials/keys in source or committed `.env`.
- **SSRF**: `file_get_contents`/cURL to an input-derived URL needs an allow-list;
  disable following to internal ranges.
- **Config**: `display_errors=On` in production leaks internals.
