# Mobile-first / responsive

Alle wijzigingen moeten mobile-first ontworpen zijn. Smallest screen eerst,
dan progressively enhancen naar groter. Pas onderstaande secties toe op
basis van wat de PR raakt (UI-component → layout & touch; new endpoint
ophalen → performance & network; form → input & keyboard).

## 1. Layout & viewport
- **Tailwind breakpoint-strategy**: base styles = mobile. `sm:` `md:` `lg:` `xl:` voor groter. Nieuwe component zonder enige breakpoint-modifier = waarschijnlijk desktop-only gedacht.
- **Geen hardcoded px-breedtes** die niet schalen: `w-[1200px]`, `min-w-[800px]`, `max-w-[600px]` op een hele pagina.
- **Geen horizontale scroll** op de body. Check op `overflow-x: hidden` workarounds — die verbergen het echte probleem.
- **Container queries** (`@container`) voor componenten die in verschillende parents zitten — beter dan viewport breakpoints.
- **Safe areas** op iPhone (notch / home-bar): `env(safe-area-inset-*)` voor fixed elements (headers, FABs, bottom-nav).
- **Orientatie**: werkt het ook in landscape op telefoon? Modals/keyboards laten weinig ruimte over.

## 2. Touch & input
- **Touch targets ≥ 44×44px** (Apple HIG) / 48×48dp (Material). Iconen, checkboxes, close-knoppen vaak te klein.
- **Spacing tussen targets** ≥ 8px om mis-taps te voorkomen.
- **Hover-only interactie** = bug op mobiel. Elke `hover:` moet een `focus:` / `active:` / `aria-expanded` tegenhanger hebben.
- **Tooltips** op `:hover` werken niet op touch — gebruik tap-toggle of long-press.
- **Drag-and-drop** zonder touch-fallback: HTML5 drag werkt niet op touch. Gebruik bibliotheken die pointer-events ondersteunen (dnd-kit etc).
- **Swipe-gestures** alleen toevoegen als ze duidelijk zijn — anders verborgen functionaliteit.
- **Pull-to-refresh** niet per ongeluk breken door `overscroll-behavior` verkeerd te zetten.

## 3. Forms & keyboard
- **`inputMode`** of `type=` voor juist toetsenbord:
  - `type="email"` → `@` zichtbaar
  - `type="tel"` → numpad
  - `inputMode="numeric"` → numbers zonder telefoon-UI
  - `inputMode="decimal"` → komma voor bedragen
- **`autocomplete`** ingevuld voor autofill (`name`, `email`, `tel`, `postal-code`, `street-address`, `bday`, `current-password`, `new-password`).
- **`enterkeyhint`** voor zinnige Enter-actie ("done", "search", "next", "send").
- **Font-size ≥ 16px** op inputs — anders zoomt iOS in.
- **Keyboard bedekt input**: scroll niet automatisch goed? Gebruik `scrollIntoView({ block: 'center' })` bij focus.
- **Submit-knop blijft bereikbaar** als keyboard open is — niet onder de keyboard verstopt.
- **`<label>`** correct gekoppeld voor screen readers + grotere tap-area.

## 4. Performance (zwakke CPU + tragere netwerk)
- **Bundle size**: dependencies van >50kB gzipped die voor 1 feature worden toegevoegd → vraag of er een lichtere is.
- **Lazy loading**: routes / zware componenten via `next/dynamic` of `React.lazy`. Grote pagina's niet in initial bundle.
- **Image optimization**:
  - `<Image>` van Next/img component i.p.v. `<img>`
  - `loading="lazy"` op below-the-fold images
  - Juiste `sizes` attr voor srcset
  - Modern format (webp/avif)
  - Niet 4K profielfoto's serveren op telefoon
- **Fonts**: `font-display: swap`, `<link rel="preload">` voor kritieke fonts, geen blocking @import.
- **Animation/transitions** op `transform`/`opacity` (GPU), niet `width`/`height`/`top`/`left` (layout).
- **Scroll listeners** met `passive: true` en gethrottled — anders janky scroll.
- **Avoid layout shifts** (CLS): images met `width`/`height`, ruimte reserveren voor async content, geen content die intvoegt boven viewport.
- **Web vitals** op mobile (3G simulated, low-end Android): LCP <2.5s, CLS <0.1, INP <200ms.

## 5. Network & data
- **Loading states**: skeleton screens of spinners — nooit langer dan 200ms blanco scherm.
- **Optimistic UI** voor mutations die meestal slagen (likes, toggles, simpele saves).
- **Error states** netjes: niet alleen "Er ging iets mis" — geef retry-knop en wat de gebruiker kan doen.
- **Offline-aware**: kritieke acties moeten queue-en bij offline, niet stilletjes falen.
- **Avoid waterfalls**: parallel data-fetching i.p.v. nested awaits.
- **Pagination/virtualization** voor lange lijsten (>50 items) — laad niet alles tegelijk.

## 6. Layout-patronen mobiel
- **Tabellen** met veel kolommen:
  - Op `<md`: cards met label-value paren
  - Of horizontal scroll met sticky first column
  - Of `hidden md:table-cell` op secundaire kolommen
- **Sidebars** verstoppen op mobiel in een drawer/sheet (Sheet-component van shadcn etc).
- **Multi-column** layouts → single column op mobiel.
- **Modals** met `max-h-[90vh] overflow-y-auto`, anders overflowen ze. Of full-screen op mobiel.
- **Sticky headers/footers**: max ~10% van viewport — telefoon heeft maar ~600px hoogte.
- **Bottom-nav** in plaats van top-tabs voor primary navigation (duim-bereik).
- **Floating Action Button** (FAB) in onderhoek voor primaire actie.

## 7. Tekst & content
- **`break-words`** of `truncate` op velden die lang kunnen zijn (e-mail, naam, URL).
- **`text-balance`** op koppen voor mooiere line-breaks.
- **Niet te veel info per scherm** — prioriteit: meest gebruikte bovenaan, secundair in expand/collapse.
- **Leesbare line-length** (max ~70 char) — op mobiel niet zo'n issue, op tablet wel.
- **Donker mode** via `dark:` modifiers waar relevant.

## 8. Accessibility op mobile
- **Contrast** ≥ 4.5:1 voor tekst (WCAG AA) — vooral bij dunne fonts op witte achtergrond.
- **`prefers-reduced-motion`**: animaties uitschakelen voor users die het aangevinkt hebben.
- **Screen reader**: VoiceOver/TalkBack labels via `aria-label` op icon-only knoppen.
- **Font-size respecteert** browser/OS instelling (gebruik `rem`, niet `px`).
- **Focus-states zichtbaar** ook op touch (niet `:focus { outline: none }` zonder vervanging).

## 9. PWA / installable (waar relevant)
- **Viewport meta** correct: `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`.
- **Theme color** voor browser-chrome: `<meta name="theme-color">`.
- **Apple touch icons** voor "Add to home screen".
- **Manifest.json** + service worker voor offline (alleen als product PWA-aspiratie heeft).

## 10. HR-specifieke aandachtspunten
- **Personeels-detail** wordt ook door medewerker zelf op telefoon bekeken — alles leesbaar op smal scherm.
- **Verlof goedkeuren / aanvragen** moet met één duim te doen zijn (manager in trein).
- **Document upload** moet camera kunnen gebruiken: `accept="image/*,application/pdf" capture="environment"`.
- **Magic-links** (zoals de ID-document update flow) worden op telefoon geopend — landing pages moeten mobile-first zijn.
- **E-mail templates** (zoals herinneringen) moeten responsive zijn — getest in Gmail mobile, Outlook mobile, Apple Mail.
- **Inloggen vanaf telefoon** moet snel: biometrische auth ondersteunen waar mogelijk.

## Wat GEEN issue is
- Admin-only pagina's (system-settings, role-management, organisatie-config) — legitiem desktop-only.
- Print-views en exports (PDF generatie) — niet relevant voor mobile.
- Internal dashboards die alleen door HR-admins in kantoor gebruikt worden — vraag wel even na of dat klopt.
- Migraties / backend-only changes.

## Output-format
- Wees concreet: noem het component, de file:line, en op welk breakpoint of welk device het breekt.
- Bij twijfel "is dit mobile relevant?": flag het, vraag of de target-user ook medewerker is (niet alleen HR-admin).
- Groepeer findings per sectie hierboven zodat het overzichtelijk leesbaar is.
