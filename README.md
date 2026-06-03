# Claude PR Reviewer

Een GitHub-bot die elke PR automatisch reviewt met Claude — gebruikt jouw
**bestaande Claude-abonnement** (Pro of Max) via OAuth, dus geen API-kosten.

> 🆕 Nieuw opzetten? Zie de stap-voor-stap **[HANDLEIDING.md](HANDLEIDING.md)**.

## Architectuur

```
┌──────────────────────────────┐    ┌──────────────────────────────┐
│   claude-pr-reviewer (deze)  │    │   target repo (jouw app)     │
│                              │    │                              │
│   skills/*.md  ← brein       │    │   .github/workflows/         │
│   templates/   ← workflow    │    │     claude-review.yml        │
│                              │◄───┤   (één klein bestand)        │
│                              │    │                              │
└──────────────────────────────┘    └──────────────────────────────┘
        ▲                                       │
        │  bij elke PR checkt de workflow       │
        └───── deze repo uit voor de skills ────┘
```

**Voordeel**: target repos bevatten één klein workflow-bestand.
Alles wat met reviewen te maken heeft — skills, prompt, gedrag — leeft hier.
Skills aanpassen = push naar deze repo, geen wijziging in target repos nodig.

## Eenmalige setup (kort)

> De volledige stap-voor-stap uitleg met schermen en troubleshooting staat in
> **[HANDLEIDING.md](HANDLEIDING.md)**. Dit is de samenvatting.

De bot draait op een **Claude-abonnement** (Pro of Max) via een OAuth-token —
geen API-kosten. De identiteit `claude[bot]` komt van de **Claude GitHub App**.

1. **OAuth-token** — `npm install -g @anthropic-ai/claude-code`, dan
   `claude setup-token`. De token (`sk-ant-oat01-...`) wordt het repo-secret
   `CLAUDE_CODE_OAUTH_TOKEN`.
2. **Claude GitHub App installeren** op je account + de target-repo's via
   `/install-github-app` (of <https://github.com/apps/claude>). Dit geeft de
   bot de rechten om op PR's te reageren en kan het secret meteen zetten.
3. **Reviewer-repo publiek houden.** Deze repo bevat alleen review-instructies
   en moet **publiek** zijn, zodat de workflow de skills zonder token kan
   ophalen.
4. **Per target-repo:** kopieer `templates/claude-review.yml` naar
   `.github/workflows/claude-review.yml`, zorg dat het secret
   `CLAUDE_CODE_OAUTH_TOKEN` er staat, en push naar `main`.
5. **Testen:** open een test-PR → binnen ~1 min verschijnt een review van
   `claude[bot]`.

> **Niet meer nodig:** een apart bot-account (`codereviewer1986`) of een
> Personal Access Token (`BOT_GITHUB_TOKEN`). Dat was de oude opzet; de Claude
> GitHub App regelt de identiteit nu.

## Skills aanpassen

Alles in `skills/*.md` is fair game. Voeg toe, pas aan, verwijder. De huidige set:

- `01-general.md` — algemene review-aanpak
- `02-security.md` — security & auth (PII, IDOR, auth-volgorde, timing-safe compare, sessie-invalidatie)
- `03-typescript.md` — TypeScript-valkuilen
- `04-react.md` — React-valkuilen
- `05-database.md` — Prisma/SQL migraties & queries (N+1, tenant-filter)
- `06-tests.md` — test-coverage & test-kwaliteit
- `07-mobile.md` — mobile-first / responsive
- `08-design-principles.md` — DRY, SRP, SOLID, KISS, YAGNI
- `09-gdpr-avg.md` — GDPR/AVG / privacy
- `10-accessibility.md` — toegankelijkheid (WCAG 2.1 AA)
- `11-nextjs.md` — Next.js App Router (server/client, caching, server actions)
- `12-error-handling.md` — error handling & observability
- `13-api-design.md` — API & server-action-ontwerp
- `14-forms.md` — forms & validatie
- `15-live-gang.md` — go-live-triage (opent issues)
- `16-file-storage.md` — file-/storage-security (path-traversal, download-authz, DB↔storage)
- `17-datetime.md` — datum/tijd/tijdzone-correctheid
- `18-concurrency.md` — concurrency & verloren updates

Workflow van wijzigen → effect: edit → `git push` in deze repo → volgende PR
in elke target repo gebruikt de nieuwe skills automatisch.

Zie [skills/README.md](skills/README.md) voor het format van een skill.

## Workflow-logica aanpassen

Staat in [templates/claude-review.yml](templates/claude-review.yml). Aanpassen
betekent: edit hier, daarna in elke target repo het overeenkomstige
`.github/workflows/claude-review.yml` updaten. (Voor één target nu niet veel
werk; bij meer targets later kun je overstappen op een reusable workflow.)

## Kosten

**Nul.** De OAuth token gebruikt je bestaande Claude-abonnement (Pro of Max).
GitHub Actions minutes voor private repos: ~30 sec per review, ruim binnen
de free tier.

## Troubleshooting

- **Workflow draait helemaal niet** → check of de PR alleen *.md / lockfiles /
  .gitignore / LICENSE wijzigt. Die zijn bewust geskipt (`paths-ignore` in de
  workflow) om abonnement-quota te sparen. Voeg één code-wijziging toe en de
  bot draait weer.
- **Workflow draait maar geen review** → check Actions log; meestal mist een
  secret of is de OAuth token verlopen (`claude setup-token` opnieuw draaien).
- **Skills niet gevonden / 403 bij checkout van reviewer repo** → de
  reviewer-repo is niet **publiek**, of de `repository:`-regel in de workflow
  wijst naar de verkeerde plek.
- **Bot reageert als `github-actions[bot]` i.p.v. `claude[bot]`** → de Claude
  GitHub App is niet op die repo geïnstalleerd.
- **Review is te oppervlakkig** → maak skills specifieker, of voeg toe aan
  `claude_args` in de workflow: `--model claude-opus-4-7` voor diepere review
  (kost meer van je abonnement-quota).
- **Bot post dezelfde review twee keer voor één commit** → kan gebeuren
  doordat het model de review meerdere keren post ("voor de zekerheid"), of
  doordat twee runs overlappen. Een *nieuwe* comment per push is prima; alleen
  dezelfde review dubbel voor dezelfde commit is de bug. Fix in
  `templates/claude-review.yml`:
  (1) elke review begint met een verborgen marker met de commit-SHA
  (`<!-- claude-review:<sha> -->`); STEP 4 laat de bot eerst checken of er al
  een review voor deze commit staat → zo ja, niet opnieuw posten. Nieuwe push
  = nieuwe SHA = nieuwe comment, maar dubbel binnen één run kan niet meer.
  (2) een `concurrency`-guard annuleert overlappende runs voor dezelfde PR.
  (3) live-gang issues worden nog maar één keer per PR aangemaakt (marker +
  `gh issue list`-check) i.p.v. bij elke push opnieuw.
  NB: de comments worden geplaatst door `claude[bot]` (de actie zelf), niet
  door `codereviewer1986`. Kopieer de bijgewerkte template naar
  `.github/workflows/claude-review.yml` in elke target repo en push naar main.
