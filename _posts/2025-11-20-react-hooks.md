---
title: "React Hooks"
updated: 2025-12-20
sort_date: 2025-12-20
---

### `useState`

**Purpose:**
To store and update reactive data in a component.

**When it runs:**
On **every render**. The setter from a previous render may schedule a new render when the state value changes, but renders can also be caused by new props, context changes, or parent renders.

**Key idea:**
Changing state triggers a re-render.

```javascript
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </div>
  );
}
```

### `useEffect`

**Purpose:**
To run side effects — anything outside React’s render flow.

Examples of side effects:
- Fetching data
- Subscribing to events
- Updating document title
- Setting intervals or timers


**When it runs:**

After React commits a render. The dependency array controls whether it runs **again**:
- `[]` → run after the initial render; cleanup still runs on unmount
- `[count]` → run after the initial render and again when `count` changes
- (no array) → run after every render


```javascript
import { useEffect, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Clicked ${count} times`;
  }, [count]);  // runs only when count changes

  return (
    <button onClick={() => setCount(count + 1)}>
      Click {count}
    </button>
  );
}
```

What happens in this example:
1. You click → `setCount(count + 1)`
2. React renders the component with the new `count` (this is the re-render)
3. React commits the UI update to the screen
4. After the render, React calls your effect → `document.title = ...`
5. If the component re-renders again, React will run the cleanup (if provided) before the next effect, and it also runs cleanup on unmount.

Important:
- Updating `document.title` does not rerender the UI.
- React can rerender because of state setters, new props, context updates, or parent renders.
- Side effects like `document.title` do not trigger re-renders.

Why updating the document title is a side effect (not rendering)?

React only considers something “UI rendering” if it happens inside the React virtual DOM — meaning React renders components into the screen using its own rendering engine.

For example:

- Changing text in a `<p>`
- Showing/hiding components
- Updating state that changes JSX

These are all React-managed UI updates. But everything outside React’s world is a side effect. React considers anything that touches the browser outside React as a side effect.

Examples:

- `document.title = "..."`
- `window.addEventListener(...)`
- `fetch(...)`
- `setInterval(...)`
- `localStorage.setItem(...)`
- Mutating the DOM manually (`document.querySelector(...)`)

These are impure because they affect the world outside the component. So updating the document title is NOT React rendering — it's modifying the real browser DOM manually, outside React's control.


### Summary

| If the update is...                 | Use                   |
| ----------------------------------- | --------------------- |
| **React-controlled UI** (JSX)       | `useState` + rerender |
| **Browser or external side effect** | `useEffect`           |



| Hook          | Purpose              | Runs When?                                                |
| ------------- | -------------------- | --------------------------------------------------------- |
| **useState**  | Store internal state | On every render; setter schedules rerenders when value changes |
| **useEffect** | Run side effects     | After render; reruns based on dependency array and cleans up on unmount |


### `useMemo` - memoize values
Purpose: Avoid expensive recalculations

```tsx
const filtered = useMemo(() => {
  return items.filter(i => i.active);
}, [items]);
```

Mental Model:
> “Recompute this value only when dependencies change”

In JavaScript, objects, arrays, and functions are compared by reference, not by value.
React uses shallow comparison in many places:
- `React.memo`
- `useEffect` dependencies
- `useCallback` / `useMemo` dependencies

Shallow comparison means:
>“Are these the same references as last time?”

Without `useMemo`,
```tsx
const filtered = items.filter(i => i.active);
```
This creates a new array **on every render**.
Even if:
- `items` didn’t change
- Filter result is identical

`filtered` is a new reference every time.

How `useMemo` fixes this:
```tsx
const filtered = useMemo(() => {
  return items.filter(i => i.active);
}, [items]);
```
Now:

- If `items` is the same reference
- React returns the same `filtered` reference

Referential equality is preserved.

Referential equality matters because React does shallow comparisons. `useMemo` preserves object identity so memoized components and effects don’t run unnecessarily.


### `useCallback` – memoize functions

Purpose: Prevent function identity changes

```tsx
const onClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```

Mental model:
- Functions are recreated every render
- `useCallback` keeps the same function reference

Used with:
- `React.memo`
- Dependency-heavy child components

### `useRef`

#### What is useRef?

```tsx
const ref = useRef(initialValue);
```

`useRef` returns a stable object:
```tsx
{ current: initialValue }
```

Two core properties:
- `ref.current` persists across renders
- Updating `ref.current` does NOT trigger a re-render

Mental model:
> `useRef` = a mutable box that React ignores for rendering
> `useRef` is for values React doesn’t need to know about

#### The 3 legitimate use cases of `useRef`

##### 1. Accessing DOM elements (most common)
```tsx
const inputRef = useRef<HTMLInputElement>(null);

<input ref={inputRef} />

<button onClick={() => inputRef.current?.focus()}>
  Focus
</button>
```
Why not state?

- DOM nodes don’t belong in React state
- You don’t want a re-render when the DOM reference changes


##### 2. Storing mutable values across renders (without re-render)

This is the most important conceptual use case.

Example: store previous value
```tsx
function Counter({ value }: { value: number }) {
  const prev = useRef<number | null>(null);

  useEffect(() => {
    prev.current = value;
  }, [value]);

  return (
    <p>Now: {value}, Before: {prev.current}</p>
  );
}
```

Why `useRef` works here
- Value survives re-renders
- Updating it doesn’t cause another render loop

##### 3. Escape hatch for non-React state

Example: interval ID, timeout ID, external library instance
```tsx
const intervalId = useRef<number | null>(null);

useEffect(() => {
  intervalId.current = window.setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => clearInterval(intervalId.current!);
}, []);
```