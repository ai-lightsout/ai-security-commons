# Ruby — Core (language & standard library) sources and sinks

Framework-independent hints for any Ruby code. Rails and other frameworks wrap
these (params, sessions) but the underlying stdlib sinks still apply. A Rails
file belongs under `hints/ruby/rails/`.

## Sources — untrusted input entering a Ruby program

| api | kind | notes |
|-----|------|-------|
| `ARGV` | cli-arg | Command-line arguments. |
| `ENV['...']` | env | Environment; may carry attacker-influenced values in some deploys. |
| `STDIN` / `$stdin.read` / `gets` / `ARGF` | stdin | Piped or interactive input. |
| `Rack::Request#params` / `req.GET` / `req.POST` | http-input | Rack layer under most frameworks; query + body. |
| `req.env['HTTP_*']` | http-input | Request headers — forgeable. |
| `Net::HTTP` / `open-uri` response bodies | network | Data from other services is still untrusted. |
| deserialized objects (`Marshal`, `YAML`, `JSON`) | derived | Trust follows the source that produced the bytes. |

## Sinks — dangerous language/stdlib operations

| api | vuln | safe-alternative |
|-----|------|------------------|
| `eval` / `instance_eval` / `class_eval` / `module_eval` (with input) | code-injection | Never eval input; redesign to data/dispatch tables. |
| `` `#{x}` `` (backticks) / `%x{...}` / `system`/`exec`/`spawn`/`IO.popen` (string form) | command-injection | Array form: `system('git', 'log', arg)` — no shell parsing. |
| `Kernel#open(input)` / `URI.open(input)` | command-injection / ssrf | `open`/`URI.open` runs a command if the arg starts `\|`; use `File.open` for files, allow-list URLs. |
| `File.read`/`File.open`/`IO.read`(input path) | path-traversal | `File.expand_path` under a root; reject `..`; verify prefix. |
| `require`/`load`/`autoload`(input) | code-injection | Never load input-derived paths (LFI/RFI); allow-list. |
| `Marshal.load` / `Marshal.restore`(input) | insecure-deserialization | Never on untrusted bytes; use JSON. |
| `YAML.load`(input) (Psych with aliases/objects) | insecure-deserialization | `YAML.safe_load` (default in Psych ≥4); restrict `permitted_classes`. |
| `send`/`public_send`/`__send__`(input_method) | unsafe-reflection | Allow-list method names; never dispatch on raw input. |
| `const_get`/`instance_variable_set`(input) | unsafe-reflection | Validate against a fixed set. |
| `ERB.new(input).result` / templates from input | template-injection (SSTI) | Fixed templates; treat input as data, never template text. |
| `Regexp.new(input)` / interpolated regex | redos | Bound/validate the pattern; timeouts (`Regexp.timeout=` in Ruby ≥3.2). |
| raw SQL string interpolation (`conn.exec("... #{x}")`, `Mysql2`/`PG`) | sql-injection | Bound params: `exec_params`/`$1`; ORM query builders. |
| `Digest::MD5`/`SHA1` for passwords | weak-crypto | `BCrypt::Password.create` / `OpenSSL` KDF. |
| `rand`/`Random` for tokens | weak-crypto | `SecureRandom.hex`/`uuid` (CSPRNG). |

## Notes

- **The `open` pipe trick**: `Kernel#open("|command")` executes a shell command —
  a classic Ruby RCE when a filename comes from input. Prefer `File.open` (and
  `URI.open` only on validated URLs).
- **The big three RCE gadgets**: `eval`, `Marshal.load`, and `YAML.load` on
  untrusted bytes. Grep for all three first.
- **`send`-based dispatch** on a user-supplied method name can call any method,
  including private ones via `__send__`.
- **Secrets**: hardcoded credentials, committed `config/master.key` or
  `config/credentials.yml.enc` decryption keys.
- **SSRF**: `Net::HTTP`/`open-uri` to an input-derived URL needs an allow-list and
  blocked internal ranges.
