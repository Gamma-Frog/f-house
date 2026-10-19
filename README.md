# Vibe Set

Type a party vibe and get an ordered DJ set. Export it to Spotify, copy it as text, save it as a CSV, or open a SoundCloud search for any track.

It's a single file (`index.html`) with no build step and no backend.

## How it picks and orders songs

Spotify can't drive this anymore. Since February 2026, Spotify apps in development mode no longer get track `popularity` or recommendations, can't read other people's playlists, and get at most 10 search results. So the "vibe brain" uses Deezer's public API instead, which needs no key:

1. **Find the vibe.** Search public playlists for your vibe text. Each comma-separated phrase is also searched separately. Take up to 12 playlists, at most 2 per curator.
2. **Count curator votes.** A song gets one vote per curator that includes it. Playlists that share over half their tracks count as one curator, which stops label spam. Artists that several curators play get a bonus, which keeps picks on-genre.
3. **Favor popular songs.** Deezer's rank (0–1M plays-based score) is weighted by the "Deep cuts ↔ Big crowd-pleasers" slider. At the default (80%), well-known songs that fit the vibe come first, and stock filler sinks.
4. **Order the set.** Each song's **heat** (popularity within the set) is matched to a set shape:
   - **Energy arc**: warm-up → build → peak → last call
   - **Hits first**: biggest songs first, deeper cuts after
   - **Slow build**: rising all the way
   Songs that curators place next to each other are then kept together, since curators often sequence for flow. The same artist never plays twice within 3 tracks.

Karaoke, tribute, and "in the style of" versions are filtered out, as are tracks under 1:30 or over 10 minutes.

## Spotify export setup (one time)

Spotify only accepts an **HTTPS** redirect URI, or `http://127.0.0.1` for local testing. To use the app on your phone, host it somewhere with HTTPS:

- **GitHub Pages**: make a repo, add `index.html`, then turn on Settings → Pages. Your URL looks like `https://<you>.github.io/<repo>/`.
- **Netlify Drop**: drag the folder onto app.netlify.com/drop.

Then:

1. Open the app and tap ⚙️. It shows the exact **Redirect URI** to use (the page's own URL). Copy it.
2. At developer.spotify.com/dashboard, create an app, tick **Web API**, and paste that Redirect URI.
3. Copy the **Client ID** into the app and tap **Save & connect**. No client secret is needed (this uses PKCE).
4. Spotify requires the app owner to have **Premium**. Anyone else who logs in must be added under **User Management** (up to 5 users in development mode).

When you export, each track is matched on Spotify by ISRC (exact recording) first, then by title + artist. Tracks that can't be matched are listed with SoundCloud links. The playlist is created private unless you toggle it to public.

## Run locally

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Then open http://127.0.0.1:8000/ and register `http://127.0.0.1:8000/` as a redirect URI. Use `127.0.0.1`, because Spotify rejects `localhost`.

## Notes

- Your client ID, Spotify tokens, and last set are stored only in your browser's localStorage.
- Song previews are Deezer's 30-second clips. If one stops working after a few hours, tap **Remix** or rebuild the set to refresh the links.
- There's no BPM or key data. Neither Spotify nor Deezer reliably provides it anymore.
