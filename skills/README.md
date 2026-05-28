# Skills

Elk `.md` bestand in deze map wordt automatisch geladen en als instructie aan
Claude meegegeven bij elke PR review. Zo werkt het:

- Schrijf elke skill als een korte, scherpe set regels — geen lange uitleg.
- Eén skill = één onderwerp (security, react, db migraties, etc).
- Voeg gerust nieuwe `.md` files toe; ze worden automatisch opgepikt.
- Files worden in alfabetische volgorde geladen — gebruik `01-`, `02-` prefixes
  als de volgorde uitmaakt.
- `README.md` wordt overgeslagen (dit bestand).

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
