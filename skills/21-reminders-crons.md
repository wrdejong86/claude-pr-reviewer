# Reminders & achtergrondjobs

De reminder-engine raakt steeds meer flows (certificaten, toolboxen,
onboarding, VOG, DBA, bandendruk, actiepunten). De klassieke bugs: verkeerde
doelgroep, dubbele of juist gemiste reminders, en reminders die blijven
spoken nadat het doel al bereikt is.

## Wat te checken
- **Doelgroep-filter expliciet en volledig**: alleen status ACTIEF; sluit
  uit-dienst-medewerkers uit, en onderaannemers waar de flow niet voor hen
  bedoeld is. Dit ging meermaals mis — benoem in de review wélke filters
  de query heeft en welke ontbreken.
- **Dedupe-key** per (soort, doelwit, periode) in het ReminderLog-patroon —
  een her-run van de cron mag niet opnieuw mailen.
- **Venster i.p.v. exacte dag**: een tier-check op precies dag-X mist bij
  één gemiste cron-run de hele tier. Gebruik een venster (bv. 2 dagen)
  zodat een gemiste run inhaalt — maar begrens de catch-up zodat een lange
  downtime geen reminder-storm veroorzaakt.
- **Self-closing**: is het doel bereikt (getekend, verlengd, geregistreerd)
  → sluit de bijbehorende taak/reminder in dezelfde flow. Een reminder die
  blijft staan na afhandeling is een bug, geen detail. Omgekeerd geldt ook:
  afgehandeld ≠ spoorloos — de historie blijft zichtbaar.
- **Retry + alarm**: tijdelijke fout (netwerk/DB) → opnieuw proberen; en
  structureel falen van de cron moet een alert geven (heartbeat/push-
  monitor), niet stil overslaan.
- **Idempotent per stap**: elke pass een eigen try/catch zodat één falende
  deel-taak de rest van de run niet meesleurt.
- **Tijdzone-vast**: draaimoment en dag-grenzen in de bedoelde tijdzone
  (NL), niet impliciet UTC — anders verschuiven "vandaag"-checks.
- **Escalatie-pad klopt**: wie krijgt de reminder, en wie krijgt het
  signaal als er niet gereageerd wordt (medewerker → OL/MT)?

## Wat GEEN issue is
- Eenmalige handmatige scripts (wel: idempotent en dev+prod-robuust).
- Bewust gekozen frequenties.

## Severity
- **BLOCKING**: verkeerde doelgroep (bv. uit-dienst gemaild), dubbele
  bezorging bij her-run, reminder-storm mogelijk na downtime.
- **SHOULD FIX**: exacte-dag-tier zonder venster, ontbrekende
  self-closing, geen retry/alert, tijdzone-drift.
- **NICE TO HAVE**: logging-verrijking, mooiere escalatie.

## Output
Schets het scenario (wie krijgt wat, wanneer, hoe vaak) + file:line + fix.
