# Externe commando's (`Process`, `ssh`, `gh`, `git`)

Voor apps die gereedschap op de machine aanroepen in plaats van een API. Dit is
het gevaarlijkste deel van zo'n app: je geeft tekst aan een shell, je krijgt tekst
terug, en beide kanten kunnen je verrassen.

## 1. Nooit een shell-string bouwen

### Wat te checken
- **`Process` met `arguments: [...]` is de enige juiste vorm.** Elk argument een
  eigen element in de array.
- **`/bin/sh -c "…"` met string-interpolatie is een blocking finding.** Een
  projectnaam, tak, pad of tokenwaarde uit een configbestand komt dan in een shell
  terecht. Eén backtick, `;` of `$(…)` en er draait iets anders dan bedoeld.
- **Ook bij `ssh` geldt dit**, met een extra laag: wat je na de alias meegeeft
  wordt op de **server** door een shell gehaald. Interpoleer daar nooit
  gebruikers- of configwaarden in.
- **Paden met spaties of aanhalingstekens** horen gewoon als los argument mee te
  gaan, niet met `\"` eromheen gefrommeld.

## 2. Absolute paden

### Wat te checken
- **Een GUI-app erft de shell-`PATH` niet.** Een app die door Finder of launchd
  gestart is heeft `/usr/bin:/bin:/usr/sbin:/sbin` — Homebrew
  (`/opt/homebrew/bin`) zit daar níet in.
- **Gevolg**: `gh` is onvindbaar, en de fout luidt "launch path not accessible" —
  niets dat naar `PATH` verwijst. **En het werkt wél vanuit Xcode**, want die geeft
  soms jouw eigen omgeving door. Dus dit breekt pas bij de echte gebruiker.
- **Resolve gereedschap één keer bij het starten** en bewaar het absolute pad.
  `which` aanroepen vanuit de app lost het niet op — dat heeft hetzelfde probleem.

## 3. Pipes en time-outs

### Wat te checken
- **`waitUntilExit()` vóór het lezen van de pipe is een deadlock.** De pipebuffer
  is ~64 KB; is de uitvoer groter, dan blokkeert het kindproces op schrijven
  terwijl de app op het einde wacht. `git log` of een build-log haalt die 64 KB
  makkelijk. Lees terwijl het proces loopt.
- **Elk commando heeft een time-out**, en bij het verstrijken wordt het proces
  echt beëindigd (`terminate()`, daarna `kill` als het moet). Zonder time-out
  hangt de app op "bezig" zonder dat iemand weet waarom.
- **`terminationStatus` wordt gecontroleerd.** Exitcode 0 is niet
  vanzelfsprekend, en lege uitvoer met code 1 is een fout, niet "niets gevonden".
- **`stderr` wordt opgevangen en in de foutmelding gebruikt** — daar staat de reden.

## 4. `ssh` in het bijzonder

### Wat te checken
- **`-o BatchMode=yes`** is verplicht. Zonder tty wacht een wachtwoord- of
  host-key-vraag eeuwig, en niemand ziet die vraag. Met BatchMode faalt het
  meteen en leesbaar.
- **`-o ConnectTimeout=…`** erbij.
- **`StrictHostKeyChecking=no` of `UserKnownHostsFile=/dev/null` is blocking.** Dat
  is de veiligheid uitzetten om een inrichtingsprobleem te omzeilen. De host hoort
  één keer met de hand in `known_hosts`.
- **Geen root-alias voor leeswerk.** Draait het commando ook als gewone gebruiker,
  gebruik die dan. Root voor het ophalen van een lijstje is een onnodig grote hamer.
- **Eén verbinding per ronde.** Vijf losse aanroepen naar dezelfde server is vijf
  keer de handshake; bundel de commando's met een markeerregel ertussen die de
  parser splitst.

## 5. Geheimen en productie

### Wat te checken
- **Nooit een heel `.env` van productie inlezen.** Daar staan secrets naast de
  waarde die je zoekt. Wil je één vlag weten, zoek dan gericht op een vooraf
  benoemde naam uit de projectconfig.
- **Sleutel*namen* mogen in bulk, waarden niet.** Namen zijn geen geheim.
- **Commando's die gelogd worden mogen geen geheim bevatten** — geen token, geen
  pad naar een key, geen `.env`-inhoud. Controleer de logregel, niet alleen het
  commando.
- **Een muterend commando hoort achter een bevestiging** die zegt wát er gaat
  gebeuren, niet alleen de naam van de knop.

## 6. De uitvoer is invoer van buiten

### Wat te checken
- **Parsen gebeurt in een pure functie met tests**, met echte opgevangen uitvoer
  als fixture. Verandert de uitvoer na een upgrade op de server, dan zie je precies
  wat.
- **Locale beïnvloedt uitvoer.** Zet `LC_ALL=C` in de omgeving als je op vaste
  woorden of scheidingstekens parseert, anders breekt het op een andere taal of
  decimaalteken.
- **Tijden staan vaak in UTC** of in de zone van de server, niet die van de
  gebruiker. `systemctl list-timers` geeft de systeemklok terug. Reken om of toon
  de zone erbij.
- **Nooit uitvoer van een commando als code of pad uitvoeren.** Behandel het als
  onbetrouwbare tekst.
- **Geen commando per item in een lus** — dat is N keer een proces starten. Vraag
  het in één aanroep op.

## Wat geen issue is (om false positives te voorkomen)
- Een vast, in code geschreven commando zonder enige interpolatie — daar kan geen
  injectie in.
- `sh -c` voor iets dat écht een pipeline of shell-constructie nodig heeft, zolang
  er geen variabele in de string staat en dat in een comment is verantwoord.
- Een korte time-out die voor een snel lokaal `git`-commando ruim genoeg is.
- Root gebruiken voor iets dat echt root vereist, mits beperkt tot dat ene
  commando en toegelicht.

## Voorbeeld

```swift
// FOUT — shell-string met interpolatie, geen time-out, deadlock bij grote uitvoer
let p = Process()
p.executableURL = URL(filePath: "/bin/sh")
p.arguments = ["-c", "cd \(project.lokaleMap) && git log \(vanaf)..main"]
let pipe = Pipe(); p.standardOutput = pipe
try p.run()
p.waitUntilExit()                                    // deadlock > 64 KB
let uit = pipe.fileHandleForReading.readDataToEndOfFile()

// GOED — losse argumenten, absoluut pad, time-out, pipe gelezen tijdens de run
let resultaat = try await runner.draai(
    Gereedschap.git,                                 // /usr/bin/git, één keer geresolveerd
    ["-C", project.lokaleMap, "log", "--oneline", "\(vanaf)..main"],
    timeout: .seconds(10)
)
guard resultaat.code == 0 else {
    throw BronFout.commandoFaalde(resultaat.stderr)  // reden blijft behouden
}
```
