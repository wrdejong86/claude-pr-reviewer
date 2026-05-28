# React

## Wat te checken
- **`useEffect` dependency arrays**: ontbrekende deps, of bewust weggelaten zonder comment.
- **State in `useEffect`**: setState in effect zonder guard → infinite re-render risico.
- **Stale closures**: callbacks die outdated state lezen (vaak fix: `useCallback` met juiste deps).
- **Key props in lijsten**: gebruik van array index als key bij dynamische lijsten.
- **Conditional hooks**: hooks aangeroepen in if/loop — overtreedt rules of hooks.
- **Inline object/array props** in performance-kritieke componenten (re-render trigger).
- **Onnodige `useMemo`/`useCallback`** op trivial calculations.
- **Direct DOM manipulatie** (`document.getElementById`) waar refs horen.
- **Forms zonder controlled state** of mixing controlled/uncontrolled.
- **Niet awaited async in event handlers** zonder error handling.
- **Server components vs client components** (Next.js): `"use client"` per ongeluk vergeten of overal.

## Wat GEEN issue is
- Missing `useMemo` op simpele waardes (premature optimization)
- "Je had context kunnen gebruiken" als props prima werken

## Voorbeeld
```tsx
// FOUT — infinite loop
useEffect(() => {
  setCount(count + 1);
}, [count]);

// FOUT — stale closure
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, []); // count blijft 0
```
