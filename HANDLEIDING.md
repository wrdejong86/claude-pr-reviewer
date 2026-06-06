# Handleiding — Review bot opzetten

Deze handleiding legt stap voor stap uit hoe je de **PR-review-bot** vanaf nul
opzet. De bot reviewt automatisch elke pull request met Claude en plaatst een
review-comment (in het Nederlands). Hij draait op een **Claude-abonnement**
(Pro of Max) — dus **geen API-kosten**.

> Tijd: ~20 minuten. Je hoeft geen code te schrijven.

---

## 1. Hoe het in elkaar zit (kort)

Er zijn twee soorten repo's:

```
┌────────────────────────────┐        ┌────────────────────────────┐
│  claude-pr-reviewer         │        │  target-repo (jouw app)    │
│  (de "hersenen", PUBLIEK)   │        │                            │
│                             │        │  .github/workflows/        │
│  skills/*.md  ← reviewregels│◄───────┤    claude-review.yml       │
│  templates/   ← workflow    │ checkt │  (één klein bestand)       │
└────────────────────────────┘ skills └────────────────────────────┘
```

- **`claude-pr-reviewer`** bevat de review-instructies (`skills/*.md`) en de
  workflow-template. Deze repo is **publiek**, zodat elke target-repo de skills
  kan ophalen zonder token.
- Een **target-repo** (de repo die je wilt laten reviewen) bevat alleen één
  klein workflow-bestand. Bij elke PR haalt dat bestand de skills uit
  `claude-pr-reviewer` en draait de review.

**Gevolg:** skills aanpassen = pushen naar `claude-pr-reviewer`. De target-repo's
hoef je daarna niet meer aan te raken.

---

## 2. Wat je nodig hebt (vooraf)

- [ ] Een **Claude-abonnement** (Pro of Max). De bot gebruikt jóuw abonnement.
- [ ] **Node.js** op je computer (voor de Claude Code CLI).
- [ ] Een **GitHub-account** met beheerrechten op de repo's die je wilt reviewen.
- [ ] De repo's staan op GitHub.

---

## 3. Stappenplan

### Stap 1 — Claude Code installeren + OAuth-token maken

Op je Mac (terminal):

```bash
brew install node                          # alleen als Node nog niet geïnstalleerd is
npm install -g @anthropic-ai/claude-code
claude setup-token
```

`claude setup-token` opent je browser → log in met je **eigen Claude-account** →
kopieer de token (begint met `sk-ant-oat01-...`).

➡️ Bewaar deze token even veilig. Dit wordt straks het GitHub-secret
**`CLAUDE_CODE_OAUTH_TOKEN`**.

> Windows/Linux: installeer Node via [nodejs.org](https://nodejs.org), de rest is gelijk.

---

### Stap 2 — De Claude GitHub App installeren

Dit is wat ervoor zorgt dat de bot als **`claude[bot]`** op PR's mag reageren.

**Makkelijkste manier** — in een willekeurige map met een GitHub-repo:

```bash
claude
/install-github-app
```

De wizard helpt je met:
1. De **Claude GitHub App** installeren op je account/organisatie en de
   gewenste repo's (<https://github.com/apps/claude>).
2. Het secret **`CLAUDE_CODE_OAUTH_TOKEN`** aanmaken in die repo's (plak de
   token uit stap 1).

**Lukt `/install-github-app` niet?** Doe het handmatig:
1. Ga naar <https://github.com/apps/claude> → **Install** → kies je account/org
   en de repo's die je wilt laten reviewen.
2. Voeg het secret zelf toe (zie stap 4C).

> Zonder deze app reageert de bot als `github-actions[bot]` i.p.v. `claude[bot]`,
> of kan hij helemaal niet posten. **Deze stap is verplicht.**

---

### Stap 3 — De reviewer-repo (skills) regelen

Je hebt twee opties:

**Optie A — hergebruik de bestaande (snelst).**
De repo `wrdejong86/claude-pr-reviewer` is al publiek en bevat de skills. Je hoeft
niets te doen; ga door naar stap 4.

**Optie B — eigen beheer.**
Wil je de skills in een eigen repo beheren? Fork of kopieer
`claude-pr-reviewer` naar je eigen account en zet die repo op **Public**
(Settings → General → Change visibility → Public).

> ⚠️ De reviewer-repo **moet publiek zijn**. Anders kan de workflow de skills niet
> ophalen (de checkout gebruikt geen token). Er staan geen geheimen in deze repo —
> alleen review-instructies — dus publiek is prima.

Onthoud de volledige naam `eigenaar/repo` (bv. `<jouw-account>/claude-pr-reviewer`).

---

### Stap 4 — Per target-repo de workflow toevoegen

Doe dit voor **elke repo** die je wilt laten reviewen.

**A. Kopieer het workflow-bestand.**
Neem `templates/claude-review.yml` uit de reviewer-repo en plaats het in de
target-repo als:

```
.github/workflows/claude-review.yml
```

**B. (Alleen bij Optie B) wijs naar jóuw reviewer-repo.**
Pas in dat bestand deze regel aan naar je eigen reviewer-repo:

```yaml
      - name: Checkout reviewer (for skills)
        uses: actions/checkout@v4
        with:
          repository: wrdejong86/claude-pr-reviewer   # ← jouw eigenaar/repo
          path: .reviewer
```

Bij Optie A laat je deze regel staan zoals hij is.

**C. Zorg dat het secret bestaat.**
Ga naar de target-repo → **Settings → Secrets and variables → Actions** en
controleer dat **`CLAUDE_CODE_OAUTH_TOKEN`** er staat (de token uit stap 1).
Heeft `/install-github-app` dit al gedaan? Dan ben je klaar.

> Tip: je kunt het secret ook op **organisatie-niveau** zetten, dan delen alle
> repo's het automatisch.

**D. Commit + push** het workflow-bestand naar de **main**-branch van de
target-repo.

---

### Stap 5 — Testen

1. Open een **test-PR** in de target-repo (bv. een kleine wijziging).
2. Binnen ~1 minuut verschijnt een review-comment van **`claude[bot]`**.
3. Check eventueel de voortgang onder **Actions → Claude PR review**.

Werkt het? 🎉 Klaar. Je kunt de test-PR weer sluiten.

---

## 4. Gedrag van de bot aanpassen (skills)

Alle reviewregels staan in `skills/*.md` in de reviewer-repo. De huidige set:

| Bestand | Onderwerp |
|---------|-----------|
| `01-general.md` | algemene review-aanpak |
| `02-security.md` | security & auth (PII, IDOR, auth-volgorde, timing-safe) |
| `03-typescript.md` | TypeScript-valkuilen |
| `04-react.md` | React-valkuilen |
| `05-database.md` | Prisma/SQL migraties & queries |
| `06-tests.md` | test-coverage & test-kwaliteit |
| `07-mobile.md` | mobile-first / responsive |
| `08-design-principles.md` | DRY, SRP, SOLID, KISS, YAGNI |
| `09-gdpr-avg.md` | GDPR/AVG / privacy |
| `10-accessibility.md` | toegankelijkheid (WCAG 2.1 AA) |
| `11-nextjs.md` | Next.js App Router |
| `12-error-handling.md` | error handling & observability |
| `13-api-design.md` | API & server-action-ontwerp |
| `14-forms.md` | forms & validatie |
| `15-live-gang.md` | go-live-aandachtspunten (opent issues) |
| `16-file-storage.md` | file-/storage-security (path-traversal, download-authz) |
| `17-datetime.md` | datum/tijd/tijdzone-correctheid |
| `18-concurrency.md` | concurrency & verloren updates |

**Aanpassen:** edit een `.md`-bestand → commit & push naar de reviewer-repo's
`main`. De **volgende** PR-review in elke target-repo gebruikt het direct. Je
hoeft de target-repo's niet aan te raken.

Een nieuwe skill toevoegen? Maak `skills/<naam>.md` aan; hij wordt automatisch
opgepikt. Zie `skills/README.md` voor het format.

---

## 5. Model kiezen

De bot draait standaard op **Sonnet** — dit staat vast in `claude_args` in de
workflow. Sonnet houdt het abonnement-verbruik laag; de bot reviewt elke
PR-push opnieuw met de volledige diff en alle skills.

**Handmatig een diepere review starten (bv. Opus):**
1. Ga naar GitHub → **Actions** → "Claude PR review"
2. Klik rechts op **"Run workflow"**
3. Vul het PR-nummer in en kies het model: `sonnet` / `opus` / `haiku`
4. Klik **"Run workflow"**

De bot plaatst dan een nieuwe review-comment op die PR met het gekozen model.
Automatische runs bij PR-pushes blijven altijd op Sonnet.

---

## 6. Wat NIET (meer) nodig is

De oude opzet gebruikte een apart bot-account (`codereviewer1986`) met een
Personal Access Token. **Dat is niet meer nodig.** De Claude GitHub App regelt
de identiteit en de schrijfrechten. Negeer oude instructies die je vragen een
bot-account of PAT (`BOT_GITHUB_TOKEN`) aan te maken.

---

## 6. Troubleshooting

| Symptoom | Oorzaak / oplossing |
|----------|---------------------|
| Workflow draait, maar geen review | Check de **Actions**-log. Meestal mist het secret `CLAUDE_CODE_OAUTH_TOKEN`, of de token is verlopen → draai `claude setup-token` opnieuw en update het secret. |
| Skills niet gevonden / 403 bij checkout | De reviewer-repo is niet **publiek**, of de `repository:`-regel (stap 4B) wijst naar de verkeerde plek. |
| Bot reageert als `github-actions[bot]` i.p.v. `claude[bot]` | De **Claude GitHub App** is niet op die repo geïnstalleerd (stap 2). |
| Helemaal geen workflow-run | Het bestand staat niet op de **main**-branch van de target-repo, of de PR is een **draft** (draft-PR's worden overgeslagen), of de PR wijzigt alleen `*.md` / lockfiles / `.gitignore` / `LICENSE` (bewust geskipt via `paths-ignore` om quota te sparen). |
| Review te oppervlakkig | Maak de skills specifieker, of gebruik de handmatige `workflow_dispatch`-trigger om de PR op Opus te reviewen (zie sectie "Model kiezen" hierboven). |
| Dezelfde review dubbel voor één commit | Mag niet meer gebeuren (recent gefixt met een SHA-marker). Een **nieuwe** comment per push is wél normaal en gewenst. Draait de target-repo nog een oude workflow? Kopieer de nieuwste `templates/claude-review.yml` opnieuw. |

---

## 7. Kosten

**Nul.** De review draait op je Claude-abonnement (Pro/Max) via de OAuth-token.
GitHub Actions-minuten: ~30 sec per review, ruim binnen de free tier (ook voor
private repo's).

---

## Naslag

- Diepere uitleg en architectuur: [`README.md`](README.md)
- Skill-format: [`skills/README.md`](skills/README.md)
- Officiële docs Claude GitHub Action:
  <https://code.claude.com/docs/en/github-actions>
