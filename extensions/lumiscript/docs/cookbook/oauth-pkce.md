# Call an OAuth-protected API (PKCE)

Let a user connect a third-party account — GitHub, Google, a self-hosted service — and call its API on their behalf, using the PKCE flow so there's no client secret to leak. This is the full round-trip: kick off authorization, catch the redirect, exchange the code for a token, store it encrypted, and use it.

## What you'll use

- [`api.oauth.*`](../guides/oauth.md) — `createState()` (CSRF nonce), `getCallbackUrl()` (your redirect path), `onCallback(handler)` (catch the redirect). Requires `oauth`.
- `api.utils.http.post` / `.get` — token exchange + the authenticated call. **Positional**: `post(url, body, options?)`. Requires `cors_proxy` + `allowDangerous`.
- `api.enclave.*` — encrypted-at-rest token storage (`put` / `get` / `delete`). Requires `allowDangerous`. ([Storage model](../concepts/storage-model.md))
- `crypto.subtle` — to compute the PKCE challenge. Available in the sandbox.
- `api.commands.*` + `api.chat.sendMessage` — a slash command to start the flow and post the link (scripts can't open a browser themselves). Needs `chat_mutation`.

The idea up front: **PKCE replaces the client secret with a one-time proof.** You generate a random `code_verifier`, send its SHA-256 hash (`code_challenge`) in the authorize URL, then send the original verifier back during token exchange. The provider checks they match — so possession of the verifier, not a baked-in secret, authenticates your client. Nothing secret lives in your script source.

## The script

```js
// @triggers ls:startup
// Connect a PKCE OAuth provider and call its API. Registers on enable.
// Permissions: oauth · cors_proxy + allowDangerous (http) · allowDangerous (enclave) · chat_mutation.

const CLIENT_ID          = 'YOUR_CLIENT_ID';
const AUTHORIZE_ENDPOINT = 'https://provider.example.com/oauth/authorize';
const TOKEN_ENDPOINT     = 'https://provider.example.com/oauth/token';
const API_ENDPOINT       = 'https://provider.example.com/api/me';
const SCOPE              = 'read';
const LUMIVERSE_ORIGIN   = 'https://your-host.example.com';   // your Lumiverse origin

// base64url(SHA-256(input)) — the PKCE challenge transform. crypto.subtle, btoa,
// and TextEncoder are all available in the sandbox; this is the only crypto needed.
async function base64UrlSha256(input) {
  const digest = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(input));
  return btoa(String.fromCharCode(...new Uint8Array(digest)))
    .replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');
}

// ── Start the flow: build the authorize URL, stash the verifier ───────────
async function startAuth() {
  const verifier  = crypto.randomUUID() + crypto.randomUUID();   // 64+ chars of entropy
  const challenge = await base64UrlSha256(verifier);
  const state     = await api.oauth.createState();               // host verifies this on return

  // The verifier must survive the user's round-trip to the provider — use enclave,
  // not globalThis (the worker may restart in between). NOTE: enclave keys allow
  // only [A-Za-z0-9_-.] (1–128 chars) — no colons; namespace with dots.
  await api.enclave.put('pkce.verifier', verifier);

  return `${AUTHORIZE_ENDPOINT}?` + new URLSearchParams({
    response_type:         'code',
    client_id:             CLIENT_ID,
    redirect_uri:          LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl()),
    scope:                 SCOPE,
    state,
    code_challenge:        challenge,
    code_challenge_method: 'S256',
  }).toString();
}

// ── Catch the redirect: exchange code + verifier for a token ──────────────
// Registered at top of body so it persists between fires. The host has ALREADY
// verified `state` before this runs, so `params` can be trusted.
api.oauth.onCallback(async (params) => {
  if (params.error) return { html: `<h1>Authorization failed</h1><p>${params.error}</p>` };

  const verifier = await api.enclave.get('pkce.verifier');
  const body = new URLSearchParams({
    grant_type:    'authorization_code',
    code:          params.code,
    redirect_uri:  LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl()),  // must byte-match
    client_id:     CLIENT_ID,
    code_verifier: verifier,                                               // the PKCE proof — no secret
  }).toString();

  const resp = await api.utils.http.post(TOKEN_ENDPOINT, body, {
    headers: { 'Content-Type': 'application/x-www-form-urlencoded', Accept: 'application/json' },
  });
  const tokens = JSON.parse(resp.body);

  await api.enclave.put('access_token', tokens.access_token);
  if (tokens.expires_in) {
    await api.enclave.put('expires_at', String(Date.now() + tokens.expires_in * 1000));
  }
  await api.enclave.delete('pkce.verifier');                 // one-time use — clean up

  api.broadcast.emit('myapp:connected', {});                 // tell the rest of your extension
  return { html: `<h1>Connected!</h1><p>You can close this tab.</p>` };
});

// ── Call the API with the stored token ────────────────────────────────────
async function callApi() {
  const token = await api.enclave.get('access_token');
  if (!token) throw new Error('not connected — run the "Connect account" command first');
  const resp = await api.utils.http.get(API_ENDPOINT, {
    headers: { Authorization: `Bearer ${token}` },
  });
  return JSON.parse(resp.body);
}

// ── Slash commands: one to start the flow, one to test the call ───────────
api.commands.register([
  { id: 'oauth-connect', label: 'Connect account', description: 'Start the OAuth (PKCE) flow', scope: 'chat' },
  { id: 'oauth-whoami',  label: 'Who am I',        description: 'Call the connected API',       scope: 'chat' },
]);

api.commands.onInvoked(async (commandId) => {
  // onInvoked fires for EVERY command the extension registers — filter to ours.
  if (commandId === 'oauth-connect') {
    const url = await startAuth();
    await api.chat.sendMessage(`[Click to authorize](${url})`, { role: 'assistant' });
  } else if (commandId === 'oauth-whoami') {
    const me = await callApi();
    await api.chat.sendMessage(`Connected as: ${me.login ?? me.name ?? JSON.stringify(me)}`, { role: 'assistant' });
  }
});
```

Enable the script, run **/Connect account** in a chat, click the posted link, authorize on the provider's page, and you bounce back to a "Connected!" tab. Then **/Who am I** calls the API with the stored token and posts the result.

## How it works

**PKCE in three lines.** `base64UrlSha256(verifier)` produces the `code_challenge`; the authorize URL carries the *challenge* plus `code_challenge_method: 'S256'`; the token exchange carries the original *verifier*. The provider hashes your verifier and checks it equals the challenge it stored — proving the same client that started the flow is finishing it, with no shared secret. `crypto.subtle.digest('SHA-256', …)` and `crypto.randomUUID()` are both available in the sandbox.

**`createState()` + host-verified CSRF.** `api.oauth.createState()` mints a nonce the host remembers; you put it in the authorize URL as `state`. When the provider redirects back, the host checks the returned `state` against its store and *rejects mismatches before your handler runs* — so inside `onCallback` you can trust `params` without verifying state yourself.

**The verifier crosses a process boundary — store it.** Between `startAuth()` and the callback the user leaves to the provider, and the worker subprocess may restart. `globalThis` wouldn't survive that; `api.enclave` does (and it's the right home for flow secrets anyway). **Enclave keys are restricted to `[A-Za-z0-9_\-.]` (1–128 chars) — no colons** — so namespace with dots (`pkce.verifier`, not `pkce:verifier`). Values must be printable ASCII ≤ 64 KB; JWT-style tokens fit easily.

**The handler persists; the command fans out.** `onCallback` registers once and survives between trigger fires (a host-side slot, one per extension). `onInvoked`, by contrast, fires for *every* command your extension registered — so filter on `commandId`. Both sit at the top of the body, re-asserting whenever `ls:startup` runs.

**`http.post` is positional.** `post(url, body, options?)` — the form-encoded body is the second argument, headers go in `options.headers`. A frequent slip is nesting the body inside `options`.

## Make it yours

- **Refresh before expiry.** If the provider issues a `refresh_token`, store it and wrap reads in a refresh-on-expiry check (`expires_at` minus a safety margin → re-exchange with `grant_type: 'refresh_token'`). The [OAuth guide](../guides/oauth.md#pairing-with-apienclave) has the full `getValidAccessToken` pattern, including rotating refresh tokens.
- **Keep the handler alive across restarts.** A worker restart between authorize and redirect drops the callback handler. Mitigate by pinning the worker (subscribe to any non-`ls:*` broadcast) or re-registering `onCallback` on a periodic trigger — see [Handler lifecycle](../guides/oauth.md#handler-lifecycle).
- **Multiple providers, one handler.** Only one `onCallback` per extension. Stash a "which flow" marker in `api.scriptStorage` before opening the authorize URL and branch on it in the handler — see [the guide](../guides/oauth.md#multi-provider-extensions).
- **A confidential client instead.** For a private, never-shared personal script you *can* use a classic `client_secret` (no challenge needed). Don't ship that — the secret is readable in your script source. PKCE is the publishable path.

## Gotchas

- **Enclave keys exclude colons.** The documented key charset is `[A-Za-z0-9_\-.]{1,128}` — so namespace with dots (`svc.token`) or underscores (`svc_token`), not colons (`svc:token`).
- **`redirect_uri` must byte-match.** Most providers reject the token exchange if its `redirect_uri` differs even slightly from the authorize URL's. Build it once (`LUMIVERSE_ORIGIN + (await api.oauth.getCallbackUrl())`) and reuse the exact string in both places.
- **Return HTML, not a redirect.** The `onCallback` handler renders a page in the user's browser — return `{ html }` telling them to close the tab. Reopening Lumiverse is a separate concern (a broadcast your UI reacts to). Keep the handler fast; providers time the redirect out in seconds.
- **There's no server-side "my origin".** Scripts run on the backend, so `window.location.origin` isn't available — hard-code `LUMIVERSE_ORIGIN` (fine for personal scripts) or capture it once from a frontend interaction into `api.scriptStorage`.
- **Always send `state`.** Skipping `createState()` opens a CSRF / login-hijack hole. It costs one `await`; never omit it.

## See also

- [OAuth callbacks](../guides/oauth.md) — the full `api.oauth` surface, refresh patterns, handler lifecycle, and multi-provider dispatch.
- [Permissions](../concepts/permissions.md) — `oauth`, `cors_proxy` + `allowDangerous` (http), `allowDangerous` (enclave), `chat_mutation` (the posted messages).
- [Storage model](../concepts/storage-model.md) — why secrets belong in `api.enclave`, not `variables` / `scriptStorage`.
