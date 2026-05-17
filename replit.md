# Star Wars: Galaxy of Collectors

A Star Wars character collector RPG where players collect iconic characters into their roster, upgrade them through levels/stars/gear tiers, and assemble squads of exactly 5 to build power.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/swgoh run dev` — run the frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS, wouter routing, TanStack Query
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI contract (source of truth)
- `lib/db/src/schema/` — Drizzle schema: characters, roster, teams, team_members, journey_tiers (with enemySquad jsonb)
- `artifacts/api-server/src/routes/` — characters, roster, teams, journey, admin, campaign, summary routes
- `artifacts/api-server/src/seed.ts` — seeds 25 Star Wars characters on first run; `ensureNewCharacters()` idempotently adds new factions
- `artifacts/swgoh/src/` — React frontend
  - `pages/` — Dashboard, Roster, Characters, Teams, Upgrade, Journey, Campaign, Admin
  - `components/` — Layout (sidebar nav), CharacterCard, StarRating

## Architecture decisions

- **Single-player, no auth**: All roster/team data is global (no user accounts). Easy to add Clerk auth later.
- **Power formula**: `level * 120 + stars * 800 + gearLevel * 500 + level * stars * 15` — recalculated on every upgrade.
- **Seed guard**: `seed.ts` checks if characters already exist before inserting, so it's safe on hot reloads. `ensureNewCharacters()` is idempotent by name check.
- **Team member cascade delete**: deleting a team removes all its team_member rows via FK cascade.
- **Max 5 teams**: enforced in the POST /teams route.
- **Journey battle**: tiers with `enemySquad` configured require a real auto-battle win (via `runBattle`). Tiers without an enemy squad use the legacy `POST /journey/:id/tiers/:tierNumber/complete` flow.
- **Vite fs.strict = false**: required so Vite can serve shared `lib/` workspace packages.

## Product

- **Command Center**: Dashboard with total power, character/squad counts, 7-star count, top 5 leaderboard, squad overview
- **Character Catalog**: Browse all 25 characters, filter by faction/role, unlock any to roster
- **Roster**: View collected characters sorted by power, filter, click to upgrade
- **Squad Builder**: Create/edit/delete up to 5 squads of 5 characters each
- **Upgrade Screen**: Level up (1-85), promote stars (1-7), upgrade gear (G1-G13) per character
- **Campaign**: Planet-based node battles with credit/XP rewards
- **Journey Units**: Tiered legendary-character unlock journeys — tiers with an enemy squad require a squad battle win; tiers without use the direct complete flow. Per-tier admin edit panel (gear icon) lets you configure enemy squad + credit cost inline.

## Characters (25 total)

Original 5: Din Djarin, Grogu, Bo-Katan (Exile), IG-12, The Armorer

New factions added:
- **212th Attack Battalion**: Commander Cody, 212th Clone Trooper, 212th Airborne Trooper, Clone Medic, AT-RT Unit
- **501st Legion**: Anakin Skywalker (501st), Ahsoka Tano (501st), Clone Captain Rex, 501st Trooper
- **Trench Droids**: Admiral Trench, BX Commando Droid, Magnaguard Elite, Tactical Droid
- **Sith Empire**: Darth Vader, Emperor Palpatine, Sith Assassin
- **Old Republic**: Revan, Bastila Shan, Old Republic Soldier
- **Imperial Disguise**: Luke Skywalker (Imperial Disguise)

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Always run `pnpm --filter @workspace/api-spec run codegen` after changing openapi.yaml
- Seed runs on every server start but skips if characters table is populated; `ensureNewCharacters()` always runs to add any missing new characters
- `gear_level` is the SQL column name; `gearLevel` is the JS/TS key in Drizzle
- Journey admin route: `PUT /api/admin/journey/tiers/:tierId` (mounted at `/admin`)
- Journey play route: `POST /api/journey/:id/tiers/:tierNumber/play`
- `enemySquad` is a jsonb column in `journey_tiers` (array of `{ characterId, level?, stars? }`)

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
