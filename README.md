# What's the Word

A daily one-word journaling app with a social layer. Each day you answer a
single prompt ("What's one word to describe your mood?") with one word —
words stack up into your personal word cloud on your profile, and public
words flow into a shared global word cloud. There's a 20-submissions-per-day
cap enforced by the server.

Stack:

- **client/** — React 19.2 + TypeScript + Vite 7 + Tailwind CSS (v3). Routes:
  `/` home (daily prompt + word entry), `/global` (global word cloud),
  `/profile` (your cloud + stats).
- **server/** — SpacetimeDB Rust module (`server/src/lib.rs`) with tables
  `User`, `WordEntry`, `DailySubmission` and the `submit_word` reducer.

> Note: the client currently ships with a **mock backend** in
> `client/src/spacetime.ts` (localStorage-based, marked as a mock at the top
> of the file) so it runs with no backend at all. To go live, point the
> client at the SpacetimeDB module published below and regenerate real client
> bindings with `spacetime generate`.

## Client setup

Requires Node.js — see `.nvmrc` (pinned to Node 22).

```sh
cd client
npm install        # or: npm ci
npm run dev        # vite dev server, default http://localhost:5173
npm run lint       # eslint
npm run build      # tsc -b && vite build → client/dist
```

## SpacetimeDB server setup

Pinned CLI version for this repo: **2.10.2** (the 2.x line). If you upgrade,
update this pin.

```sh
# 1. Install the CLI, then pin the repo's version
curl -sSf https://install.spacetimedb.com | sh
spacetime version install 2.10.2
spacetime version use 2.10.2

# 2. Start a local node and scaffold/verify the module
spacetime start
cd server
spacetime build          # compiles the Rust module to WASM

# 3. Publish (local dev or Maincloud)
spacetime publish whats-the-word
# Maincloud: spacetime login, then
#   spacetime publish --server maincloud whats-the-word

# 4. Regenerate the TypeScript client bindings and drop them into the client
spacetime generate --lang typescript --out-dir ../client/src/module_bindings
```

After that, wire `client/src/spacetime.ts` to the generated bindings
(instead of the mock) and set the database host/name in the client.

## CI

`.github/workflows/ci.yml` runs the client pipeline on every push/PR
(Node 22 → `npm ci` → `npm run lint` → `npm run build`). The server module is
not built in CI (the SpacetimeDB CLI is not installed on GitHub runners);
run `spacetime build` in `server/` locally before publishing.
