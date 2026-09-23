diff --git a/README.md b/README.md
index ac3dd5e63d4b5a7b0d1ec4235c3ce6dd6f916dc4..a5d405058ee8a535c14b71104008095a36ba4990 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,70 @@
-# Music-6
\ No newline at end of file
+# Synthetiq YouTube Connector
+
+A standalone, GitHub-ready service that lets **Synthetiq Music** connect a user's YouTube account and import entire YouTube playlists (up to 500 tracks per import) as normalized music tracks. It uses Google OAuth 2.0 with the minimal `youtube.readonly` scope; the browser never receives a Google access token.
+
+## What is included
+
+- `src/` — zero-dependency Node.js OAuth service and YouTube Data API client.
+- `public/synthetiq-youtube.js` — small client adapter for the Synthetiq web application.
+- `public/` — a connection and playlist-import screen for local verification.
+- `test/` — automated tests for OAuth URL construction and Synthetiq track conversion.
+- `.github/workflows/ci.yml` — GitHub Actions checks on pushes and pull requests.
+
+## Prerequisites
+
+- Node.js 20 or newer.
+- A Google Cloud project with **YouTube Data API v3** enabled.
+- OAuth consent screen configured, and an OAuth 2.0 **Web application** client created.
+
+## Run locally
+
+1. Copy the configuration template: `cp .env.example .env`.
+2. In Google Cloud Console, add `http://localhost:3000/api/youtube/callback` to the client's **Authorized redirect URIs**.
+3. Fill `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET`, and `SESSION_SECRET` in `.env`.
+4. Load the environment and start it:
+
+   ```bash
+   set -a && source .env && set +a
+   npm start
+   ```
+
+5. Open `http://localhost:3000`, select **Connect YouTube**, complete consent, then return to Synthetiq Music.
+
+> `.env` is intentionally ignored. Never commit a Google client secret or session secret.
+
+## Add it to Synthetiq Music
+
+Deploy this service behind HTTPS, set `YOUTUBE_REDIRECT_URI` to its public callback URL, and register that exact URL with Google. If Synthetiq is hosted on a different origin, set `ALLOWED_ORIGINS` to that exact origin, set `COOKIE_SAME_SITE=none` (HTTPS is required), and call the adapter with the connector URL. Then copy `public/synthetiq-youtube.js` into Synthetiq Music (or serve it from this connector) and use:
+
+```js
+import { connectYouTube, importYouTubePlaylist } from "./synthetiq-youtube.js";
+
+const connectorUrl = "https://youtube-connector.example.com";
+
+// On a “Connect YouTube” click:
+connectYouTube(connectorUrl);
+
+// After connection, pass a YouTube playlist URL or playlist ID:
+const tracks = await importYouTubePlaylist("https://www.youtube.com/playlist?list=PLAYLIST_ID", connectorUrl);
+// Send `tracks` to Synthetiq's existing library/import flow.
+```
+
+Each returned item has `provider`, `externalId`, `title`, `artist`, `artworkUrl`, and `url`. The service returns no audio stream: playback and downloads remain subject to YouTube's terms and API policies.
+
+## API
+
+| Route | Purpose |
+| --- | --- |
+| `GET /health` | Health check returning `{ "ok": true }`. |
+| `GET /api/youtube/connect` | Starts Google OAuth. |
+| `GET /api/youtube/callback` | OAuth callback; validates a signed, browser-bound state cookie and stores an HTTP-only session. |
+| `GET /api/youtube/playlist?id=PLAYLIST_ID` | Returns up to 500 normalized tracks for the connected user. |
+
+Sessions are in memory, suitable for local development or a single process. For a multi-instance deployment, use a shared encrypted session store and HTTPS (`NODE_ENV=production`).
+
+## Verify
+
+```bash
+npm test
+npm run check
+```
