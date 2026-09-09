# Changelog

All notable changes to RICHAT are documented in this file. Versions match the Docker image tags (`dotnetdummy/richat:<version>`).

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
