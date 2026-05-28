# TypeScript

## Wat te checken
- **`any` types**: nieuwe `any` of `as any` casts. Vraag waarom — vaak verbergen ze een echte bug.
- **`@ts-ignore` / `@ts-expect-error`** zonder uitleg.
- **Non-null assertion (`!`)** op waardes die wel degelijk null kunnen zijn.
- **Niet-exhaustive switch** op union types — gebruik `never` check in `default`.
- **Onveilige type assertions** (`as SomeType`) waar een type guard beter zou zijn.
- **Optional chaining die fouten verbergt**: `user?.organization?.id` waar `user` altijd zou moeten bestaan.
- **`Record<string, any>`** als return type — meestal te vaag.
- **Async functies die geen `await` gebruiken** maar wel `async` zijn gemarkeerd.
- **Promises die niet ge-awaited worden** (fire-and-forget) zonder `.catch()`.

## Wat GEEN issue is
- `any` in gegenereerde code of in third-party type declarations
- Korte type assertions waar het type aantoonbaar correct is

## Voorbeeld
```ts
// FOUT
const user = data as User; // wat als data verkeerd is?

// BETER
if (!isUser(data)) throw new Error("invalid user payload");
const user = data; // type is nu User
```
