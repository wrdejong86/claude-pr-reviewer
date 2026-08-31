# Meertaligheid & vertalingen

De app is meertalig geworden (onboarding/opleidingen in 7–8 talen incl.
Arabisch/RTL, QR-machinehulp, meertalige mails). Vertaal-bugs zijn stil:
de NL-versie werkt altijd, het gat zit bij de anderstalige medewerker.

## Wat te checken
- **Lege of ontbrekende vertaling** mag nooit een lege string of crash
  geven — expliciete fallback naar NL, en het gat moet in beheer zichtbaar
  zijn (dekkingsmeter), niet stil wegvallen.
- **Koppel vertalingen aan stabiele id's**, niet aan titels, volgorde of
  array-index. Herordenen of hernoemen mag geen vertaling losmaken.
- **Nieuwe UI-tekst in een medewerker-/klant-flow** die al vertaald is →
  moet direct via de vertaal-laag lopen, geen hardcoded NL ertussen.
  Nieuw onderdeel? Vóórvertalen vóór livegang, niet erna.
- **Import/seed van vertalingen is alles-of-niets**: check aantallen op
  élk niveau (hoofdstukken, vragen, opties) — een ontbrekend blok valt
  stil terug op NL en is dan onzichtbaar.
- **Mails in de taal van de ontvanger** (of bewust tweetalig), niet in de
  taal van de verzender.
- **RTL (Arabisch)**: layout spiegelt mee (`dir="rtl"`), chevrons/pijlen
  en uitlijning draaien om; geen hardcoded `text-left`/`ml-*` waar het
  richtingsgevoelig is.
- **Toon**: geen spreektaal-samentrekkingen ('ie, 'm, 't) in tekst die
  medewerkers of klanten lezen; aanspreekvorm consistent per doelgroep.
- **Interpolatie**: variabelen in vertaalstrings via placeholders, geen
  string-concatenatie (woordvolgorde verschilt per taal).

## Wat GEEN issue is
- MT-only beheer-schermen die bewust NL-only zijn.
- Log-teksten en code-comments.

## Severity
- **BLOCKING**: lege string/crash bij ontbrekende vertaling in een
  medewerker-flow; import die stil een heel hoofdstuk overslaat.
- **SHOULD FIX**: hardcoded NL in een vertaalde flow, vertaling aan
  volgorde/titel gekoppeld, mail in verkeerde taal, RTL-breuk.
- **NICE TO HAVE**: dekkingsmeter-verfijning, toon-polish.

## Output
Noem de taalvariant/het scenario waarin het misgaat + file:line + fix.
