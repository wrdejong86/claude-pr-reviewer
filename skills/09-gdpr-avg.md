# GDPR / AVG / Privacy

HR-data is bijzondere persoonsgegevens onder AVG/GDPR. Schending = boete tot
4% van wereldwijde omzet of €20M. Wees streng. Bij twijfel → blocking.

## 1. Bijzondere persoonsgegevens (AVG art. 9)
Extra-streng te beschermen:
- **BSN** (mag wettelijk alleen in HR-context)
- **Gezondheidsgegevens** (ziekteverzuim, ARBO, zwangerschap)
- **Religie** (mag in HR alleen voor feestdagen-roosters)
- **Strafrechtelijke gegevens** (VOG-data)
- **Biometrische data** (vingerafdruk, gezichtsherkenning)
- **Vakbondslidmaatschap**

### Wat te checken
- Worden deze velden ergens **gelogd** of in **error messages** geëxposed?
- Worden ze in **URLs** of **query params** gezet? (Komt in server logs, browser history.)
- Komen ze in **client-side state** zonder noodzaak?
- Zijn ze toegankelijk via een **publiek endpoint**?

## 2. Data minimization (AVG art. 5)
- **Selecteer alleen nodige velden** — geen `SELECT *` of `include: { everything: true }` in queries die naar de UI gaan.
- **API-responses**: stuur niet meer terug dan de frontend nodig heeft. Liever meerdere kleine endpoints dan één giga-payload.
- **Forms**: vraag niet om data "voor later" — pas bij behoefte.
- **Logging**: log entity-IDs, geen volledige objecten met PII.

## 3. Audit logging
HR-systemen moeten kunnen aantonen WIE WAT WANNEER heeft gezien/gewijzigd.

### Wat te checken
- Nieuwe endpoint die PII opvraagt — wordt het **gelogd in een audit table** (wie, wat, wanneer)?
- Wijzigingen aan persoonsgegevens (salaris, contract, ziekteverzuim) — wordt de **oude waarde** bewaard voor historie?
- Exports/downloads van PII — gelogd?
- Inlog-events, mislukte inlog-pogingen — gelogd?

## 4. Right to deletion (AVG art. 17)
- Heeft elk model met PII een **delete-flow**? Niet alleen `deleted_at = now()` — dat is soft-delete, geen verwijdering.
- Bij delete: ook gerelateerde data (documenten, tijdlijn-events, notificaties)?
- Backups worden niet direct geraakt, maar bij restore moet verwijderde data niet terugkomen.
- Anonimisering als alternatief: PII vervangen door hashes/dummy, integriteit van rapporten blijft.

## 5. Right of access (AVG art. 15) — Subject Access Request
- Kan een medewerker zijn **eigen data exporteren** (PDF/JSON)? Onder AVG moet je dat binnen 30 dagen leveren.
- Bevat de export ALLE data over die persoon, incl. audit-logs over hem/haar?

## 6. Retentie / bewaartermijnen
- Personeelsdossier: tot **2 jaar na uitdiensttreding** (loonheffingen 7 jaar).
- Sollicitanten (niet aangenomen): **4 weken** standaard, of 1 jaar met toestemming.
- Sick leave: tot 2 jaar na herstel.
- Wordt er een retention-policy gehandhaafd in de DB (cron die oude data verwijdert/anonimiseert)?

## 7. Consent & doelbinding
- Data verzameld voor doel A mag niet zomaar voor doel B gebruikt. Check of nieuwe features data herbestemmen.
- Bij opt-in features (nieuwsbrieven, beoordelingen, surveys): is er een **consent-tracking** model?
- Wordt consent **revocable** opgeslagen?

## 8. Data sharing & sub-processors
- Nieuwe externe service (e-mail provider, OCR, AI) → is die in de **verwerkersovereenkomst** opgenomen?
- Data over de grens (US): is er een SCC/DPF basis? (Vooral relevant bij US-cloud-providers.)
- Logging naar third-party tools (Sentry, LogRocket): mag dat PII bevatten? Nee.

## 9. Magic links & tokens
- **Token in URL** (zoals `/document-update/?token=xxx`) — komt in browser history, server logs, referer headers.
- Beter: token als POST body of header, of zeer kortlevend (bv. 5 min) + one-time-use.
- Bij e-mail magic links: expireert binnen redelijke termijn (max 14 dagen voor jouw use case).
- Wordt het token na gebruik **ongeldig gemaakt** (`usedAt = now()`)?

## 10. Cookies & tracking
- Nieuwe cookies → consent banner nodig? Functionele cookies mag, analytics/marketing niet zonder opt-in.
- Geen analytics/tracking pixels in e-mail templates zonder consent.

## 11. Data breach (AVG art. 33)
- Bij vermoede breach: meld binnen **72 uur** aan AP (Autoriteit Persoonsgegevens).
- Bevat de code logging die forensisch onderzoek mogelijk maakt? (Zonder zelf PII te lekken.)

## Wat GEEN issue is
- Interne tooling die alleen door geautoriseerde admins benaderbaar is met audit-trail.
- Test-fixtures met fake PII (duidelijk gelabeld als `test_`/`dummy_`).
- Reguliere applicatie-logs (info-level) zonder PII.

## Severity
- **BLOCKING**: BSN/gezondheidsdata in logs, externe API zonder verwerkersovereenkomst, missing tenant/auth check op endpoint met PII, token in URL voor langlevende sessie.
- **SHOULD FIX**: ontbrekend audit log, overcomplete API-response, missing retention policy.
- **NICE TO HAVE**: kleinere data-minimization optimalisaties.

## Output
Bij elke vondst noem: AVG-artikel waar mogelijk + concrete oplossing + severity.
