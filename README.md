# RICHAT - voice and messaging

Self-hosted Discord-style chat. This repository is the **install bundle** and the **issue tracker**. It does not contain application source.

- **Run it:** Docker Compose in this repo pulls [`dotnetdummy/richat`](https://hub.docker.com/r/dotnetdummy/richat) and PostgreSQL.
- **Product site:** [RICHAT.net](https://richat.net)
- **Issues:** file bugs and requests here.

The app runs as a PWA. It supports public and private channels, threads, emoji reactions, custom emojis, GIF search, pinned messages, link previews, and push notifications. Voice is a huddle on an existing text channel (screen share, webcams, soundboard, party mode, poker, and pictionary).

## Requirements

- Docker with Compose
- A login provider: a Steam Web API key, email/password, or a generic OAuth2 provider (exactly one)
- VAPID keys (push notifications)
- HTTPS in production (PWA, push, and microphone)

## Install

```bash
git clone https://github.com/dotnetdummy/richat.git
cd richat
cp .env.example .env
```

Fill in `.env`:

1. Set `POSTGRES_PASSWORD` to a strong password.
2. Set `APP_URL` to the public origin (`https://chat.example.com`).
3. Set `INITIAL_ADMIN` to your email (email/password or OAuth2) or Steam persona name. The first matching sign-in becomes admin.
4. Generate `AUTH_SECRET` and paste it:

   ```bash
   openssl rand -base64 32
   ```

   Keep the same secret. Rotating it invalidates every session. With email/password login it also breaks sign-in, because stored emails are HMAC'd with this value.

5. Generate Web Push VAPID keys and paste `VAPID_PUBLIC_KEY` / `VAPID_PRIVATE_KEY`:

   ```bash
   npx --yes web-push generate-vapid-keys
   ```

   Set `VAPID_SUBJECT` to a `mailto:` address or `https:` URL that identifies this app. Keep the same key pair; rotating it invalidates every device's push subscription.

6. Enable **exactly one** login provider: `STEAM_API_KEY`, `AUTH_EMAIL_PASSWORD=true`, or OAuth2 (`OAUTH2_CLIENT_ID` + `OAUTH2_CLIENT_SECRET` + a discovery URL or explicit endpoints). The app refuses to start with zero or more than one.

Then:

```bash
docker compose up -d
```

Open `http://localhost:1987` (or your `APP_URL`) and sign in as `INITIAL_ADMIN`. After that, manage the whitelist from **Admin** (Shield icon next to the logo in the channel menu). The seed creates public channel `general`.

Chat data lives in the `pgdata` volume. `GET /health` is the health check. Pending migrations run when the app starts.

Override the image with `RICHAT_IMAGE` if you want a pinned tag (default `dotnetdummy/richat:latest`).

### Portainer

Deploy this `docker-compose.yml` as a stack that **pulls** `RICHAT_IMAGE`. Do not add a `build:` key — Portainer has no app source tree, and compose build fails with `mkdir /.docker: permission denied`.

### Reverse proxy

Terminate TLS in front of port 1987. Set `APP_URL` (and `AUTH_TRUSTED_ORIGINS` if the origin list is wider than `APP_URL`) to the public HTTPS URL. Production needs HTTPS for install-as-app, push, and huddle microphones.

## Upgrade

```bash
docker compose pull && docker compose up -d
```

Pending migrations run when the app starts. Pin `RICHAT_IMAGE` to a version if you do not want to follow `latest`.

## Voice TURN

Optional. Needed on hard NATs. Coturn is a **separate** compose file in [`coturn/`](coturn/) (host networking, public IP). It is not in the app image or the root compose file. See [`docs/turn.md`](docs/turn.md).

## Environment

Put these in `.env`. See `.env.example`.

| Variable                                              | Required                             | Purpose                                               |
| ----------------------------------------------------- | ------------------------------------ | ----------------------------------------------------- |
| `POSTGRES_PASSWORD`                                   | Yes                                  | Postgres password (compose also builds `DATABASE_URL`) |
| `AUTH_SECRET`                                         | Yes                                  | Session signing secret                                |
| `APP_URL`                                             | Yes.                                 | Public origin of the app                              |
| `AUTH_TRUSTED_ORIGINS`                                | No (defaults to `APP_URL`)           | Comma-separated allowed origins                       |
| `STEAM_API_KEY`                                       | One login provider                   | Steam OpenID login                                    |
| `AUTH_EMAIL_PASSWORD`                                 | One login provider                   | `true` enables email/password login                   |
| `OAUTH2_CLIENT_ID` / `OAUTH2_CLIENT_SECRET`           | One login provider                   | Generic OAuth2 / OIDC login                           |
| `OAUTH2_DISCOVERY_URL`                                | With OAuth2                          | OIDC discovery URL (or set the explicit URLs)         |
| `OAUTH2_AUTHORIZATION_URL` / `OAUTH2_TOKEN_URL`       | With OAuth2 (no discovery URL)       | Explicit OAuth2 endpoints                             |
| `OAUTH2_USERINFO_URL`                                 | No                                   | Explicit userinfo endpoint                            |
| `OAUTH2_SCOPES`                                       | No (default `openid,profile,email`)  | Comma-separated scopes                                |
| `OAUTH2_PROVIDER_NAME`                                | No (default `OAuth`)                 | Login button label                                    |
| `VAPID_PUBLIC_KEY`                                    | Yes                                  | Web Push                                              |
| `VAPID_PRIVATE_KEY`                                   | Yes                                  | Web Push                                              |
| `VAPID_SUBJECT`                                       | Yes                                  | Web Push subject (`mailto:` or `https:` URL)          |
| `KLIPY_APP_KEY`                                       | No                                   | KLIPY GIF search. Without it, only uploaded GIFs.     |
| `PORT`                                                | No (default `1987`)                  | Host port mapped to the app                           |
| `INITIAL_ADMIN`                                       | First setup                          | Email or Steam name of the first admin                |
| `POSTGRES_USER` / `POSTGRES_DB`                       | No (default `richat`)                | Postgres role and database name                       |
| `DB_MIGRATE_ON_START`                                 | No (default `true`)                  | Run pending migrations when the app starts            |
| `RICHAT_IMAGE`                                        | No                                   | App image tag                                         |
| `EMAIL_SALT_ROUNDS`                                   | No (default `10`)                    | bcrypt rounds for hashed emails                       |
| `MESSAGE_RETENTION_DAYS`                              | No (default `90`)                    | Last-edit age after which unpinned messages are deleted |
| `PINNED_MESSAGE_RETENTION_DAYS`                       | No (default `365`)                   | Last-edit age after which pinned messages are deleted |
| `USER_RETENTION_DAYS`                                 | No (default `365`)                   | Inactivity after which accounts are deleted (`0` off) |
| `TURN_HOST` / `TURN_USERNAME` / `TURN_CREDENTIAL`     | No                                   | TURN for voice on hard NATs. See [`docs/turn.md`](docs/turn.md). |
| `MAX_VOICE_PARTICIPANTS`                              | No (default `8`)                     | Max people in a voice huddle                          |

With email/password there is no email infrastructure: password resets go through **Admin → Users → Reset password**, which generates a random password the admin hands over out-of-band; the user can then change it in Settings → Profile.

## Privacy

- Emails are hashed at registration; the plaintext is never stored. Steam and OAuth2 use bcrypt. Email/password uses a deterministic HMAC-SHA256 (keyed with `AUTH_SECRET`) because sign-in must look the user up by email.
- Sessions store no IP address and no user agent.
- Whitelist invites hold a plaintext email, Steam name, or `@domain`. Person rows are consumed at first sign-in. Domain rows stay until they expire. Every row is purged at its chosen TTL (24 hours to 1 year).
- The Steam ID in `auth.account` stays in plaintext: it is the login identity and is needed for lookup at sign-in.
- A daily retention job (and on demand from Admin → Retention) purges old unpinned and pinned messages, orphan uploads, expired sessions, stale push subscriptions, and inactive accounts. The owner and system user are never auto-deleted. Deleted accounts keep chat shown as Deleted user.
- Push payloads are encrypted to the device. Each user can hide channel names, sender names, and message text from push payloads (Settings → Notifications).
- Link previews are fetched server-side (public addresses only) and cached in memory only.
- The TURN sidecar runs with stdout logging off, because coturn allocation logs contain client IPs.
- Messages and files are stored unencrypted in PostgreSQL; there is no end-to-end encryption. Database dumps contain plaintext content.

## Support

- Issues: this repository
- Product site: [richat.net](https://richat.net)
