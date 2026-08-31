# E-mail & notificaties

Mail is een eigen subsysteem geworden (wachtrij, verstuurvenster, bounces,
mail-logboek, WhatsApp/SMS-kanaal). De terugkerende bugs zitten niet in de
tekst maar in de bezorging en de links.

## Wat te checken
- **Elke link in een mail naar een externe partij werkt ZONDER sessie.**
  Magic-link/token verplicht — en óók op elke sub-resource achter die
  pagina (PDF-download, foto's, exports). Dit ging al 3× mis (403 op
  gemailde PDF-links). Denk-test: werkt de link in een incognito-venster?
- **Afbeeldings-URLs in mails zijn absoluut** en gebouwd op de publieke
  app-URL (`appBaseUrl`/`APP_URL`) — niet relatief, niet op
  `NEXTAUTH_URL` of localhost. Mailclients laden geen relatieve paden.
- **Wachtrij-idempotency**: een retry, herstart of dubbel gedraaide cron
  mag niet twee keer bezorgen — dedupe-key of status-transitie die een
  tweede verzending uitsluit.
- **Verstuurvenster** gerespecteerd voor bulk/reminders (werkdag-venster);
  wat buiten het venster valt wordt geparkeerd, niet gedropt.
- **Bounce-pad aanwezig**: nieuwe mailstroom naar medewerkers/externen →
  komt een bounce in het bounce-log terecht en wordt iemand genotificeerd?
- **Header-injection via het choke-point**: ontvanger/onderwerp met
  user-input alleen via de gedeelde gesanitiseerde mail-helper, nooit een
  eigen nodemailer-aanroep ernaast.
- **Mail-falen blokkeert de business-flow niet** (best-effort + logging);
  maar stil falen zonder log/warning is óók fout — zeker als een
  4-ogen-flow ervan afhangt.
- **Geen PII in het onderwerp** (namen mag, BSN/gezondheid/dossier-inhoud
  nooit); taal en aanhef passend bij de ontvanger.

## Wat GEEN issue is
- Dev-mails naar MailHog; test-fixtures.
- Interne notificaties (in-app) zonder mail-component.

## Severity
- **BLOCKING**: sessie-vereiste link in een mail naar een externe partij;
  dubbele bezorging mogelijk; eigen mail-pad buiten het choke-point.
- **SHOULD FIX**: relatieve/verkeerde beeld-URLs, ontbrekend bounce-pad,
  bulk buiten het verstuurvenster, stil mail-falen.
- **NICE TO HAVE**: toon/aanhef-polish, logboek-verrijking.

## Output
Beschrijf wat de ontvanger ziet als het misgaat + file:line + fix.
