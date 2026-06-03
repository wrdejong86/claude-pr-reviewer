# Security

## Wat te checken
- **Secrets in code**: API keys, tokens, passwords, connection strings hardcoded.
- **SQL injection**: string-concatenated queries, ontbrekende parameterized queries.
- **XSS**: `dangerouslySetInnerHTML`, `innerHTML =`, ongetrimde user input in DOM.
- **Auth bypass**: endpoints zonder auth-check, missing `requireAuth` middleware.
- **Authorization**: endpoints die niet checken of de user toegang heeft tot het
  betreffende `tenant`/`organization`/`employee` record (IDOR risk).
- **CSRF**: state-changing endpoints zonder CSRF token bij cookie-based auth.
- **Open redirects**: redirects naar user-controlled URLs zonder whitelist.
- **Logging van gevoelige data**: passwords, tokens, BSN, gezondheidsdata in logs.
- **Dependency wijzigingen**: nieuwe packages — check of het bekende/onderhouden packages zijn.
- **HR-specifiek**: persoonlijke data (BSN, salaris, ziekteverzuim) moet altijd
  geauthorizeerd opgevraagd worden — nooit in een publiek endpoint.
- **Auth-volgorde**: de auth/authz-check moet de **allereerste** stap zijn in een
  server action / route handler — vóór status-checks, reads of side-effects.
  Werk doen of data lezen vóór de check lekt info of voert ongeautoriseerde
  acties deels uit.
- **Timing-safe vergelijking**: secrets, tokens en HMAC-signatures vergelijken
  met een constant-time functie (`crypto.timingSafeEqual`), niet met `===`
  (timing-attack). Vooral op cron-/webhook-endpoints met een Bearer/shared secret.
- **Sessie-invalidatie / session-versioning**: bij wachtwoord-wijziging,
  rol-/permissie-wijziging of "log overal uit" moeten bestaande sessies en
  tokens ongeldig worden (bv. een session-version die je bumpt). Anders blijft
  een oude sessie geprivilegieerd.
- **Header-injection**: user-input in response-headers (redirect `Location`,
  `Content-Disposition` filename, e-mail-headers) zonder CR/LF te strippen →
  header/response-splitting.

## Wat GEEN issue is
- Test fixtures met fake secrets (label `test_` of `dummy_`)
- Verwijderen van security-checks bij refactor waar de check elders is verplaatst

## Severity guidance
- Secrets gecommit = **Blocking** + flag dat de secret geroteerd moet worden.
- IDOR / auth bypass = **Blocking**.
- Auth-check ná reads/side-effects, of timing-unsafe compare op auth-token = **Blocking**.
- Sessie niet geïnvalideerd bij rol/wachtwoord-wijziging, header-injection = **Should fix**.
- Logging van PII = **Should fix**.
