# Database & migrations

## Wat te checken bij migraties (Prisma/SQL files)
- **Destructieve operaties**: `DROP COLUMN`, `DROP TABLE`, `ALTER TYPE` — kost dit data?
- **`NOT NULL` toevoegen aan bestaande kolom** zonder default of backfill — breekt op productie als rijen bestaan.
- **`UNIQUE` constraint** toevoegen zonder eerst te checken of bestaande data uniek is.
- **Indexes op grote tabellen** zonder `CONCURRENTLY` (Postgres) — kan tabel locken.
- **Renames** van kolommen/tabellen — breekt elke app instance die nog oude versie draait (rolling deploy).
- **Foreign keys zonder `ON DELETE` policy** — wat gebeurt er bij cascading delete?
- **Migraties + data backfill in dezelfde transactie** op grote tabellen → time-outs.

## Wat te checken bij queries
- **N+1 queries**: loop met `findUnique`/`findFirst` binnen — gebruik `findMany` met `where: { id: { in: [...] } }`.
- **`include`/`select`** die hele tabellen meeladen waar maar 1 veld nodig is.
- **Missing tenant/organization filter** in een multi-tenant context — IDOR risico.
- **Raw SQL** zonder parameterization.
- **Transacties** rondom meerdere writes die atomic moeten zijn.

## Wat GEEN issue is
- Migraties die alleen iets toevoegen (nieuwe kolom met default, nieuwe tabel)
- Performance-zorgen op tabellen die aantoonbaar klein zijn

## Severity
- Destructieve migratie zonder explicit acknowledgment = **Blocking**.
- Missing tenant filter = **Blocking**.
- N+1 in user-facing endpoint = **Should fix**.
