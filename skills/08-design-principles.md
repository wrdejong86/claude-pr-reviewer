# Design principles (DRY, SRP, SOLID, KISS, YAGNI)

De fundamentele software engineering principes. Niet dogmatisch toepassen —
ze zijn gereedschap, geen religie. Soms is een beetje duplicatie beter dan de
verkeerde abstractie. Wees expliciet over WHY iets fout zit.

## 1. DRY — Don't Repeat Yourself

### Wat te checken
- **Copy-paste van logica**: dezelfde 5+ regels op meerdere plekken. Vooral business-rules (BSN-validatie, datum-berekeningen, status-mappings).
- **Magic numbers/strings herhaald**: `if (status === "ACTIVE")` op 8 plekken → constante. `const DAYS_IN_REMINDER = 30` ipv literal in 5 functions.
- **Identieke validatie-schema's** in frontend en backend zonder gedeelde source-of-truth (zod schema's exporteren en hergebruiken).
- **Bijna-identieke React componenten** met 1-2 verschillen → één component met props.
- **Database-query patterns** die per endpoint opnieuw geschreven worden → repository/service-laag.
- **API-response shapes** die per endpoint opnieuw gedefinieerd worden → gedeelde types.

### Wat GEEN DRY-issue is (false positives!)
- **Coincidental duplication**: twee functies LIJKEN hetzelfde maar evolueren onafhankelijk (bv. `formatEmployeeName` en `formatCustomerName` — verschillende business-context). Niet samenvoegen.
- **Tests** mogen redundant zijn voor leesbaarheid. Setup-code dupliceren is vaak duidelijker dan een ingewikkelde test-factory.
- **Configuratie/constanten** mogen herhaald lijken als ze conceptueel anders zijn (zelfde getal, andere betekenis).
- **Migrations**: nooit DRY maken, elk is een snapshot in tijd.

### Regel
Rule of three: pas extraheren bij de **derde** keer dat je dezelfde logica nodig hebt. Niet bij de tweede.

## 2. SRP — Single Responsibility Principle

### Wat te checken
- **God-components**: React component die >300 regels is, of >5 hooks heeft, of zowel data-fetching, business-logic én rendering doet → splitsen.
- **God-functions**: functies >50 regels die meerdere onafhankelijke dingen doen. "and" in functienaam is een signaal: `validateAndSaveEmployee` → twee functies.
- **God-files**: één file met losse logica (utility, types, component, hook door elkaar).
- **Service-class** die zowel ophaalt, transformeert, mailt én logt → splits in `EmployeeRepository`, `EmployeeFormatter`, `EmployeeMailer`.
- **API-routes** die meer dan 1 ding doen: een POST /employees die ook een welkom-mail stuurt + invite-token genereert → trek mail/token uit naar aparte service.

### Wat GEEN SRP-issue is
- **Kleine, samenhangende functies** die alleen samen zinvol zijn. Niet alles fragmenteren in 1-regel functies — leesbaarheid lijdt.
- **Componenten met meerdere render-secties** die conceptueel bij elkaar horen (header + body + footer van een kaart).

### Vraag bij twijfel
"Als deze functie/component moet veranderen, hoeveel verschillende REDENEN zijn er?" Eén reden = SRP OK. Meerdere = splitsen.

## 3. SOLID (de andere vier, kort)

### OCP — Open/Closed
- **Wat checken**: nieuwe functionaliteit die ALLE bestaande consumers van een type breekt. Bijv. enum uitbreiden zonder TypeScript exhaustive check → silent bugs.
- **Beter**: typen zo ontwerpen dat uitbreiden = nieuwe variant toevoegen, niet bestaande gedrag wijzigen.

### LSP — Liskov Substitution
- **Wat checken**: subtypes die de contracten van hun parent breken. Bijv. een `ReadOnlyList` die toch een `.push()` heeft die throwt.
- In TypeScript meestal door type-system afgevangen.

### ISP — Interface Segregation
- **Wat checken**: één grote `EmployeeService` interface waar de meeste consumers maar 1-2 methods van gebruiken. Splits in kleinere interfaces (`EmployeeReader`, `EmployeeWriter`).
- Vooral relevant voor testability — mocks worden kleiner.

### DIP — Dependency Inversion
- **Wat checken**: hoge-level modules die direct concrete classes importeren (DB clients, file-systems, time/random). Vraag of dat injectabel/mockbaar moet.
- **Niet dogmatisch**: niet elke functie heeft een interface nodig. Pas toe waar testbaarheid of vervangbaarheid een doel is.

## 4. KISS — Keep It Simple

### Wat te checken
- **Onnodige abstractie**: een factory die 1 type maakt. Een interface met 1 implementatie. Een hook die alleen `useState` wrapt.
- **Onnodige design patterns**: Strategy pattern voor 2 varianten die een `if/else` ook af kon.
- **Clever code**: nested ternaries, reduce-met-side-effects, regex die niemand snapt zonder uitleg.
- **Voortijdige optimalisatie**: useMemo/useCallback op trivial computaties, micro-optimaties in hot paths zonder gemeten probleem.

### Regel
Code wordt 10× vaker gelezen dan geschreven. Schrijf voor de lezer.

## 5. YAGNI — You Aren't Gonna Need It

### Wat te checken
- **Speculative configuratie-opties** waar nu maar één waarde voor gebruikt wordt.
- **Hooks/callbacks** "voor het geval dat" — niet ingebouwd tot het nodig is.
- **Generieke base-classes** voor 1 subklasse.
- **`metadata` velden** in DB voor toekomstig gebruik — voeg toe wanneer je het nodig hebt (migratie is goedkoop).
- **Multi-tenancy support** in een single-tenant codebase (jouw geval!) — niet nu inbouwen.

### Vraag
"Wordt dit deze week, deze sprint, of dit kwartaal gebruikt?" Anders = YAGNI.

## 6. Separation of concerns

### Wat te checken
- **Business logic in UI components**: berekeningen, validaties, formatting → naar hooks of utils.
- **UI logic in business layer**: backend services die HTML/JSX strings genereren.
- **Database logic in routes**: raw Prisma queries in een API route i.p.v. een service-laag → moeilijker te testen.
- **Mixing async concerns**: een functie die zowel fetcht, side-effects doet (mail/notification), én iets returnt — splits.

## 7. Pure functions & side effects

### Wat te checken
- **Hidden side effects**: een "getter" functie die ook iets schrijft (mutates global, logs to db, sends email).
- **Time/random** in pure logica: gebruik dependency injection (pass `now` als arg) — anders niet testbaar.
- **Mutation van inputs**: functies die hun arguments wijzigen i.p.v. nieuwe waardes returnen.
- **`useEffect` zonder cleanup**: subscriptions, timers, listeners die blijven hangen.

## 8. Naming

### Wat te checken
- **Vage namen**: `data`, `result`, `temp`, `obj`, `handler`. Wees specifiek: `employeeRecords`, `validatedFormData`.
- **Misleidende namen**: `getUser()` die ook iets schrijft. `isActive` die soms `null` retourneert.
- **Inconsistente terminologie**: `employee` en `worker` en `staff` voor hetzelfde concept.
- **Hongaarse notatie**: `strName`, `iCount` → weg ermee, TypeScript geeft je dit gratis.
- **Booleans** met negatie: `isNotDisabled` → `isEnabled`.

## 9. Composition over inheritance

### Wat te checken
- **Class-hierarchieen** dieper dan 2 niveaus (vooral in een TS-codebase die toch al functional is). Vraag of compositie + interfaces niet beter past.
- **React class components** in een nieuwe PR — convert naar functional.
- **Mixins** of decorators die state delen — vaak teken van verkeerde structuur.

## 10. Coupling & cohesion

### Wat te checken
- **High coupling**: één wijziging vereist dezelfde wijziging in 5+ files. Suggestie: gedeelde abstractie of betere boundaries.
- **Low cohesion**: één file/module bevat losse functies die niets met elkaar te maken hebben. Splits.
- **Circular imports**: file A importeert van B, B importeert van A. Refactor naar een derde module.

## Output-format
- Voor elke finding: noem PRINCIPLE (DRY/SRP/etc), file:line, en WAAROM dit een probleem is in deze specifieke context.
- Wees pragmatisch: een kleine duplicatie in een 2-keer-gebruikte util is geen blocker. Een 200-regel god-component wel.
- False positives flaggen: als je twijfelt of het echt een issue is, schrijf "low confidence — kan ook bewuste keuze zijn".
