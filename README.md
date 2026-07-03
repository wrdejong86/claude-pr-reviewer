# Claude PR Reviewer

Een GitHub-bot die elke PR automatisch reviewt met Claude — gebruikt jouw
**bestaande Claude-abonnement** (Pro of Max) via OAuth, dus geen API-kosten.

> 🆕 Nieuw opzetten? Zie de stap-voor-stap **[HANDLEIDING.md](HANDLEIDING.md)**.

> 🔔 **Update 2026-07-03 (v2):** de bot is omgebouwd van een agentische
> actie naar een **review-pijplijn** — één model-call zonder tools, posten
> gebeurt door de workflow zelf. ~10× goedkoper per review, sneller, en
> read-only/éénmalig-posten zijn nu structureel gegarandeerd. Draai je de
> bot al ergens? **Kopieer
> [`templates/claude-review.yml`](templates/claude-review.yml) opnieuw** naar
> `.github/workflows/claude-review.yml` en push naar `main`. Details in
> **[CHANGELOG.md](CHANGELOG.md)**.

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
geen API-kosten. Sinds v2 (2026-07) post de workflow zelf de review; comments
verschijnen als `github-actions[bot]`.

1. **OAuth-token** — `npm install -g @anthropic-ai/claude-code`, dan
   `claude setup-token`. De token (`sk-ant-oat01-...`) wordt het repo-secret
   `CLAUDE_CODE_OAUTH_TOKEN`.
2. **Reviewer-repo publiek houden.** Deze repo bevat alleen review-instructies
   en moet **publiek** zijn, zodat de workflow de skills zonder token kan
   ophalen.
3. **Per target-repo:** kopieer `templates/claude-review.yml` naar
   `.github/workflows/claude-review.yml`, zorg dat het secret
   `CLAUDE_CODE_OAUTH_TOKEN` er staat, en push naar `main`.
4. **Testen:** open een test-PR → binnen ~2 min verschijnt een review van
   `github-actions[bot]`.

> **Niet meer nodig:** een apart bot-account (`codereviewer1986`), een
> Personal Access Token (`BOT_GITHUB_TOKEN`), óf de **Claude GitHub App**
> (die was alleen nodig voor de oude agentische actie). Alleen het
> OAuth-secret volstaat.

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

## Model

De bot draait standaard op **Sonnet**. Sonnet houdt de kosten laag — de bot
draait op jóuw abonnement en reviewt elke PR-push inclusief de volledige diff
en alle skills.

**Handmatig een ander model kiezen:**
Ga naar GitHub → Actions → "Claude PR review" → **Run workflow**. Vul het
PR-nummer in en kies het gewenste model (sonnet / opus / haiku). Handig voor
grote of kritieke PRs die een diepere Opus-review verdienen.

Automatische runs bij PR-pushes gebruiken altijd Sonnet.

## Kosten

**Geen API-kosten.** De OAuth token gebruikt je bestaande Claude-abonnement
(Pro of Max). Elke review verbruikt wél abonnement-quota — sinds v2 typisch
**~$0,15–0,25 equivalent per review** (één model-call). Het exacte bedrag per
review staat in de **Actions job summary**. GitHub Actions minutes: ~2 min
per review, ruim binnen de free tier.

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
- **Review komt van `github-actions[bot]`** → dat is sinds v2 normaal: de
  workflow post zelf. De Claude GitHub App is niet meer nodig.
- **Review is te oppervlakkig** → maak skills specifieker, of gebruik de
  handmatige `workflow_dispatch`-trigger om de PR op Opus te reviewen (zie
  sectie "Model" hierboven).
- **Bot post dezelfde review twee keer voor één commit** → kan sinds v2 niet
  meer (bash post exact 1× en een dedup-gate stopt re-runs op dezelfde
  commit vóór de model-call). Zie je het toch, dan draait die target-repo
  nog een oude workflow-versie → kopieer de template opnieuw. Een *nieuwe*
  comment per push is normaal en gewenst.
- **Model-call faalt ("Lege model-output" in de log)** → meestal is de
  OAuth-token verlopen: draai `claude setup-token` opnieuw en ververs het
  secret. Anders: check de Actions-log van de stap "Review".
