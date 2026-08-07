# livepolls

A real-time audience-engagement tool (Slido / Mentimeter style). Anyone creates a topic room, and
participants submit short comments that render as a **live word cloud** updating in real time across
every connected client.

## Features

- **Create a topic room** and share it; participants join instantly.
- **Live word cloud** — submitted comments aggregate by word frequency and animate live (`@visx/wordcloud`).
- **Real-time fan-out** — updates pushed to all clients in a room over WebSockets.
- **Per-room connection counts** with join/disconnect cleanup.

## Architecture

```
Next.js client ──socket.io──▶ Node/Express + socket.io server ──▶ Redis pub/sub (Upstash)
     ▲                                                                    │
     └────────────── live word-cloud updates fan out to all room clients ─┘
```

- **Client:** Next.js 14 (React 18), TanStack Query, `socket.io-client`, `@visx/wordcloud`,
  Radix/shadcn, Upstash Redis. Server actions validate + store topics/comments and aggregate word frequencies.
- **Server (separate):** Express + `socket.io` + `ioredis` using **Redis pub/sub** to broadcast room
  updates across clients (and, in principle, across horizontally-scaled server instances).

## Getting started

```bash
# server
cd server && npm install && npm start        # socket.io + Redis bridge

# client
cd client && npm install && npm run dev       # Next.js app
```

Set the env vars for both sides — see `.env.example`.

## Deploy

**Local / any Docker host** — `cp .env.example .env`, fill in the Upstash creds, then
`docker compose up --build`. Runs Redis (server pub/sub) + the socket.io server (:8080) + the
Next.js client (:3000).

**Render (one blueprint)** — [`render.yaml`](render.yaml): *New → Blueprint → this repo*. It
provisions a Redis (key-value) for the server plus Docker web services for the server and client.
Set in the dashboard: `NEXT_PUBLIC_SOCKET_URL` (→ deployed server URL), `CLIENT_ORIGIN` (→ deployed
client URL), and `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` (a free Upstash instance the
client's server actions use for storage).

CI is provided at [`deploy/ci.yml.example`](deploy/ci.yml.example) — copy it to
`.github/workflows/ci.yml` via the GitHub UI to enable it.

> Socket URL and CORS origin are now env-driven (were hardcoded to localhost).

## Project structure

```
livepolls/
├── client/   # Next.js app: create/join rooms, submit comments, live word cloud
└── server/   # Express + socket.io + Redis pub/sub fan-out
```

## Roadmap

- Auth + presenter mode; more visualization types (bars, live polls) and moderation.
- Prove horizontal scale (multiple server instances over the shared Redis pub/sub) with a load test.
