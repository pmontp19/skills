# Anti-patterns

What NOT to do. Each is a real bug from a reference project; the fix is in the corresponding `regressions_test.go`.

## Token handling

### ❌ Env-sourced token leaks to disc
A `Session` loaded with `Account.Token` from `SIRENA_ACCESS_TOKEN`, when persisted by `SaveSession`, writes the env secret to `session.json`. Anyone with read access to the user's home directory now has the agent's CI token.

**Fix:** unexported `envSourced bool` field (invisible to JSON) set by the env-overlay path; `SaveSession` checks it and writes a snapshot with `Account = nil` if true. sirena-cli: `internal/config/config.go`.

### ❌ Files created 0644 due to relaxed umask
`os.WriteFile(path, b, 0600)` honors the process umask. With umask 0022, the file lands 0644 — world-readable secrets.

**Fix:** `writeSecret` pattern: write to a temp file, `Chmod(0600)` explicitly, atomic rename. All three reference projects.

### ❌ Token expiry stored as `expires_in` relative seconds
Stored at login, read later — the relative number is meaningless without the original timestamp.

**Fix:** store `ExpiresAt` as RFC3339 absolute. Always. Auth libraries return relative; convert before persisting.

## Cart / spending guard

### ❌ Spending guard covers only items, ignores shipping
`OrderForm.TotalCents()` returns only the items totalizer. Once `region set <cp>` binds a shipping totalizer, the guard approves adds that push the all-in charge over the cap.

**Fix:** `TotalCents()` returns the orderForm `Value` (all-in = items+shipping+tax) when > 0, falling back to items-only. New `ItemsCents()` for display. Document the behavior change in godoc. sirena-cli H4.

### ❌ `cart add <sku>` looks up the parent product's default SKU price
`lookupUnitPrice` did `GetProduct(itemID)` (queries by productId) and read `items[0].Price()` (default SKU). For a multi-SKU product, `cart add 201 1` either missed or read the wrong SKU's price into the spending guard.

**Fix:** `GetSKU(ctx,c,skuID)` filters catalog search on the platform's SKU filter (VTEX: lowercase `fq=skuId:`); `Product.SKUByID(id)` walks `items[]` for the exact match. Lookup falls back to default SKU only on a catalog race, logged. sirena-cli H3.

### ❌ `cart clear` snapshot race
`clear` snapshotted `of.Items`, built `{0..N:0}`, POSTed. A concurrent invocation shifting items[] between snapshot and POST would zero the wrong rows.

**Fix:** `clear` re-fetches the orderForm immediately before building updates (refuses to clear if re-fetch fails — safer than zeroing stale indices). sirena-cli H10. True atomicity needs server-side support; document the limitation.

## JSON fidelity

### ❌ Dynamic attribute keys collapse non-string scalars to text
A JSON value of `42` (number) and `"42"` (string) both ended up as the string `"42"` in `Attributes`, losing the distinction.

**Fix:** reversible prefixes for non-string scalars: `number:42`, `bool:true`, `null:`. Objects and non-string arrays fall back to raw JSON text (documented). sirena-cli M11.

### ❌ `json:"-"` on a field the JSON output needs
Marking `Attributes map[string][]string` with `json:"-"` makes it invisible in `--json` output, silently dropping data the discovery doc promised.

**Fix:** keep the field exported in JSON. Round-trip regression test that asserts the field is populated after `Marshal → Unmarshal`. sirena-cli C1.

### ❌ Format-only barcode validation
EAN lookup validates only the format (length + digits). A transposed digit passes format, hits the network, returns "not found" for a barcode that's structurally invalid.

**Fix:** validate the GS1 check digit before any network call. Length-parity-aware weight pattern so EAN-8/13/14/UPC-A all validate. sirena-cli L2.

## Discovery / process

### ❌ HAR or cookie export committed to the repo
HAR files contain live session tokens, `Set-Cookie` headers, and POST bodies (email, OTP, payment details). Cookie exports from `agent-browser cookies get --json` are 100% credentials. A committed HAR is a leaked session anyone with read access can replay.

**Fix:** keep HARs and cookie exports under `/tmp/` (or outside the repo). Delete when discovery is done. `templates/.gitignore.tmpl` blocks `*.har`, `cookies.json`, and `/captures/` as a safety net — never remove those lines. (Borrowed rule from `agent-browser`'s `derive-client` skill.)

### ❌ Phase 0 skipped — "we know it's Cloudflare-walled"
Decathlon's website LOOKS Turnstile-walled to a naive `curl`. A TLS-impersonating client (`curl_cffi(impersonate="chrome")`) passes clean. The "blocker" was an environment artifact.

**Fix:** Phase 0 mandates an executable repro of the assumed blocker BEFORE treating it as real.

### ❌ Phase 6 adversarial review run in parallel with implementers
The reviewer inspects work-in-progress and reports finished work as "not fixed". False negatives everywhere.

**Fix:** reviewer runs AFTER implementers, on the consolidated state. sirena-cli AGENTS.md rule #6.

### ❌ Adversarial reviewer flags a bug, fix lands, no regression test
The same bug returns three weeks later when a refactor reverts the fix.

**Fix:** one regression test per bug, commented with the bug ID. The test is the long-term memory of why the code is shaped the way it is.

## Multi-agent coordination

### ❌ Big-bang rewrite of `root.go` to add a command
Two agents both rewrite `root.go` to add their command. Merge conflict, or one silently overwrites the other.

**Fix:** `root.go` is a shared file. Only the `AddCommand(...)` block is editable, and only by adding lines. Never rewrite it. sirena-cli AGENTS.md rule #3.

### ❌ Refactoring a file another agent might own
Agent A "cleans up" `cart.go` while agent B is adding the spending guard to the same file. Conflict.

**Fix:** add, don't refactor. If you must refactor a shared file, coordinate in `AGENTS.md` first. sirena-cli AGENTS.md rule #2.

### ❌ Silent schema drift in `Parse*` helpers
`ParseNutrition` / `ParseAllergens` / etc. silently return `nil` when their JSON decode fails. The CLI keeps working but the field is missing — the user has no idea the platform changed its schema.

**Fix:** package-level `parseDebugLogger` hook, default nil (preserves silent-skip). Wire it from `runtime.go` when `--verbose` is set. Truncation (256 bytes) built into the helper so a multi-KB payload doesn't flood a log line. sirena-cli L6.
