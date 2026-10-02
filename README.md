# True Shuffle

A single static web page that shuffles a Spotify playlist with a *uniformly
random* order, bypassing Spotify's built-in shuffle (which tends to replay a
small subset on large playlists).

**Live page:** https://bert-q-matheson.github.io/true-shuffle/

## How it works

1. Pick a source playlist (defaults to Liked Songs). It is never modified —
   only read.
2. Hit **Shuffle**. The page fetches every track, randomizes the order with a
   Fisher–Yates shuffle (`crypto.getRandomValues`), and writes the result into
   a dedicated playlist named `<source> 🔀`, created private on first use.
3. Re-shuffling **replaces** the 🔀 playlist's contents in place — same name
   every time, no dated copies piling up. New likes are picked up because the
   source is re-read (with a cache — see below).
4. Play the 🔀 playlist in Spotify with Spotify's own shuffle **off** — the
   randomness is baked into the track order.

No server, no build step. Spotify login uses PKCE (OAuth for apps that can't
keep a secret), so the whole thing is one `index.html`.

## Setup (one time)

1. Go to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
   and log in with your Spotify account. Click **Create app**.
2. Name it anything (e.g. "True Shuffle"), description anything.
3. Under **Redirect URIs**, add the page's URL **exactly**, trailing slash
   included:
   `https://bert-q-matheson.github.io/true-shuffle/`
   (The page also displays the exact URI on its connect screen, so you can
   copy it from there.)
4. Under "Which API/SDKs are you planning to use?" check **Web API**, then
   **Save**.
5. Open the app, go to **Settings**, copy the **Client ID**. It is a public
   identifier, not a secret — PKCE means there is no client secret involved.
6. Paste it into `CONFIG.clientId` at the top of `index.html` and redeploy.

Note: new Spotify apps start in Development Mode (fine for personal use, up
to 5 users). The app owner needs an active Spotify Premium subscription.

## Usage

- **Connect Spotify** — one click, approve the permissions.
- **Source playlist** — Liked Songs or any of your playlists.
- **Shuffle** — writes a fresh random order into `<source> 🔀`.
- **Refresh track cache** — the track list is cached in the browser
  (localStorage) and only re-fetched when the source changes (track count for
  Liked Songs, snapshot id for playlists). Use this if the cache ever looks
  stale.
- **Disconnect** — clears the stored tokens.

## Notes & limitations

- Spotify's February 2026 Web API migration renamed the playlist endpoints
  (`/tracks` → `/items`, `track` → `item`); this page uses the current ones.
- Apps can only read track listings for playlists you **own or collaborate
  on**. Picking a followed playlist as the source will fail with an
  explanatory error.
- Tokens live in the browser's localStorage (same as Spotify's own web
  player). They are narrowly scoped (`playlist-read-private`,
  `user-library-read`, `playlist-modify-private`) and can be revoked anytime
  from your Spotify account page under "Apps with access".
