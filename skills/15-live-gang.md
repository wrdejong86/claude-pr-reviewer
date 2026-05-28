# Live-gang triage

Sommige wijzigingen vereisen extra aandacht vóór ze in productie kunnen.
Detecteer deze tijdens de review, en maak voor ELK relevant item een
GitHub issue zodat het inzichtelijk is in een centrale checklist.

## Wanneer een live-gang issue aanmaken?

Een wijziging is **live-gang relevant** als één of meer van deze waar is:

### 1. Nieuwe infrastructuur / configuratie
- Nieuwe **env vars** die in productie gezet moeten worden.
- Nieuwe **externe service** (mail provider, OCR, AI, payment) → verwerkersovereenkomst nodig.
- **DNS / SSL** wijzigingen.
- **Maintenance window** nodig (grote migratie, schema-rename, etc.).
- **Capaciteit** moet omhoog (DB pool, queue workers, etc.).

### 2. Cron jobs / background workers
- **Nieuwe cron** toegevoegd → moet ergens geschedulet worden (Vercel cron, GitHub Actions, etc.).
- Bestaande cron-frequentie gewijzigd → eerder/vaker draaien.
- Cron raakt **veel data** (full-table scan) → eerst capacity check.

### 3. Database migraties (zie ook 05-database)
- **Destructieve operaties** (DROP, ALTER TYPE) → maintenance window of careful ordering.
- **Backfill van bestaande data** → kan minuten/uren duren bij grote dataset.
- **NOT NULL toevoegen** met default → kan tabel locken.
- **Indexes** zonder `CONCURRENTLY` → tabel-lock.
- Migratie die **niet rollback-bare** is.

### 4. Feature flags / gradual rollout
- Grote nieuwe feature → vraag of er een **feature flag** is voor gradual enable.
- **Kill-switch** beschikbaar als het misgaat?
- Backwards compatibility (oude versies van de app/cron draaien nog tijdens deploy)?

### 5. Monitoring & alerts
- **Nieuwe metric** die getrackt moet worden (success rate, latency, kosten).
- **Nieuwe alert** nodig (escalatie-flow, dead-letter).
- **Dashboard** updaten zodat MT/dev het kan zien.

### 6. Performance / capacity
- Nieuwe **N+1 query** in user-facing endpoint.
- **Externe API call** in hot path zonder cache.
- **Bundle size** stijgt significant (>50KB).
- Verwachte verkeer: kan het systeem het aan?

### 7. Security / authentication
- Nieuwe **publieke endpoint** zonder bestaande auth.
- Nieuwe **role/permission** geïntroduceerd → wie krijgt 'm?
- **Magic links** of tokens met lange levensduur → security review.
- Nieuwe **CORS-origin** toegestaan.

### 8. GDPR / privacy (zie 09-gdpr-avg)
- Nieuwe **persoonsgegevens** opgeslagen → DPIA nodig? Retentie afgesproken?
- Nieuwe **export-functie** voor PII → audit log + access control.
- Nieuwe **e-mail/SMS** naar medewerker → consent of legitimate interest?

### 9. Communicatie / training
- User-facing wijziging die **release notes** behoeft.
- HR/MT **briefing** nodig (nieuwe workflow, gewijzigde knoppen).
- Medewerkers krijgen nieuwe **mail-template** → kort intern doorgeven.
- Documentatie/handleiding **update**.

### 10. Third-party dependencies
- Nieuwe **paid SaaS** → kosten + verwerkersovereenkomst.
- Nieuwe **dependency** met >50KB impact of risk-score (npm audit).
- **Licentie-check** voor copyleft (GPL) packages.

### 11. Data backfill / migratie van bestaande data
- Bestaande employees krijgen nieuwe velden → zijn ze ingevuld?
- Bestaande relaties moeten correct gemigreerd.
- Een-malige **catch-up cron** nodig (bv. notify alle expired ID's vandaag)?

### 12. Staging-test scenario's
- Welke **specifieke scenario's** moeten in staging gevalideerd?
- **Load test** nodig?
- **Backwards compat** test met oude data?

## Format van het issue

Gebruik dit format voor elk live-gang issue:

```
TITEL: 🚀 Live-gang: <korte omschrijving>

LABEL: live-gang

BODY:
> Gedetecteerd door reviewer-bot in PR #<nummer>.

**Categorie:** <een van hierboven, bv. "Cron jobs / background workers">

**Context:**
<2-3 zinnen wat deze PR doet en waarom het live-gang-aandacht vereist>

**Actie-items vóór live-gang:**
- [ ] <concrete actie 1>
- [ ] <concrete actie 2>
- [ ] <etc>

**Waar gevonden:**
- `file:line` — `<korte beschrijving>`

**Eigenaar:** _(invullen door wrdejong86)_
```

## Hoe issues aanmaken

Gebruik:
```
gh issue create --repo <owner/repo> \
  --title "🚀 Live-gang: <titel>" \
  --label "live-gang" \
  --body "<body>"
```

Of equivalent met heredoc voor multiline body.

## Wanneer GEEN issue aanmaken
- Pure bug-fixes zonder nieuwe infrastructuur.
- UI-tweaks (kleur, padding, copy).
- Refactors zonder gedragswijziging.
- Test-toevoegingen.
- Documentatie-updates.

## Hoeveel issues per PR?
- **Maximaal 5** per PR — beter prioriteren dan een lijst van 15 die niemand leest.
- **Groepeer waar logisch**: alle infrastructuur in één issue, alle monitoring in één.
- Als er **niets** live-gang relevant is, **maak geen issues aan**.

## Belangrijke regel
- Vermeld **altijd in de review-comment** dat je een live-gang issue hebt aangemaakt, met link.
- Voorbeeld in review: "🚀 Live-gang issue aangemaakt: [#123](https://...)" — zodat de auteur het direct ziet.
