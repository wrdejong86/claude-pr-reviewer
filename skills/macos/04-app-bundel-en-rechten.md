# App-bundel, rechten en opslag (macOS)

Voor de buitenkant van een Mac-app: de bundel, de signatuur, de rechten die hij
vraagt, en waar hij z'n spullen neerzet. Dit is het equivalent van
`15-live-gang.md` voor een desktop-app — er is geen deploy, maar er zijn wél
dingen die pas op de machine van de gebruiker breken.

## 1. Bundel en `Info.plist`

### Wat te checken
- **`CFBundleIdentifier` en `CFBundleExecutable`** aanwezig en consistent met de
  binaire naam. Zonder geldige identiteit krijgt de app geen meldingsrechten en
  geen eigen plek voor z'n voorkeuren.
- **`LSUIElement`** op `true` voor een app die alleen in de menubalk hoort —
  anders staat hij ook in het Dock.
- **`LSMinimumSystemVersion`** gezet en gelijk aan wat het project echt
  ondersteunt.
- **Wijzigt de bundle-identifier?** Dan verliest de app z'n meldingstoestemming en
  z'n opslagmap. Dat is een migratie, niet een naamswijziging — moet in de
  PR-beschrijving staan.

## 2. Signatuur en sandbox

### Wat te checken
- **Ad-hoc ondertekenen (`codesign --sign -`) is prima voor een app die alleen op
  de eigen machine draait.** Voor verspreiding is het niet genoeg; dan hoort er
  een Developer ID en notarisatie bij.
- **App Sandbox aanzetten breekt `Process`.** Een gesandboxte app mag geen externe
  commando's starten, en de fout verwijst niet naar de sandbox. Zet een project
  dat shelt niet in de sandbox — en als iemand het tóch doet, moet dat een bewuste,
  toegelichte keuze zijn.
- **Nieuwe entitlements zijn een aandachtspunt**, ook zonder sandbox: elke
  toevoeging vergroot wat de app mag. Vraag waarom.
- **Hardened Runtime** met `allow-unsigned-executable-memory` of
  `disable-library-validation`: blocking tenzij er een harde reden is.

## 3. Meldingen

### Wat te checken
- **Toestemming wordt één keer gevraagd**, op een moment dat de gebruiker begrijpt
  waarom — niet bij de eerste start voordat hij iets gezien heeft.
- **Weigering wordt netjes opgevangen.** De app blijft werken, alleen zonder
  meldingen. Geen herhaalde prompts, geen foutmelding.
- **Meldingen alleen bij een verándering**, niet als herhaling van wat al
  zichtbaar is. Een app die elke ronde hetzelfde meldt, wordt uitgezet.
- **Een stiltevenster is geen excuus om iets ernstigs in te slikken.** Wat kapot
  is mag altijd door; wat kan wachten, wacht.
- **`UNAuthorizationStatus` wordt gecontroleerd** vóór het plannen van een melding.

## 4. Opslag op schijf

### Wat te checken
- **`~/Library/Application Support/<bundle-id>/`** voor gegevens die de app zelf
  beheert. **Niet** `~/Documents`, niet de map naast de binary, niet `/tmp` voor
  iets dat moet blijven.
- **De map wordt aangemaakt als hij niet bestaat**, met een nette fout als dat
  niet lukt.
- **Ontbrekend of corrupt configbestand** is geen crash. Een half weggeschreven
  JSON hoort een leesbare melding te geven met het pad erin.
- **Schrijven is atomair**: naar een tijdelijk bestand en dan verplaatsen. Anders
  levert een crash midden in het schrijven een onbruikbaar bestand op.
- **Geen geheimen op schijf.** Is het ontwerp "de app bewaart geen tokens", dan is
  élke vorm van tokenopslag — ook in de Keychain — een afwijking die besproken moet
  worden, niet stil ingevoerd.
- **Machine-specifieke config staat niet in git.** Paden verschillen per machine;
  een voorbeeldbestand hoort er wel in.

## 5. Meelopen met het systeem

### Wat te checken
- **Automatisch starten bij inloggen via `SMAppService`** (ServiceManagement), niet
  via de verouderde login-items of een launchd-plist met de hand.
- **Slaap en ontwaken**: een `Timer` of `Task.sleep` vuurt na slaap niet
  betrouwbaar op het verwachte moment. Abonneer op
  `NSWorkspace.didWakeNotification` en verwerk dan meteen, anders kijkt de
  gebruiker na het openklappen naar oude gegevens zonder dat te weten.
- **Netwerk kan weg zijn.** Een bron die niets kan ophalen hoort dat te zeggen, niet
  een lege lijst te tonen.
- **Focus / Niet storen respecteren** — dat doet `UserNotifications` zelf; probeer
  het niet te omzeilen.

## Wat geen issue is (om false positives te voorkomen)
- Geen notarisatie bij een app die bewust niet verspreid wordt.
- Sandbox uit, als dat een vastgelegde en toegelichte keuze is voor een app die
  externe commando's draait.
- Een cachebestand in `~/Library/Caches/` dat verloren mag gaan.
- Geen automatisch starten bij inloggen — dat mag een handmatige
  systeeminstelling blijven.

## Voorbeeld

```swift
// FOUT — naast de binary, niet atomair, crasht op onleesbare inhoud
let pad = URL(filePath: "projecten.json")
let projecten = try! JSONDecoder().decode([Project].self, from: Data(contentsOf: pad))

// GOED — juiste plek, leesbare fout, atomair schrijven
let map = URL.applicationSupportDirectory.appending(path: "nl.dejongschoonmaak.flightcontrol")
try FileManager.default.createDirectory(at: map, withIntermediateDirectories: true)

let pad = map.appending(path: "projecten.json")
do {
    projecten = try JSONDecoder().decode([Project].self, from: Data(contentsOf: pad))
} catch {
    stand.fouten[.config] = "Kan \(pad.path()) niet lezen: \(error.localizedDescription)"
    projecten = []
}

// schrijven: eerst tijdelijk, dan verplaatsen
let tijdelijk = pad.appendingPathExtension("tmp")
try data.write(to: tijdelijk)
_ = try FileManager.default.replaceItemAt(pad, withItemAt: tijdelijk)
```
