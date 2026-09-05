# Voice TURN

Voice huddles work for many home NATs with the default public STUN servers (Cloudflare and Google). Symmetric NAT, CGNAT, and networks that block UDP still need TURN.

TURN is a sidecar (`coturn`). It is not part of the app image or the root compose file. It needs host networking and a public IPv4. Relayed audio uses TURN bandwidth. The huddle size cap (`MAX_VOICE_PARTICIPANTS`, default 8) does not change.

## App env

Set these on the RICHAT app (the root `.env`). They must match the coturn user:

```
TURN_HOST=turn.example.com
TURN_USERNAME=richat
TURN_CREDENTIAL=the-same-password-as-coturn
```

That adds `stun:turn.example.com:3478` plus UDP and TCP TURN on port 3478, after the default public STUN servers.

Optional overrides:

- `TURN_URLS` — comma-separated URLs instead of the host defaults. Example: `turn:turn.example.com:3478?transport=udp,turn:turn.example.com:3478?transport=tcp`
- `ICE_SERVERS` — JSON array. Replaces STUN and TURN env entirely.

```
ICE_SERVERS=[{"urls":"stun:stun.l.google.com:19302"},{"urls":"turn:turn.example.com:3478","username":"richat","credential":"secret"}]
```

Do not put TURN credentials in a client bundle. The browser gets them from the app after sign-in.

## Run coturn

Use a Linux host with a public IPv4. Point `turn.example.com` at that IP (not at the chat hostname unless they share the host). Docker Desktop on macOS cannot use host networking; local voice does not need TURN.

```bash
cd coturn
cp .env.example .env
# set TURN_PASS and TURN_EXTERNAL_IP (the host public IPv4)
docker compose --env-file .env --project-name richat-turn up -d
```

`TURN_USER` / `TURN_PASS` in that `.env` must match `TURN_USERNAME` / `TURN_CREDENTIAL` on the app.

Firewall:

| Port        | Proto   | Why                    |
| ----------- | ------- | ---------------------- |
| 3478        | UDP/TCP | STUN + TURN allocation |
| 49152–49200 | UDP     | Media relay            |

`TURN_EXTERNAL_IP` must be the address peers can reach. If it is wrong, ICE completes on paper and you still get no audio.

This image runs without TLS (`turn:`, not `turns:`). Add certificates and drop `--no-tls` / `--no-dtls` later if you need TURN over TLS on 5349 or 443.

Coturn runs with `--no-stdout-log`: its allocation logs record client IP addresses, which Richat does not keep anywhere. To debug a relay problem, temporarily switch back to `--log-file=stdout --simple-log` and remove it again when done.
