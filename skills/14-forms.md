# Forms & validation

Forms zijn het meest dichte interactie-oppervlak van een HR-app. Hier zit
het verschil tussen "werkt prima" en "klant belt support". Aanvullend op
de `accessibility` skill (form labels, errors-koppelen).

## 1. Validatie-laag

### Wat te checken
- **Zod schema** als single source of truth voor types + validatie.
- **Gedeeld tussen client en server** — schema importeren in beide kanten, niet dupliceren.
- **Server valideert altijd opnieuw** — frontend-validatie is UX, geen security.
- **Async validatie** (bv. "is dit e-mailadres uniek?") via custom resolver of `refine`.

## 2. Default values

### Wat te checken
- **`defaultValues`** prop op `useForm` — niet `undefined` voor controlled inputs (React warning).
- **Edit-mode**: defaults vullen vanuit bestaande data, niet leeg starten.
- **Date pickers**: default value als `Date` of ISO-string consistent — geen mix.
- **Selects**: default als geldige optie (anders blanco eerste keer).

## 3. Field-level state

### Wat te checken
- **`touched`** — toon validatie-error pas na first blur, niet meteen bij typen.
- **`dirty`** — knop "Opslaan" alleen actief als er iets gewijzigd is, of confirmation dialog bij navigatie weg met onopgeslagen wijzigingen.
- **`isSubmitting`** — submit-knop disable + spinner tijdens submit.
- **`isValid`** — eventueel submit-knop disable als form invalid (niet altijd nodig).

## 4. Error display

### Wat te checken
- **Per-field error** onder het veld, gekoppeld via `aria-describedby` (zie a11y skill).
- **Form-level error** (bv. "Server-fout") bovenaan, met `role="alert"`.
- **Errors verdwijnen** zodra user corrigeert, niet pas bij volgende submit.
- **Nederlandse berichten**, niet "Field required" maar "Vul een waarde in".
- **Specifiek**: niet "Invalid input", wel "E-mailadres moet een @-teken bevatten".
- **Server-errors** mappen naar juiste field (bv. "Naam is al in gebruik" → bij naam-veld).

## 5. Submit UX

### Wat te checken
- **Disable submit-knop tijdens submit** — voorkomt dubbele submit.
- **Loading state** zichtbaar (spinner in knop, of overlay).
- **Success feedback** — toast, redirect, of inline message.
- **Form reset** na success (of niet, afhankelijk van flow).
- **Focus terug naar eerste foute veld** bij failed submit.
- **Scroll naar eerste error** als deze niet in viewport is.

## 6. Multi-step forms

### Wat te checken
- **State preserved** tussen stappen (niet via component-state die unmount).
- **Back-button werkt** — niet alleen "Vorige" knop.
- **Progress indicator** zichtbaar.
- **Validatie per stap** — geen stap overslaan met invalid data.
- **Draft-save** bij afsluiten — gebruiker mag niet 5 stappen kwijt zijn.

## 7. File uploads

### Wat te checken
- **`accept`** attribuut (MIME-types, niet extensies).
- **Max file size** validatie client + server.
- **Progress indicator** voor grote files.
- **Resume** bij failed upload (bonus).
- **Mobile**: `capture="environment"` voor camera.
- **Multiple uploads**: list met remove-knoppen voor selected files vóór submit.
- **Drag-and-drop** met touch-fallback (zie mobile skill).
- **Server**: validate file content (magic bytes), niet alleen extension. Voorkom RCE via .exe als .pdf.

## 8. Date / time inputs

### Wat te checken
- **Native `<input type="date">`** voor simpele cases — werkt overal, mobiel toetsenbord goed.
- **Custom date picker** alleen als nodig (date range, week picker, etc.). Library: react-day-picker, headless options.
- **Locale**: dd-mm-yyyy voor NL gebruikers, niet US-formaat.
- **Time zones**: opslaan als UTC, tonen in user's tijdzone. Niet door elkaar.
- **Min/max** voor logische bereiken (geen geboortedatum in toekomst).

## 9. Conditional fields

### Wat te checken
- **Show/hide fields** moeten ook **unregister** uit form state, anders worden ze toch gevalideerd/submitted.
- React Hook Form: `{ shouldUnregister: true }` of expliciet `unregister(...)`.
- **Animations** subtiel — geen jarring jumps in layout.

## 10. Autosave / draft

### Wat te checken
- Voor lange forms: autosave naar localStorage of server elke X seconden.
- **Conflict resolution** als user op meerdere tabs werkt.
- **Indicator** ("Draft opgeslagen 2 min geleden").

## 11. Confirmation patterns

### Wat te checken
- **Destructieve acties** (delete, archive) → modal met expliciete bevestiging.
- **Type-the-name** voor super-destructive (delete organization).
- **Onopgeslagen wijzigingen** bij navigatie weg → `beforeunload` of in-app modal.

## 12. Specifiek voor HR-forms

### Wat te checken
- **BSN-veld**: validatie (11-proef), masking tijdens typen (display laatste 4 cijfers).
- **IBAN-veld**: format + checksum validatie.
- **Telefoonnummer**: Nederlandse format-helper (`libphonenumber-js`).
- **Adres**: postcode-API koppelen (PostNL of vergelijkbaar) — autofill straat + plaats.
- **Geboortedatum**: leeftijd-derivation voor jeugd-loonschalen.
- **Contracttype/uren**: validatie tegen wettelijke regels (max uren onder bepaalde leeftijd, etc.).

## 13. Accessibility (zie ook a11y skill)
- `<label>` correct gekoppeld.
- Errors via `aria-describedby` + `aria-invalid`.
- `fieldset/legend` voor groepen.
- `autocomplete` attribuut voor browser-autofill.

## Wat GEEN issue is
- Eenmalige interne forms zonder gebruikersinteractie (admin scripts).
- Quick proof-of-concepts.

## Severity
- **BLOCKING**: server-side validatie ontbreekt, file upload zonder type/size check.
- **SHOULD FIX**: dubbele submit mogelijk, errors niet aan field gekoppeld (a11y).
- **NICE TO HAVE**: autosave, betere format-hints, conditional unregister.

## Output
Bij elke vondst: noem het form/veld + concrete UX-impact + fix.
