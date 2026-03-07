# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**beep terminal** is a single-file, zero-dependency PWA that emulates the Linux `beep` command in the browser using the Web Audio API. The entire application lives in `index.html` — there is no build step, no package manager, and no framework.

## Development

Open `index.html` directly in a browser, or serve it with any static file server:

```bash
python3 -m http.server 8080
# or
npx serve .
```

No build, lint, or test commands exist — there are no toolchain config files.

## Architecture

Everything is in `index.html`:

- **CSS**: Terminal aesthetic with IBM Plex Mono, CRT scanline overlay via `::before`/`::after` pseudo-elements. All colors are CSS custom properties (`--green`, `--green-dim`, `--amber`, `--red`, `--cyan`, `--bg`, `--bg2`, etc.) defined in `:root` (dark) and overridden in `body.light` (light mode). Dark/light preference is saved in `localStorage` under `beep_theme`.
- **HTML**: A `.terminal` container with a fake macOS-style titlebar (including SVG theme toggle button), a scrollable output log, two tabs (Play / Presets), speed slider row, button row, and a toast notification element.
- **JavaScript**: All logic inline in a `<script>` tag:
  - `parseBeep(cmd)` — tokenizes a `beep` command string into an array of `{freq, dur, delay}` note objects; splits on `-n` separators.
  - `applySpeed(notes)` — scales `dur` and `delay` of each note by the current speed factor.
  - `speededCmd(cmd)` — rebuilds the command string with scaled `-l`/`-d` values for sharing/export.
  - `runBeep()` — async playback loop using `AudioContext`, `OscillatorNode` (square wave) and `GainNode` with attack/release envelopes. Interruptible via `stopRequested` flag checked inside a custom `sleep()`.
  - `downloadWav()` — renders the speed-adjusted notes offline via `OfflineAudioContext` and encodes a 16-bit mono PCM WAV via `audioBufferToWav()`.
  - `showHelp()` — writes `beep -h` into the textarea and calls `runBeep()`.
  - `sleep(ms)` — Promise-based delay with a 20ms polling interval for stop detection.
  - Presets are stored in `localStorage` as JSON under key `beep_presets`.
  - Sharing encodes the speed-adjusted command into a `?cmd=` URL query parameter.
  - Service worker (`sw.js`) is registered for offline/PWA support.

## Supporting Files

- `sw.js` — cache-first service worker caching all static assets under cache key `beep-v1`.
- `manifest.json` — PWA manifest; `start_url` is `/beep/` (deployed at a `/beep/` subpath).
- `404.html` — served by GitHub Pages for unknown routes.
- `.nojekyll` — disables Jekyll processing on GitHub Pages.

## Deployment

The app is deployed as a GitHub Pages static site. The `start_url` in `manifest.json` assumes a `/beep/` subpath, so keep this in mind if changing the deployment path.

## beep Command Syntax

```
beep [-f FREQ] [-l MS] [-d MS] [-n ...]

  -f FREQ   frequency in Hz  (default: 440)
  -l MS     duration in ms   (default: 200)
  -d MS     delay before note in ms (default: 0)
  -n        separator for the next note
```

## Speed Slider — intentional design

The speed slider deliberately does **not** modify the textarea content. The original command is always preserved as-is. The speed factor is applied at runtime in `applySpeed(notes)` (scales `-l` and `-d` values) and `speededCmd(cmd)` (rebuilds the command string with scaled values for sharing).

This means: Play, Download WAV, and Share all use the speed-adjusted values — but the textarea always shows the original. **Do not "fix" this** by writing scaled values back into the textarea.

## PWA / Service Worker Cache

When modifying static assets, bump the cache version string in `sw.js` (`const CACHE = 'beep-v1'`) to force clients to pick up the new files.
