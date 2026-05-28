# API & Server Actions design

Geldt voor zowel Next.js route handlers (`route.ts`) als Server Actions
(`"use server"`). Consistency = onderhoudbaar + voorspelbaar voor frontend.

## 1. HTTP status codes

### Wat te checken
- **`200 OK`** — succesvolle GET/PUT/PATCH.
- **`201 Created`** — succesvolle POST die een resource maakt; include `Location` header met de URL.
- **`204 No Content`** — succesvolle DELETE / actie zonder body.
- **`400 Bad Request`** — invalid input (validation errors).
- **`401 Unauthorized`** — niet ingelogd (NB: betekent NIET "geen rechten").
- **`403 Forbidden`** — ingelogd maar geen rechten op deze resource.
- **`404 Not Found`** — resource bestaat niet (of mag user niet zien — voorkomt enumeration).
- **`409 Conflict`** — state-conflict (bv. dubbele entry, version mismatch).
- **`422 Unprocessable Entity`** — semantisch onjuist (validatie failed).
- **`429 Too Many Requests`** — rate limited.
- **`500`** — alleen bij echte server-side bugs, niet bij verwachte fouten.

### Anti-patronen
- **Altijd 200 met `{ error: "..." }`** — frontend kan niet uniform met `response.ok` werken.
- **200 met `error` in body voor validatie** — gebruik 400/422.
- **500 voor "not found"** — gebruik 404.

## 2. Error response shape (consistent!)

### Wat te checken
- **Eén vorm** door hele API: bv. `{ error: { code: string, message: string, details?: any } }`.
- **`message` in Nederlands** (voor gebruiker tonen), **`code` machine-leesbaar** (voor switches).
- **Field-errors** voor forms: `details: { field: "email", message: "..." }` of array.
- **Geen stack traces in productie responses** (security leak).

## 3. Server Action response shape

### Wat te checken
- Server actions kunnen niet "throwen" zoals route handlers. Gebruik return-shape:
  ```ts
  type ActionResult<T> =
    | { success: true; data: T }
    | { success: false; error: string; fieldErrors?: Record<string, string> }
  ```
- **Consistente shape** door alle actions heen.
- **Niet alleen `void` returnen** — frontend kan dan niet zien of het werkte.

## 4. Pagination

### Wat te checken
- **Cursor-based** voor grote lijsten (oneindig scroll): `{ items, nextCursor }`. Stabieler dan offset bij wijzigingen.
- **Offset-based** OK voor tabel-paginering met page numbers: `{ items, total, page, pageSize }`.
- **Default page size** redelijk (10-50), **max page size** afgedwongen (anders DoS via `pageSize=999999`).
- **Filters/sortering** in query params, niet body op een GET.

## 5. Idempotency

### Wat te checken
- **GET/PUT/DELETE** moeten idempotent zijn — meerdere keren uitvoeren = zelfde resultaat.
- **POST** is meestal NIET idempotent (bv. payment, bestelling) — gebruik `Idempotency-Key` header om dubbele-clicks op te vangen.
- Server actions die mail versturen / bestellen: dubbele submit-protectie.

## 6. Authentication & authorization

### Wat te checken
- **Auth-check op elk endpoint** — geen "vergeten" admin-routes.
- **Authorization (RBAC)** — heeft user de rol/permissie voor deze actie?
- **Resource-level auth (IDOR)** — heeft user toegang tot juist déze employee/organization/document?
- **CSRF**: server actions doen het automatisch (origin-check). Route handlers met cookie-auth → gebruik double-submit cookie of SameSite.
- **Token type**: ben je session-cookie of bearer-token verwachten? Niet beide door elkaar.

## 7. Input validation

### Wat te checken
- **zod schema** (of vergelijkbaar) op elke input — voor zowel route handlers als server actions.
- **Schema gedeeld** tussen frontend en backend (single source of truth).
- **Coercion** waar nodig: `z.coerce.number()` voor query-strings die binnenkomen als string.
- **Max lengths** afgedwongen — geen 10MB strings naar DB sturen.
- **Sanitize HTML** als input ergens als HTML wordt weergegeven (DOMPurify).

## 8. Rate limiting

### Wat te checken
- **Per-user / per-IP rate limits** op publieke endpoints (login, password-reset, magic-link request).
- **Burst vs sustained** beleid.
- **429 response** met `Retry-After` header.
- Upstash, Redis, of in-memory rate limiter.

## 9. CORS

### Wat te checken
- **Whitelist** specifieke origins — geen `Access-Control-Allow-Origin: *` voor authenticated endpoints.
- **Credentials**: `Access-Control-Allow-Credentials: true` alleen waar nodig.
- Next.js: standaard same-origin, alleen problematisch als je een externe SPA toestaat.

## 10. Versioning

### Wat te checken
- **Breaking changes** in publieke API → versie bumpen (`/api/v2/...`) of nieuw veld toevoegen i.p.v. wijzigen.
- **Interne API's** (server actions, alleen door eigen frontend gebruikt) → minder strikt, refactor vrij.

## 11. Response shapes

### Wat te checken
- **Geen lege strings i.p.v. null** — verwarrend (`""` is een waarde, `null` is "geen waarde").
- **Dates als ISO 8601 strings** in JSON (`"2026-05-28T13:00:00Z"`), niet Unix-timestamps zonder uitleg.
- **Snake_case vs camelCase**: kies één + houd consistent. (TS frontend → camelCase.)
- **Booleans heten `isX`/`hasX`** — niet `disabled` (verwarrend bij negatie).
- **Enums** liever als strings dan ints (leesbaar in DB/logs).

## 12. Pagination/list semantics

### Wat te checken
- **`total_count`** is duur (vereist COUNT query) — overweeg `has_more` boolean.
- **Sortering** stabiel — voeg secundaire sort op `id` toe als primaire kan ties hebben.
- **Empty array**, niet `null` voor "geen results".

## 13. Caching headers

### Wat te checken
- **`Cache-Control: private, no-cache`** voor user-specific data.
- **`Cache-Control: public, max-age=3600`** voor statische resources.
- **ETags / `Last-Modified`** voor conditional GETs op grote payloads.

## 14. Webhooks (waar van toepassing)

### Wat te checken
- **Signature verification** (HMAC met secret) op inkomende webhooks.
- **Idempotency** — dezelfde webhook kan meerdere keren binnenkomen.
- **Async processing** — webhook returnen 200 binnen X seconden, doe werk in background.

## Wat GEEN issue is
- Interne tooling waar je beide kanten controleert (refactor vrij).
- Een proof-of-concept endpoint.
- Server actions waarvan alle callers bekend zijn (geen API-contract).

## Severity
- **BLOCKING**: missing auth-check, no input validation, stack traces in 500-response, IDOR.
- **SHOULD FIX**: inconsistente error shape, missing rate limit op login, geen idempotency op critical POST.
- **NICE TO HAVE**: betere pagination, schoonmaakbare enum-naming.

## Output
Bij elke finding: noem de conventie + waarom de huidige aanpak problemen geeft + concrete fix.
