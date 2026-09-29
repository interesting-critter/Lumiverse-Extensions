<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Datacat Browser |
| Source repository | https://github.com/datacat-run/datacat-lumiverse |
| Original link (from Lumiverse-Extensions README) | https://github.com/datacat-run/datacat-lumiverse |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`c70e1f4`) |

---

# Datacat Browser for Lumiverse

Bring the [Datacat](https://datacat.run) character catalogue into Lumiverse.
Browse a large, constantly refreshed selection of characters, find exactly what
you want, import it, and start chatting without leaving your app.

## For Lumiverse users

### What you can do

- Discover new characters in Fresh, or search and filter the full catalogue.
- Explore tags and creator profiles to find more of what you like.
- Import a character directly into Lumiverse, then open a chat immediately.
- Keep track of your most recent imports in one compact list.
- Leave kudos and comments through your connected Datacat account.

### Install

1. Open Lumiverse's extension manager.
2. Choose the option to install an extension from a Git repository.
3. Paste `https://github.com/datacat-run/datacat-lumiverse` and install it.
4. Approve the `characters` and `cors_proxy` permissions when Lumiverse asks.
5. Select the yellow cat button beside Settings to open Datacat.

When Lumiverse asks you to connect an account or complete a quick verification,
follow the Datacat page and return to Lumiverse when it finishes. You can then
continue browsing and importing normally.

## For app and extension developers

This project is a working, auditable proof of concept for using the
[Datacat Client API](https://datacat.run) from a chat app or extension. It
demonstrates an end-to-end flow: public discovery, character transfer, account
connection, verification, and community actions.

The editable implementation is in `src/`; `dist/` is the generated package
that Lumiverse loads. Build it with `bun run build` and compare the output when
auditing a release.

### API quick start

The Client API base URL is:

```text
https://datacat.run/api/client/v1
```

Apply for a Client ID at
[Datacat → Creator Management → API Access](https://datacat.run/creatortools/api-access):

1. Sign in to Datacat and submit your app or extension, platform, homepage,
   intended user flow, and requested permissions.
2. After review, copy the approved Client ID from the application page.
3. Send it with every Client API request:

   ```bash
   curl "https://datacat.run/api/client/v1/capabilities" \
     --header "X-Datacat-Client-Id: <approved-client-id>"
   ```

The Client ID is also called an access code. It identifies your approved app
for permissions, attribution, and revocation; it is not a private API secret.

### Endpoints used by this extension

Paths below are relative to `https://datacat.run/api/client/v1`.

| Use case | Endpoints |
| --- | --- |
| Check available features | `GET /capabilities` |
| Browse and search characters | `GET /fresh`, `GET /characters`, `GET /characters/:characterId`, `GET /tags` |
| Browse a creator | `GET /creators/:creatorRef/bootstrap`, `GET /creators/:creatorRef/characters` |
| Import character content and image | `GET /characters/:characterId/card`, `GET /characters/:characterId/avatar` |
| Connect or switch a user | `GET /account/status`, `POST /account-links`, `POST /account-links/:linkId/token`, `DELETE /account` |
| Complete a protected import | `POST /verifications`, `GET /verifications/:verificationId`, `POST /verifications/:verificationId/token` |
| Read or contribute to community | `GET /characters/:characterId/community`, `POST /characters/:characterId/community/kudos`, `POST /characters/:characterId/community/comments` |

### Recommended integration flow

1. Call `GET /capabilities`, then build discovery with `/fresh`, `/characters`,
   `/tags`, and the creator endpoints.
2. Let the user choose `Anonymous account (rate limits apply)` or `Log in to
   Datacat` before opening the account-link URL. For an explicit anonymous link,
   append `anonymous=1`; for login, append `login=1` (and `switch=1` when
   changing accounts). Poll the matching token endpoint until it completes.
3. Before a protected card or avatar transfer, start the hosted verification
   flow when the API asks for it, then exchange the completed verification for
   a lease.
4. Fetch `/card` and `/avatar`, import the result into your host app, and keep
   account tokens, installation IDs, device codes, and verification leases in
   protected application storage rather than frontend page code.

Use the source here as a reference implementation, not as a requirement to
copy the Lumiverse UI. Other apps can use the same API to build their own
catalogue browser, import workflow, or Datacat-connected experience.

## Development

```bash
bun run build
```
