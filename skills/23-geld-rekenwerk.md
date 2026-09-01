# Geld & rekenwerk

Er zitten inmiddels meerdere rekenkernen in de app (tarievenblad, budgetten,
leningsovereenkomst met aflosschema, btw-correcties, borg). Rekenfouten in
documenten die klanten of de Belastingdienst zien, zijn geen detail.

## Wat te checken
- **Geen floats voor geld.** Opslag en rekenwerk in centen (integer) of
  Decimal; `toFixed`/format alleen bij weergave. `0.1 + 0.2`-drift in een
  contract of tarief is een blocking-waardige bug.
- **Afrondingsmoment expliciet**: per regel afronden en dan optellen geeft
  een ander totaal dan optellen en dan afronden. De keuze moet bewust
  zijn, consistent, en in een test vastgelegd.
- **Percentages: benoem de grondslag.** Een %-opslag is betekenisloos
  zonder te weten welke componenten meetellen (klassieker: de %-kolom die
  de sociale premies niet meenam). Bij elke nieuwe percentage-berekening:
  waar wordt het percentage óver genomen, en staat dat in code én
  document-tekst gelijk?
- **Btw expliciet**: excl./incl. per bedrag benoemd, correctieflow werkt
  door naar het document én de mail naar de tegenpartij.
- **Schema-invarianten testen**: som van aflos-/verbruik-termijnen ==
  hoofdsom/budget; laatste termijn vangt het afrondingsrestant. Een
  zelf-aangeleverd schema valideren vóór het in een ondertekend document
  belandt.
- **Eenmalig vs. periodiek gescheiden**: eenmalige bedragen (borg) niet
  per gebeurtenis dupliceren; periodiciteit (maand vs. 4-weken) expliciet.
- **Wettelijke drempels als geteste constanten** (bv. Wft-signalen bij
  leningen): de drempelwaarde en de vergelijking (≥ vs >) in een unit-test.
- **Import van financiële data**: verkeerde kolom/format moet luid falen,
  niet stil een 0 of een oude waarde opleveren.
- **Vastgestelde/ondertekende bedragen zijn onveranderlijk** — correcties
  via een nieuwe versie of correctieboeking, nooit in-place (sluit aan op
  de versionering uit skill 16/het agreement-patroon).

## Wat GEEN issue is
- Niet-financiële tellers en percentages zonder geldgevolg.
- Bewuste weergave-afronding in dashboards (mits berekening exact blijft).

## Severity
- **BLOCKING**: float-rekenwerk op geld in een document/DB, verkeerde
  grondslag in een klant-/personeelsdocument, in-place wijzigen van een
  vastgesteld bedrag.
- **SHOULD FIX**: onbewust afrondingsmoment, onduidelijke btw-behandeling,
  ongetest schema-invariant of drempel.
- **NICE TO HAVE**: format-consistentie in weergave.

## Output
Reken het mis-scenario concreet voor (bedrag in → fout bedrag uit) +
file:line + fix.
