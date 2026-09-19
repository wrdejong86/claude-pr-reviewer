# SwiftUI op macOS

Voor native Mac-apps in SwiftUI, inclusief menubar-apps. Aanvullend op
`10-accessibility.md`; `04-react.md` en `07-mobile.md` gelden hier **niet** — dit
is geen webapp en geen telefoon.

## 1. `body` is geen plek om werk te doen

Een `body` kan tientallen keren per seconde opnieuw draaien. Alles wat erin staat
draait dan ook opnieuw.

### Wat te checken
- **Geen I/O, geen `Process`, geen netwerk, geen `FileManager` in `body`** of in
  een berekende property die `body` gebruikt.
- **Geen zware berekening of sortering in `body`** — zet die in het model, of
  cache het resultaat.
- **Geen `DateFormatter()` of `JSONDecoder()` in `body`** — instantieer één keer.
- **Werk starten hoort in `.task { }`**, niet in `.onAppear` met een losse `Task`.
  Gebruik `.task(id:)` als het opnieuw moet bij een gewijzigde waarde.

## 2. Menubar-apps

### Wat te checken
- **Het `label` van `MenuBarExtra` moet goedkoop zijn** — dat wordt vaak
  hertekend. Geen berekening, geen formatter.
- **`.menuBarExtraStyle(.window)`** is nodig voor een eigen view; de standaard
  (`.menu`) accepteert alleen menu-achtige inhoud.
- **`LSUIElement`** in `Info.plist` als de app alleen in de menubalk hoort — anders
  staat hij ook in het Dock.
- **Het icoon draagt informatie** (rust / wacht / stuk). Kleur alléén is niet
  genoeg: zorg voor een vorm- of symboolverschil, of een tekstlabel in het menu.
  Zie `10-accessibility.md`.
- **Een menubar-app draait uren.** Let op groeiend geheugen: caches zonder
  plafond, arrays die alleen maar aangroeien, observers die nooit opgeruimd worden.

## 3. Licht en donker

### Wat te checken
- **Semantische kleuren**: `Color.primary`, `.secondary`,
  `Color(nsColor: .controlBackgroundColor)`. **Geen vaste hex** voor iets dat
  betekenis draagt.
- **Een eigen kleur hoort in de asset catalog met een licht- én donkervariant**,
  niet als losse `Color(red:green:blue:)`.
- **Getest in beide standen.** Een PR die kleur aanraakt zonder te melden dat
  beide standen bekeken zijn: should fix.
- **Contrast ≥ 3:1** voor betekenisdragende iconen (WCAG 1.4.11). `.orange` op wit
  is een grensgeval.

## 4. Staten: leeg, fout, bezig

Dit is waar Mac-apps het vaakst liegen tegen hun gebruiker.

### Wat te checken
- **Een lege lijst en een mislukte ophaal zijn niet hetzelfde.** "Geen resultaten"
  waar eigenlijk "kon niet ophalen" hoort, is een bug — geen cosmetisch punt.
- **Elke bron die kan falen heeft een zichtbaar foutpad**, met de reden erin.
- **Bezig-staat** bij werk dat langer dan een halve seconde duurt.
- **Versheid tonen** bij gegevens van buiten: "laatst ververst …". Een overzicht
  zonder stempel wordt op een gegeven moment niet meer vertrouwd.

## 5. Structuur

### Wat te checken
- **Geen bedrijfslogica in een view.** Een view leest de stand en toont 'm.
- **`AnyView` zonder reden** — verliest type-informatie en dwingt hertekenen.
  Gebruik `@ViewBuilder` of `Group`.
- **Stabiele identiteit in lijsten**: `Identifiable` met een echte id, niet de
  array-index. Bij een index schuift de selectie mee zodra de lijst wijzigt.
- **Vaste `.frame(width:)`** op iets dat in een verkleinbaar venster staat → de
  inhoud wordt afgekapt. Denk aan minimum- in plaats van vaste maten.
- **`@Observable`** (Observation) is in Swift 6 te prefereren boven
  `ObservableObject` + `@Published` voor nieuwe modellen.

## Wat geen issue is (om false positives te voorkomen)
- Een vaste breedte op een popover of menubar-venster dat bewust niet meeschaalt.
- Een simpele `Text(getal.formatted())` in `body` — dat is goedkoop.
- Kleuren zonder donkervariant in een decoratief element dat geen betekenis draagt.
- `.onAppear` voor iets dat echt eenmalig en synchroon is.
- Geen responsive-eisen zoals bij web: 375px breed en touch-targets zijn hier niet
  van toepassing.

## Voorbeeld

```swift
// FOUT — werk in body, en leeg is niet hetzelfde als mislukt
var body: some View {
    let prs = try? shell("gh pr list --json title")     // I/O in body
    List(prs ?? []) { Text($0.title) }                  // faalt = "leeg"
}

// GOED
var body: some View {
    Group {
        if let fout = stand.fouten[.github] {
            Label(fout, systemImage: "exclamationmark.triangle")
                .foregroundStyle(.secondary)
        } else if stand.wachtOpJou.isEmpty {
            Text("Niets wacht op jou").foregroundStyle(.secondary)
        } else {
            List(stand.wachtOpJou) { WachtregelRij($0) }
        }
    }
    .task { await ververs() }
}
```
