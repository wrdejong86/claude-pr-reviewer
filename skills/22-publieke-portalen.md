# Publieke portalen & magic-links

Magic-link-portalen zijn hét architectuurpatroon van de app geworden
(ondertekenen, voortgangsoverleg, budget, QR-machinehulp, aanvul-links,
surveys, incident-melden). Er is een vast huispatroon — wijk je ervan af,
dan is dat een bevinding.

## Wat te checken (het huispatroon)
- **Nieuwe publieke route → middleware-whitelist bijwerken** (`proxy.ts` /
  `isPublicRoute`). Vergeten = de klant wordt naar /login gestuurd vóór je
  handler ooit draait. Dit is meermaals gebeurd; check het bij élke nieuwe
  publieke route én bij nieuwe publieke API-sub-routes.
- **Rate-limit VÓÓR de token-lookup**, niet erna (anders is de lookup zelf
  het DoS-oppervlak).
- **Length-guard op het token** (bv. 32-char minimum) vóór de DB-query —
  defense-in-depth tegen enumeration.
- **Token-levenscyclus compleet**: `expiresAt` gecheckt, `revokedAt`
  gerespecteerd, `usedAt` waar one-time bedoeld is, en het token wordt
  **ingetrokken wanneer het bron-object sluit** (archiveren/afronden van
  overleg, offerte, traject).
- **404, geen 403** op onbekend/ingetrokken token — geen side-channel die
  verklapt dat iets bestaat.
- **Geen interne lekkage op klant-pagina's**: geen interne namen/rollen,
  ids, interne navigatie, signaleringen of andere klanten. De klant ziet
  alleen wat voor de klant bedoeld is; huisstijl en echt logo horen erbij.
- **Sub-resources dragen het token mee**: foto's, PDF's en exports achter
  een publieke pagina hebben dezelfde token-check als de pagina zelf.
- **Sessie-loze test**: werkt de hele flow (incl. downloads) in een
  incognito-venster? Zo nee → bevinding.
- **Scope per link expliciet**: een link geeft toegang tot precies één
  resource(-boom); check dat de query op het token-record scopet, niet op
  een client-meegegeven id.
- **`lastUsedAt`/`visitCount` best-effort** — tracking mag de flow nooit
  breken.

## Wat GEEN issue is
- Interne (ingelogde) pagina's; MT-beheerschermen voor de links zelf.
- Bewust permanente tokens (meldlinks) mét revocatie-mogelijkheid.

## Severity
- **BLOCKING**: publieke route zonder rate-limit of token-scope-check,
  middleware-whitelist vergeten, interne PII/identiteit op een
  klant-pagina, sub-resource zonder token-check.
- **SHOULD FIX**: 403 i.p.v. 404, token niet ingetrokken bij afronden,
  ontbrekende length-guard.
- **NICE TO HAVE**: huisstijl-polish, tracking-verrijking.

## Output
Beschrijf wat een buitenstaander met alleen de URL kan + file:line + fix.
