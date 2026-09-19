# TuneFlip (Kae-Pop) — Plan (2026-09-19)

A Hitster-style guess-the-year card game, built as a gift. Not affiliated with Hitster.

Static site on GitHub Pages (`zymsys/tuneflip` → https://zymsys.github.io/tuneflip/ (Carolyn can fork to her own account)).
Vanilla HTML/JS, no build step. Nothing server-side; decks live in the builder's localStorage.

## Core design

- **Song card QR** = `https://zymsys.github.io/tuneflip/#t=<itunesTrackId>&y=<year>`
  - In-app scanner parses `t` and `y`; native camera scan also works (opens app → tap-to-play fallback).
  - Player phone does live `https://itunes.apple.com/lookup?id=<t>&callback=cb` (JSONP, no key, no CORS)
    → `previewUrl` (30s m4a), `trackName`, `artistName`, `artworkUrl100`.
  - Player needs **no deck data**. Cards are self-describing.
- **App card QR** = `https://zymsys.github.io/tuneflip/?name=Kae-Pop&c1=ff007f&c2=2b1055&icon=🎤`
  - Player app saves branding to localStorage as "current deck". Logo image won't fit a QR → emoji/preset.
- **Year**: iTunes `releaseDate` is the *album version's* date (remasters/compilations lie).
  Builder shows it as default, user overrides per track. Override is printed on card AND baked into QR.
- **Audio unlock**: one tap on "Start scanning" plays a silent src on the `<audio>` element; after that,
  swapping `src` + `play()` on scan is allowed on iOS. Scan → flip phone → clip plays.

## Files

- `index.html` — player. Branded home → Scan (getUserMedia, `playsinline`, jsQR on canvas ~10fps)
  → stop camera on decode → auto-play preview → 30s progress bar → Replay / Scan next.
  Fallback route when opened from native camera with `#t=`: PLAY button.
  Lib: jsQR (cdnjs).
- `build.html` — deck builder. localStorage (`decks` JSON) + export/import JSON backup.
  Per deck: name, c1, c2, icon, tracks[{id, title, artist, year, yearFetched}].
  Add tracks by: Apple Music song link (`?i=` param), Apple Music album link (lookup collectionId
  &entity=song → all tracks), or pasted `Title - Artist` lines (search API, `limit=1`).
  **Throttle searches ~3s apart** (API ≈ 20 calls/min) with a progress bar.
- `print.html` — card sheet via print CSS (`@page { size: letter; margin: 0 }`, page-break per sheet).
  3×3 grid, 63mm square cards, cut marks. Fronts: big year, artist, title. Backs: QR.
  Back pages **mirror columns** for long-edge duplex. Plus one app card per deck.
  Lib: qrcode-generator (cdnjs). jsPDF only if browser print alignment is unreliable.

## Won't work / limits

- Playlist URLs: Apple needs MusicKit ($99 dev token); Spotify needs API app. Not in the web app.
  Stretch: local Python helper that scrapes a playlist page into a `Title - Artist` list.
- Locking the phone may pause Safari audio. Face-down only; raise auto-lock for game night. Test.

## Build order

1. Player: scan + lookup + auto-play. Prove it with one hand-made QR first.
2. Builder: deck CRUD, add-by-link, year override, branding.
3. Print sheet; duplex test on the real printer.
4. Text-list paste, album links, JSON export/import, polish.
