# OAuth callbacks

`api.oauth.*` is the only inbound-HTTP hook Spindle exposes to LumiScript extensions. It's there for OAuth-style authorization flows specifically — the case where a user grants your script access to a third-party service (GitHub, Discord, a self-hosted API, etc.), and the provider redirects the user's browser back to a URL your script needs to intercept.

The surface is three methods: `getCallbackUrl()` returns the URL to register with the provider, `createState()` mints a CSRF nonce, and `onCallback(handler)` registers the handler that runs when the redirect arrives. Everything *else* — authorize-URL construction, token exchange, persistence, refresh — is script-owned. The host gives you the hook and verifies state; your script orchestrates the flow.

Requires the `oauth` permission. All three methods throw `Error: PERMISSION_DENIED:oauth — grant this permission to use this API` when the permission isn't granted.

## The big idea

OAuth 2.0 authorization code flow has eight phases. `api.oauth.*` is involved in two of them; everything else is script-side:

| Phase | What happens | Owner |
|---|---|---|
| 1 — Authorize URL | Build URL with `client_id`, `redirect_uri`, `scope`, `state` | **Script** |
| 2 — User in browser | Direct the user to the authorize URL | **Script** |
| 3 — Grant | User clicks "Authorize" on the provider's page | Provider + user |
| 4 — Redirect | Provider redirects to `<redirect_uri>?code=…&state=…` | Provider |
| 5 — State verification | Host checks `state` matches a recently-minted nonce | **Host** |
| 6 — Handler dispatch | Host invokes your callback handler with the query params | **Host** + script |
| 7 — Token exchange | POST to the provider's token endpoint with the `code` | **Script** (via `api.utils.http`) |
| 8 — Persistence + refresh | Store tokens at rest; refresh before expiry | **Script** (via `api.enclave`) |

The host's responsibilities are narrow: provide the inbound-HTTP hook, verify CSRF state, dispatch to your handler. Token exchange, persistence, and refresh are on you because they're provider-specific — every provider's token endpoint takes slightly different parameters, and storing tokens correctly depends on what scopes you need and for how long.

## Quick start

A GitHub OAuth flow, using `api.utils.http.post` for the token exchange and `api.enclave.put` for persistence:

```js
// @triggers ls:startup, CHAT_SWITCHED
// @description GitHub OAuth example.

const CLIENT_ID     = 'YOUR_CLIENT_ID';
const CLIENT_SECRET = 'YOUR_CLIENT_SECRET';      // see Public clients + PKCE below
const LUMIVERSE_ORIGIN = 'https://your-host.example.com';

// 1. Register the callback handler at top of body — survives between fires.
api.oauth.onCallback(async (params) => {
  if (params.error) {
    return { html: `<h1>Auth failed</h1><p>${params.error}</p>` };
  }

  // 2. Exchange the code for tokens.
  // api.utils.http.post is POSITIONAL: post(url, body, options?). The body
  // is the second arg (not nested in options); headers live in options.
  const body = new URLSearchParams({
    client_id:     CLIENT_ID,
    client_secret: CLIENT_SECRET,
    code:          params.code,
    redirect_uri:  LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl()),
  }).toString();
  const resp = await api.utils.http.post(
    'https://github.com/login/oauth/access_token',
    body,
    {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        Accept:         'application/json',
      },
    },
  );
  const { access_token, refresh_token, expires_in } = JSON.parse(resp.body);

  // 3. Persist encrypted.
  await api.enclave.put('github.access_token',  access_token);
  if (refresh_token) {
    await api.enclave.put('github.refresh_token', refresh_token);
  }
  if (expires_in) {
    await api.enclave.put('github.expires_at', String(Date.now() + expires_in * 1000));
  }

  // 4. Notify the rest of the extension.
  api.broadcast.emit('myapp:github-connected', {});

  // 5. Render the success page in the user's browser.
  return { html: `<h1>GitHub connected!</h1><p>You can close this tab.</p>` };
});

// Helper a UI surface can call to start the flow.
async function buildAuthorizeUrl() {
  const state       = await api.oauth.createState();
  const callbackUrl = await api.oauth.getCallbackUrl();

  return `https://github.com/login/oauth/authorize?` + new URLSearchParams({
    client_id:    CLIENT_ID,
    redirect_uri: LUMIVERSE_ORIGIN + callbackUrl,
    scope:        'repo user',
    state,
  }).toString();
}
```

To actually *start* the flow, your script needs to surface the authorize URL to the user — scripts run server-side and can't open a browser tab directly. Common patterns: register a chat command that posts the URL as a clickable link, inject an "authorize" button into a panel via `api.ui.dom.*`, or surface it through a settings UI.

About `CLIENT_SECRET`: hard-coding it in script source means anyone with read access to your scripts can extract it. For published or shared scripts, use the PKCE flow ([Public clients + PKCE](#public-clients--pkce) below) — there's no `client_secret` to leak.

## Surface

```ts
interface OAuthAPI {
  onCallback(
    handler: (params: Record<string, string>) => Promise<{ html?: string } | void>,
  ): () => void;

  getCallbackUrl(): Promise<string>;
  createState():    Promise<string>;
}
```

All three methods require the `oauth` permission.

### `onCallback(handler)`

Register the callback handler. Returns an unsubscribe function (sync — the return value is the unsub, not a promise).

```ts
onCallback(
  handler: (params: Record<string, string>) => Promise<{ html?: string } | void>,
): () => void
```

- **`handler`** — async function called when the provider redirects to your callback URL. By the time your handler fires, the host has **already verified** the CSRF state — mismatched state values are rejected before your handler ever sees them.
- **`params`** — query-string parameters from the redirect. For a successful code flow: `{ code, state }`. For an error: `{ error, error_description?, state }`. Provider-specific extensions appear as additional fields.
- **Handler return value** — optional. Return `{ html: '...' }` to render a custom page in the user's browser. Return nothing (`void`) to show the host's default "auth complete" page. The handler should return promptly — providers expect the redirect to complete in seconds, not minutes.

```js
const unsub = api.oauth.onCallback(async (params) => {
  // ...
  return { html: '<h1>Done!</h1>' };
});

// Later, to remove the registration:
unsub();
```

**Only one handler per extension.** Last-wins. Cross-script collision logs a warn but doesn't throw — see [Cross-script collision](#cross-script-collision).

**Handler persists between fires.** Like broadcast subscriptions, the registration lives on the host side and survives between trigger runs. Cleared on script disable, delete, or worker subprocess restart — see [Handler lifecycle](#handler-lifecycle).

### `getCallbackUrl()`

Returns the **host-relative** path to your callback endpoint:

```js
const path = await api.oauth.getCallbackUrl();
// → "/api/spindle-oauth/lumiscript/callback"
```

Stable per-extension — all your scripts share one callback URL. Treat the path as opaque; the exact format is host-determined.

To form the **absolute URL** providers need as `redirect_uri`, prefix with the Lumiverse origin:

```js
const callbackUrl  = await api.oauth.getCallbackUrl();
const absoluteUrl  = LUMIVERSE_ORIGIN + callbackUrl;
// → "https://lumiverse.example.com/api/spindle-oauth/lumiscript/callback"
```

There's no universal "get my Lumiverse origin" API on the server side. Common patterns to obtain the origin:

- Hard-code it (acceptable for personal scripts).
- Stash it in `api.scriptStorage` once via a one-time UI command that captures `window.location.origin` from a frontend interaction.
- Read it from `api.utils.http` headers when running a token exchange (some providers echo it back).

### `createState()`

Mint a CSRF state nonce.

```js
const state = await api.oauth.createState();
// → "f4a8c1b2..."
```

The host stores the nonce and **verifies it at callback time**. If the `state` parameter the provider sends back doesn't match a recently-minted nonce, the host rejects the redirect before invoking your handler.

This is essential. Without state verification an attacker could trick a user's browser into hitting your callback URL with a code they obtained for *their own* account, hijacking the user into the attacker's session. Always include `state=<value>` in your authorize URL:

```js
const state = await api.oauth.createState();
const authorizeUrl = `https://provider.example.com/authorize?...&state=${state}`;
```

You don't need to verify state yourself — the host has already done it by the time your handler fires.

## Pairing with `api.utils.http`

Token exchange is a POST to the provider's token endpoint. Requires the `cors_proxy` permission ([`concepts/permissions.md`](../concepts/permissions.md)).

```js
const body = new URLSearchParams({
  grant_type:    'authorization_code',
  code:          params.code,
  redirect_uri:  LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl()),
  client_id:     CLIENT_ID,
  client_secret: CLIENT_SECRET,
}).toString();
const resp = await api.utils.http.post(
  'https://provider.example.com/oauth/token',
  body,
  {
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      Accept:         'application/json',
    },
  },
);
const tokens = JSON.parse(resp.body);
```

Two things to watch:

- **`redirect_uri` must match exactly.** Most providers require the `redirect_uri` in the token exchange to byte-match the one in the authorize URL. Use the same `LUMIVERSE_ORIGIN + callbackUrl` value in both places.
- **Content-Type matters.** Providers vary. GitHub, Discord, Google use `application/x-www-form-urlencoded` for the token endpoint. Some others (Spotify, Twitch) accept JSON. Check your provider's docs.

## Pairing with `api.enclave`

Tokens belong in the enclave. AES-256-GCM encrypted at rest; requires `allowDangerous`. Keys match `[A-Za-z0-9_\-.]{1,128}`; values must be printable ASCII ≤ 64 KB. Standard JWT-style tokens fit easily.

```js
await api.enclave.put('github.access_token',  access_token);
await api.enclave.put('github.refresh_token', refresh_token);
await api.enclave.put('github.expires_at',    String(Date.now() + expires_in * 1000));
```

Reading the token back, with a refresh-on-expiry pattern:

```js
async function getValidAccessToken() {
  const token  = await api.enclave.get('github.access_token');
  const expiry = parseInt(await api.enclave.get('github.expires_at') ?? '0', 10);
  if (token && Date.now() < expiry - 60_000) return token;          // 60s safety margin
  return await refreshAccessToken();
}

async function refreshAccessToken() {
  const refresh_token = await api.enclave.get('github.refresh_token');
  if (!refresh_token) throw new Error('no refresh token — user must re-authorize');

  const body = new URLSearchParams({
    grant_type:    'refresh_token',
    refresh_token,
    client_id:     CLIENT_ID,
    client_secret: CLIENT_SECRET,
  }).toString();
  const resp = await api.utils.http.post(
    'https://provider.example.com/oauth/token',
    body,
    { headers: { 'Content-Type': 'application/x-www-form-urlencoded', Accept: 'application/json' } },
  );
  const next = JSON.parse(resp.body);

  await api.enclave.put('github.access_token', next.access_token);
  await api.enclave.put('github.expires_at',   String(Date.now() + next.expires_in * 1000));
  if (next.refresh_token) {                                          // some providers rotate refresh tokens
    await api.enclave.put('github.refresh_token', next.refresh_token);
  }
  return next.access_token;
}
```

Some providers issue rotating refresh tokens (the refresh response includes a *new* `refresh_token` that supersedes the old one). The conditional `await api.enclave.put('github.refresh_token', ...)` handles both rotating and non-rotating providers.

## Public clients + PKCE

Hard-coding `client_secret` in script source is **safe only for private personal scripts that you'll never share**. For published cross-user scripts, use OAuth 2.0's *public client* + PKCE flow — there's no `client_secret`, so there's nothing to leak.

1. Register the OAuth app as a public client with the provider (no `client_secret` issued).
2. In the authorize URL, include `code_challenge` (SHA-256 of a random verifier) and `code_challenge_method=S256`.
3. In the token exchange, send the original `code_verifier` (not the challenge) — the provider verifies the SHA-256 matches.

```js
// Authorize-URL construction:
const verifier  = crypto.randomUUID() + crypto.randomUUID();        // 64 chars of entropy
const challenge = base64UrlSha256(verifier);                        // SHA-256, then base64url
const state     = await api.oauth.createState();

// Persist the verifier so the callback can recover it:
await api.enclave.put('pending_verifier', verifier);

const authorizeUrl = `https://provider.example.com/authorize?` + new URLSearchParams({
  client_id:             CLIENT_ID,
  redirect_uri:          LUMIVERSE_ORIGIN + callbackUrl,
  scope:                 'user.read',
  state,
  code_challenge:        challenge,
  code_challenge_method: 'S256',
}).toString();

// Token exchange — no client_secret, but with code_verifier:
const verifier = await api.enclave.get('pending_verifier');
const body = new URLSearchParams({
  grant_type:    'authorization_code',
  code:          params.code,
  redirect_uri:  LUMIVERSE_ORIGIN + callbackUrl,
  client_id:     CLIENT_ID,
  code_verifier: verifier,                                           // proves you started this flow
}).toString();
const resp = await api.utils.http.post(
  'https://provider.example.com/oauth/token',
  body,
  { headers: { 'Content-Type': 'application/x-www-form-urlencoded', Accept: 'application/json' } },
);
await api.enclave.delete('pending_verifier');                        // clean up
```

The `base64UrlSha256` helper isn't built in — implement using `crypto.subtle.digest('SHA-256', ...)` (available in the script runtime) and the standard "replace `+` with `-`, `/` with `_`, strip `=` padding" base64url munge.

## Handler lifecycle

The callback handler registration lives on the host side. Lifecycle boundaries:

| Event | Registration |
|---|---|
| Script body finishes a run | **Persists** — handler survives between trigger fires. |
| Script edited and re-saved | **Persists** — same extension-level slot. |
| Script disabled | **Cleared** — handler unregistered via the script-runner's teardown. |
| Script deleted | **Cleared** — same path as disable. |
| Worker subprocess restarts | **Cleared** — registration state lives in the worker; restart wipes it. |
| Another `onCallback` call (any script) | **Replaced** — last-wins; warn logged on cross-script collision. |

The "cleared on worker restart" entry is the one to think about. If the worker subprocess restarts between the user clicking "Authorize" and the provider redirecting back, your handler won't be there to receive the callback and the user sees a confusing default-error page.

Two mitigations:

- **Pin the worker.** Subscribe to any non-`ls:*` broadcast event in your script body (see [`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md) for the pinning policy). A self-subscribed event you never emit works — the policy only cares that there's a non-`ls:*` subscription.
- **Re-register on every fire.** Idempotent — calling `onCallback` twice with the same handler just replaces it with itself. Putting `api.oauth.onCallback(...)` at the top of your body means as long as *some* trigger fires periodically, the slot stays alive.

The second pattern is the standard. Worker restarts are uncommon enough that paired with re-register-on-fire, callbacks just work.

## Cross-script collision

LumiScript supports **one OAuth callback handler per extension**, across all scripts. Calling `onCallback` while another script's handler is registered silently replaces it; a warn lands in the backend log:

```
[script-runner] api.oauth.onCallback: script "<scriptName>" is replacing the
OAuth callback handler previously registered by script "<otherScriptName>". Only one
OAuth callback handler is supported per extension — the prior handler will
no longer fire.
```

(`src/script-runner/host-dispatcher.ts:2888-2904` — non-terminating; the new registration proceeds with last-wins.)

Same-script re-registration gets a separate (less alarming) warn — useful as a heads-up during dev when you forget to remove a top-of-body `onCallback` while reloading the script.

### Multi-provider extensions

If your extension needs to support multiple OAuth providers (GitHub *and* Discord, say), consolidate them into a single handler that dispatches based on which flow is in progress. The cleanest pattern: stash a "current flow" marker in storage before opening the authorize URL, read it in the callback:

```js
async function startGithubAuth() {
  await api.scriptStorage.set('oauth_flow', 'github');
  // ... open authorize URL ...
}

async function startDiscordAuth() {
  await api.scriptStorage.set('oauth_flow', 'discord');
  // ... open authorize URL ...
}

api.oauth.onCallback(async (params) => {
  const flow = await api.scriptStorage.get('oauth_flow');
  await api.scriptStorage.delete('oauth_flow');                      // clean up

  switch (flow) {
    case 'github':  return handleGithubCallback(params);
    case 'discord': return handleDiscordCallback(params);
    default:        return { html: `<h1>Unknown OAuth flow</h1>` };
  }
});
```

Don't try to encode flow identity into the `state` nonce — the host's state verification expects exact-match against what `createState()` returned, and tacking extra segments onto the nonce will fail verification.

## Common pitfalls

- **`redirect_uri` mismatch between authorize and token-exchange.** Most providers reject the token request if the `redirect_uri` doesn't byte-match the one in the original authorize URL. Build it once via `LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl())` and reuse the same string in both places.

- **The handler returns an `html` page, not a redirect.** Don't try to redirect the user back into Lumiverse from inside the handler — return an HTML page that tells them to close the tab. Reopening the Lumiverse UI is a UX detail you can handle separately (e.g., via a broadcast event your UI script reacts to).

- **`client_secret` in script source.** Fine for personal scripts you'll never share. For anything published, switch to PKCE.

- **No CSRF state.** Tempting to skip `createState()` for "internal" or "personal" scripts. Don't — the cost is nothing (`await api.oauth.createState()` once per flow start) and the failure mode is account-takeover. Always pass `state=…`.

- **Forgetting `code_verifier` between authorize and callback (PKCE).** The verifier needs to survive across the user's round-trip to the provider. Use `api.enclave` or `api.scriptStorage` to persist it; don't rely on `globalThis` since the worker may restart between the two events.

- **Long-running token exchange in the handler.** Providers wait at most a few seconds for the redirect response. If your token exchange takes longer (slow provider, retry logic), the user's browser may give up before your handler returns. Keep handler work tight — exchange the code, persist the token, return the HTML response. Defer slow follow-up work to a broadcast subscriber.

- **No refresh on expiry.** Long-lived sessions need a refresh-on-expiry wrapper around the access token (the `getValidAccessToken` pattern above). Without it, your script silently breaks the first time the token expires.

- **Worker eviction during the user's authorize roundtrip.** Mitigate with the pin-or-re-register pattern in [Handler lifecycle](#handler-lifecycle) above.

## See also

- **In-app Reference, "API Functions → api.oauth" section** — auto-generated method list.
- **[`concepts/permissions.md`](../concepts/permissions.md)** — the `oauth` permission plus `cors_proxy` (for `api.utils.http.*`) and `allowDangerous` (for `api.enclave.*`), all needed for a typical OAuth flow.
- **[`concepts/handler-lifetime.md`](../concepts/handler-lifetime.md)** — the eviction-pinning policy referenced above.
- **[Broadcasting events between scripts](broadcast.md)** — the natural way to notify other parts of your extension when a token becomes available.
- **`api.enclave.*` reference** — surface details for the encrypted-at-rest storage tier (keys, value sizes, the `put`/`get`/`delete`/`has`/`list` surface).
