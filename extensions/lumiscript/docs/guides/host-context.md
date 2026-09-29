# Host & user context

Two small free-tier namespaces let a script ask about the environment it's running in: `api.users` (is the user looking? what's their role?) and `api.version` (which Lumiverse build is this?). Both are read-only and scoped to the active user automatically — you never pass a user id.

## `api.users` — is the user present, and who are they?

```js
const visible = await api.users.isVisible();   // boolean
const role    = await api.users.getRole();      // 'operator' | 'admin' | 'user'
```

- `isVisible()` — `true` if the user has the app visible in at least one session; `false` if every session is hidden / backgrounded, or there's no open session.
- `getRole()` — the user's Lumiverse role. Internal owners report as `'operator'`; admins as `'admin'`; everyone else as `'user'`.

### Choose push vs. in-app

The classic use of `isVisible()` is deciding *how* to reach the user — an OS push when they're away, a quiet in-app toast when they're looking:

```js
const msg = 'Your long generation finished.';
if (await api.users.isVisible()) {
  api.ui.toast(msg, 'success');                       // free tier
} else {
  await api.ui.pushNotification('Done', msg);          // needs push_notification
}
```

### Gate privileged actions on role

```js
if ((await api.users.getRole()) === 'operator') {
  // expose an operator-only diagnostic surface
}
```

## `api.version` — which Lumiverse build is this?

```js
const backend  = await api.version.getBackend();    // e.g. '1.0.2'
const frontend = await api.version.getFrontend();   // e.g. '1.0.2'
```

Both return a semver string. The use case is **feature-gating** — branch on whether the host is new enough for an event or API your script depends on.

### Compare numerically, not as strings

`'1.10.0' > '1.9.0'` is `false` as a string comparison — compare the numeric tuple instead. A minimal helper:

```js
function atLeast(version, min) {
  const norm = (v) => v.split('-')[0].split('+')[0].split('.').map(Number);   // drop -rc/+build, then split
  const a = norm(version), b = norm(min);
  for (let i = 0; i < Math.max(a.length, b.length); i++) {
    const x = a[i] ?? 0, y = b[i] ?? 0;
    if (x !== y) return x > y;
  }
  return true;   // equal
}

if (atLeast(await api.version.getBackend(), '1.0.0')) {
  // safe to use a feature introduced in 1.0.0
}
```

(LumiScript itself uses the same numeric-compare approach internally for its `minimum_lumiverse_version` check — there's no host comparison API to call, so a small helper like the one above is the pattern.)

## Notes

- **Both free tier.** No permission, no user id.
- These are live reads — `isVisible()` in particular reflects the user's session state *right now*, so call it at the moment you need to decide, not at script start.

See the in-app **Reference tab** for `api.users` and `api.version` alongside the rest of the surface.
