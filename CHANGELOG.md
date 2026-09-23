# Changelog

All notable changes to RICHAT are documented in this file. Versions match the Docker image tags (`dotnetdummy/richat:<version>`).

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.7.0] - 2026-09-23

### Added

- `PG_POOL_MAX` (default 20) sets the database pool size. `PG_STATEMENT_TIMEOUT_MS` (default 30000, `0` disables) caps each query.
- `/health` returns JSON readiness (`engine`, `database`, `listener`) and answers 503 until the engine has started or while the database is unreachable.
- `LOG_FORMAT=json|pretty` (JSON by default in production). Production logs a metrics line each minute: open SSE streams, pool usage, and message load times.

### Changed

- Dependencies are now updated to latest, including Material UI 9, Vite 8, and a newer Nitro nightly. Generic OAuth2 sign-in now uses `/api/auth/callback/oauth2` with PKCE on. Update the identity provider redirect URI if it still points at `/api/auth/oauth2/callback/oauth2`.
- The Richat Overlay helper is a Tauri app, so the macOS and Windows downloads are much smaller. macOS may ask again for Input Monitoring and Accessibility, because the app identity changed.
- The server validates env values at boot and refuses to start with a list of every malformed setting (numbers, booleans, URLs, `ICE_SERVERS`, partial VAPID keys). Missing VAPID keys only log a warning.
- Graceful shutdown on `SIGTERM`/`SIGINT`: pending last-seen writes are flushed and database connections are closed (5 s limit).
- Migrations take a Postgres advisory lock, so two containers starting together no longer race.
- Upgrade runs two migrations: `0005_slim_channel_notify` (smaller channel change notifications) and `0006_thread_and_retention_indexes` (thread and retention indexes; drops indexes duplicated by primary keys).
- Retention deletes in batches of 1000 rows to avoid long table locks.
- Uploaded files, GIFs, sounds, avatars, and emojis support `ETag`/304 and byte ranges, so audio and video seeking does not re-download the file.
- Sessions are cached in memory for up to 30 seconds; role changes and removed users apply immediately.
- Games and party mode load their code only when active in the joined huddle.
- The Docker image no longer installs a second copy of the dependencies; the server output is self-contained.
- Reactions and file attaches reload the message without thread and quote queries.

### Fixed

- The overlay window’s macOS permission note is hidden on Windows and Linux.
- Realtime recovers after the Postgres listener connection drops: it reconnects with backoff and clients refetch channels, messages, users, and voice state.

## [2.6.0] - 2026-09-20

### Added

- Switch microphone, speaker, and camera from the huddle header without opening Settings. The dock stays mute, deafen, and leave.

### Fixed

- Trusted JWT login recovers with a full reload when the reverse-proxy token expires.
- Voice device pickers show Default instead of a blank value.

## [2.5.0] - 2026-09-18

### Added

- Word Bomb on the huddle: type a word that contains the shown syllable before the timer; last player with lives wins.
- Pictionary tweaks: first player in the lobby chooses 1–8 rounds and starts the game.
- Mute and volume for screen-share audio on the huddle stage and pop-out.

### Changed

- Poker raise uses +/- by the big blind plus Min / half-pot / pot / all-in presets instead of a slider.
- Poker shows remaining players' hole cards on their seats at a contested showdown.
- Poker cards, felt, deal/showdown motion, and the turn countdown are easier to follow.

### Fixed

- Pictionary guesses overlay no longer clips the latest line when more than two guesses are in.

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
