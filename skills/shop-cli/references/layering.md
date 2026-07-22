# Layering

The 4-package layering is the architectural invariant. Every shipped CLI follows it.

```
cmd/<name>/main.go    ← 20 lines: cli.NewRoot + Execute + stderr exit 1
       │
       ▼
internal/cli          ← cobra commands, runtime memo, rendering
       │  calls
       ▼
internal/api          ← typed operations: (ctx, *client.Client, ...) → (T, error)
       │  calls
       ▼
internal/client       ← HTTP transport: headers, cookies, refresh, TLS fingerprint
       │  reads/writes
       ▼
internal/config       ← ~/.<name>/{session,config}.json (0600 atomic); env overlay

internal/auth         ← protocol only: JWT decode, OAuth/PKCE, magic-link.
                         Knows nothing about transport, disc, or UX.
```

## Direction rules (invariants)

- `cli` **never** touches the network directly. Always via `api` or `client`.
- `api` **never** touches the disc. It only calls `*client.Client` and returns typed results.
- `client` **never** knows endpoint shapes, only hosts + headers + auth mechanism.
- `config` **never** does network. Pure file I/O + env.
- `auth` is protocol-pure: no UX, no storage, no cobra. The `cli` layer composes it.

## Why

- **Testability.** `api` can be tested with `httptest.NewServer` inline OR with fixture files — no real network, no real disc. `config` can be tested by pointing it at a tmpdir.
- **Surface isolation.** When the platform's API drifts, only `api` + fixtures change. When the auth flow changes, only `auth` + `cli/auth.go` change. When the storage shape changes, only `config` changes.
- **Multi-agent safety.** With this layering, two agents can work on different surfaces (cart vs catalog) in parallel without touching shared files except the `AddCommand(...)` block in `root.go` (additive only).

## Runtime memo (the `*rtHolder` pattern)

Every cobra `RunE` starts with `rt := ctxValue(ctx)`. The first call constructs the runtime once:

1. `config.LoadSession()` (empty on missing — never an error).
2. `client.New(sess, logger)`.
3. `config.LoadConfigFrom(flags.Config)`.
4. Resolve every overlay (lang, max, store, etc.) in the documented precedence (flag > env > config).
5. **Overlay env-sourced account token** if set, marking `envSourced=true` on the session.

Subsequent calls in the same invocation return the memoized runtime. The `*rtHolder` lives in `context.Context`, set by `PersistentPreRunE`.

## PersistentPostRunE — the only place session persists

```go
PersistentPostRunE: func(cmd *cobra.Command, _ []string) error {
    rt := ctxValue(cmd.Context())
    if rt.client != nil && rt.client.SyncSession() {  // dirty?
        return config.SaveSession(rt.sessionPath(), rt.sess)
    }
    return nil
}
```

`SyncSession()` returns `true` only if the session mutated during this invocation (segment cookie refreshed, account cookie captured, etc.). This avoids needless disc writes and TOCTOU windows.

## Why `api` is stateless and per-file-per-resource

- One file per resource (`catalog.go`, `cart.go`, `region.go`, `ean.go`, `brand.go`, …). Easy to find, easy to diff, easy to assign to a subagent.
- Functions take `(ctx, *client.Client, ...)` — no shared state, no globals, no init. Pure data-in/data-out.
- Response types live next to their operations. Custom `UnmarshalJSON` for dynamic-key platforms (VTEX dynamic attribute columns) lives on the type, not in a parser helper.
