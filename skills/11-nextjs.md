# Next.js (App Router)

Specifiek voor de Next.js App Router en server-/client-componenten. Aanvullend
op de algemene `react.md` skill — die blijft gelden, dit gaat over Next-only
valkuilen.

## 1. Server vs Client components

### Wat te checken
- **`"use client"` per ongeluk vergeten** op een component met state/hooks/event handlers → build error.
- **`"use client"` te hoog in de tree** → hele subtree wordt client, bundle groeit. Splits zodat alleen interactieve leaves client zijn.
- **Server-only code in client component**: imports van `fs`, `path`, `db`, env vars zonder `NEXT_PUBLIC_` → bundle-leak of build error.
- **Server component die client-only hooks importeert** (useState, useEffect) → build error.
- **Async server components**: `async function Page()` mag, maar **niet** in client components.
- **Children pattern**: een server component kan client components als children renderen (ze worden geslotted in), maar niet importeren en gebruiken in props zonder serialiseerbaarheid.

## 2. Server Actions

### Wat te checken
- **`"use server"` directive** bovenaan een server action file of in een function. Vergeten = client-side aanroep faalt.
- **Auth-check** in elke server action — net zo verplicht als in route handlers.
- **Authorization**: heeft de user toegang tot de resource die wordt gewijzigd? (tenant/organization/employee scope.)
- **Input validation** met zod schema — never trust client.
- **Return type**: gebruik `{ success: true, data }` of `{ success: false, error }` pattern, niet rauwe throws.
- **FormData** correct geparset (gebruik `formData.get(...)`, parse met zod).
- **`revalidatePath`/`revalidateTag`** na mutations — anders blijft cache oud.
- **Niet** server actions misbruiken voor reads — gebruik server components of route handlers.

## 3. Route handlers (route.ts)

### Wat te checken
- HTTP method exports correct: `export async function POST(req: Request)`.
- **`NextResponse.json(data, { status: 4xx })`** voor errors — niet `throw`.
- **`request.headers.get(...)`** voor auth tokens, niet uit body.
- **`request.json()`** kan throwen → wrappen in try/catch.
- **Dynamic params**: `{ params }: { params: Promise<{ id: string }> }` in Next 15+, await before use.
- **Streaming responses**: gebruik `Response` met `ReadableStream`.

## 4. Caching

### Wat te checken
- **`fetch()` in server component** wordt default gecached. Voor real-time data:
  - `fetch(url, { cache: "no-store" })` of
  - `fetch(url, { next: { revalidate: 60 } })` voor ISR
- **Database queries** worden NIET automatisch gecached — wrap in `unstable_cache` als gewenst.
- **`revalidatePath("/employees")`** na een wijziging — anders blijft de lijst stale.
- **`revalidateTag("employees")`** voor fijnmazige cache invalidation (met `fetch(..., { next: { tags: ["employees"] } })`).
- **`dynamic = "force-dynamic"`** export bovenaan page voor altijd-vers.
- **`generateStaticParams`** voor static generation van dynamic routes.

### Caching valkuilen
- Cache-key bestaat alleen uit URL + method — POST/PUT worden NIET gecached.
- `cookies()`/`headers()` aanroepen in een page maakt 'm dynamic → cache uit.
- Tussen requests kan een fetch op een private endpoint **per ongeluk gedeeld worden** als de cache niet user-scoped is.

## 5. Middleware

### Wat te checken
- **Matcher config** specifiek — niet per ongeluk static assets meepakken.
- **Auth middleware**: redirect bij niet-ingelogd, met `NextResponse.redirect(new URL("/login", req.url))`.
- **Tenant resolution** in middleware → headers/cookies meegeven aan downstream.
- **Locale routing** (i18n) — `next-intl` of `next-i18next` patronen.
- **Edge runtime constraint**: geen Node API's (geen `fs`, beperkte crypto, alleen Web APIs).

## 6. Metadata

### Wat te checken
- **`generateMetadata`** voor dynamic pages — anders default title overal.
- **Title** specifiek per pagina (niet alleen app-naam).
- **OG-tags** voor pages die gedeeld worden (employee profile niet — privacy!).
- **Robots**: `noindex` op privé-pagina's.

## 7. Loading, error, not-found UI

### Wat te checken
- **`loading.tsx`** per route segment voor skeleton/spinner.
- **`error.tsx`** met `reset()` knop — een client component, krijgt error + reset prop.
- **`not-found.tsx`** + aanroep van `notFound()` bij missende data.
- **Global error**: `app/global-error.tsx` als laatste vangnet.

## 8. Suspense & streaming

### Wat te checken
- **`<Suspense>` boundaries** rond slow data-fetches → rest van pagina rendert eerder.
- **Geen waterfall**: parallelle data-fetches met `Promise.all`, niet nested awaits.
- **Streaming** automatisch in app router — meeste `loading.tsx` werkt via Suspense onder de motorkap.

## 9. Image & font optimization

### Wat te checken
- **`next/image`** ipv `<img>` — automatic responsive + lazy + format optimization.
- **`width`/`height`** required om CLS te voorkomen.
- **`priority` prop** op LCP-image (above the fold).
- **`next/font`** voor fonts — geen FOIT/FOUT, automatic subsetting.
- **`placeholder="blur"` + `blurDataURL`** voor smooth load van content-images.

## 10. Environment variables

### Wat te checken
- **`NEXT_PUBLIC_*`** voor variabelen die in client mogen. Andere lekken NIET in bundle.
- **Secrets** (`DATABASE_URL`, `JWT_SECRET`) nooit met `NEXT_PUBLIC_` prefix.
- **Runtime check**: zod-schema bovenaan voor env-validatie bij startup.
- **`.env.example`** bijgewerkt bij nieuwe vars.

## 11. Data fetching patterns

### Wat te checken
- **Server component fetcht data** — geen `useEffect` data-fetch in client component als het ook server kan.
- **TanStack Query / SWR** voor client-side data die muteert vaak.
- **Prefetching**: `<Link prefetch>` standaard aan voor zichtbare links.
- **Parallel routes** (`@slot`) en **intercepting routes** (`(.)path`) — gebruik bewust, niet voor elke modal.

## 12. Server-only / Client-only utilities

### Wat te checken
- Gevoelige helpers (DB, secrets) markeren met `import "server-only"` bovenaan → fail loud als per ongeluk client-side gebruikt.
- Client-only browser-API's: `import "client-only"` om server-render te voorkomen.

## Wat GEEN issue is
- Pages router (legacy) — andere regels, sommige bovenstaande niet van toepassing.
- Static landing pages zonder dynamiek — caching defaults zijn prima.
- Marketing pages buiten de app.

## Severity
- **BLOCKING**: auth-check ontbreekt in server action, env-secret met `NEXT_PUBLIC_` prefix, server-only code in client component (build break).
- **SHOULD FIX**: ontbrekende `revalidatePath`, ontbrekend `loading.tsx` op slow route, image zonder dimensions.
- **NICE TO HAVE**: Suspense boundary opsplitsen voor betere streaming.

## Output
Verwijs naar Next.js-feature/concept + concrete fix met code-voorbeeld waar bondig kan.
