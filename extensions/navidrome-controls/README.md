<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | Navidrome Controls |
| Source repository | https://github.com/prolix-oc/Lumiverse-SubsonicControls |
| Original link (from Lumiverse-Extensions README) | https://github.com/prolix-oc/Lumiverse-SubsonicControls.git |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`c3c4a27`) |

---

# Lumiverse Subsonic Controls

A Lumiverse extension for Subsonic and OpenSubsonic-compatible servers such as Navidrome, Airsonic, Gonic, and other implementations that support the common REST API. It can also control a running Feishin desktop player through Feishin's Remote WebSocket server.

## What this first version supports

- Secure per-user connection settings: server details and preferences in user storage, with Subsonic and Feishin passwords in the Lumiverse secure enclave.
- Subsonic `ping`, `search3`, `getNowPlaying`, cover-art, and lyrics endpoints.
- OpenSubsonic `getLyricsBySongId` with a fallback to classic `getLyrics`.
- Optional server-side Jukebox play, pause, previous, next, play-from-search, and queue actions.
- Feishin desktop Remote playback state, artwork, and play/pause/previous/next controls.
- The recommended Subsonic token authentication scheme (`t=md5(password + salt)`) with a fresh salt per request.

## Important protocol behavior

The Subsonic protocol does not provide a universal remote-control API for playback happening in another client. Its portable now-playing endpoint reports active users, while playback controls are available through the optional server-side Jukebox endpoint. Choose **Now playing only**, **Server-side Jukebox**, or **Feishin Desktop Remote** independently from the configured Subsonic server.

Album artwork is fetched through Lumiverse's backend image proxy and delivered to the UI as an inline image. The browser does not need direct access to the music server, so an HTTPS Lumiverse page can display artwork from a private HTTP Navidrome server. Lumiverse's backend must be able to reach the configured server. Missing artwork does not prevent track metadata from loading.

## Feishin desktop Remote

Keep the Subsonic server configured, select **Feishin Desktop Remote** under **Playback controls**, then enter the URL shown by Feishin's **Settings → Window → Remote** section (normally `http://<computer-lan-ip>:4333`). Feishin must be running on the same reachable network; its browser build does not expose the Remote server.

This connection is a direct WebSocket control channel to the desktop app. It supplies live now-playing artwork and basic transport controls, while library search and lyrics continue through the configured Subsonic server. Feishin's HTTP server serves the Remote page and a credentials endpoint only; it does not offer HTTP endpoints for playback commands or state. If an HTTPS-hosted Lumiverse instance cannot connect to Feishin's insecure `ws://` endpoint, use a TLS-terminating WebSocket reverse proxy and configure its `https://` URL instead. Feishin's Remote API cannot play an arbitrary Subsonic search result, so those search-result transport buttons are hidden while this remote-control option is active. Remote credentials are stored per user in the Lumiverse secure enclave and restored with the connection.

## Development

```sh
bun run build
```

The extension expects a server URL such as `https://music.example.com` or `https://music.example.com/rest`; it will add `/rest` when necessary.
