# {{project-name}} Chat

A simple real-time chat application built with Ankurah, demonstrating distributed
reactive updates across a Leptos (Rust/WASM) frontend and a Rust backend.

## Features

- Real-time message synchronization
- Persistent storage (Sled on server, IndexedDB in browser)
- Automatic user creation with localStorage persistence
- WebSocket-based peer communication
- Reactive UI updates (Leptos + ankurah-signals)

## Architecture

- **model/** — Shared data models (User, Room, Message)
- **server/** — Rust server with Sled storage and WebSocket connector
- **leptos-app/** — Leptos (CSR) frontend, compiled to WASM with [trunk](https://trunkrs.dev/)

## Quick Start

The easiest way to run everything is the background dev runner. It builds and
supervises the server and the Leptos app on randomized ports and publishes status
files for a [Sutra](https://github.com/synestheticsystems/sutra) dashboard:

```bash
./dev.sh            # start (prints the server + web URLs)
./dev.sh --status   # show status
./dev.sh --logs     # tail combined logs
./dev.sh --stop     # stop
```

### Or run the pieces manually

Requires [trunk](https://trunkrs.dev/) (`cargo install trunk`) and the wasm target
(`rustup target add wasm32-unknown-unknown`).

```bash
# 1. Server (WebSocket backend, Sled storage) — listens on 127.0.0.1:9898 by default
cargo run -p {{project-name}}-server

# 2. Leptos app — compiled to WASM, served by trunk which proxies /ws to the server
cd leptos-app
trunk serve --proxy-backend ws://127.0.0.1:9898/ws --proxy-ws
```

## Models

### User
- `display_name`: String — the user's display name

### Room
- `name`: String — the room name

### Message
- `user`: Ref<User> (LWW) — reference to the sending user
- `room`: Ref<Room> (LWW) — reference to the room
- `text`: String — message content
- `timestamp`: i64 (LWW) — Unix timestamp in milliseconds
- `deleted`: bool (LWW) — soft-delete flag

## Developing against a local Ankurah

`./akdev` swaps the published Ankurah crates for a local checkout or a git branch
(the patch is written to `.cargo/config.toml`, which is gitignored):

```bash
./akdev enable                 # use ../ankurah (local path)
./akdev enable --pr 201        # use a specific ankurah PR (resolved via gh)
./akdev enable --git URL --branch NAME
./akdev disable                # back to published crates
```

## End-to-end tests

```bash
cd e2e
npm install
npm run test:e2e               # picks free ports, runs Playwright (chat + multi-user)
```

## License

MIT or Apache-2.0
