# Technical Overview

What happens from a click to a stored result. No source code.

## Signup
Email + password, optional avatar. Supabase handles identity and hands back a JWT. The browser keeps it and sends it as `Bearer` on every API call. The backend verifies the same token server-side, then saves the profile.

## Library
You upload an EPUB. The API saves the metadata and puts the cover in Supabase Storage. The book gets parsed in the browser (text split into words) and cached in IndexedDB, so it opens instantly next time.

## Reading
The reader renders the book and makes every word clickable. Clicking looks the word up in the bundled dictionary: part of speech, glosses, gender, synonyms, etymology, frequency. No network. One more click saves it to a deck.

## Reviewing
Decks open in an SM-2 review flow: again / hard / good / easy. The scheduler runs on the client so it feels instant. The result (ease, interval, next review) gets saved, and the API checks and clamps it first, so bad values can't get in.

## Tracking
You log a session: reading, watching, listening, or study, with duration, words, episodes, finished. The API saves it and runs gamification in one pass:
- XP from duration × type multiplier + bonuses.
- Aggregate counters: words read, books finished, words learned.
- Level recompute and badges (awarded idempotently).
- Streak advance.

## Progression
The same progress powers the level bar, totals, streak, leaderboard, and public profiles. Profiles are read-only and computed per request.

## Where things run

| Thing | Where |
|---|---|
| EPUB parse, word split, SRS | browser |
| Auth check, validation, gamification | API |
| Users, books, decks, cards, activities, progress | Postgres |
| Book cache, reading position | IndexedDB |
| Covers, avatars | Supabase Storage |

## One line

Browser (Next.js) → HTTPS + JWT → Express API → Prisma → Postgres. The API also talks to Supabase Auth and Storage.

## Not doing

- No server-side dictionary calls. It's bundled.
- No custom passwords or sessions. Supabase Auth only.
- No background jobs. Everything is a normal request.