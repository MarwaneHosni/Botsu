# Botsu

Learn Spanish by reading real books.

You open a Spanish book, click any word, and see what it means. Words you don't know go to flashcards with one click. Later you review them with spaced repetition. Everything you do (reading, watching, listening, studying) earns XP, levels, badges, and a streak.

## Why

Most language tools make you juggle four apps: a reader, a dictionary, a flashcard app, a habit tracker. They don't talk to each other. You learn a word while reading and never see it again.

Botsu puts it all in one place.

## Features

- **Reader**: reads EPUBs in the browser.
- **Click to define**: tap a word and get the meaning, part of speech, gender, synonyms, and how common it is. The dictionary ships with the app, so there's no server round-trip.
- **Save to deck**: send a word straight to a flashcard deck. No duplicates.
- **Spaced repetition**: review decks with an Anki-style SM-2 scheduler.
- **Immersion tracker**: log reading, watching, listening, and study sessions. See them on a heatmap.
- **Gamification**: XP, levels, badges, streaks.
- **Leaderboard and profiles**: see where you rank, share a public profile.
- **Library**: manage your EPUBs and reading progress.
- **YouTube subtitles**: pull subtitles from videos to read along.
- **Auth**: email/password via Supabase. You only ever touch your own data.

## Architecture

```text
Browser (Next.js)
  ├─ Reader      EPUB parse + IndexedDB cache
  ├─ Study       SM-2 scheduling
  └─ Tracker / Progress / Leaderboard
        │ JSON + JWT (CORS)
        ▼
API (Express on Vercel Functions)
  ├─ Auth check (Supabase JWT)
  ├─ Gamification, leveling, validation
  └─ Routers: books, decks, cards, activities, profiles, uploads
        │ Prisma
        ▼
Supabase: Postgres + Auth + Storage
```

More: [architecture/architecture.md](architecture/architecture.md). Diagram: [architecture/architecture.svg](architecture/architecture.svg).

## Stack

| Layer | Tech |
|---|---|
| Frontend | Next.js (App Router), React, JS/JSX, Tailwind, lucide-react, epubjs, idb-keyval, Supabase JS |
| Backend | Node, Express, Prisma, Supabase admin SDK, helmet, cors, formidable |
| Database | PostgreSQL (Supabase), Prisma migrations |
| Auth | Supabase Auth. Email/password, JWT verified server-side |
| Storage | Supabase Storage. Private covers, public avatars |
| SRS | SM-2, computed in the browser, checked on the server |
| Dictionary | Bundled Spanish–English corpus with frequency data |
| Hosting | Vercel (frontend + API functions), Supabase (DB/Auth/Storage) |

No AI or LLM. The dictionary ships with the app, so the core features work offline and give the same result every time.

## Engineering notes

- **The browser does the heavy work.** EPUB parsing, splitting text into words, and review scheduling all run client-side. Reading stays fast.
- **One pass updates everything.** XP, counters, level-ups, badges, and streaks are computed in a single server pass after an activity is saved. Nothing gets counted twice.
- **One level formula.** The curve lives in one shared module. The gamification engine, the profile API, and the frontend bars all use it, so they can't disagree.
- **Streaks are stored, not guessed.** `currentStreak` is its own column. It goes up or resets per activity, and the longest streak follows from it.
- **Everything a user writes is whitelisted.** Field by field, type-checked, values clamped. Anything computed (XP, level, streak) can't be written by the client.
- **Auth is stateless.** The API only trusts Supabase JWTs. No password storage, no sessions of our own.
- **Runs on free tiers.** Frontend and API on Vercel. Postgres/Auth/Storage on Supabase. Cold starts on the API are the trade-off.

## Layout

```text
Frontend
├─ Pages      landing, auth, reader, library, study, tracker, progress, leaderboard, profile
├─ UI         components, design tokens, mascot
├─ Lib        supabase client, api client, epub engine, SRS, IndexedDB
└─ Config

Backend
├─ Routes     books, decks, cards, progress, activities, stats, sync, gamify, profile, users, uploads
├─ Auth       JWT verify + optional-auth
├─ Services   gamification, leveling, validation
├─ Prisma     models + migrations
└─ Integrations  Supabase Auth + Storage
```

No source tree here. This is a description of it.

## Decisions

Write-ups: [docs/engineering-decisions.md](docs/engineering-decisions.md).

## Problems I hit

1. **Stats counted twice.** The activity route and the gamification service both bumped the same counters. Fixed by giving the gamification service sole ownership.
2. **Longest streak never moved.** It was recomputed in a way that did nothing. Fixed by storing `currentStreak` and backfilling it with a migration.
3. **Two level formulas.** Different pages showed different levels for the same XP. Fixed by pulling the curve into one module.
4. **A route let users write their own XP.** `PATCH /progress` accepted anything. Now it rejects server-owned fields.
5. **Signup was broken in three places.** Avatar upload ran before the token existed, the profile save hit a route that didn't exist, and the session call pointed at the wrong host. Reordered and fixed.
6. **`//api/...` URLs.** A trailing slash in config made double-slash URLs. A CORS preflight can't follow Vercel's 308 redirect, so the API looked dead in the browser. Normalized the base URL and the CORS origin on both ends.
7. **Schema drift.** The schema referenced a `User` table the migrations never created. Wrote a migration that creates it and backfills rows before adding the foreign key.

## Status

- Works end to end: signup, upload a book, read with definitions, build decks, study, track, leaderboard, profiles.
- Live on Vercel + Supabase, free tier.
- This repo is docs only.

## Demo

Public deployment and notes: [demo/README.md](demo/README.md).

## Running it

There's no source here, so there's nothing to install. Use the demo.

## Source

Not included. This repo is for the write-up and the demo.

## License

MIT. See [LICENSE](LICENSE).