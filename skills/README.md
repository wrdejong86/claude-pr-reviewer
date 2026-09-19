# Skills

Elk `.md` bestand in deze map wordt automatisch geladen en als instructie aan
Claude meegegeven bij elke PR review. Zo werkt het:

- Schrijf elke skill als een korte, scherpe set regels — geen lange uitleg.
- Eén skill = één onderwerp (security, react, db migraties, etc).
- Voeg gerust nieuwe `.md` files toe; ze worden automatisch opgepikt.
- Files worden in alfabetische volgorde geladen — gebruik `01-`, `02-` prefixes
  als de volgorde uitmaakt.
- `README.md` wordt overgeslagen (dit bestand).

## Platform-specifieke skills staan in een submap

De skills in de **hoofdmap** gaan over de webstack (Next.js, React, Prisma) en
worden door die targets geladen met `cat skills/[0-9]*.md` — **alles wat genummerd
is en in de hoofdmap staat**.

Skills die alleen voor één platform gelden horen daarom in een submap, zodat ze
niet per ongeluk in een review van een ander project belanden:

```
skills/
├── 01-general.md … 23-geld-rekenwerk.md    ← webstack, door alle targets geladen
└── macos/                                  ← alleen voor native Mac-apps
    ├── 01-swift.md
    ├── 02-swiftui-macos.md
    ├── 03-externe-commandos.md
    └── 04-app-bundel-en-rechten.md
```

**Waarom:** `[0-9]*.md` recurst niet. Een Swift-skill als `24-swift.md` in de
hoofdmap zou in élke Next.js-review meegeladen worden, met regels over actors en
`Process` die daar niets betekenen. Een submap voorkomt dat zonder dat de bestaande
targets iets hoeven te wijzigen.

Een target die zo'n platform-set wél wil, laadt hem expliciet:
`cat skills/<subset>.md skills/macos/*.md`. Zie
`templates/claude-review-macos.yml`.

## Een skill toevoegen

1. Maak `skills/<naam>.md`
2. Schrijf: wat moet Claude zoeken? Welke regels gelden? Wat is GEEN issue?
3. Commit & push — volgende PR review gebruikt de nieuwe skill direct.

## Voorbeeld-structuur per skill

```markdown
# <Onderwerp>

## Wat te checken
- bullet
- bullet

## Wat geen issue is (om false positives te voorkomen)
- bullet
- bullet

## Voorbeeld
<korte code-snippet van goed vs fout>
```
