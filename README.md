# beep terminal

Browser-based `beep` command player — plays tones via Web Audio API.
Works on desktop and mobile, installable as a PWA.

## Features
- Full `beep` syntax: `-f` (freq Hz), `-l` (length ms), `-n` (next note), `-d` (delay ms)
- Speed control: scale playback speed from 10% to 300% — the textarea always shows the original command, but the speed is applied to playback, WAV export, and shared URLs
- WAV export: download the current command as a lossless audio file (speed-adjusted)
- URL sharing: `?cmd=...` loads a command directly (speed-adjusted values baked in)
- Presets: save/load named commands in localStorage
- Dark/light mode toggle with preference saved in localStorage
- PWA: installable on Android/iOS, works offline
- Keyboard: `Ctrl+Enter` to run

## Syntax reference

```
beep [-f FREQ] [-l MS] [-d MS] [-n ...]

  -f FREQ   frequency in Hz         (default: 440)
  -l MS     duration in ms          (default: 200)
  -d MS     delay before note in ms (default: 0)
  -n        separator for next note

Example:
  beep -f 440 -l 500 -n -f 880 -l 300 -n -f 659 -l 460
```
