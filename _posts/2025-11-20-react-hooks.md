---
title: "React Hooks"
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
