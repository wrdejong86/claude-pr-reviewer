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

## Wat GEEN issue is
- Test fixtures met fake secrets (label `test_` of `dummy_`)
- Verwijderen van security-checks bij refactor waar de check elders is verplaatst

## Severity guidance
- Secrets gecommit = **Blocking** + flag dat de secret geroteerd moet worden.
- IDOR / auth bypass = **Blocking**.
- Logging van PII = **Should fix**.
