---
title: "Two-Way Data Flow in React: Hooks + Redux"
updated: 2026-01-10
sort_date: 2026-01-10
---

While learning React hooks, I focused a lot on what each hook does (`useEffect`, `useCallback`, etc.).
But after working on a real client-facing React application with Redux, I realized that hooks are not just APIs — they form a **data-flow architecture**.

### 1. Redux State Must Be the Single Source of Truth

In a real app, multiple components and widgets may read and write shared data.
Only one place should be authoritative.

In our case:
- Redux holds the ground truth (`selectedSecurities`)
- UI components and widgets (search bar, table, pills, etc.) must derive their state from Redux — not own it

Rule:
- UI state should be reconciled from Redux, not partially owned by event handlers.


### 2. `useEffect` Is a Reconciler, Not Just “Side Effects”

Effect = “Make UI match the model”.

```ts
useEffect(() => {
  // reconcile UI with Redux state
}, [reduxState]);
```

### 3. UI Events Should Only Express Intent — Not Mutate UI State

Handlers should:
- Dispatch Redux actions
- Express what happened

**They should not directly mutate UI bookkeeping state.**

```ts
// Good
dispatch(removeSecurity({id: security.id}));

// Not ideal
displayedIdsRef.current.delete(security.id); // double ownership
```

Let Redux update → let `useEffect` reconcile UI.   
This avoids double ownership, race conditions, and stale UI bugs.


### 4. Dependency Arrays Are About Closure Correctness — Not Performance
`useCallback` and `useEffect` dependencies are not “performance hints”.
They define what version of state your function is allowed to see.
If a handler reads `selectedSecurities`, it must include it in deps — otherwise it becomes logically incorrect.


### 5. Stable Dependencies Still Belong in the Dependency Array

Redux’s `dispatch` is stable — but still belongs in deps:

```ts
useCallback(() => {
  dispatch(...)
}, [dispatch]);
```
This keeps hooks semantically correct, future-proof, and concurrency-safe.


### 6. Real Apps Are Bidirectional Data Flows

Real systems have two directions:

|      Direction                      |    Hook                                  |
| ----------------------------------- | ---------------------------------------- |
| Redux -> UI       | `useEffect` (reconciliation)             |
| UI -> Redux       | `useCallback` (intent dispatch)          |

UI events dispatch → Redux updates → effect reconciles UI/ref.

React Hooks are not just tools — they are data-flow constraints. They enforce how data moves, who owns truth, and how UI stays consistent.
