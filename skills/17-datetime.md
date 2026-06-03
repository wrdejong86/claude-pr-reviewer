# Datum, tijd & tijdzones

Datum/tijd-logica is in deze codebase een terugkerende bron van subtiele bugs
(afgekapte datums, off-by-one, foute vervaldatum-checks). Dit gaat over de
**logica**, niet over date-*inputs* in forms (dat staat in `14-forms` §8).

## 1. UTC opslaan, lokaal tonen

### Wat te checken
- Opslag in **UTC**, weergave in de tijdzone van de gebruiker — niet door elkaar.
- **Vergelijkingen** doen op UTC/`Date`-objecten, niet op geformatteerde strings.
- Geen handmatige `+1 uur`-correcties verspreid door de code (symptoom van een
  tijdzone-misverstand).

## 2. Date-only vs datetime (midnight-truncatie)

### Wat te checken
- Een **datum** (geboortedatum, vervaldatum, certificaat-datum) is géén tijdstip.
  Een vergelijking als `date <= now()` met een tijdcomponent kapt geldige
  waarden af op middernacht (zie `parseDocumentDate kapt toekomst af op
  middernacht`).
- Bewaar/vergelijk date-only op **dag-niveau** (begin/eind van de dag bewust
  kiezen), of gebruik een echt date-type.
- Let op `new Date("2026-06-03")` → dat is **UTC-middernacht**, in NL al de
  vorige avond → off-by-one bij weergave.

## 3. Verleden/toekomst-validatie

### Wat te checken
- **Geboortedatum** niet in de toekomst; leeftijd plausibel.
- **`validUntil` / vervaldatum** bij submit niet in het verleden (zie
  `valideer validUntil in offerte-submit`).
- **Start ≤ eind** bij periodes (contract, certificaat: `createCertificate/
  updateCertificate`).
- Valideer op de **server** met server-tijd, niet alleen client.

## 4. Ranges: inclusief vs exclusief

### Wat te checken
- Wees expliciet over `[start, end)` vs `[start, end]`.
- **"Geldig t/m <datum>"** betekent het **einde** van die dag, niet `00:00` —
  anders verloopt iets een dag te vroeg.
- Overlap-checks (afspraken, roosters): randen correct meenemen.

## 5. Vervaldatum / expiry-randgevallen

### Wat te checken
- Tokens, offertes, certificaten, sessies: vergelijk op het **juiste moment**
  en in de **juiste tijdzone** ("verloopt vandaag" hangt van de zone af).
- "Bijna verlopen"-reminders: off-by-one in dag-berekening is hier klassiek.

## 6. DST / zomertijd & duur-berekeningen

### Wat te checken
- Duur over een DST-grens met `ms = dagen * 86400000` is **fout** (een dag is
  niet altijd 24u). Gebruik een datum-bibliotheek (`date-fns` / `date-fns-tz` /
  `luxon`) voor add/subtract en verschillen.

## 7. Server-tijd, niet client-tijd

### Wat te checken
- `createdAt`, `expiresAt`, audit-timestamps: zet ze **server-side**. Vertrouw
  geen client-`Date` voor iets dat beveiliging of integriteit raakt.

## 8. Locale-format (zie ook 14-forms §8)
- NL-weergave `dd-mm-jjjj`; geen US `mm/dd`. Parser en formatter consistent.

## Wat GEEN issue is
- Pure UI-formatting zonder logische gevolgen.
- Vaste, bewust gekozen cron-tijden.
- Test-fixtures met vaste datums.

## Severity
- **BLOCKING/SHOULD FIX**: foute expiry-/vervaldatum-vergelijking met security-
  of business-impact (token blijft geldig, offerte ten onrechte (on)geldig).
- **SHOULD FIX**: midnight-truncatie die geldige data weggooit, ontbrekende
  start ≤ eind-check, verleden/toekomst niet gevalideerd.
- **NICE TO HAVE**: DST-randgeval in niet-kritieke duur-weergave.

## Output
Bij elke vondst: noem de datum-waarde + file:line, welk randgeval misgaat (en
in welke tijdzone), en de concrete fix.
