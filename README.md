# Claude PR Reviewer

Een GitHub-bot die elke PR automatisch reviewt met Claude — gebruikt jouw
**bestaande Claude Code Max abonnement** via OAuth, dus geen API-kosten.

## Architectuur

```
┌──────────────────────────────┐    ┌──────────────────────────────┐
│   claude-pr-reviewer (deze)  │    │   target repo (hr-hub etc.)  │
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

**Voordeel**: target repos (hr-hub etc.) bevatten één klein workflow-bestand.
Alles wat met reviewen te maken heeft — skills, prompt, gedrag — leeft hier.
Skills aanpassen = push naar deze repo, geen wijziging in target repos nodig.

## Eenmalige setup

### Stap 1 — Claude Code OAuth token

Op je Mac:
```
brew install node
npm install -g @anthropic-ai/claude-code
claude setup-token
```
Browser → inloggen met jouw eigen Claude Max account → token kopiëren
(`sk-ant-oat01-...`). Bewaar tijdelijk veilig.

### Stap 2 — Bot GitHub account

Bot bestaat al: username `codereviewer1986`, e-mail `scanner@smbdj.nl`.

Personal Access Token (ingelogd als bot):
- https://github.com/settings/tokens?type=beta → Generate new token
- Naam: `pr-review-bot`, expiration: 1 year
- Repository access: **All repositories** (zodat de bot zowel deze repo
  als target repos kan lezen)
- Permissions:
  - Pull requests: Read and write
  - Contents: Read-only
  - Issues: Read and write

### Stap 3 — Push deze repo naar GitHub

Op je Mac, ingelogd als jezelf in `gh`:
```
cd ~/Desktop/Controlroom/claude-pr-reviewer
git init -b main
git add .
git commit -m "Initial reviewer setup"
gh repo create claude-pr-reviewer --private --source=. --push
```

### Stap 4 — Nodig bot uit in claude-pr-reviewer

Zodat de workflow van target repos de skills kan checkout-en:
https://github.com/wrdejong86/claude-pr-reviewer/settings/access →
**Add people** → `codereviewer1986` → Read access is genoeg.

### Stap 5 — Voor elke target repo (zoals hr-hub)

A. **Nodig bot uit als collaborator** (Write):
   https://github.com/wrdejong86/<target>/settings/access

B. **Kopieer template workflow** naar `.github/workflows/claude-review.yml`:
   ```
   cp ~/Desktop/Controlroom/claude-pr-reviewer/templates/claude-review.yml \
      ~/Desktop/Controlroom/<target>/.github/workflows/
   ```

C. **Voeg secrets toe** aan de target repo:
   https://github.com/wrdejong86/<target>/settings/secrets/actions
   - `CLAUDE_CODE_OAUTH_TOKEN` = token uit stap 1
   - `BOT_GITHUB_TOKEN`        = PAT uit stap 2

D. **Commit + push** de workflow naar de target repo's main branch.

E. **Log in als bot** en accepteer de uitnodiging via
   https://github.com/notifications.

### Stap 6 — Test

Open een test-PR in de target repo. Binnen ~1 minuut zie je een review-comment
van `codereviewer1986`.

## Skills aanpassen

Alles in `skills/*.md` is fair game. Voeg toe, pas aan, verwijder. Voorbeelden:

- `01-general.md` — algemene review-aanpak
- `02-security.md` — security checks (incl. HR-specifieke PII)
- `03-typescript.md` — TS gotchas
- `04-react.md` — React valkuilen
- `05-database.md` — Prisma/SQL migraties & queries
- `06-tests.md` — wat er in tests mis kan zijn

Workflow van wijzigen → effect: edit → `git push` in deze repo → volgende PR
in elke target repo gebruikt de nieuwe skills automatisch.

Zie [skills/README.md](skills/README.md) voor het format van een skill.

## Workflow-logica aanpassen

Staat in [templates/claude-review.yml](templates/claude-review.yml). Aanpassen
betekent: edit hier, daarna in elke target repo het overeenkomstige
`.github/workflows/claude-review.yml` updaten. (Voor één target nu niet veel
werk; bij meer targets later kun je overstappen op een reusable workflow.)

## Kosten

**Nul.** De OAuth token gebruikt je bestaande Claude Code Max subscription.
GitHub Actions minutes voor private repos: ~30 sec per review, ruim binnen
de free tier.

## Troubleshooting

- **Workflow draait maar geen review** → check Actions log; meestal mist een
  secret of is de OAuth token verlopen (`claude setup-token` opnieuw draaien).
- **403 bij checkout van reviewer repo** → bot is niet uitgenodigd in
  claude-pr-reviewer (zie stap 4).
- **403 bij PR comment** → bot is niet uitgenodigd in target repo, of PAT mist
  Pull requests: write permission.
- **Review is te oppervlakkig** → maak skills specifieker, of voeg toe aan
  `claude_args` in de workflow: `--model claude-opus-4-7` voor diepere review
  (kost meer van je Max-quota).
