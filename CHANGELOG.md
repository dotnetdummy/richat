# Changelog

All notable changes to RICHAT are documented in this file. Versions match the Docker image tags (`dotnetdummy/richat:<version>`).

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.4.0] - 2026-09-11

### Added

- Quote a message in a channel or thread. The original author is mentioned for alerts even if the body has no `@`.

### Changed

- The "general" public channel stays first in the sidebar, with a divider under it when there are other public rooms.
- Other live huddles in the sidebar use the incoming-call color without the pulse, distinct from the green huddle you are in.

### Fixed

- Screen share no longer stays on Connecting after a drop: a connected peer with no screen track is rebuilt, prolonged ICE disconnects rebuild instead of only restarting ICE, and a single remote video is treated as the share when the camera flag is stale.

## [2.3.0] - 2026-09-10

### Added

- `KLIPY_PRIVACY=true` restores server-side KLIPY search, the GIF proxy, mixed Explore results.
- Bare email addresses in messages become mailto links.

### Changed

- KLIPY GIF search and images load in the browser by default. Explore is KLIPY-only, with a Search KLIPY field. Existing operators who want the previous IP-hiding proxy must set `KLIPY_PRIVACY=true`.
- Noise suppression in Settings → Voice & Video runs on this device and falls back to the browser if that cannot start.

### Fixed

- Voice huddle ICE from a rolled-back or later offer is held until the remote SDP matches, so `addIceCandidate` no longer throws during glare or renegotiation.
- Voice activity and speaking rings use an AudioWorklet clock instead of the deprecated ScriptProcessorNode.
- Screen-share overlay selects the only matching display automatically.

## [2.2.0] - 2026-09-09

### Added

- Trusted JWT login via a reverse-proxy header (`AUTH_TRUSTED_JWT_HEADER`). When that header is present, visitors are signed in automatically.

## [2.1.1] - 2026-09-08

### Added

- `DATABASE_URL_ENV` to read the runtime Postgres URL from a differently named variable (default `DATABASE_URL`).
- Optional `DATABASE_MIGRATE_URL` / `DATABASE_MIGRATE_URL_ENV` so boot migrations can use a different Postgres role than the app.

## [2.1.0] - 2026-09-07

### Added

- Screen share encoding (Quality or Compressed) next to frame rate, in Settings and in a popover before you share.
- Names on screen-share annotations at each person's latest mark, including on the presenter's overlay.

### Changed

- Starting a screen share from the huddle opens a popover with last-used frame rate and encoding, then Share screen runs the browser picker.

### Removed

- Laser tool from screen-share annotations. Pen is the default.

### Fixed

- The local speaking ring now follows what you are sending (mute, push-to-talk, and push-to-mute), not raw mic level.
 
## [2.0.1] - 2026-09-06

### Added

- Short description of the app under `About` in settings. 

## [2.0.0] - 2026-09-06

- Public release. Read more at [RICHAT.net](https://richat.net)
