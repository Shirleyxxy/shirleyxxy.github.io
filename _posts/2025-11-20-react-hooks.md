---
title: "React Hooks"
---

### `useState`

**Purpose:**
To store and update reactive data in a component.

**When it runs:**
When the component renders (initially) and anytime the state updates.

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

Depends on the dependency array:
- `[]` → run once on mount
- `[count]` → run when count changes
- (no array) → run on every render


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

Important: 
- Updating `document.title` does not rerender the UI.
- React only rerenders when you update React state with `setState`.
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



| Hook          | Purpose              | Runs When?                     |
| ------------- | -------------------- | ------------------------------ |
| **useState**  | Store internal state | On render + when state updates |
| **useEffect** | Run side effects     | Based on dependency array      |
