# Accessibility (WCAG 2.1 AA)

In Nederland wettelijk verplicht voor veel software (Tijdelijk besluit
digitale toegankelijkheid, voor publieke + veel B2B). Plus: 1 op 5 mensen
heeft een vorm van beperking. Toegankelijke code = bredere bruikbaarheid.

## 1. Semantic HTML

### Wat te checken
- **`<div>` of `<span>` met `onClick`** zonder rol/tabindex → niet bereikbaar via keyboard. Gebruik `<button>` of voeg `role="button" tabIndex={0} onKeyDown` toe.
- **`<a href="#">`** als knop → gebruik `<button>`. Links navigeren, knoppen acteren.
- **Headings (`<h1>`-`<h6>`) overslaan** of meerdere `<h1>` per pagina.
- **Tabellen** met `<div>` opgebouwd i.p.v. `<table>` → screen readers kunnen niet door rijen/kolommen navigeren.
- **Lijsten** als losse `<div>`'s i.p.v. `<ul>`/`<ol>` + `<li>`.
- **Forms** zonder `<form>` element — submit op Enter werkt niet, browser-autofill werkt slecht.

## 2. ARIA — alleen waar nodig

### Wat te checken
- **`aria-label`** op icon-only buttons (`<Button><X /></Button>` → screen reader hoort niets).
- **`aria-live="polite"`** op dynamische content (toast notifications, status updates).
- **`aria-expanded`** op collapsibles, `aria-haspopup` op menus.
- **`aria-describedby`** om fouten/hints aan inputs te koppelen.
- **`aria-current="page"`** op actieve nav-item.

### Anti-patronen
- **Overuse**: `role="button"` op een `<button>` is redundant. Liever native dan ARIA.
- **`aria-hidden="true"`** op interactieve elementen → onbereikbaar.
- **Fake screen-reader-only text** terwijl `aria-label` ook had gekund.

## 3. Keyboard navigation

### Wat te checken
- Alle interactieve elementen bereikbaar met **Tab**, in logische volgorde.
- **Escape** sluit modals, menus, popovers.
- **Enter** activeert primaire actie in form.
- **Arrow keys** in radio groups, menus, comboboxes (Headless UI / shadcn doet dit gratis).
- **Skip link** ("naar inhoud") bovenaan pagina voor screen-reader users.
- Geen **keyboard traps**: kun je een modal verlaten zonder muis?

### Anti-patronen
- `outline: none` zonder vervangende focus-style → onzichtbaar waar je bent met keyboard.
- `tabindex > 0` → verstoort natuurlijke tab-order. Gebruik 0 of -1.

## 4. Focus management

### Wat te checken
- **Modal opent**: focus naar eerste interactieve element binnen modal.
- **Modal sluit**: focus terug naar trigger-element.
- **Route change**: focus naar `<h1>` of `<main>` van nieuwe pagina (screen reader leest dan opnieuw context).
- **Form submit error**: focus naar eerste foute veld (of summary).
- **Dynamic content toegevoegd**: aankondigen via `aria-live`, eventueel focus verplaatsen.

## 5. Contrast & visueel

### Wat te checken
- **Tekst-contrast ≥ 4.5:1** voor normale tekst, **≥ 3:1** voor grote tekst (18pt+ of 14pt+ bold).
- **UI-componenten (knopranden, focus indicators) ≥ 3:1**.
- **Niet alleen kleur** als indicator (rood/groen status, links alleen door kleur onderscheiden). Voeg icoon of tekst toe.
- **Donker mode** ook getest op contrast (vaak grijs-op-donkergrijs te laag).

### Tools
- Chrome DevTools → Lighthouse audit
- axe DevTools extensie

## 6. Forms & labels

### Wat te checken
- Elk input heeft een **`<label>`** met `htmlFor` of wrap → click op label focust input + grotere tap-area.
- **Placeholder is geen label** — verdwijnt bij typen, vaak te laag contrast.
- **Required fields** zichtbaar gemarkeerd én via `aria-required` of `required` attribute.
- **Error messages** gekoppeld via `aria-describedby={errorId}` en `aria-invalid="true"` op input.
- **Field hints** ook via `aria-describedby` (niet alleen visueel).
- **Fieldset/legend** voor radio/checkbox groups.

## 7. Images & media

### Wat te checken
- **`<img>` met `alt`** — beschrijvend, of `alt=""` voor decoratieve (niet weglaten).
- **Icons als img**: alt-tekst of `aria-label` op parent.
- **Avatars/profielfoto's**: alt = naam van persoon.
- **Charts/graphs**: alt-tekst met de "story" + langere beschrijving elders.
- **Video's**: captions/ondertiteling voor audio content.

## 8. Reduced motion

### Wat te checken
- `@media (prefers-reduced-motion: reduce)` respect — auto-playing animations uit.
- Geen heftige transities die epileptische gevoelig veroorzaken (>3 flashes per seconde).
- Parallax, auto-rotating carousels: pauzeer-knop + reduced motion off.

## 9. Schaalbaarheid

### Wat te checken
- **Font-size in `rem`**, niet `px` — respecteert browser-zoom.
- **Layout houdt stand bij 200% zoom** (WCAG vereist).
- Geen tekst in afbeeldingen (kan niet vertaald/herschaald).

## 10. Taal

### Wat te checken
- `<html lang="nl">` of correcte locale.
- Bij meertalig: `lang="en"` op losse Engelse stukken zodat screen reader correct uitspreekt.

## 11. Screen reader testing

Geen substituut voor echt testen. Maar code-signals:
- Logische DOM-volgorde (matcht visueel).
- Geen `display: contents` op interactieve wrappers (kan focus verbreken).
- Tooltip-content ook beschikbaar voor SR (`aria-describedby`).

## Wat GEEN issue is
- Interne admin-tools met getrainde users → wel relevant, maar pragmatisch minder strikt.
- Test fixtures, prototype-pagina's.
- Print-stylesheets.

## Severity
- **BLOCKING**: keyboard trap, no-text alternative on critical action, contrast onder 4.5:1 op primaire tekst.
- **SHOULD FIX**: ontbrekend `aria-label`, focus niet gemanaged in modal, foutmelding niet gekoppeld aan input.
- **NICE TO HAVE**: betere semantic HTML, focus-state visueel verbeteren.

## Output
Verwijs naar WCAG-criterium waar mogelijk (bv. "WCAG 2.4.3 Focus Order") + concrete fix.
