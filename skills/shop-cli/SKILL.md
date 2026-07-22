---
name: shop-cli
description: Reverse-engineer an e-commerce site/app and ship an unofficial, agent-friendly Go CLI for it, end-to-end. Use when the user asks to build a CLI for a specific online shop (VTEX, Ocado, mobile-app JSON API, scraped storefront, etc.), or invokes shop-cli by name, or mentions "another bonpreu/decathlon/sirena-style CLI". Produces a single static Go binary with --json on every command, fixture-pinned round-trip tests, no-login-first surface, optional account auth, and an adversarial-review pass.
---

# shop-cli

A repeatable process for shipping unofficial, agent-friendly Go CLIs for online shops. Distilled from three reference projects — `bonpreu-cli` (Ocado Smart Platform white-label), `decathlon-cli` (mobile-app JSON API + OAuth2/PKCE), and `sirena-cli` (VTEX IO + catalog/checkout, the most complete) — which share the same architecture, conventions, and build discipline even though their backends are completely different.

The skill is **process-generic** (it works for any backend) but ships **Go templates** so the agent can scaffold the project in minutes and focus on the reverse-engineering work that is actually store-specific.

## When to use

- User asks to build a CLI for a specific online shop ("build a CLI for Carrefour", "do a mercadona-cli", "one like sirena but for X").
- User invokes `shop-cli` by name.
- User mentions "another bonpreu/decathlon/sirena-style CLI".
- The target is an e-commerce site or shopping app with no public API, where the CLI must talk to the same HTTP endpoints the official client uses.

**When NOT to use:** The shop already publishes an official SDK/API (just wrap it). Or the user wants a non-Go CLI. Or the target is a marketplace with a documented partner API (use that instead of reverse-engineering).

## The non-negotiable contract

Every CLI produced by this skill MUST satisfy all of these. They are not optional polish — they are the architecture. Each reference project (bonpreu, decathlon, sirena) implements every one:

1. **Single static Go binary**, stdlib + cobra (+ optional `bogdanfinn/tls-client` only when TLS fingerprinting is required).
2. **`--json` on every command.** stdout = data (JSON or human table); stderr = diagnostics; exit 1 only on `Execute()` error. No mixed streams.
3. **No-login-first surface.** Catalog reads, guest cart, and any other unauthenticated surface ship before account login. Login is additive.
4. **State under `~/.<name>/`** (override `<NAME>_HOME`): `session.json` and `config.json`, both **0600**, atomic write (temp + rename), dir 0700.
5. **Spending guard.** `--max <eur>` / `<NAME>_MAX_EUR` / `config.max_eur` overlay (flag > env > config) refuses cart mutations over the cap, computed against the order's all-in total (items + shipping + tax) when available.
6. **Agent-first auth.** Four paths, in order of agent-friendliness: env var (`<NAME>_ACCESS_TOKEN` or `<NAME>_COOKIEJAR`) → `login --token <value>` / `login --cookies cookies.json` (CI) → interactive flow (`login --start` / `--code`, TTY-gated, never hangs). Secrets from env are **never written to disk** (env-sourced stripping).
7. **Fixture round-trip tests.** Every API type pinned to a real captured response in `internal/api/testdata/*.json`. Test: `json.Unmarshal → assert load-bearing fields → re-marshal → re-decode → assert stable`. Plus one regression test per bug, commented with the bug ID.
8. **Reverse-engineering documented** in `docs/<store>-api-discovery.md` with sectioned endpoints, literal request/response, and a per-section "Repro" with the exact CLI command.
9. **Unofficial, trademark-safe.** README disclaimer: not affiliated, reverse-engineered for interoperability, "use responsibly, sane request rate". Reference the trademark only to describe compatibility.
10. **`go test ./... && go vet ./... && go build` clean.** No warnings.

## The 7 phases

Run phases in order. Each phase has a concrete definition of done. Do not skip ahead — the discovery phase determines the architecture of everything downstream. Within a phase, work in thin vertical slices (one command, end-to-end, before the next).

### Phase 0 — Feasibility (gate, ≤1h of research)

**Output:** `docs/reference-<store>-feasibility.md` with a one-line verdict up top.

Before writing any code, confirm the project is viable. Capture:

- **Verdict in line 1.** Either "viable: <path>" or "blocked: <reason>".
- **Re-test the prior assumption.** Whatever you (or the user) assumed was the blocker — "Cloudflare will wall us", "needs TLS fingerprinting", "login requires app-only OTP" — write the **executable repro** that proves or refutes it. (`curl` first; escalate to `curl_cffi(impersonate="chrome")` or `tls-client` only if needed.) Most "blockers" are environment artifacts.
- **Backend identification.** VTEX IO? Ocado Smart Platform? Custom Next.js? Mobile-app JSON API? SAP Hybris? Shopify? This determines everything downstream.
- **Endpoint surface inventory.** Open devtools (web) or MITM the app (mobile). List the host(s), the auth header(s) (cookies? `x-api-key`? `Bearer`? CSRF?), and one example request per surface (catalog search, product detail, cart).
- **Login friction.** Does the login page gate the password step with reCAPTCHA / device OTP / custom-scheme redirect? If so, plan for the **browser-then-paste-code** pattern (`gh`/`claude`/`codex` style) and the **token-injection MVP** (`--token` / env var) as the agent/CI path.
- **Legal / ToS / bug-bounty context.** Does the target run a bug bounty? What does it cover? Document the scope you'll stay within (catalog/cart reads, sane rate). Cite the program URL.
- **Existing landscape.** Search GitHub for prior CLIs/scrappers for the same target. Note what to reuse and what to avoid.

**Done when:** the doc has a verdict, a backend ID, an executable repro of the (non-)blocker, and an endpoint inventory sufficient to start discovery. If verdict is "blocked", stop and report — don't burn cycles.

### Phase 1 — API discovery (the core reverse-engineering)

**Output:** `docs/<store>-api-discovery.md`, structured per the template. Fixture captures start landing in `internal/api/testdata/` (these become the round-trip tests in Phase 3+).

Use whatever capture method matches the backend:

- **Web storefront (preferred):** record a full HAR with **`agent-browser`** (auto-embeds JSON/HTML response bodies so you can study endpoint shapes offline):
  ```
  agent-browser open https://store.example
  # log in BEFORE starting the HAR so credentials don't land in the recording
  agent-browser network har start
  # ...drive every flow you intend to ship: search, detail, paginate, add-to-cart...
  # ...run each flow TWICE with different inputs (two search terms, two product ids)...
  agent-browser network har stop /tmp/<store>.har
  agent-browser cookies get --json > /tmp/<store>-cookies.json   # live session, separate file
  ```
  Then filter with `jq` (see `references/process.md`). Treat both files as **secrets**: keep them under `/tmp/`, never under the repo, never commit. `.gitignore` blocks them as a belt-and-braces safety.
- **Web storefront (alternative, no agent-browser):** open Chrome DevTools → Network → tick "Preserve log" → drive the flows → click the **⬇ Export HAR** button on the Network panel. Same `jq` recipes apply. Or use the DevTools MCP tools (`list_network_requests` + `get_network_request <reqid>`) to pick individual endpoints without a HAR; only the last 3 navigations are kept (`includePreservedRequests: true`), so prefer a real HAR for anything multi-page.
- **Mobile app:** MITM the official app on a rooted emulator (or rooted physical device). Note: APK/IPA decompilation surfaces candidate endpoints and static keys (`x-api-key`, `client_id`) but doesn't replace live capture.
- **VTEX IO storefront:** read `window.__INITIAL_STATE__` / `__RUNTIME__` from the homepage; the GraphQL queries are in the React bundle.

For each endpoint you'll need, capture into the doc, in this order of sections (mirror `templates/docs/api-discovery.md.tmpl`):

1. **Hosts and routing** — every host, its role (storefront / CDN / API / auth), edge CDN, observed TLS fingerprinting.
2. **Auth model** — guest session mechanism (cookie/JWT/CSRF) with the exact request that mints it; the public catalog reads; what auth IS needed for.
3. **Catalog** (unauthenticated) — search, product detail, variations, categories, brands, autocomplete/suggest, barcode lookup, sitemap.
4. **Cart** (guest, if it exists) — orderForm/basket shape, add/set/remove/clear, the spending-guard hook.
5. **Region / shipping** (if applicable) — postal-code binding, SLA lookup.
6. **Account auth** — token injection (always shippable), OAuth2/PKCE or magic-link (best-effort, blocked on real account), JWT claim shape, persistence model.
7. **Repro** — the exact CLI commands that exercise each section.

**Discipline:**
- **Run each flow twice with different inputs** (two search terms, two product ids, two postal codes). Diff the recorded URLs across runs — the parts that change are parameters, the parts that don't are the endpoint. Single-pass captures hide this distinction.
- **Log in BEFORE starting the HAR** if you need authed captures, so login credentials (POST body, OTP) don't land in the recording. The live session cookies are exported separately (`agent-browser cookies get --json`).
- Every endpoint pinned with a literal `http` code block (request + response).
- Capture **real responses** into `internal/api/testdata/<surface>_<id>.json`. Synthetic fixtures only when the live response would leak PII; mark them as such in the test.
- Note every gotcha inline (VTEX's `fq=C:/<...>/` trailing-slash, the singular `orderForm` endpoint, lowercase `skuId:`, the host-scoped `x-api-key`, the BFF `Accept: vnd.<x>.v2+json` requirement). These are the bugs that cost hours in Phase 6.
- **HAR and cookie exports are secrets.** They contain live session tokens and POST bodies. Keep them in `/tmp/` (or outside the repo), delete when done. `.gitignore.tmpl` blocks `*.har` and `cookies.json` as a safety net.

**Done when:** the doc covers sections 1-3 fully (plus 4-5 if guest cart exists), each with a real fixture. Section 6 sketched even if not yet implemented.

### Phase 2 — Scaffold (one-shot, ~30min)

**Output:** working `go build`, `go test ./...` clean, a CLI that prints help and exits 0.

Render the templates into the new repo. Variables the templates expect (and where they come from):

| Variable | Meaning | Example |
|---|---|---|
| `{{.Name}}` | binary name (lowercase, no spaces) | `sirena` |
| `{{.Pascal}}` | PascalCase (for type names) | `Sirena` |
| `{{.Upper}}` | UPPER_SNAKE (env vars, constants) | `SIRENA` |
| `{{.Module}}` | go.mod module path | `github.com/pmontp19/sirena-cli` |
| `{{.Store}}` | human-readable store name | `La Sirena` |
| `{{.StoreURL}}` | storefront URL | `https://www.lasirena.es` |
| `{{.Backend}}` | backend type (free text) | `VTEX IO + catalog/checkout` |

Files to render:

- `go.mod` (module = `{{.Module}}`, `go 1.26`, dep on `cobra`; add `bogdanfinn/tls-client` only if Phase 0 found fingerprinting is required).
- `cmd/{{.Name}}/main.go` — 20-line entrypoint.
- `internal/cli/{root,runtime,context}.go` — cobra wiring + runtime memo + flags.
- `internal/client/client.go` — transport skeleton (headers, cookies, base URL constant).
- `internal/config/config.go` — `~/.{{.Name}}/` persistence, 0600 atomic, `<NAME>_HOME` override.
- `internal/api/fixtures_test.go` + `regressions_test.go` — empty but importing cleanly.
- `internal/auth/auth.go` — JWT decode helper (signature NOT verified — bearer is opaque, server validates).
- `Makefile`, `.goreleaser.yaml`, `.gitignore`, `.github/workflows/ci.yml`.
- `README.md` (from template — disclaimer + status + quickstart placeholders).
- `AGENTS.md` (multi-agent coordination scratchpad — copy the rules from any reference project).
- `docs/<store>-api-discovery.md` (from Phase 1, already drafted).

**Done when:** `go build -o bin/{{.Name}} ./cmd/{{.Name}}` succeeds; `./bin/{{.Name}} --help` prints and exits 0; `go test ./... && go vet ./...` clean; `git init && git add -A && git commit`.

### Phase 3 — Catalog MVP (no-login surface)

**Output:** first shippable surface. `search`, `product`, `categories`/`category`, `brands`/`brand` — whatever the discovered endpoints support.

Build in vertical slices: one command, end-to-end (api op → cli cmd → fixture test → live smoke), then the next.

For each command:
1. Add the **api operation** in `internal/api/<topic>.go`: typed function `(ctx, *client.Client, ...) → (typed result, error)`. Define the response type with explicit fields for what's load-bearing; capture the rest in an `Attributes map[string][]string` (or similar) via a custom `UnmarshalJSON` that does the **double-decode** pattern (decode known fields + decode the full object into a `map[string]json.RawMessage` and walk it for dynamic keys). See `references/layering.md` — `api` never touches disc, only the `*client.Client`.
2. Add a **fixture** `internal/api/testdata/<topic>_<id>.json` from a real capture (or a synthetic one marked as such).
3. Add a **round-trip test** in `internal/api/fixtures_test.go`: unmarshal fixture into the type, assert load-bearing fields populated, re-marshal, re-decode, assert stable.
4. Add the **cobra command** in `internal/cli/<topic>.go`: `func newXxxCmd() *cobra.Command` with `SilenceUsage:true, SilenceErrors:true`, `RunE` that does `rt := ctxValue(ctx)` → `api.X(ctx, rt.client, ...)` → `if rt.json { printJSON(...) } else { renderXxx(...) }`. Register it in `internal/cli/root.go` by adding **one line** to the `AddCommand(...)` block.
5. **Live smoke** against the real site. If it works, commit. If it doesn't, the discovery doc or the fixture is wrong — go back to Phase 1 for that endpoint.

**Done when:** the no-login catalog surface ships end-to-end with fixtures, tests, and live verification. README "Success criteria" checkboxes ticked for these commands.

### Phase 4 — Guest cart + region (if applicable)

**Output:** `cart {get,add,set,remove,clear}` working on the guest cart. `region {set,clear}` if the platform exposes shipping SLAs.

This is where the **spending guard** lands. Implementation rule:

- `OrderForm.TotalCents()` (or equivalent) MUST return the **all-in** total (items + shipping + tax) when the platform exposes it, falling back to items-only. Document this in the godoc — it's a behavior change vs naive "items total" guards.
- The guard refuses mutations whose projected total exceeds `--max`/env/config. Fails **closed** if the total can't be read.
- `cart clear` re-fetches the orderForm immediately before building the zero-quantity updates (refuses to clear if the re-fetch fails — safer than zeroing possibly-stale indices). True atomicity needs server-side support; document the limitation.

SKU/productId conflation is the most common cart bug. If the platform distinguishes them, `cart add <sku>` MUST look up the **SKU** (not the parent product's default SKU) for both the cart mutation AND the spending guard's price read. Use the platform's `fq=skuId:<id>`-equivalent filter; if rejected, fall back to `GetVariations` and walk the items for the exact match.

**Done when:** guest cart works end-to-end with spending guard covering all-in total, plus region binding if the platform supports it. Regression tests for: SKU lookup, all-in total, clear race.

### Phase 5 — Account auth (additive, after no-login ships)

**Output:** `login` / `logout` / `whoami`, in order of agent-friendliness.

1. **Token injection (MVP, always shippable).** `login --token <value>` and env var `<NAME>_ACCESS_TOKEN`. Both persist to `session.json` (0600) — **except** env-sourced tokens, which live in-memory only and are stripped by `config.SaveSession` before writing (the `envSourced` unexported flag). `whoami` live-verifies the token against the platform's session probe; never reports `logged_in:true` without a server-side check. `logout` clears it.
2. **Cookie-jar injection (MVP, always shippable, when the platform is cookie-based).** `login --cookies cookies.json` (file produced by `agent-browser cookies get --json` while the live session was still open) or env var `<NAME>_COOKIEJAR` pointing at a file path. The client loads the cookies into its jar at startup; `whoami` live-verifies; `logout` clears them. Cookies loaded from a file are persisted in `session.json` (0600) just like `--token` values — **except** env-sourced paths, which are read on each invocation and never copied into `session.json` (envSourced flag, same pattern as tokens). Use this when the platform's auth is a session cookie + CSRF pair instead of a bearer token, or as a fallback when the interactive login flow is gated by reCAPTCHA/OTP/custom-scheme redirect the CLI can't satisfy.
3. **Interactive flow (best-effort).** Whatever the platform requires: OAuth2/PKCE (public client, no secret) or VTEX-ID magic-link/classic. Pattern: `login --start --json` prints `{authorize_url, verifier, state}` and saves the verifier; user completes login in a browser, pastes back the redirect URL or code; `login --code "<...>"` exchanges and stores. If login is gated by reCAPTCHA/OTP/custom-scheme redirect that a CLI can't satisfy headlessly, document it — the token/cookie MVPs are the agent path, full interactive flow is a human convenience.
4. **Auto-refresh.** Access tokens carry an absolute `ExpiresAt` (RFC3339, never `expires_in` relative). Refresh lazily before any authed call, with a margin (60s for access tokens, hours for guest tokens). If the refresh token is missing or rejected, fail with an actionable error ("re-login with `{{.Name}} login`").

State model invariants:
- `Session.LoggedIn()` / `Account.AccessTokenValid()` / guest-token-valid — all respect the margin; malformed expiry = invalid = refresh.
- `client.SyncSession()` returns a `dirty` bool; `cli`'s `PersistentPostRunE` persists only when dirty.

**Done when:** token-injection verified end-to-end live. Interactive flow wired (happy path exercised if a real account is available; otherwise documented as best-effort with the blocker cited). `whoami` truthful in all three states (logged-in-and-verified, env-overlay, expired).

### Phase 6 — Adversarial review (mandatory, after Phase 3-5 land)

**Output:** a written review classifying every issue H/M/L, and fixes for all H + most M. Regression test per fix.

Run the review **after** the implementers finish, NOT in parallel — running it on work-in-progress produces false negatives (the reviewer flags unfinished work as "broken"). Pattern from `sirena-cli/AGENTS.md`: implementers land → build/test green → THEN launch the reviewer on the consolidated state.

The reviewer asks, per surface:

- **Correctness:** does the code actually do what the discovery doc claims? Is SKU vs productID conflated anywhere? Does the spending guard cover shipping? Does `clear` race? Are silent Parse* failures swallowed? Does the auth `whoami` ever lie?
- **Fidelity:** does the JSON output drop fields the discovery doc captured? Are dynamic attribute keys with non-string JSON scalars (number/bool/null) collapsed to text? Is EAN/barcode validation only format-deep (no GS1 check digit)?
- **Robustness:** what happens on network failure mid-mutation? On malformed JSON? On expired tokens during a flow? On concurrent CLI invocations?
- **Security:** are env-sourced tokens ever written to disk? Are files actually 0600 (not just `WriteFile(0600)` — also `Chmod(0600)` after, against relaxed umask)? Are catalog-sourced strings sanitized before stderr (ANSI/OSC injection)?
- **Performance:** any obvious n² in the hot path (re-parse inside a sort comparator, per-item re-fetch)?

Classify H (must fix before ship), M (should fix), L (doc-only). Each fix lands with a regression test in `internal/api/regressions_test.go` (or the surface-local `_test.go`), commented with the bug ID and a one-line description of the original bug.

**Done when:** zero open H issues; M issues either fixed or documented in README "Open / planned"; every fix has its regression test.

### Phase 7 — Polish & ship

**Output:** tagged release; README accurate; CI green.

- README: real Quickstart (commands the user can copy-paste), "Status" reflecting what's actually shipped, "Success criteria" checklist, layout section, disclaimer.
- `.goreleaser.yaml` (linux/darwin × amd64/arm64, `CGO_ENABLED=0`, ldflags stamping `main.version`).
- CI: `go test ./... && go vet ./... && go build` on push/PR. No live smoke in CI (it talks to a real API) — that's manual.
- `git tag v0.1.0`.

## How to use this skill (for the agent)

1. **Read this file in full** before starting. Don't paraphrase the contract — implement it literally.
2. **Read the reference projects** if you need to see a working example of a phase. All three are on GitHub: [`pmontp19/sirena-cli`](https://github.com/pmontp19/sirena-cli) (private; the most complete — start there), [`pmontp19/bonpreu-cli`](https://github.com/pmontp19/bonpreu-cli) (public), `pmontp19/decathlon-cli` (not yet published; local clone at `~/Developer/decathlon-cli` while WIP).
3. **Render templates from `templates/`** for Phase 2. Use the variables above. Templates are starting points, not gospel — adapt them to the discovered backend. But the **layering** (`cli → api → client → config/auth`) and the **invariants** (no-disc-from-api, no-net-from-cli, 0600 atomic, env-token-strip) are not negotiable.
4. **Check `references/`** for the why behind each rule: `layering.md`, `conventions.md`, `process.md`, `anti-patterns.md`.
5. **Use `AGENTS.md`** (from `templates/AGENTS.md.tmpl`) for multi-agent coordination. Rules: read-before-edit shared files, no big-bang edits to `root.go` (only add lines to `AddCommand(...)`), `go test ./...` green before declaring done, no commits (human reviews), adversarial reviewer runs AFTER implementers.
6. **Phase order is not negotiable.** Skipping Phase 0/1 produces a CLI that ships against assumptions instead of evidence. Skipping Phase 6 ships latent bugs that fail in the field.

## Reference projects (read these)

- [`github.com/pmontp19/sirena-cli`](https://github.com/pmontp19/sirena-cli) — VTEX IO + catalog/checkout. The most complete reference: catalog + guest cart + region + token-injection + magic-link best-effort + TUI. Read its `AGENTS.md` for the multi-agent pattern, its `docs/sirena-api-discovery.md` for the discovery doc shape, and its `internal/api/regressions_test.go` for the regression-per-bug pattern. (Private repo — you need access.)
- [`github.com/pmontp19/bonpreu-cli`](https://github.com/pmontp19/bonpreu-cli) — Ocado Smart Platform white-label. Reference for: HAR/cURL session import, CSRF retry, delivery-slot reservation, `checkout open` hand-off (completes 3DS in the browser rather than implementing payment).
- `decathlon-cli` — Mobile-app JSON API + OAuth2/PKCE. Reference for: MITM-the-app discovery, host-scoped `x-api-key`, public-client PKCE login flow with custom-scheme redirect, multi-backend resilience design. **Not yet published to GitHub**; WIP local clone at `~/Developer/decathlon-cli`.
