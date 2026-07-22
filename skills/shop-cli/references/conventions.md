# Conventions

The 10 conventions every shipped CLI follows. None is optional polish.

## 1. Single static Go binary, stdlib + cobra

- `CGO_ENABLED=0`. Cross-compile to linux/darwin × amd64/arm64 from one command.
- Deps: `github.com/spf13/cobra` (CLI). Add `github.com/bogdanfinn/tls-client` + `fhttp` ONLY when Phase 0 confirmed TLS fingerprinting is required (VTEX authed endpoints, Cloudflare-walled sites that reject plain Go).
- NO viper, NO cobra-cli generator, NO OAuth/jwt third-party libs. Auth is stdlib-pure.
- Go 1.26 (or latest stable).

## 2. `--json` on every command, stdout/stderr/exit contract

```
stdout = data (human table OR JSON)
stderr = diagnostics, progress, warnings, "✓ logged in" confirmations
exit 0 on success, exit 1 ONLY when Execute() returns error
```

- `--json` switches the command's output to `printJSON` (JSON encode to stdout, `SetEscapeHTML(false)`, indent 2).
- `SilenceUsage:true; SilenceErrors:true` on every command — cobra never prints; `main` controls all output.
- Errors return up the stack; `main` prints `error: <err>` to stderr and exits 1.
- Catalog strings rendered to stderr/terminal must be sanitized for ANSI/OSC sequences (a hostile catalog entry could otherwise inject escape sequences). Strip C0/C1/DEL.

## 3. No-login-first surface

Catalog reads, guest cart, region, sitemap, barcode lookup — anything the platform exposes unauthenticated — ships BEFORE account login. Login is additive.

This isn't just nicer UX; it's the discovery discipline. If you can't read the catalog without login, Phase 1 wasn't complete.

## 4. State under `~/.<name>/`, 0600 atomic

- Dir: `~/.<name>/` via `os.UserHomeDir()`, override env `<NAME>_HOME`. Created with `MkdirAll(0700)`.
- Files: `session.json` (0600), `config.json` (0600). Optional `cache.json` for id-resolution caches.
- **Atomic write** (the `writeSecret` pattern): `os.CreateTemp(dir, base+".*.tmp")` → Write → `Chmod(0600)` → Close → `os.Rename(tmp, p)`. Cleanup best-effort. Avoids TOCTOU perms window + truncation under concurrent writes.
- `Chmod(0600)` AFTER WriteFile defends against a relaxed umask (some systems set 0022 → 0644 on plain WriteFile).
- File-missing on Load → struct zero value. Never an error. (Callers shouldn't special-case "no session yet".)

## 5. Spending guard

- Overlay: `--max <eur>` flag → `<NAME>_MAX_EUR` env → `config.max_eur`. Zero or absent = no guard.
- Cap computed against the **all-in** total (items + shipping + tax) when the platform exposes it. Naive items-only guards approve mutations that push the real charge over the cap once shipping is bound.
- Fails **closed**: if the total can't be read (network error, malformed response), the mutation is refused.
- Behavior change vs naive guards MUST be documented in godoc.

## 6. Agent-first auth (three paths)

In order of agent-friendliness:

1. **Env var** `<NAME>_ACCESS_TOKEN` (and optionally `<NAME>_REFRESH_TOKEN`). In-memory overlay; **never written to disc** (env-sourced stripping via an unexported `envSourced` flag on `Session`). Highest precedence for CI.
2. **`login --token <value>`** — persists to `session.json` (0600). Agent/CI path when env isn't convenient.
3. **Interactive flow** (`login --start --json` → user opens `authorize_url` in browser → `login --code "<redirect-or-code>" --json`). TTY-gated: if stdin is not a TTY and no non-interactive input is given, error with instructions — never hang.

`whoami` always exits 0 and reports `logged_in` truthfully (verified server-side, never inferred from token presence).

## 7. Fixture round-trip tests + regression-per-bug

- Every API type pinned to a real captured response in `internal/api/testdata/<surface>_<id>.json`.
- Test pattern (`fixtures_test.go`):
  ```go
  func TestXFixture(t *testing.T) {
      var resp xResponse
      mustReadFixture(t, "x.json", &resp)
      // assert load-bearing fields populated
      b, err := json.Marshal(&resp)  // re-marshal
      var again xResponse
      if err := json.Unmarshal(b, &again); err != nil { t.Fatal(...) }
      // assert field-by-field stability
  }
  ```
- Synthetic fixtures only when a live capture would leak PII; mark them in the test header.
- `regressions_test.go`: one test per bug, commented with the bug ID + a one-line description of the original bug. Bugs found in Phase 6 MUST land here before the fix is declared done.

## 8. Reverse-engineering documented

`docs/<store>-api-discovery.md` with sections 1-7 (Hosts, Auth, Catalog, Cart, Region, Account Auth, Repro). Every endpoint pinned with a literal `http` code block. Per-section "Repro" with the exact CLI command.

This is for the next human or agent who has to update the CLI when the platform drifts. Without it, every API change is a Phase-1-from-scratch event.

## 9. Unofficial, trademark-safe

README disclaimer (paraphrased, keep the spirit):

> Independent, unofficial tool for educational and research purposes. Not affiliated with, endorsed by, or sponsored by `<Store>`. Reverse-engineered for interoperability with the public-facing HTTP endpoints the official clients use. Use responsibly; comply with the platform's ToS and rate limits.

Reference the trademark only to describe compatibility. Cite any bug-bounty program scope you're staying within.

## 10. Build hygiene

- `go test ./... && go vet ./... && go build` clean, always.
- No `init()` functions.
- No panics in `api`/`client`/`config`/`auth`. Errors return up.
- No `fmt.Println` in `internal/` — only in `cmd/main.go` and `cli/` rendering.
- Deps pinned in `go.mod`; no `replace` directives pointing at local paths.
