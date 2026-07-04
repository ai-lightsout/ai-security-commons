# C# — Core (language & .NET standard library) sources and sinks

Framework-independent hints for any .NET code. Framework layers (ASP.NET, WPF,
worker services) add their own sources on top of these.

## Sources — untrusted input entering a .NET process

| api | kind | notes |
|-----|------|-------|
| `args` (`Main(string[] args)`) | cli-arg | Command-line arguments; attacker-controlled in many deployments. |
| `Environment.GetEnvironmentVariable` | env | Trust depends on who sets the environment. |
| `Console.ReadLine` / stdin | http-input | Interactive or piped input. |
| `File.ReadAllText` / `Stream` reads | file | Content is only as trusted as the file's writer. |
| `Socket` / `NetworkStream` / `TcpClient` reads | network | Raw network input; frame and bound it. |
| `JsonSerializer.Deserialize<T>(input)` | deserialization | Shape is attacker-controlled; validate post-bind. |
| `HttpClient` response bodies | network | A called service may be compromised or attacker-run (SSRF pivots). |

## Sinks — dangerous standard-library operations

| api | vuln | safe-alternative |
|-----|------|------------------|
| `Process.Start(fileName, argsString)` | command-injection | `ProcessStartInfo` with `ArgumentList`; never build a shell string. |
| `Assembly.Load` / `Activator.CreateInstance(typeName)` | code-injection | Never resolve types/assemblies from untrusted input. |
| `File.*(path)` with tainted `path` | path-traversal | Canonicalize; verify under an intended root; reject `..`. |
| `BinaryFormatter.Deserialize` | insecure-deserialization | Removed/obsolete for a reason — use `System.Text.Json`. |
| `XmlSerializer` / `DataContractSerializer` with type from input | insecure-deserialization | Fix the expected type; never bind arbitrary types. |
| `SqlCommand` / `OdbcCommand` string concat | sql-injection | Parameters, always. |
| `new Random()` for tokens/secrets | weak-crypto | `RandomNumberGenerator` (CSPRNG) for anything security-relevant. |
| `MD5` / `SHA1` for passwords or integrity | weak-crypto | PBKDF2/`Rfc2898DeriveBytes`, bcrypt, Argon2 for passwords; SHA-256+ for integrity. |
| Logging raw untrusted input | log-injection | Encode/escape newlines before logging user data. |

## Notes

- **Secrets**: hardcoded connection strings, API keys, or tokens in source are a
  finding regardless of framework. Look in config files and constants too.
- **Crypto**: static IVs/keys, ECB mode, and disabled certificate validation
  (`ServerCertificateCustomValidationCallback` returning `true`) are classic bugs.
