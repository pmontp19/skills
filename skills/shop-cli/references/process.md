# Process (expanded)

The 7-phase playbook in `SKILL.md` is the canonical reference. This file expands on the why and the edge cases, for the agent who needs to make a judgment call mid-phase.

## Phase 0 — Feasibility

### What counts as "viable"

- The target serves its data over plain HTTPS to a real client (browser or app).
- That data includes at least the catalog surface (search + product detail) without requiring account login.
- The CLI can identify itself honestly (UA, headers) and you can stay within a sane request rate without ToS violations.

### What counts as "blocked"

- The platform gates ALL reads behind an account login that requires a paid subscription.
- The platform's bot-detection can't be satisfied without actively evading it in ways that cross the line from "interoperability with the public-facing endpoints" to "defeating a security control". Stop and report; don't push.
- The legal team said no. (Real reason. Cite it.)

### Backend identification cheat sheet

| Signal | Backend |
|---|---|
| `window.__RUNTIME__` or `__INITIAL_STATE__` with `vtex` keys | VTEX IO |
| `/api/catalog_system/pub/...` paths | VTEX catalog |
| `/api/checkout/pub/orderForm` paths | VTEX checkout |
| `ecom-request-source` / `client-route-id` headers | Ocado Smart Platform |
| `*.shoppingapp.decathlon.com` hosts | Decathlon mobile BFF |
| `/api/2024-10/graphql.json` paths | Shopify Storefront API |
| SAP Hybris `YAPI`/`occ/v2/` paths | SAP Commerce |

## Phase 1 — Discovery

### Capture methods

Pick by what the target exposes. Prefer `agent-browser` for web storefronts: it produces a real HAR with response bodies embedded, so the discovery can continue offline after the browser closes.

| Backend | Method | Notes |
|---|---|---|
| Web storefront (preferred) | `agent-browser network har start` → drive → `har stop /tmp/x.har` | HAR embeds JSON/HTML bodies by default. Per-body cap 2 MB. `--content all` adds binary base64; `--content none` disables embedding. |
| Web storefront (alt) | Chrome DevTools UI → Network → "Preserve log" → ⬇ Export HAR | One-click HAR export, same `jq` recipes apply. |
| Web storefront (no HAR) | DevTools MCP `list_network_requests` + `get_network_request <reqid>` | No HAR export; only last 3 navigations kept (`includePreservedRequests: true`). Use for one-off lookups, not multi-page flows. |
| Mobile app | MITM on rooted emulator/device. APK/IPA decompile for static keys + endpoint candidates. | Static analysis doesn't replace live capture. |
| VTEX IO | read `__RUNTIME__` from homepage; GraphQL queries in React bundle. | |
| GraphQL API | devtools → Network → look for the GraphQL POST; capture query + variables + response. | |

### HAR capture workflow (agent-browser)

```
# 1. Open the site and log in BEFORE starting the HAR — keeps login POST bodies
#    (email, OTP) out of the recording. The session cookies are exported
#    separately at the end.
agent-browser open https://store.example
# ...complete login in the browser session...

# 2. Start recording.
agent-browser network har start

# 3. Drive every flow the CLI will ship, TWICE with different inputs:
agent-browser open "https://store.example/search?q=gamba"
agent-browser open "https://store.example/search?q=pollastre"
agent-browser open "https://store.example/product/1468"
agent-browser open "https://store.example/product/201"
# ...add-to-cart, paginate, region set, etc....

# 4. Stop and export.
agent-browser network har stop /tmp/store.har

# 5. Export the live session cookies to a separate file (for cookie-jar auth).
agent-browser cookies get --json > /tmp/store-cookies.json
```

While the session is still open, `agent-browser network requests --filter api` and `network request <id>` give the same data interactively — useful for quick lookups — but only the HAR survives navigation and browser close.

### Filtering the HAR with jq

```sh
# All JSON API calls: method, status, URL
jq -r '.log.entries[]
  | select(.response.content.mimeType | test("json"))
  | "\(.request.method) \(.response.status) \(.request.url)"' /tmp/store.har

# Filter noise (analytics, third-party)
jq -r '.log.entries[]
  | select(.response.content.mimeType | test("json"))
  | select(.request.url | test("google-analytics|segment|sentry|datadog|doubleclick|/collect|/track|/beacon|/log") | not)
  | "\(.request.method) \(.response.status) \(.request.url)"' /tmp/store.har

# Diff two requests to the same endpoint to find which parts are parameters
jq '.log.entries[] | select(.request.url | test("api/search")) | .request.url' /tmp/store.har | sort -u

# Full detail for one endpoint: headers, POST body, response body
jq '.log.entries[] | select(.request.url | test("api/search"))
  | {request: {method: .request.method, headers: .request.headers,
     postData: .request.postData.text},
     response: .response.content.text}' /tmp/store.har
```

### Reading the capture

- **Response schema**: `.response.content.text` is the real payload — derive Go types from it.
- **Auth headers**: diff request headers across endpoints. Look for `authorization`, `cookie`, `x-csrf-token`, `x-api-key`, and site-specific `x-*` headers. Replay only the ones the API actually requires — verify by omission in Phase 3.
- **Cookies**: the live session is in the separate `cookies.json` from step 5. Load that into the client for cookie-jar auth (Phase 5 §2). Never copy cookie values into source.

### Fixture discipline

- Real captures are best. They pin the response↔type drift.
- Synthetic fixtures (marked in the test header) only for PII: account-specific data, real names, real addresses.
- One fixture per shape variation: `search_<term>.json`, `product_<id>.json`, `category_<id>.json`. Don't make a single "example.json".
- Re-fetch and replace fixtures when the platform's schema drifts (Phase 6 will catch this).

### Gotcha documentation

Every Phase 1 gotcha becomes a Phase 6 review item. The discovery doc is not just for the human — it's the spec the adversarial reviewer audits against.

Examples (real, from reference projects):
- VTEX `fq=C:/<rootId>/<...>/<leafId>/` — leading AND trailing slash. Missing either → empty result, no error.
- VTEX `orderForm` singular, not `orderForms`.
- VTEX `fq=skuId:` lowercase. `SkuId:` rejected silently.
- Decathlon: `postToken` MUST NOT include `x-api-key` (trips the gateway).
- Decathlon: BFF endpoints require `Accept: application/vnd.dkt.shoppingapp.v2+json`.
- Decathlon: redirect URI `dktappmobile://auth` whitelisted; `localhost` rejected server-side.

## Phase 2 — Scaffold

### Template variables

The templates use Go-template syntax `{{.Var}}`. The agent must resolve these before writing the file. Cheatsheet:

```
{{.Name}}      sirena
{{.Pascal}}    Sirena
{{.Upper}}     SIRENA
{{.Module}}    github.com/pmontp19/sirena-cli
{{.Store}}     La Sirena
{{.StoreURL}}  https://www.lasirena.es
{{.Backend}}   VTEX IO + catalog/checkout
```

### When to add `bogdanfinn/tls-client`

Add it (and the `fhttp` fork) ONLY if Phase 0 confirmed TLS fingerprinting is required. Plain Go `net/http` is the default. Indicators you need fingerprinting:

- Authed endpoints return 401 to plain Go even with a valid cookie.
- The site's bot detection scores on ClientHello + HTTP/2 SETTINGS + header order.
- `curl` gets a JS challenge but `curl_cffi(impersonate="chrome")` passes.

If you add it, pin the `profiles.Chrome_XXX` constant matching the UA you claim to be (stale profile + fresh UA = bot signal). sirena-cli uses `profiles.Chrome_146`.

### CI

`.github/workflows/ci.yml`: trigger on push/PR touching `cmd/**, internal/**, go.mod, go.sum, .github/workflows/ci.yml`. Job `test`: setup-go (version-file=go.mod, cache) → `go test ./...` → `go vet ./...` → `go build -o /tmp/{{.Name}} ./cmd/{{.Name}}`. NO live smoke in CI (talks to real API); that's manual pre-release.

## Phase 3-5 — Implementation

### Endpoint verification triage

When a generated command fails against the live API, this is the triage table (borrowed from `agent-browser`'s `derive-client` skill — it nails the common shapes):

| Symptom | Cause | Fix |
|---|---|---|
| 401 / 403 | Expired or missing session | Re-login via `agent-browser` / browser, re-export cookies/token. |
| 403 / 419 on writes | CSRF token is per-session or per-form | Fetch the CSRF endpoint first; or keep that flow browser-driven. (sirena-cli: `segment` cookie drives the CSRF check.) |
| Works once then breaks | Signed or expiring request params (HMAC, `_ts`, rotating nonces) | Fall back to the browser for that step; derive the rest. Don't try to reproduce the signing client-side unless it's documented. |
| Response shape differs from HAR | A/B tests, geo-dependent responses, locale variants | Re-record and treat the union of fields as optional. Custom `UnmarshalJSON` (see below) is what makes this tolerable. |
| 406 / 415 | Missing `Accept` with vendor version, e.g. `application/vnd.dkt.shoppingapp.v2+json` | Reproduce the exact `Accept`/`Content-Type` headers the official client sends. |
| 400 on `POST` | Body shape drifted, extra/missing required field | Diff the captured `postData.text` against your struct's marshaled output byte for byte. |

### Vertical slice shape

One command, end-to-end, then the next:

1. `internal/api/<topic>.go` — typed op + response type + custom `UnmarshalJSON` if needed.
2. `internal/api/testdata/<topic>_<id>.json` — fixture.
3. `internal/api/fixtures_test.go` — round-trip test.
4. `internal/cli/<topic>.go` — cobra command. Register in `root.go` AddCommand block (one line).
5. Live smoke. If broken, back to Phase 1 for that endpoint.
6. Commit.

### Custom `UnmarshalJSON` for dynamic keys

VTEX-like platforms put dynamic per-product columns as top-level JSON keys. The double-decode pattern:

```go
type rawProduct struct {
    ProductID string `json:"productId"`
    // ... all known fields ...
    Items []Item `json:"items"`
}

var knownProductKeys = map[string]struct{}{
    "productId": {}, /* ... all known keys ... */ "items": {}, "attributes": {},
}

func (p *Product) UnmarshalJSON(data []byte) error {
    var raw rawProduct
    if err := json.Unmarshal(data, &raw); err != nil { return err }
    p.ProductID = raw.ProductID
    // ... copy known fields ...

    var obj map[string]json.RawMessage
    if err := json.Unmarshal(data, &obj); err != nil { return err }

    p.Attributes = nil
    if rawAttrs, ok := obj["attributes"]; ok {
        _ = json.Unmarshal(rawAttrs, &p.Attributes)  // our own form
    }
    if p.Attributes == nil {
        p.Attributes = make(map[string][]string, len(obj))
    }
    for k, v := range obj {
        if _, ok := knownProductKeys[k]; ok { continue }
        // try []string, then string, then number:/bool:/null: prefixes,
        // then raw JSON text for objects/non-string arrays.
    }
    return nil
}
```

### The `printJSON` helper

```go
func printJSON(w io.Writer, v any) error {
    enc := json.NewEncoder(w)
    enc.SetEscapeHTML(false)
    enc.SetIndent("", "  ")
    return enc.Encode(v)
}
```

`SetEscapeHTML(false)` so `&` in product names doesn't become `\u0026` in `--json` output (matters for piping to `jq`).

## Phase 6 — Adversarial review

### When to run

After Phase 3-5 are GREEN (`go build && go test && go vet` clean), on the consolidated state. NOT in parallel with implementers.

### What the reviewer reads

1. `docs/<store>-api-discovery.md` — the spec.
2. Every `internal/api/*.go` — does the code do what the doc claims?
3. Every `internal/api/fixtures_test.go` entry — does the test assert the load-bearing fields the doc captured?
4. `internal/config/config.go` — env-sourced stripping actually works? Files 0600?
5. `internal/client/client.go` — auth headers attached on the right hosts? Refresh margins correct? Errors actionable?
6. `internal/cli/runtime.go` — overlay precedence correct (flag > env > config)?

### Classification

- **H** (must fix before ship): correctness or security bugs. Token leaks, spending guard wrong, `whoami` lies, silent data loss.
- **M** (should fix): robustness bugs, fidelity drops, missing regression tests for known-bad inputs.
- **L** (doc-only): confusing code, missing godoc, comment-only fixes.

Each fix → regression test in `regressions_test.go`, commented with the bug ID + one-line description.

## Phase 7 — Polish

### README structure (mandatory sections)

1. **Title + one-paragraph description.** What it is, what it talks to, single binary + `--json`.
2. **Disclaimer block.** Unofficial, trademark-safe, use responsibly.
3. **Status.** What's shipped, what's planned. Be honest.
4. **Success criteria.** A checklist of end-to-end working commands. Tick as they ship.
5. **Quickstart.** Copy-pasteable commands grouped by surface (Session / Catalog / Cart / Delivery / Auth).
6. **What's distinctive about `<Store>`** (if applicable). Store-specific quirks worth highlighting.
7. **Layout.** The `internal/` tree with one-line responsibilities.
8. **Disclaimer** (footer repeat).

### goreleaser

linux/darwin × amd64/arm64. Skip windows/arm64 if you don't test it. `CGO_ENABLED=0`. ldflags: `-s -w -X main.version={{.Version}}`. Archives tar.gz (zip for windows).

### Tag

`git tag v0.1.0` once Phase 6 has zero open H issues and README is accurate.
