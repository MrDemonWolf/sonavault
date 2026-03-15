# SonaVault — Claude Context

## Project Overview
SonaVault is a self-hosted fursona vault app for managing sona profiles, artwork, and artist attribution.

**Owner:** Nathanial / MrDemonWolf, Inc.

---

## Stack

| Concern | Solution |
|---|---|
| Frontend | Next.js (standalone output for Docker) |
| Backend | Hono on Node.js |
| API Layer | tRPC |
| Auth | Better Auth |
| Database | Postgres via self-hosted Supabase on Coolify |
| File Storage | Supabase Storage (S3-compatible, self-hosted) |
| ORM | Drizzle ORM |
| Monorepo | Turborepo + pnpm workspaces |
| Docs | FumaDocs |
| Primary Deploy | Coolify |
| Generic Deploy | Docker Compose |

---

## Monorepo Structure

```
sonavault/
├── apps/
│   ├── web/        # Next.js frontend
│   └── api/        # Hono backend
├── packages/
│   ├── db/         # Drizzle ORM schemas + client
│   ├── auth/       # Better Auth config
│   └── ui/         # Shared shadcn/ui components
├── turbo.json
├── pnpm-workspace.yaml
├── docker-compose.yaml
└── .env.example
```

---

## Development

- Use `pnpm` — never npm or yarn
- Run `pnpm dev` from the repo root to start all apps via Turborepo
- TypeScript throughout — no plain JS files

---

## Environment Variables

| Variable | Service | Description |
|---|---|---|
| `DATABASE_URL` | api | Supabase Postgres connection string |
| `SUPABASE_URL` | api | Self-hosted Supabase URL (e.g. `https://supabase.yourdomain.com`) |
| `SUPABASE_ANON_KEY` | api | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | api | Supabase service role key (server-only) |
| `BETTER_AUTH_SECRET` | api | Secret for Better Auth session signing |
| `BETTER_AUTH_URL` | api | Public URL of the API service |
| `NEXT_PUBLIC_API_URL` | web | Public URL of the Hono API |
| `MAX_STORAGE_GB` | api | Per-user storage warning threshold (default: `10`) |

---

## Supabase Storage

- Primary bucket: `sonavault-artwork` (private — serve via signed URLs, 1hr TTL)
- Public bucket: `sonavault-public` (SFW artwork for public sona profiles only)
- Object path pattern: `{userId}/{sonaId}/{uuid}.{ext}`
- Allowed types: `jpeg`, `png`, `webp`, `gif` — max 20 MB per file
- Upload endpoint: `POST /api/upload` (multipart/form-data, auth required)

---

## Database Schema (Drizzle)

```
users             — id, email, username, createdAt, updatedAt
sessions          — Better Auth managed
accounts          — Better Auth managed
verifications     — Better Auth managed
user_preferences  — id, userId, safeMode (bool, default true), theme (enum: dark/light, default dark)
sonas             — id, userId, name, species, description, tags (text[]), isPublic (bool), slug, createdAt, updatedAt
artwork           — id, sonaId, userId, artistId (nullable FK), title, storagePath, contentRating (enum: sfw/nsfw), mimeType, fileSizeBytes, uploadedAt
artists           — id, userId, name, notes, isFavorite (bool, default false), createdAt, updatedAt
artist_handles    — id, artistId, platformId, handle, url
platforms         — id, userId, name, urlPattern, iconSlug, createdAt
tags              — id, userId, name (unique per user)
sona_tags         — sonaId, tagId
artwork_tags      — artworkId, tagId
post_logs         — id, artworkId, platformId, userId, postedAt, postUrl (nullable); unique(artworkId, platformId)
collections       — id, userId, name, description, createdAt, updatedAt
collection_items  — id, collectionId, artworkId, addedAt; unique(collectionId, artworkId)
```

---

## tRPC Routers

| Router | Procedures |
|---|---|
| `auth` | `register`, `login`, `logout` |
| `sona` | `create`, `list`, `getById`, `update`, `delete`, `setPublic`, `updateSlug` |
| `artwork` | `create`, `listBySona`, `getSignedUrl`, `assignArtist`, `delete` |
| `artwork` (Phase 4+) | `list` (full filters: sonaId, artistId, tagIds, contentRating, dateFrom, dateTo, sortBy) |
| `artist` | `create`, `list`, `getById`, `update`, `delete`, `toggleFavorite` |
| `platform` | `list`, `create`, `update`, `delete` |
| `tag` | `list`, `create`, `delete` (if unused) |
| `postLog` | `add`, `listByArtwork`, `delete`, `dashboardSummary` |
| `collection` | `create`, `list`, `getById`, `update`, `delete`, `addArtwork`, `removeArtwork` |
| `preferences` | `get`, `setSafeMode`, `setTheme` |
| `settings` | `updateUsername`, `changePassword` |
| `storage` | `getUsage` (cached 5min) |
| `search` | `query(q)` — grouped results: sonas, artwork, artists |

---

## App Routes (Next.js)

```
/                           → redirect to /dashboard or /login
/login                      → public
/register                   → public
/dashboard                  → home, posting summary widget
/dashboard/sonas            → sona list
/dashboard/sonas/[id]       → sona detail + artwork gallery
/dashboard/artists          → artist directory
/dashboard/collections      → collections list
/dashboard/collections/[id] → collection detail
/dashboard/search           → full-text search results
/dashboard/settings         → account, storage, platform management, danger zone
/u/[username]/[slug]        → public sona gallery (no auth, SFW only)
/art/[id]                   → public artwork share page (if sona isPublic + SFW)
```

- Middleware protects all `/dashboard/*` routes — redirect unauthenticated to `/login`
- Session user is passed into tRPC context on the Hono backend

---

## API Routes (Hono)

```
POST /api/upload            → multipart upload to Supabase Storage
GET  /api/export            → full data export as JSON (auth required)
GET  /api/health            → health check
/api/auth/**                → Better Auth handler (GET + POST)
/api/trpc/**                → tRPC handler
```

---

## Platform Seeds (on first user auth)

FurAffinity, Twitter/X, Bluesky, DeviantArt, Inkbunny, SoFurry, Weasyl

---

## Design Tokens

| Token | Dark (default) | Light |
|---|---|---|
| `--color-bg` | `#091533` | `#EEF5FF` |
| `--color-accent` | `#0FACED` | `#0088AA` |
| `--color-text` | `#ffffff` | `#091533` |
| `--color-card` | `#0d1f45` | `#ffffff` |

- Theme applied via `[data-theme]` on `<html>` — all components use CSS variables, no hardcoded colors
- Default theme: dark

---

## Key Implementation Notes

- **Scaffold command:** `pnpm create better-t-stack@latest sonavault --frontend next --backend self --runtime none --api trpc --auth better-auth --payments none --database postgres --orm drizzle --db-setup supabase --package-manager pnpm --no-git --web-deploy none --server-deploy none --no-install --addons fumadocs pwa turborepo --examples none`
- **Next.js:** set `output: 'standalone'` in `next.config.ts` for Docker
- **Dockerfiles:** multi-stage (deps → builder → runner) using `node:20-alpine`; expose `PORT` env var
- **Safe mode filter:** `WHERE contentRating = 'sfw'` when `safeMode = true`; NSFW items show blurred thumbnail + badge, no signed URL fetched
- **Repost warning:** `postLog.add` returns a warning payload instead of inserting if `(artworkId, platformId)` already exists
- **Cascade deletes:** deleting a sona removes artwork DB rows + Supabase Storage objects; deleting an artist removes artist_handles only
- **Public sona:** copy SFW objects to `sonavault-public` bucket on `setPublic(true)`; remove on `setPublic(false)`
- **Export format:** `{ exportedAt, user, sonas, artwork (with storagePath), artists, platforms, postLogs, collections }`
- **Storage usage:** `SUM(fileSizeBytes)` from artwork table — not live Supabase API call
- **Search:** Postgres `tsvector` generated columns on sonas (name + species + description) and artwork (title)
- **Coolify:** two app services (web + api) + separate self-hosted Supabase service; run migrations via Coolify one-off command `pnpm db:migrate`

---

## Phase Roadmap

### Phase 1 — MVP (Core Vault) ← current focus
- Project scaffold: Turborepo monorepo, Supabase Postgres + Storage, Docker + Coolify setup
- Auth: Better Auth with Postgres adapter — register, login, session, protected routes
- Sona Management: create / edit / delete fursona profiles (name, species, description, tags)
- Artwork Upload: upload images to Supabase Storage, attach to a sona, basic metadata
- SFW / NSFW Toggle: content rating per artwork; NSFW hidden by default behind global safe-mode toggle
- Basic Gallery View: grid of artwork per sona, lightbox viewer using Supabase Storage public URLs
- Settings — Account: change username, change password, storage usage, export data, danger zone

### Phase 2 — Artist Attribution
- Artist profiles with social media handles (FA, Twitter, Bluesky, etc.)
- Link artwork → artist; artist directory with search and favorites
- Platform management in settings (seeded defaults + custom)

### Phase 3 — Posting Tracker
- Post log per artwork per platform (date + optional URL)
- Repost warning on duplicate platform; dashboard posting summary widget

### Phase 4 — Organization & Search
- Normalized tags on sonas and artwork; Postgres FTS across sonas/artwork/artists
- Gallery filters (artist, tag, rating, date); collections/albums; bulk actions

### Phase 5 — Sharing & Public Profiles
- Per-sona `isPublic` + `slug`; public gallery at `/u/[username]/[slug]` (SFW only)
- Shareable artwork links; OG meta tags for Discord/social previews

### Phase 6 — Polish & Production Hardening
- Light/dark theme (CSS variables, persisted in Postgres); PWA (manifest + service worker)
- Storage dashboard with configurable warning threshold; manual JSON export + restore guide
- Danger zone (typed DELETE confirmation); FumaDocs docs site
- Coolify first-class deploy guide + generic Docker Compose guide; changelog (semver)
