# Architecture

How the whole thing works, end to end. No code.

Diagram: [architecture.svg](architecture.svg).

## Pieces

```text
┌──────────────────────────────────────────────────────────────┐
│ Browser: Next.js app (Vercel)                               │
│   Reader · Study · Tracker · Library · Leaderboard           │
│   Supabase session · IndexedDB cache                         │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTPS · JSON · JWT · CORS
┌───────────────────────────▼──────────────────────────────────┐
│ API: Express (Vercel Functions)                             │
│   Auth middleware → routers → services → persistence         │
└───────────────────────────┬──────────────────────────────────┘
                            │ Prisma · Supabase admin client
┌───────────────────────────▼──────────────────────────────────┐
│ Supabase                                                     │
│   Postgres · Auth (JWT) · Storage (covers, avatars)          │
└──────────────────────────────────────────────────────────────┘
```

## What each piece does

### Browser
- **Reader** parses and renders EPUBs. Splits the text into words. Caches the book and parsed text in IndexedDB, so reopening is instant.
- **Dictionary** is a bundled Spanish–English corpus with frequency counts. Clicking a word looks it up locally. No server call.
- **Study** runs SM-2 on the client. Each grade produces the next ease and interval, then gets saved.
- **Tracker** logs sessions and shows stats plus a heatmap.
- **Library / decks** manage books and saved words.
- **Leaderboard / profiles** show rankings and public pages.
- **Auth** keeps the Supabase session and sends the token on every API call.

### API
- **Auth middleware** verifies the Bearer token server-side and attaches the user to the request. There's a permissive variant for read-only views.
- **Routers** expose REST endpoints for books, decks, cards, progress, activities, stats, sessions, sync, profiles, users, and uploads.
- **Services** handle gamification (XP, level, badges, streaks), aggregation, and SRS validation.
- **Ownership** is checked on every write. Public reads are a small, explicit set.

### Supabase
- **Postgres** via Prisma. Tables: users, books, decks, cards, activities, progress, badges, word stats, sessions.
- **Auth** issues the JWTs.
- **Storage** holds covers (private) and avatars (public).

## Flows

### Logging a session
1. Tracker posts the activity with the user's token.
2. API checks the token and saves the activity.
3. Gamification runs once: XP from duration, type, and bonuses; aggregate counters; level recompute; badges (idempotent); streak advance.
4. The result shows up on the level bar, profile, and leaderboard.

### Reading a book
1. Upload an EPUB. API saves the metadata. The cover goes to Storage.
2. Browser parses and renders it, splits the words.
3. Click a word → local lookup → panel with meaning, gender, synonyms, frequency. One click saves it to a deck.
4. Reading position is cached locally and can sync.

### Studying
1. Study screen loads a deck and builds a queue of due and new cards.
2. SM-2 runs on the client per grade.
3. API validates and clamps the schedule fields, then saves them.

### Leaderboard and profiles
1. API reads progress rows and merges Supabase user info (name, avatar).
2. Frontend renders the ranking, podium, and profile pages.

## External services

| Service | What for |
|---|---|
| Supabase Auth | accounts, JWTs |
| Supabase Postgres | all app data |
| Supabase Storage | covers, avatars |
| Vercel | hosting frontend + API |
| YouTube captions | subtitle tracks for read-alongs |

No AI/LLM. Dictionary and frequency data ship with the app.

## Boundaries

- **Auth**: the API never stores passwords. It only verifies Supabase tokens.
- **Authorization**: ownership is checked on every write. Public endpoints expose a minimal field set.
- **Integrity**: client-writable fields are whitelisted and clamped. Computed values (XP, level, streak) aren't writable.
- **Portability**: the same API code runs as a plain Node process and as a Vercel Function.
- **Schema**: changes ship as Prisma migrations with backfills. Upgrading a live DB doesn't destroy data.

## Deployment

```text
Vercel              Vercel Functions      Supabase
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│ Next.js app │ ──► │ Express API  │ ──► │ Postgres · Auth   │
└─────────────┘     └──────────────┘     │ · Storage         │
                                         └──────────────────┘
```

- Frontend: Next.js on Vercel.
- API: Express as a Vercel Function.
- DB/Auth/Storage: Supabase.

Trade-offs: cold starts, a ~4.5 MB function body limit (so the avatar cap is 4 MB), and no background workers.

## Decisions

[docs/engineering-decisions.md](../docs/engineering-decisions.md).