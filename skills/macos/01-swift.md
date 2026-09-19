# Swift 6 — taal en concurrency

Voor projecten in Swift 6 met strikte concurrency aan. Aanvullend op
`18-concurrency.md` — die gaat over verloren updates in een database, dit gaat
over data races en isolatie in Swift zelf.

## 1. Strikte concurrency

### Wat te checken
- **Types die een actor-grens oversteken zijn `Sendable`.** Een model dat van een
  bron-actor naar de UI reist en niet `Sendable` is → compilerfout of, erger, een
  `@unchecked Sendable` die het probleem wegmoffelt.
- **`@unchecked Sendable` is een rode vlag.** Alleen acceptabel bij een type dat
  z'n eigen slot beheert, met een comment dat uitlegt waarom. Zonder uitleg:
  should fix.
- **`nonisolated(unsafe)`** idem — dat is "zet de controle uit". Vraag waarom.
- **`@preconcurrency import`** is een tijdelijke pleister voor oude API's. Bij
  nieuwe code zonder externe afhankelijkheid: onnodig.
- **Een actor die de UI aanraakt.** SwiftUI-views zijn `@MainActor`; een bron-actor
  die direct een view of `@State` bijwerkt hoort niet. Lever een `Sendable`
  resultaat op en werk de stand op de main actor bij.
- **`@MainActor` op een heel type** waar alleen één methode het nodig heeft →
  trekt onnodig werk naar de hoofdthread.

## 2. Taken en levensduur

### Wat te checken
- **`Task { }` zonder eigenaar** in een langlevende app: wie annuleert die? Een
  taak die aan een view hangt hoort `.task { }` te zijn, zodat hij meegaat met de
  levensduur van de view.
- **`Task.detached`** verliest de context (actor, prioriteit, cancellatie). Bijna
  altijd verkeerd; vraag om de reden.
- **Lange lussen zonder `Task.checkCancellation()`** of `try await` — die kun je
  niet meer stoppen.
- **Retain cycles**: een closure die `self` vasthoudt in een `Timer`, een
  `NotificationCenter`-observer of een langlevende `Task` → `[weak self]`.
- **Observers worden opgeruimd.** Een `NotificationCenter`-abonnement zonder
  tegenhanger blijft vuren nadat het object weg is.

## 3. Crashes zijn hier duurder dan elders

Een menubar-app draait uren achter elkaar. Een crash is niet "een pagina die
faalt" maar **de app die stil verdwenen is** — de gebruiker merkt het pas als hij
iets mist.

### Wat te checken
- **Force unwrap (`!`)**, `try!`, `as!` en `array[i]` zonder grensbewaking op een
  pad dat door externe data loopt (commando-uitvoer, JSON, netwerk) → weiger.
- **`fatalError` / `preconditionFailure`** in een normaal pad. Alleen acceptabel
  voor programmeerfouten die nooit kunnen voorkomen, met comment.
- **`try?` die een fout stil wegslikt.** Als het resultaat optioneel wordt
  genegeerd, verdwijnt de reden. Log 'm of geef 'm door.
- **Een lege `catch { }`** — zelfde punt, scherper.

## 4. Kleine dingen die vaak fout gaan

### Wat te checken
- **`DateFormatter` in een lus of in `body`** — dat is duur. Hergebruik één
  instantie, of gebruik `ISO8601DateFormatter` / `Date.FormatStyle`.
- **Synchrone I/O op de main actor**: `Data(contentsOf:)`, `String(contentsOf:)`,
  `Process.waitUntilExit()` → de UI bevriest. Naar een achtergrondcontext.
- **`class` waar een `struct` kan.** Referentietypes delen staat; dat is precies
  wat je in een concurrency-model niet wil.
- **Exhaustiviteit**: een `switch` over een enum zonder `default` laat de compiler
  een nieuwe case vangen. Een `default` die stil iets doet, verbergt 'm.
- **Gedwongen `DispatchQueue.main.async`** in nieuwe code — gebruik
  `await MainActor.run { }` of markeer de functie `@MainActor`.

## Wat geen issue is (om false positives te voorkomen)
- Force unwrap op een letterlijke, in code vastgelegde waarde
  (`URL(string: "https://example.com")!`) — dat kan niet falen en is leesbaarder.
- `@MainActor` op een view-model dat inderdaad alleen de UI voedt.
- `Task { }` in een knop-actie die kort is en bewust niet annuleerbaar hoeft.
- `as!` in een test, waar een crash de juiste uitkomst is.

## Voorbeeld

```swift
// FOUT — bron-actor werkt de UI bij, en de fout verdwijnt
actor GitHubBron {
    func ververs(_ stand: Stand) async {
        let regels = try? await haalOp()      // fout weggeslikt
        stand.wachtOpJou = regels!            // crash op nil, én verkeerde actor
    }
}

// GOED — levert een Sendable resultaat op, de aanroeper zet het op de main actor
actor GitHubBron {
    func haalWachtregelsOp() async throws -> [Wachtregel] { … }
}

@MainActor
func ververs() async {
    do   { stand.wachtOpJou = try await bron.haalWachtregelsOp() }
    catch { stand.fouten[.github] = "GitHub niet bereikbaar: \(error.localizedDescription)" }
}
```
