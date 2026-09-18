# Engineering Decisions

Real choices, and why. Short.

## 1. Free tiers: Vercel + Vercel Functions + Supabase
**Decision**: Frontend and API on Vercel. Postgres, Auth, and Storage on Supabase.
**Context**: No budget. Wanted something that looks like a real setup, not a toy.
**Options**: VPS (control, but patching, backups, TLS, a bill). Paid PaaS (needs a card). Free tiers.
**Chose**: Vercel free + Supabase free.
**Why**: No card, no ops for DB/Auth/Storage, and a realistic split between a public tier and a serverless API.
**Trade-offs**: Cold starts. A ~4.5 MB body limit, so avatars cap at 4 MB. No background workers.

## 2. Stateless auth via Supabase
**Decision**: The API verifies Supabase JWTs. No sessions or passwords of our own.
**Context**: Needed secure auth without running auth.
**Options**: Custom password store, self-signed JWT, managed provider.
**Chose**: Supabase Auth.
**Why**: No password storage to get wrong. One provider. Ownership checks become `resource.userId === user.id`.
**Trade-offs**: Tied to a provider. The canonical path is the Bearer header; the cookie fallback is dead cross-origin.

## 3. SM-2 on the client, validated on the server
**Decision**: The browser computes the schedule. The API checks and clamps it.
**Context**: Reviews are interactive. Round-tripping every grade is slow.
**Options**: Server-authoritative SRS, trust the client, or client-compute plus server-validate.
**Chose**: Client computes, server validates.
**Why**: Fast UX, and bad values can't corrupt the model.
**Trade-offs**: A user can pick favorable-but-valid numbers. It's their own data, so that's an accepted limit.

## 4. One level curve
**Decision**: `100 * n^2`, in one shared module.
**Context**: Two formulas existed, so the same user saw different levels on different pages.
**Options**: Keep both, pick one per layer, or extract one.
**Chose**: Extract one.
**Why**: Level shows up everywhere. It can't disagree with itself.
**Trade-offs**: Changing the curve later is a one-file change.

## 5. One pass owns the counters
**Decision**: XP, aggregates, levels, badges, and streaks all update in one server pass.
**Context**: The route and the gamification service both bumped counters. Stats doubled.
**Options**: Compensate, tag idempotently, or consolidate.
**Chose**: Consolidate.
**Why**: "Who updates XP?" has one answer.
**Trade-offs**: Activity creation does a bit more work. Worth it.

## 6. Store the streak
**Decision**: `currentStreak` is a real column.
**Context**: "Longest streak" never grew. Deriving streaks from dates was fragile.
**Options**: Derive on read, or store a counter.
**Chose**: Store it, and backfill with a migration.
**Why**: Simple to reason about and test.
**Trade-offs**: Redundant state, but one owner keeps it consistent.

## 7. Migrations with backfills
**Decision**: Schema changes ship as Prisma migrations that backfill data.
**Context**: The schema referenced a `User` table the migrations never created. Adding the foreign key to a live DB would fail.
**Options**: `db push`, reseed, or an authored migration.
**Chose**: Authored migration: create, backfill, then add the FK.
**Why**: `migrate deploy` works on empty and populated databases.
**Trade-offs**: Hand-written SQL, so review it carefully.

## 8. Whitelist inputs
**Decision**: Every writable field is whitelisted, type-checked, and clamped.
**Context**: "Update with whatever the client sent" is a mass-assignment hole.
**Options**: Trust the client, blocklist, or allowlist.
**Chose**: Allowlist.
**Why**: Self-documenting, and hard to widen by accident.
**Trade-offs**: New fields need an explicit change. That's the point.

## 9. Normalize the base URL and CORS origin
**Decision**: Strip trailing slashes on both ends.
**Context**: A trailing slash made `//api/...` URLs. Vercel 308-redirects those, and a preflight can't follow a redirect, so the API looked dead in the browser but worked in curl.
**Options**: Rely on config, document it, or enforce it.
**Chose**: Enforce it in code.
**Why**: The failure was silent and confusing.
**Trade-offs**: A tiny bit of indirection.

## 10. One Supabase client, one auth pass
**Decision**: One shared admin client. `verifyAuth` runs once per request.
**Context**: Several modules made their own client, and auth ran at the mount and again in the route. Two Supabase calls per request.
**Options**: Leave it, or refactor.
**Chose**: Refactor.
**Why**: Half the auth calls, and one place to audit.
**Trade-offs**: A shared singleton. It's a stateless HTTP client, so it's fine.

## The one rule
Own each mutable value in one place. Validate at the boundary. Let the deployment shape the design.