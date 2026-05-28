# Mobile-first / responsive

## Wat te checken
- **Hardcoded breedtes/heights** in px die niet schalen: `width: 1200px`, `min-width: 800px`. Gebruik `max-w-*`, `w-full`, container queries.
- **Tabellen** met veel kolommen zonder mobiele fallback (horizontal scroll, card-layout op `<md`, of `hidden md:table-cell` op secundaire kolommen).
- **Tailwind breakpoints**: nieuwe componenten zonder `sm:`/`md:`/`lg:` modifiers — meestal teken dat alleen desktop in scope was.
- **Touch targets te klein**: knoppen/iconen < 44×44px zijn lastig te tikken. Check `h-10 w-10` minimum voor tap-targets.
- **Hover-only interactie**: `hover:` zonder `focus:` of `active:` equivalent — werkt niet op touch.
- **Modals/dialogs** die niet `max-h-[90vh]` + scroll hebben — overflowen op kleine schermen.
- **Forms**: ontbrekende `inputMode` / `type="email|tel|number"` → verkeerd toetsenbord op mobiel.
  Ontbrekende `autocomplete=` → autofill werkt niet.
- **Sticky headers/footers** die te veel verticale ruimte pakken op mobiel (telefoon heeft maar ~600px hoogte).
- **Drag-and-drop** zonder touch-fallback — werkt niet op telefoon.
- **Iframes / video's** zonder `max-w-full` of `aspect-ratio` → breken het layout.
- **Lange teksten** zonder `break-words` / `truncate` → overflow op smal scherm (vooral e-mailadressen, lange namen).
- **Sidebars** die altijd zichtbaar zijn op mobiel — meestal moeten die in een drawer/sheet.

## HR-specifieke aandachtspunten
- Personeels-detail kaarten worden ook op telefoon bekeken (medewerker checkt eigen gegevens) — alle info moet leesbaar op smal scherm.
- Acties zoals "verlofaanvraag goedkeuren" moeten op telefoon met één duim te doen zijn.
- Document-upload moet camera kunnen gebruiken op mobiel (`accept="image/*" capture="environment"`).

## Wat GEEN issue is
- Admin-only pagina's (bijv. systeem-instellingen, role-management) — die zijn legitiem desktop-only.
- Print-views, exports — niet relevant voor mobile.
- Charts/grafieken die op mobiel kleiner zijn maar wel werken.

## Format
- Wees concreet: noem het specifieke component/pagina en op welk breakpoint het breekt.
- Bij twijfel: vraag of dit component ook door medewerkers (niet alleen HR-admins) gebruikt wordt.
