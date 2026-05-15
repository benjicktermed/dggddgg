# Adonis HuB

A Roblox-themed access key management system with device binding, admin panel, and Roblox API integration.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/astrix run dev` — run the frontend (auto-port)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Optional env: `DISCORD_WEBHOOK_URL` — Discord webhook for key event notifications
- Optional env: `ADMIN_SECRET` — Secret to protect admin API endpoints

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS v4 (`artifacts/astrix`)
- API: Express 5 (`artifacts/api-server`)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/astrix/src/` — React frontend (login, home, admin pages)
- `artifacts/astrix/src/context/AppContext.tsx` — global state (session, profile, game)
- `artifacts/api-server/src/routes/keys.ts` — key verify/create/list/delete/reset-hwid
- `artifacts/api-server/src/routes/roblox.ts` — Roblox API proxy (search, user, friends, games, avatar)
- `artifacts/api-server/src/lib/discord.ts` — Discord webhook notifications
- `lib/db/src/schema/keys.ts` — DB schema (keys, key_hwids tables)
- `attached_assets/` — image assets used by the frontend

## Architecture decisions

- Keys use HWID binding: first device to use a key locks it to that device
- Admin PIN is hardcoded in the frontend (`021114`) for zero-config setup; backend optionally protects create/list/delete with `ADMIN_SECRET` env var
- Roblox API is proxied through the backend to avoid CORS and enable caching (TTL cache per endpoint)
- Avatar images are re-proxied through `/api/roblox/avatar` to bypass Roblox CDN CORS restrictions
- Balance is client-side only (cosmetic display)

## Product

- **Login**: Enter an access key → verified against DB + HWID bound to device
- **Home**: Roblox-style dashboard with balance display, package browser, FAQ, and send Robux modal
- **Admin**: PIN-gated panel (PIN: `021114`) to create, view, reset HWID, and delete keys
- **Settings**: Set Roblox username, fetch profile, pick a game to display

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Run `pnpm --filter @workspace/db run push` after any schema change
- The frontend uses `@assets/*` alias → `../../attached_assets/` (Vite only; also in tsconfig paths)
- ADMIN_SECRET env var is optional — if not set, admin API endpoints are publicly accessible (protected only by the frontend PIN gate)

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
