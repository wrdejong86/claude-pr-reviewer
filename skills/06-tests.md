# Tests

## Wat te checken
- **Nieuwe feature zonder tests** — vraag waar de coverage is.
- **Bug fix zonder regression test** — de bug komt terug.
- **Tests die de implementatie testen i.p.v. gedrag** (te veel mocking van interne functies).
- **Skipped tests** (`.skip`, `.only`, `xit`) die per ongeluk gecommit zijn.
- **Hardcoded waits** (`setTimeout(... 1000)`) i.p.v. proper async wait helpers.
- **Tests die alleen happy path testen** — waar zijn de edge cases / error paths?
- **Snapshot tests** zonder review — vaak gewoon "accepted whatever came out".
- **Test data die echt naar productie kan lekken** (echte e-mails, echte API keys).
- **Tests die elkaar beïnvloeden** (state niet gereset tussen tests).

## Wat GEEN issue is
- Pure refactors waar tests onveranderd blijven en blijven slagen
- Tooling/config wijzigingen die geen test nodig hebben
- Triviale wijzigingen (typo in copy, kleur aanpassing)

## Format
- Als tests ontbreken, geef concreet aan WELKE scenarios getest moeten worden.
- Niet alleen "voeg tests toe" — wees specifiek over de cases.
