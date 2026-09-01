# Veranderlog — review bot

Belangrijke updates aan de review bot. Nieuwste bovenaan.

> **Heb je de bot al draaien in een eigen repo?** Dan hoef je voor
> **skill-updates** niets te doen — die worden bij elke review live opgehaald.
>
> Voor updates aan de **workflow-template** (gemarkeerd met 🔧 hieronder) moet
> je het bestand `.github/workflows/claude-review.yml` in je target-repo
> opnieuw kopiëren uit `templates/claude-review.yml` en pushen naar `main`.

---

## 2026-08-31 — 📚 Vijf nieuwe skills + drie aanscherpingen (audit op ~830 PR's)

**Wat:** Skill-audit tegen de PR-historie sinds juni (hr-hub #290 → #1116)
plus de pre-flight checklist van terugkerende review-bevindingen. De app is
sindsdien fundamenteel veranderd (meertalig, mail-wachtrij, klant-portalen
overal, drie geld-rekenmodules) — daar waren geen skills voor.

Nieuw:
- `19-i18n-vertalingen` — meertaligheid: fallbacks bij lege vertaling,
  id-gebonden keys, alles-of-niets imports met niveau-checks, RTL, mails in
  de taal van de ontvanger.
- `20-mail-notificaties` — token-dragende links naar externen (3× 403-
  incident), absolute beeld-URLs via APP_URL, wachtrij-idempotency,
  verstuurvenster, bounce-pad, mail via het choke-point.
- `21-reminders-crons` — doelgroep-filters (uit-dienst/onderaannemers!),
  dedupe-keys, venster i.p.v. exacte dag, self-closing, retry + alert.
- `22-publieke-portalen` — het magic-link-huispatroon: middleware-whitelist,
  rate-limit vóór token-lookup, 404 niet 403, token-levenscyclus incl.
  intrekken bij afronden, geen interne lekkage, sub-resources met token.
- `23-geld-rekenwerk` — geen floats voor geld, afrondingsmoment,
  %-grondslag benoemen, schema-invarianten (som termijnen == hoofdsom),
  wettelijke drempels als geteste constanten.

Aangescherpt:
- `11-nextjs` — `server-only`-sentinel niet in CLI-geladen helpers (2×
  incident); geen non-function exports uit `"use server"`-files (2×).
- `15-live-gang` — nieuwe env-var óók in de `environment`-lijst van
  `docker-compose.prod.yml` (3× vergeten).
- `17-datetime` — weergave via gedeelde datum-helpers met vaste tijdzone;
  geen inline `Intl.DateTimeFormat` (drie opruim-PR's).

**Actie nodig?** Nee — skills worden bij elke review live opgehaald.

---

## 2026-07-04 — 🔧 v2-pijplijn teruggedraaid naar v1-actie (hr-hub #697)

**Wat:** De single-shot pijplijn (v2) is in hr-hub kort live geweest en
bewust teruggedraaid: goedkoper in tokens, maar significant trager in
doorlooptijd. De bot draait weer op `claude-code-action@v1` mét de
behouden verbeteringen: skills inline via de pre-step, `workflow_dispatch`
met modelkeuze, SHA-marker-dedup, concurrency-guard en `paths-ignore`.

---

## 2026-06-17 — 🔧 Skills inline via pre-step (omzeil v1-actie-bug)

**Wat:** De skills worden nu in een **pre-step** (`Inline reviewer skills voor
de prompt`) ingeladen en direct als string in de prompt gezet, in plaats van
dat het model ze tijdens de review zelf moet ophalen.

**Waarom:** [Bug in `anthropics/claude-code-action@v1`](https://github.com/anthropics/claude-code-action/issues/690)
negeert de `--allowedTools` / `--disallowedTools` die we in de workflow
meegeven. Het model kreeg daardoor standaard de **Read**-tool en opende de
18 skill-bestanden alsnog één voor één — ~15 extra tool-rondjes per review.

**Effect (gemeten baseline → verwacht na fix):**
| Metric | Voor | Verwacht |
|--------|------|----------|
| Cost/review | $0,96 | ~$0,30 |
| Duur | 201s | ~90s |
| Turns | 15 | ~5 |
| Skills-dekking | 100% | 100% (sterker geborgd) |

**Actie nodig?** Ja: kopieer `templates/claude-review.yml` opnieuw naar
`.github/workflows/claude-review.yml` in elke target-repo en push naar `main`.

---

## 2026-06-03 — 🔧 Quota-besparing via `paths-ignore`

**Wat:** Pull requests die alleen niet-code-bestanden raken (`*.md`,
lockfiles, `.gitignore`, `LICENSE`, `CODEOWNERS`) triggeren geen review
meer. Mixed PR's (code + docs) draaien wél.

**Effect:** ~20% minder abonnement-quota-gebruik op een actieve codebase,
zonder kwaliteitsverlies.

**Actie nodig?** Ja: bij bestaande target-repo's de template opnieuw
kopiëren naar `.github/workflows/claude-review.yml`.

---

## 2026-06-03 — 🔧 Dubbele review-comments definitief opgelost

**Wat:** Drie aanvullende veiligheden tegen dubbele reviews op één commit:

1. Elke review krijgt een **verborgen SHA-marker**
   (`<!-- claude-review:<commit-sha> -->`). Vóór posten checkt de bot of er al
   een review met die marker bestaat — zo ja, niet opnieuw posten.
2. **`concurrency`-guard** annuleert overlappende runs voor dezelfde PR
   (snel opeenvolgende pushes maken niet meer twee reviews tegelijk).
3. **Live-gang issues worden nog maar 1× per PR aangemaakt** (marker +
   `gh issue list`-check) i.p.v. bij elke push opnieuw.

Een **nieuwe** comment per push blijft normaal en gewenst — alleen dezelfde
review dubbel voor dezelfde commit was de bug.

**Actie nodig?** Ja: template opnieuw kopiëren in target-repo's.

---

## 2026-06-03 — 📚 Drie nieuwe skills + security-aanvullingen

**Wat:** Drie nieuwe skill-bestanden in `skills/`, op basis van terugkerende
bug-patronen in echte PR-historie:

- `16-file-storage.md` — file- & storage-security (path-traversal op file-
  routes, ownership/rol-check bij downloads, token-scope + expiry, DB↔storage-
  consistentie, blob-over-fetch)
- `17-datetime.md` — datum/tijd/tijdzone-correctheid (midnight-truncatie,
  UTC vs lokaal, ranges, vervaldatum-randgevallen, DST)
- `18-concurrency.md` — concurrency & verloren updates (last-write-wins,
  optimistic locking, autosave-op-mount, dubbele submit, volgnummer-races)

Plus aanvullingen in `02-security.md`: auth-volgorde, timing-safe compare,
session-invalidatie/-versioning, header-injection.

**Actie nodig?** Nee — skills worden bij elke review live opgehaald.

---

## 2026-06-03 — ⚡ Snellere reviews (oorspronkelijke poging)

**Wat:** Skills laden in 1 `cat`-commando i.p.v. 15 losse Reads, PR-diff
eerst lezen, voortgangs-tracker spaarzaam gebruiken.

**Status:** Werkte minder dan gehoopt door de v1-actie-bug — vervangen door
de **pre-step-fix** van 2026-06-17 hierboven, die het probleem structureel
omzeilt.

---

## 2026-06-02 — 📖 Handleiding voor Dennie + README opgeschoond

**Wat:** Nieuwe stap-voor-stap [HANDLEIDING.md](HANDLEIDING.md) voor wie de
bot vanaf nul opzet. README bijgewerkt naar de juiste, actuele setup:
identiteit komt van de **Claude GitHub App** (`claude[bot]`), reviewer-repo
moet **publiek** zijn, en alleen `CLAUDE_CODE_OAUTH_TOKEN` is nodig.

Het oude apart bot-account (`codereviewer1986`) + Personal Access Token
(`BOT_GITHUB_TOKEN`) is **niet meer nodig** — die stappen zijn uit de docs.

**Actie nodig?** Nee.

---

## Hoe verifieer ik dat mijn target-repo de laatste workflow-versie draait?

Vergelijk de eerste regels van je `.github/workflows/claude-review.yml` met
[`templates/claude-review.yml`](templates/claude-review.yml). De huidige
versie heeft een step met de naam **"Inline reviewer skills voor de
prompt"** — als die ontbreekt, draai je een oudere versie.
