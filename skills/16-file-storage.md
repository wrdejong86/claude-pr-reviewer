# File- & storage-security

Een HR-app verstuurt én serveert documenten, foto's, handtekeningen en PDF's.
Dit is een aparte risico-as náást upload-validatie (zie `14-forms` §7): het
gaat hier vooral om het **serveren/downloaden** van files en de consistentie
tussen database en opslag. Dit is in deze codebase het meest voorkomende type
beveiligingsbug.

## 1. Path traversal op file-routes

### Wat te checken
- **User-controlled id/pad** in een file-route (`/files/[id]`, `/onboarding-file?id=...`):
  wordt het pad opgebouwd met user-input? `../`, absolute paden, of een
  `id` die direct als bestandsnaam wordt gebruikt = path traversal.
- **Geen pad uit input**: zoek het bestand op via een **DB-record op id**, niet
  door de input als (deel van) een pad te plakken.
- Bij een onvermijdelijk pad: **normaliseer** (`path.normalize`) én verifieer
  dat het resultaat **binnen** de verwachte basismap valt (`resolved.startsWith(baseDir)`).
- **Extensie/segmenten whitelisten** waar van toepassing.

### Voorbeeld
```ts
// FOUT — id direct in pad
const buf = await fs.readFile(`/uploads/${params.id}`); // ?id=../../.env

// BETER — DB-lookup, pad nooit uit user-input
const doc = await db.document.findUnique({ where: { id: params.id } });
if (!doc) notFound();
// + ownership-check (zie §2), dan pas de (door ons opgeslagen) storageKey gebruiken
```

## 2. Authorization op elke download/serve

### Wat te checken
- **Auth + ownership/rol-check op de file-route zelf** — niet alleen op de
  pagina die ernaar linkt. Een directe GET op de file-URL moet óók beschermd zijn.
- **IDOR op file-id**: mag deze user juist dít document/foto/dossier zien?
  (tenant/organization/employee-scope.)
- **Rol-hercheck bij gevoelige downloads** (dossier-export, salaris-PDF):
  niet vertrouwen op een eerdere check elders in de flow.
- **404 i.p.v. 403** waar enumeration een risico is.

## 3. Token-scope + expiry op file-routes (magic links)

### Wat te checken
- Token-beveiligde file-routes (Agreement-PDF, onboarding-file, attachments):
  check **niet alleen "token bestaat"**, maar ook:
  - **`expiresAt`** — verlopen token weigeren.
  - **scope** — hoort dit token bij déze resource (programId/employeeId/document)?
  - **one-time-use** — `usedAt` zetten waar van toepassing.
- Zie ook `09-gdpr-avg` §9 en `02-security` (auth bypass).

## 4. Content-validatie van uploads (aanvullend op 14-forms §7)

### Wat te checken
- **Magic bytes**, niet alleen extensie/MIME (voorkom `.exe` als `.pdf`).
- **Server-side size cap** — niet vertrouwen op client.
- **EXIF strippen** bij foto's (locatie/GPS = PII) en bij voorkeur **re-encoden**.
- **SVG** alleen toestaan na sanitatie (kan script bevatten → stored XSS).

## 5. DB ↔ storage consistentie

### Wat te checken
- **Create**: als de DB-rij wordt aangemaakt en de upload faalt (of omgekeerd),
  blijft er rommel achter. Rollback de DB-rij bij mislukte `storeUpload`, of
  schrijf pas naar DB ná geslaagde upload.
- **Delete**: verwijder bijbehorende files op disk/in de bucket — anders
  **wees-files** (orphans) die blijven groeien en PII bevatten.
- **Volgorde & atomiciteit**: blob-store en DB zijn geen gedeelde transactie;
  kies een veilige volgorde + compensatie bij falen (zie `12-error-handling` §12).

## 6. Niet over-fetchen van binaire velden

### Wat te checken
- Grote binaire/base64-velden (bv. `Signature.drawnImage`, foto-blobs) **niet**
  meeladen in lijst-/detail-queries die ze niet tonen — `select` expliciet,
  geen brede `include`. Scheelt geheugen, latency én onnodige PII-exposure
  (zie `05-database` en `09-gdpr-avg` §2).

## 7. Opslag-locatie

### Wat te checken
- **Geen user-files in een publiek/web-served pad** zonder auth (anders
  raden = downloaden).
- Bij object-storage: **private bucket** + **signed, kortlevende URLs**, geen
  permanente publieke links naar PII.

## Wat GEEN issue is
- Statische publieke assets (logo's, marketing-afbeeldingen).
- Test-fixtures met dummy-files.
- Interne admin-tooling met audit-trail en getrainde gebruikers (wel even
  nagaan of de file echt PII bevat).

## Severity
- **BLOCKING**: path traversal, IDOR / ontbrekende ownership-check op een
  file-route, token-route zonder expiry/scope-check.
- **SHOULD FIX**: wees-files bij delete, geen rollback bij mislukte upload,
  over-fetch van blob-velden, EXIF/PII niet gestript.
- **NICE TO HAVE**: re-encode van uploads, signed URLs i.p.v. auth-proxy.

## Output
Bij elke vondst: noem de route/actie + file:line, wat een aanvaller of bug kan
bereiken, en de concrete fix.
