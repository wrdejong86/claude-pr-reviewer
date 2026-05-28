# General review approach

## Wat te checken
- Klopt de PR met de PR-beschrijving? Doet de code wat de titel/beschrijving zegt?
- Zit er dode code, ongebruikte imports, of console.logs in die er niet horen?
- Worden er TODOs of FIXMEs toegevoegd zonder uitleg?
- Zijn er hardcoded waardes (URLs, IDs, magic numbers) die in config horen?
- Wordt bestaande code onnodig herschreven (scope creep)?
- Zijn er typos in variabele/functie namen die later pijn doen?

## Wat GEEN issue is
- Stijl-issues die een linter al pakt (laat dat aan ESLint/Prettier)
- "Ik zou het anders gedaan hebben" zonder concrete reden
- Refactors voorstellen in een bugfix PR

## Format
- Verwijs altijd naar `filename:line` zodat ik direct kan klikken.
- Wees concreet: "Op `auth.ts:45` mist een null-check op `user`."
