# Project Structure

How it's organized. This is the logical layout, not the real file tree.

```text
Botsu

Frontend (Next.js)
├─ Pages / Routes
│   ├─ Landing
│   ├─ Auth            sign up, sign in, email confirmation
│   ├─ Reader          EPUB reader + dictionary
│   ├─ Library         books, upload, progress
│   ├─ Decks / Study   cards + SM-2 review
│   ├─ Tracker         session logging, stats, heatmap
│   ├─ Progress        XP, level, badges
│   ├─ Leaderboard     rankings + podium
│   └─ Profiles        public pages
├─ UI / Design
│   ├─ Tokens          colors, spacing, radii, type
│   ├─ Components      buttons, cards, bars, badges
│   └─ Mascot          character + motion
├─ Client Libs
│   ├─ Supabase client
│   ├─ API client      one base URL, authenticated fetch
│   ├─ EPUB engine     parse, render, split words
│   ├─ SRS             SM-2
│   └─ IndexedDB       book and progress cache
└─ Config              env, framework, lint

Backend (Express API)
├─ API Layer
│   ├─ Books           metadata, covers, reading progress
│   ├─ Decks / Cards   decks, cards, bulk import, SRS fields
│   ├─ Progress        read-only progress projection
│   ├─ Activities      create and list sessions
│   ├─ Stats           aggregated summary
│   ├─ Sync            books and cards → activities
│   ├─ Gamification    badges, levels, leaderboard
│   ├─ Profiles/Users  public profiles, edits, avatars
│   └─ Uploads         image uploads
├─ Auth
│   ├─ verifyAuth          required
│   └─ verifyOptionalAuth  read-only views
├─ Services
│   ├─ Gamification    XP, level, badges, streaks, aggregates
│   ├─ Validation      field whitelists + SRS clamping
│   └─ Leveling        the one XP→level curve
├─ Persistence
│   ├─ Prisma models   users, books, decks, cards, activities,
│   │                  progress, badges, word stats, sessions
│   └─ Migrations      schema changes + data backfills
└─ Integrations
    ├─ Supabase Auth
    └─ Supabase Storage

Infra (managed)
├─ Vercel      Next.js app + API functions
└─ Supabase    Postgres · Auth · Storage
```

## Notes

- The browser does the interactive work: parsing EPUBs, splitting words, scheduling reviews.
- The API stays thin and strict. It owns persistence, gamification, and integrity.
- XP and level read from one place, so the tracker, profiles, and leaderboard always agree.
- This repo is the description, not the source.