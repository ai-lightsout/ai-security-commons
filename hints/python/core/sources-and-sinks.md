# Python — Core (language & standard library) sources and sinks

Framework-independent hints for any Python code. Web frameworks (Django, Flask,
FastAPI) add their own request sources on top of these.

## Sources — untrusted input entering a Python process

| api | kind | notes |
|-----|------|-------|
| `sys.argv` | cli-arg | Command-line arguments. |
| `input()` / stdin | http-input | Interactive or piped input. |
| `os.environ` / `os.getenv` | env | Trust depends on who sets the environment. |
| `open(path).read()` | file | Content is only as trusted as the file's writer. |
| `socket.recv` | network | Raw network input; frame and bound it. |
| `json.loads(data)` | deserialization | Shape is attacker-controlled; validate post-parse. |
| `pickle.loads` / `marshal.loads` | deserialization | **Arbitrary code execution** on untrusted input — treat any pickle source as a sink too. |
| `requests.get(...).text/.json()` | network | A called service may be attacker-run (SSRF pivots). |

## Sinks — dangerous standard-library operations

| api | vuln | safe-alternative |
|-----|------|------------------|
| `os.system(cmd)` | command-injection | `subprocess.run([...], shell=False)` with an argument list. |
| `subprocess.*(cmd, shell=True)` | command-injection | `shell=False` + list args; never interpolate input into a shell string. |
| `eval(s)` / `exec(s)` | code-injection | Parse explicitly; `ast.literal_eval` for literals only. |
| `pickle.loads(data)` | insecure-deserialization | Never unpickle untrusted data; use JSON. |
| `yaml.load(data)` (no SafeLoader) | insecure-deserialization | `yaml.safe_load`. |
| `open(path)` with tainted `path` | path-traversal | Canonicalize (`os.path.realpath`); verify under an intended root; reject `..`. |
| `__import__(name)` / `importlib` from input | code-injection | Never import module names from untrusted input. |
| `cursor.execute("... %s ..." % input)` | sql-injection | Parameter substitution: `execute(sql, (params,))`. |
| `hashlib.md5` / `sha1` for passwords | weak-crypto | `hashlib.pbkdf2_hmac`, bcrypt, or argon2 for passwords. |
| `random.random()` for tokens/secrets | weak-crypto | `secrets` module (CSPRNG). |
| `tempfile.mktemp()` | path-traversal | `tempfile.mkstemp` / `NamedTemporaryFile` (race-safe). |

## Notes

- **Secrets**: hardcoded API keys, tokens, or passwords in source are a finding.
- **SSRF**: any `requests`/`urllib` call to a URL derived from input needs a
  destination allow-list and blocking of internal/link-local ranges.
