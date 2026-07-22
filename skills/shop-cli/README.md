# shop-cli-skill

A [skill](https://opencode.ai/docs/skills) (opencode / Claude Code) that reverse-engineers an e-commerce site or app and ships an unofficial, agent-friendly Go CLI for it, end-to-end.

Distilled from three reference projects — [`pmontp19/sirena-cli`](https://github.com/pmontp19/sirena-cli) (VTEX IO), [`pmontp19/bonpreu-cli`](https://github.com/pmontp19/bonpreu-cli) (Ocado Smart Platform), and `decathlon-cli` (mobile-app JSON API + OAuth2/PKCE) — which share architecture, conventions, and build discipline even though their backends are completely different.

The skill is **process-generic** (works for any backend) but ships **Go templates** so the agent scaffolds the project in minutes and focuses on the reverse-engineering work that's actually store-specific.

## Install

### As an opencode / Claude Code skill

Symlink this repo into your skills directory:

```sh
ln -s "$PWD" ~/.agents/skills/shop-cli
# Claude Code users get it automatically via the .claude/skills/ symlink
```

Once installed, the skill auto-triggers when the user asks to build a CLI for a specific online shop ("build a CLI for X", "another one like sirena but for Y", or invokes `shop-cli` by name).

### Standalone (no agent framework)

Just read [`SKILL.md`](./SKILL.md) and follow the 7 phases. Templates live under [`templates/`](./templates/). References under [`references/`](./references/).

## What the skill produces

A Go CLI satisfying the **non-negotiable contract** (see `SKILL.md`):

- Single static Go binary, stdlib + cobra (+ optional `bogdanfinn/tls-client` only when TLS fingerprinting is required).
- `--json` on every command. stdout = data, stderr = diagnostics, exit 1 only on `Execute()` error.
- **No-login-first surface.** Catalog + guest cart ship before account login.
- State under `~/.<name>/` (0600 atomic), override `<NAME>_HOME`.
- **Spending guard** (`--max` / env / config), computed against the all-in total.
- **Agent-first auth** (env var → `login --token` → interactive flow); env tokens never written to disk.
- **Fixture round-trip tests** against real captures + one regression test per bug.
- Reverse-engineering documented in `docs/<store>-api-discovery.md`.
- Unofficial, trademark-safe disclaimer.

## The 7 phases

1. **Phase 0 — Feasibility.** One-line verdict, executable repro of the assumed blocker, backend ID, endpoint inventory, legal context. ≤1h of research.
2. **Phase 1 — API discovery.** Hosts, auth model, catalog, cart, region, account auth, repro. Real captures land in `internal/api/testdata/`.
3. **Phase 2 — Scaffold.** Render templates. `go build && go test && go vet` clean.
4. **Phase 3 — Catalog MVP.** `search`, `product`, `categories`, `brands` (whatever's unauthenticated).
5. **Phase 4 — Guest cart + region.** Spending guard covers all-in total; `clear` re-fetches.
6. **Phase 5 — Account auth.** Token injection (MVP) + interactive flow (best-effort).
7. **Phase 6 — Adversarial review.** H/M/L classification; fixes + regression tests.
8. **Phase 7 — Polish & ship.** README accurate, goreleaser, CI, `git tag v0.1.0`.

See [`SKILL.md`](./SKILL.md) for the full playbook and [`references/`](./references/) for the why behind each rule.

## Templates

```
templates/
├── go.mod.tmpl
├── cmd/main.go.tmpl
├── internal/
│   ├── client/client.go.tmpl       # transport, headers, auth attach, SyncSession dirty flag
│   ├── config/config.go.tmpl       # ~/.<name>/{session,config}.json (0600 atomic), env overlay, envSourced strip
│   ├── auth/auth.go.tmpl           # JWT decode (signature NOT verified — display only)
│   ├── api/
│   │   ├── catalog.go.tmpl         # Search + Product type + Parse* debug hook
│   │   ├── fixtures_test.go.tmpl   # round-trip test pattern
│   │   ├── regressions_test.go.tmpl# one test per bug, commented with bug ID
│   │   └── testdata/search_placeholder.json
│   └── cli/
│       ├── root.go.tmpl            # NewRoot + AddCommand block + PreRun/PostRun
│       ├── runtime.go.tmpl         # ctxValue memo + overlay + maxEUR guard
│       ├── context.go.tmpl         # Flags + rtHolder context plumbing
│       └── search.go.tmpl          # example cobra command (canonical shape)
├── docs/
│   ├── feasibility.md.tmpl        # Phase 0 output
│   ├── api-discovery.md.tmpl      # Phase 1 output (sections 1-7)
│   └── README.md.tmpl             # the new CLI's README (disclaimer + quickstart)
├── AGENTS.md.tmpl                  # multi-agent coordination scratchpad
├── Makefile.tmpl
├── .goreleaser.yaml.tmpl
├── .gitignore.tmpl
└── .github/workflows/ci.yml.tmpl
```

Template variables (`{{.Var}}` Go-template syntax):

| Variable | Meaning | Example |
|---|---|---|
| `{{.Name}}` | binary name (lowercase) | `sirena` |
| `{{.Pascal}}` | PascalCase (type names) | `Sirena` |
| `{{.Upper}}` | UPPER_SNAKE (env/const) | `SIRENA` |
| `{{.Module}}` | go.mod module path | `github.com/pmontp19/sirena-cli` |
| `{{.Store}}` | human store name | `La Sirena` |
| `{{.StoreURL}}` | storefront URL | `https://www.lasirena.es` |
| `{{.Backend}}` | backend type | `VTEX IO + catalog/checkout` |

## References

- [`layering.md`](./references/layering.md) — the 4-package invariant (`cli → api → client → config/auth`).
- [`conventions.md`](./references/conventions.md) — the 10 rules every shipped CLI follows.
- [`process.md`](./references/process.md) — the 7 phases expanded with edge cases.
- [`anti-patterns.md`](./references/anti-patterns.md) — real bugs from the reference projects, with fixes.

## Reference projects

- [`pmontp19/sirena-cli`](https://github.com/pmontp19/sirena-cli) (private) — VTEX IO + catalog/checkout. The most complete reference: catalog + guest cart + region + token-injection + magic-link best-effort + TUI. Read its `AGENTS.md` for the multi-agent pattern.
- [`pmontp19/bonpreu-cli`](https://github.com/pmontp19/bonpreu-cli) (public) — Ocado Smart Platform white-label. HAR/cURL session import, CSRF retry, `checkout open` hand-off.
- `decathlon-cli` — mobile-app JSON API + OAuth2/PKCE. Not yet published to GitHub.

## License

MIT.
