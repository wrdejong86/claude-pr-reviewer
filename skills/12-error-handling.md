# Error handling & observability

Robuuste error handling = verschil tussen "het werkt op localhost" en "het
werkt onder druk in productie".

## 1. Try/catch granularity

### Wat te checken
- **Try/catch zonder doel**: gewoon `catch (e) { console.log(e) }` → fout wordt gegeten, gebruiker krijgt niets, geen alarm. **Anti-pattern.**
- **Try-catch om hele functie** zonder onderscheid tussen verschillende errors → onhandig om te debuggen.
- **Specifieke error-types** vangen i.p.v. alles: `if (e instanceof PrismaClientKnownRequestError && e.code === "P2002")` → handle unique constraint specifiek.
- **Re-throw waar nodig** als je niet kan herstellen — laat hogere laag het afhandelen.

## 2. Custom error classes

### Wat te checken
- Domain-errors als classes: `class EmployeeNotFoundError extends Error`, `class UnauthorizedError extends Error`, `class ValidationError extends Error`.
- Server actions kunnen instanceof checks gebruiken om passende HTTP status / UI te kiezen.
- Geef errors een `code` (string) en `cause` (origineel) — beter logbaar.

## 3. React error boundaries

### Wat te checken
- **`error.tsx`** in elke significante route segment in Next App Router.
- **Class-based ErrorBoundary** voor non-route React subtrees waar fouten gefocaliseerd opgevangen moeten.
- Boundary's tonen een **vriendelijke fallback** + "Reset" knop + optioneel "Stuur naar support" (met error-ID).
- Boundary **logt** de error naar monitoring tool — niet alleen UI tonen.

## 4. Server-side error handling

### Wat te checken
- **Route handlers** gooien geen rauwe errors — return `NextResponse.json({ error: ... }, { status: 500 })`.
- **Server actions** retourneren `{ success: false, error: "..." }` — niet throwen (Next serialiseert anders raar).
- **Database errors** worden NIET 1-op-1 doorgegeven (lekt schema). Map ze naar gebruiker-vriendelijke messages.
- **Onverwachte errors** worden gelogd met **stack + context** (user-id, request-id) maar zonder PII.

## 5. User-friendly errors (Nederlands)

### Wat te checken
- Geen technische jargon ("HTTP 500", "TypeError"). Wel: "Er ging iets mis bij het opslaan, probeer het later opnieuw."
- **Welke actie kan de gebruiker doen?** "Probeer opnieuw" / "Neem contact op met support" / "Refresh de pagina".
- **Foutmelding bij forms**: specifiek bij het veld dat fout was, niet generiek bovenaan (zie ook `forms` + `accessibility` skills).
- **Optimistische updates** die falen: tonen de oorspronkelijke state weer + duidelijke melding.

## 6. Logging strategy

### Wat te checken
- **Structured logging**: `logger.info({ event: "employee.created", employeeId, by: actorId })` ipv `console.log("created " + id)`. Doorzoekbaar.
- **Log levels** correct: `error` voor onverwachte, `warn` voor recoverable, `info` voor business events, `debug` voor verbose.
- **Geen `console.log`** in productie-paden — gebruik centralized logger (pino, winston).
- **Request-ID / correlation-ID** door request keten — anders niet traceerbaar.
- **No PII in logs**: zie ook GDPR skill. Hash/redact e-mails, namen, BSN.

### Anti-patronen
- `console.log(employee)` waar `employee` BSN bevat → logged in production.
- Errors in stilte: `.catch(() => {})`.

## 7. Monitoring / Sentry / observability

### Wat te checken
- **Sentry (of equivalent) configured?** — bij nieuwe error-paden, wordt het opgevangen?
- **`Sentry.captureException(error, { extra: { ... } })`** met context — niet zomaar dumpen.
- **Sample rate** voor performance niet 100% (te duur).
- **Source maps** geupload zodat stack traces leesbaar zijn.
- **User context** (id, NOT email) gezet voor traceability.
- **Geen breadcrumbs** met PII in requests.

## 8. Retry & timeouts

### Wat te checken
- **External calls** (e-mail provider, OCR, etc.) → timeout + retry met exponential backoff. Geen oneindig wachten.
- **Idempotent** retries — anders dubbele e-mails / charges. Gebruik idempotency-keys.
- **Circuit breaker** patroon voor herhaaldelijk falende externe services (bv. p-retry, p-timeout).
- **Database transactions** met timeout — geen `BEGIN` die uren blijft hangen.

## 9. Graceful degradation

### Wat te checken
- Als de e-mail-service down is, faalt de hele employee-create dan? Beter: queue de mail in achtergrond.
- Als analytics endpoint faalt, breekt dat de UI? Nee, het mag stilletjes falen.
- **Niet-kritieke features** mogen falen zonder de hoofdfunctie te breken.

## 10. Toast vs inline errors

### Wat te checken
- **Inline error** bij forms / specifieke acties (gebruiker weet welk veld/actie).
- **Toast** voor algemene events ("Opgeslagen!", "Verzonden!"). Niet voor errors die actie vereisen.
- **Modal** voor blocking errors die instructie geven (sessie verlopen).
- Errors die de gebruiker mist als ze niet kijken → gebruik geen toast. Persistent.

## 11. Async patterns

### Wat te checken
- **Promises zonder catch** — fire-and-forget kan throws missen → unhandled rejection.
- **Promise.all** faalt zodra één faalt — gebruik `Promise.allSettled` als parts mogen falen.
- **AbortController** bij component unmount voor lopende fetches.

## 12. Data-integriteit

### Wat te checken
- Schrijven naar meerdere stores zonder transactie → inconsistente state bij fout halverwege. Wrap in transactie.
- "Sync" operaties (DB + e-mail + cache invalidation) → óf alles slaagt, óf alles wordt teruggedraaid + retried.
- **Eventual consistency** is OK, maar dan moet de UI dat aankunnen (refresh, "wordt verwerkt" state).

## Wat GEEN issue is
- Test fixtures die expres throwen.
- Quick debug logs in development-only paden (`if (process.env.NODE_ENV === "development")`).
- Fail-loud bij startup config errors (juist goed — fail fast).

## Severity
- **BLOCKING**: gefloten errors (`catch {}`), PII in logs, geen auth-error handling.
- **SHOULD FIX**: generic 500 zonder context, ontbrekende error boundary, geen retry op flaky external call.
- **NICE TO HAVE**: meer specifieke error-classes, betere user-facing message.

## Output
Bij elke vondst: wat KAN er fout gaan + wat ziet de gebruiker nu + wat zou je doen.
