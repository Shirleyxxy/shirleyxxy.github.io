---
title: "Redux in Practice: Slices, Thunks, and Data Flow"
updated: 2025-12-13
sort_date: 2025-12-13
---

These notes focus on how Redux actually works in a modern React codebase using Redux Toolkit, based on what I’ve seen at work.


## The one thing Redux cares about: predictable data flow

Redux exists to manage **global application state** in a predictable way.
At a high level, Redux enforces a single rule: 
> State can only change in response to actions, and those changes are defined by reducers.

This constraint is what makes large applications easier to reason about.


## The full data flow

Everything in Redux follows this loop:

```
React component
  ↓ dispatch(action or thunk)
Thunk (optional, async)
  ↓ dispatch actions
Reducer
  ↓ produce new state
Store
  ↓
useSelector
  ↓
React re-render
```

## `createSlice`: defining how state changes

In modern Redux, state is organized into slices.
A slice represents:
- One domain of the app
- Its piece of state
- All valid state transitions

`createSlice` packages three things together:
- Initial state
- Reducer logic
- Action creators

Example:
```ts
const userSlice = createSlice({
  name: "user",
  initialState,
  reducers: {
    userLoaded(state, action) {
      state.data = action.payload;
    },
    userLoggedOut(state) {
      state.data = null;
    },
  },
});
```

What this gives you:
- A slice reducer: `userSlice.reducer` → used by the store
- Auto-generated actions: `userSlice.actions.userLoaded` → dispatched by thunks/components

**Reducers: the rules of state changes**
Reducers are:
- Pure functions
- Synchronous
- The only place where Redux state can change

Conceptually:
```
(state, action) => newState
```
Even though Redux Toolkit allows “mutating” syntax, it uses Immer under the hood to keep state immutable.
> Reducers don’t decide when state changes — only how.

## `createAsyncThunk`: handling async logic

Reducers cannot:
- Call APIs
- Await promises
- Perform side effects

Async logic lives in thunks. A thunk is an async function that orchestrates work and dispatches actions.

Example: 
```ts
export const fetchUser = createAsyncThunk(
  "user/fetch",
  async () => {
    return api.getUser();
  }
);
```

## The three automatic states of an async thunk

Every async thunk automatically generates three actions:

- `fetchUser.pending`
- `fetchUser.fulfilled`
- `fetchUser.rejected`

These are handled in the slice via `extraReducers`:

```ts
extraReducers: builder => {
  builder
    .addCase(fetchUser.pending, state => {
      state.loading = true;
    })
    .addCase(fetchUser.fulfilled, (state, action) => {
      state.loading = false;
      state.data = action.payload;
    })
    .addCase(fetchUser.rejected, state => {
      state.loading = false;
    });
}
```

## How React connects to Redux

From a React component:

```tsx
const dispatch = useDispatch();
const user = useSelector(selectUser);

useEffect(() => {
  dispatch(fetchUser());
}, [dispatch]);

```
Components dispatch events, Redux owns the data, and `useSelector` subscribes components to specific state.
It’s common to see `useEffect` and `useSelector` in the same component — `useEffect` controls when work happens, while `useSelector` controls what data the component depends on.



