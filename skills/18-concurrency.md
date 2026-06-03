# Concurrency & verloren updates

Meerdere acties, tabs of requests op dezelfde data kunnen elkaars wijzigingen
overschrijven (last-write-wins). Dit is subtiel én datadestructief, en komt in
deze codebase terug bij autosave en "complete"-flows.

## 1. Lost update / last-write-wins

### Wat te checken
- Een write die het **hele record** overschrijft met (mogelijk verouderde)
  clientdata wist tussentijdse serverwijzigingen (zie `completeOnboarding
  overschrijft handtekening niet`).
- **Partial update**: schrijf alleen de velden die de actie echt bedoelt — niet
  het volledige object terugschrijven.
- Velden die door een ándere flow gezet worden (handtekening, status, audit)
  niet onbedoeld meenemen in een onverwante update.

## 2. Optimistic concurrency

### Wat te checken
- Bij resources die door meerdere mensen/tabs bewerkt kunnen worden: check een
  **`updatedAt`/version** bij de update. Conflict → 409 + herladen/mergen i.p.v.
  blind overschrijven.
- Prisma-patroon: `update({ where: { id, updatedAt: knownUpdatedAt }, ... })` →
  0 rijen geraakt = conflict.

## 3. Autosave die serverdata overschrijft bij mount

### Wat te checken
- Autosave die **bij mount** de (nog lege of verouderde) form-state naar de
  server pusht en zo verse serverdata overschrijft (zie `wizard-autosave
  overschrijft serverdata niet bij mount`).
- Fix: niet autosaven vóór de form **gehydrateerd** is; guard op de eerste
  render; alleen saven als de form echt **dirty** is door de gebruiker.

## 4. Stale closures in autosave-deps (zie 04-react)

### Wat te checken
- `useEffect`/interval/debounce die outdated state schrijft door een ontbrekende
  of foute dependency (zie `isConcept in afspraken-autosave deps`). Verkeerde
  deps → autosave schrijft een verouderde snapshot.

## 5. Dubbele submit / idempotency

### Wat te checken
- Submit-knop **disablen** tijdens submit (zie `14-forms` §5) én **server-side**
  bescherming: idempotency-key of natuurlijke uniqueness (zie `13-api-design` §5).
- Twee snelle POSTs (dubbelklik, retry) mogen geen dubbele record/mail/charge geven.

## 6. Read-modify-write races op tellers/nummers

### Wat te checken
- `findMax + 1` patroon voor volgnummers (zie `nextPersonnelNumber magnitude-
  bug`): twee gelijktijdige requests krijgen hetzelfde nummer. Gebruik een
  **DB-sequence**, atomic increment, of een transactie met de juiste lock.

## 7. Transacties voor samengestelde writes (zie 12-error §12, 05-database)

### Wat te checken
- Meerdere afhankelijke writes die samen consistent moeten zijn → `prisma.
  $transaction([...])`. Halverwege falen mag geen halve state achterlaten.

## 8. Niet-atomic "check-then-act"

### Wat te checken
- Uniqueness via **eerst SELECT, dan INSERT** is een race. Vertrouw op een
  **DB unique constraint** en vang `P2002` (Prisma) netjes af — niet alleen een
  applicatie-check (zie `duplicate-check op klanten/onderaannemers`).

## Wat GEEN issue is
- Single-user admin-scripts / migraties.
- Append-only logs (geen update-conflict mogelijk).
- Data die aantoonbaar maar door één actor tegelijk bewerkt wordt.

## Severity
- **BLOCKING**: datadestructieve lost-update (overschrijft andermans/eerdere data).
- **SHOULD FIX**: ontbrekende optimistic-lock op concurrent-bewerkbare resource,
  read-modify-write race op volgnummer, dubbele submit zonder server-guard,
  check-then-act zonder DB-constraint.
- **NICE TO HAVE**: nettere conflict-UX (merge i.p.v. herladen).

## Output
Bij elke vondst: schets het race-scenario (twee acties/tabs/requests), noem
file:line, en de concrete fix (partial update / version-check / constraint /
transactie).
