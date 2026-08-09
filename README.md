# e2dub

Lightweight live dubbing plugin for DreamOS / Enigma2 on Dreambox One.

The Enigma2 UI and process supervisor are Python 2.7 compatible. Gemini Live
API communication uses a small standard-library-only helper, so no Google SDK
or third-party Python package is installed on the receiver.

Runtime data:

- API key: `/root/apikey.txt`
- Log: `/tmp/E2LivDub.log`
- Output language: Persian (`fa`)
- Gemini model: `gemini-3.5-live-translate-preview`

The pipeline reads the selected DVB service from Enigma2 streamproxy, sends
16-bit mono 16 kHz PCM to Gemini in 100 ms chunks, encodes the returned 24 kHz
PCM as AAC, and remuxes it with the original video into a local MPEG-TS stream.

Build the DreamOS arm64 Debian package with:

```sh
python build_deb.py
```

The resulting package is written to `dist/`.

## 0.1.2

- Uses the DreamOS `eTimer.timeout.connect()` API and retains signal connections.
- Adds a guarded plugin entry point so screen-construction errors cannot escape
  into Enigma2's ActionMap.
- Removes stale Python bytecode during installation.
- Closes WebSocket resources on every reconnect path.

## 0.1.3

- Adds a compact 300x110 plugin-browser logo with safe inner margins.
- Creates `/tmp/E2LivDub.log` during installation.
- Schedules an automatic Enigma2 GUI restart after `dpkg` finishes.

## 0.1.4

- Creates an empty root-owned `/apikey.txt` with mode `0600` on first install.
- Preserves an existing API key during package upgrades.
- Deletes `/apikey.txt` when the package is removed or purged.

## 0.1.5

- Corrects the API-key location to `/root/apikey.txt` everywhere.
- Migrates a key left at the incorrect 0.1.4 path without truncating it.
- Deletes `/root/apikey.txt` when the package is removed or purged.

## 0.1.6

- Treats Python 2.7/OpenSSL read timeouts as normal idle WebSocket reads instead
  of disconnecting the Gemini session every two seconds.
- Logs when source PCM starts streaming and when translated PCM first arrives.
- Moves the pipeline into a persistent controller so the plugin window closes
  immediately after Start while translation continues in the background.
- Reopening the plugin shows the current background status and allows Stop.

## 0.1.7

- Allows up to 60 seconds for a slow Gemini `setupComplete` response.
- Keeps the original DVB service playing while Gemini connects.
- Switches Enigma2 to the local translated stream only after the first real
  translated PCM block arrives, preventing a false LIVE state with silence.

## 0.1.8

- Adds timed Enigma2 notifications for starting, waiting, ready, and stopped.
- Replaces the service reference with channel name, frequency, and satellite.
- Refreshes the screen with a compact dark design and green live-status accent.
- Tunes Gemini automatic speech detection for shorter phrase-end latency.

## 0.1.9

- Replaces the unhelpful 60-second wait for a stalled initial Gemini setup with
  a 10-second watchdog and a one-second retry.
- Shows `Gemini is busy - retrying...` while rotating a stuck connection.
- Keeps the original channel audible until translated PCM is actually received.

## 0.2.0

- Adds strict WebSocket frame validation and preserves server close codes,
  reasons, and structured Live API errors in sanitized diagnostics.
- Publishes an atomic JSON state containing the setup profile, attempt number,
  and last non-secret failure detail; legacy plain state files remain readable.
- Tries the managed long-session setup first, then falls back to the minimal
  official Live Translation setup with exponential retry backoff.
- Removes the speculative `Gemini is busy` status. The screen now reports the
  actual setup phase or server error without exposing the API key.
- Detects channel changes even while the initial Gemini session is connecting.
- Extends package-removal cleanup to capture, relay, helper, and mux processes.

## 0.2.1

- Reduces the audio-capture probe window after an on-device benchmark showed
  first PCM arriving in one second instead of three on the tested DVB service.

## 0.2.2

- Adds a balanced low-latency VAD profile with 350 ms end-of-speech silence;
  the minimal official setup remains the automatic compatibility fallback.
- Reduces local MPEG-TS mux delay from 700 ms to 100 ms and translated-service
  readiness polling from 500 ms to 100 ms.

## 0.2.3

- Treats Python 2/OpenSSL `EAGAIN` (`Errno 11`) as a transient idle read
  condition instead of tearing down a healthy Gemini session.
- Resets reconnect backoff after every successfully established session.
- Restores the proven 700 ms MPEG-TS mux delay to prevent decoder starvation
  and periodic buffering on DreamOS.

## 0.2.4

- Handles the second Python 2/OpenSSL nonblocking-read form, `The operation did
  not complete (read)`, as an idle socket condition rather than a disconnect.

## 0.2.5

- Displays the installed e2dub version directly in the plugin screen.
- Limits FFmpeg interleave buffering from its 10-second default to 500 ms and
  flushes local UDP packets immediately.
- Repeats H.264 parameter data at keyframes, publishes PAT/PMT and SDT tables
  periodically, marks the initial discontinuity, and enlarges the UDP socket
  buffer to improve DreamOS decoder recovery and video continuity.

## 0.2.6

- Keeps the original DVB channel name when switching to the local translated
  transport stream.
- Closes the plugin browser and parent menus after Start, returning directly to
  fullscreen television.
- Adds a persisted 0-100 original-channel volume setting (default 0). When it is
  enabled, FFmpeg plays the original channel during translated silence and
  automatically ducks it beneath translated speech.

## 0.2.7

- Fixes an Enigma2 menu regression introduced in 0.2.6. Parent plugin menus
  are now closed only through their normal `Screen.close()` lifecycle.
- Never mutates `session.dialog_stack` and never destroys parent dialogs with
  `deleteDialog`, so Plugin Browser and the main menu remain reusable.

## 0.2.8

- Moves original-channel volume to the main plugin screen as a LEFT/RIGHT
  progress bar and always shows the exact numeric percentage beside it.
- Saves every volume change immediately. If e2dub is already active, changes
  are debounced and the audio pipeline is rebuilt once with the new mix level.
- Returns briefly to the original DVB service during a live audio-mix rebuild,
  avoiding a stale pipeline that was started with the previous volume.

## 0.2.9

- Replaces the skin-dependent ProgressBar with a directly resized eLabel fill,
  so LEFT/RIGHT visibly changes the bar on DreamOS skins without a bar pixmap.
- Adds `Telegram: t.me/routekernel1` and a centered plugin-version line below
  the main controls.

## 0.2.10

- Replaces the unpainted empty-label fill with 20 individually painted volume
  segments. Each LEFT/RIGHT step visibly toggles exactly one 5% segment.
- Uses a non-empty label in every segment so DreamOS paints its green
  background consistently across skins.

## 0.2.11

- Uses a DreamOS `MultiPixmap` backed by 21 pre-rendered PNG levels (0-100%).
  This bypasses the image's non-painting Label and ProgressBar implementations.
- Generates the compact RGB bar PNGs deterministically while building the DEB,
  without adding Pillow or any runtime dependency to the receiver.

## 0.2.12

- Duplicates the final translated MPEG-TS with FFmpeg's tee muxer: the existing
  local TV output remains on UDP 19876 and a laptop output is sent to
  `192.168.2.3:19877`.
- Marks the laptop tee slave `onfail=ignore`, so an offline laptop or closed VLC
  cannot interrupt Dreambox playback or translation.
