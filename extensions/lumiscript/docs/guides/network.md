# Network egress & local hosts

By default, LumiScript scripts **cannot reach private network addresses** — localhost, your LAN, link-local, cloud-metadata endpoints. This page explains why, and how to allow the specific local host you actually want (a local model server, a home-lab device).

> **New in v2.0.** Private-host egress is blocked by default as of LumiScript 2.0. Upgrading from 1.x? If a script talked to a local model server (Ollama, ComfyUI, LM Studio) or a LAN device, it will now be rejected until you allowlist that host — jump to [The fix](#the-fix-allow-specific-private-hosts).

## The default: SSRF-safe egress

Every outbound HTTP request a script makes — through `api.utils.http.*` or a bare `fetch` (the `allowDangerous` feature) — is routed through the host's **SSRF-safe proxy**. That proxy resolves and pins DNS, then blocks any request whose target resolves to a private address:

- loopback (`127.0.0.1`, `::1`, `localhost`)
- LAN ranges (`10.*`, `172.16–31.*`, `192.168.*`)
- link-local (`169.254.*`, including the `169.254.169.254` cloud-metadata endpoint)

This is deliberate hardening: it stops a script — yours, or one bundled in a shared character card — from being tricked into probing your internal network or reading a cloud instance's credentials. Public hosts are unaffected; they go straight through.

## The wall: your local model server is blocked

The moment you point a script at a local endpoint — Ollama on `localhost:11434`, ComfyUI on `localhost:8188`, LM Studio, a NAS — the default block rejects it. That's the SSRF guard doing its job; it can't tell your intentional local call from a malicious one.

## The fix: allow specific private hosts

**Settings → Network → Allowed private hosts.** Add the exact host (and optionally port) you want scripts to reach directly, bypassing the block:

```
localhost:11434
127.0.0.1:8188
192.168.1.50
[::1]:3000
```

Now scripts may reach those — and only those — private addresses.

### The rules (and why)

- **Fixed addresses only.** An entry must be an **IP literal** or **`localhost`**, optionally with a port. A plain hostname (`my-nas.local`, `ollama.internal`) is rejected — a name resolves at connect time with no pinning, which would re-open the DNS-rebinding hole the guard exists to close.
- **Literal host match — no rebinding bypass.** The allowlist is matched against the URL's host *literally*. A public hostname that merely *resolves* to an allowlisted IP does **not** match; it stays on the hardened path. So allowlisting `127.0.0.1` can't be abused by an `evil.com → 127.0.0.1` DNS trick.
- **Port scoping.** An entry with a port (`localhost:11434`) matches only that port. An entry without one (`192.168.1.50`) matches any port on that host.
- **http/https only.** Even an allowlisted host is reachable only over http/https — a `file://127.0.0.1/…` URL is refused, so the allowlist can't become an arbitrary local-file read.
- **User-owned.** The list lives in your settings and is **never writable by a script**. A script can't add its own escape hatch; only you can.

It pairs with the per-script **`allowDangerous`** toggle: `allowDangerous` grants a script the HTTP capability at all; the allowlist decides which *private* hosts that capability may reach. A script needs both to hit your local server.

## Worked example: calling a local Ollama

1. Enable **`allowDangerous`** on the script (it needs outbound HTTP).
2. Add `localhost:11434` to **Settings → Network → Allowed private hosts**.
3. In the script:

```js
const res = await api.utils.http.post(
  'http://localhost:11434/api/generate',
  JSON.stringify({ model: 'llama3', prompt: 'Say hello.', stream: false }),
  { headers: { 'content-type': 'application/json' } },
);
const { response } = JSON.parse(res.body);
console.log(response);
```

Without step 2 the request is blocked by the SSRF guard; with it, it goes straight to your local server. (`api.utils.http.post` takes a **string** body — `JSON.stringify` your payload — and returns `{ status, statusText, headers, body }` with `body` a string you parse yourself.)

## See also

- [Permissions](../concepts/permissions.md) — `allowDangerous` and the `cors_proxy` permission that gate HTTP.
- [Calling the LLM](llm.md) — for hosted models, prefer connection profiles over raw HTTP.
- [API stability → Engine divergence](../api-stability.md#engine-divergence-quickjs-isolate-opt-in) — how bare `fetch` behaves across engines, including this allowlist's direct path (`AbortSignal` and per-cookie `Set-Cookie` work there).
