[<< Day 26](../26_Day_use_Hook_Server_Components/26_use_hook_server_components.md) | [Day 28 >>](../28_Day_Testing/28_testing.md)

# Day 27 – Global State Management

## Table of Contents
- [When Do You Need Global State](#when-do-you-need-global-state)
- [Your Options in 2026](#your-options-in-2026)
- [Zustand: The Default Recommendation](#zustand-the-default-recommendation)
- [Zustand with Selectors](#zustand-with-selectors)
- [Zustand with `immer`](#zustand-with-immer)
- [Zustand with `persist`](#zustand-with-persist)
- [Jotai: Atomic State](#jotai-atomic-state)
- [Redux Toolkit](#redux-toolkit)
- [Comparing Zustand, Jotai, and Redux Toolkit](#comparing-zustand-jotai-and-redux-toolkit)
- [Server State vs Client State](#server-state-vs-client-state)
- [Recommended Decision Guide](#recommended-decision-guide)
- [💻 Exercises](#-exercises)

## When Do You Need Global State

Not all shared state should become “global state.”

Use local component state first when the data belongs to one component or one small subtree:

- an open/closed dropdown
- a single form input
- tab selection inside one page

Global state becomes useful when the same state is needed by many unrelated components across the app:

- authenticated user
- theme preference
- shopping cart
- notifications
- feature flags
- app-wide filters

### Warning sign: prop drilling everywhere

If you are passing the same state through three or more layers just so a distant child can use it, your design may be ready for:

- Context
- a state library

### Another warning sign: giant Context values

Context is great, but if a single provider holds many unrelated state values and actions, maintenance can get messy and render performance may suffer.

---

## Your Options in 2026

Here are the main choices for global client state:

| Tool | Best for | Pros | Trade-offs |
|--|--|--|--|
| Context API | simple shared state | built into React, no package | can become verbose, provider nesting |
| Zustand | most apps | tiny, ergonomic, no provider required | fewer enforced patterns |
| Jotai | fine-grained reactive state | atomic model, derived atoms feel natural | different mental model |
| Redux Toolkit | large teams / complex apps | strong conventions, DevTools, middleware ecosystem | more setup than Zustand |

### Quick recommendation

- **Start with Context** for simple theme/auth toggles
- **Use Zustand** for most real-world apps
- **Use Jotai** when atomic composition is the best fit
- **Use Redux Toolkit** when your app or team needs stricter structure

---

## Zustand: The Default Recommendation

Zustand is popular because it is:

- small
- fast
- readable
- provider-free
- easy to adopt incrementally

Install it:

```bash
npm install zustand
```

### First store: shopping cart

```jsx
import { create } from 'zustand'

export const useCartStore = create((set, get) => ({
  items: [],

  addItem: item =>
    set(state => ({
      items: [...state.items, item],
    })),

  removeItem: id =>
    set(state => ({
      items: state.items.filter(item => item.id !== id),
    })),

  clearCart: () => set({ items: [] }),

  getItemCount: () => get().items.length,
}))
```

Use it in components:

```jsx
import { useCartStore } from './stores/useCartStore'

function ProductCard({ product }) {
  const addItem = useCartStore(state => state.addItem)

  return (
    <article>
      <h2>{product.name}</h2>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>Add to cart</button>
    </article>
  )
}
```

```jsx
import { useCartStore } from './stores/useCartStore'

function CartSummary() {
  const items = useCartStore(state => state.items)
  const clearCart = useCartStore(state => state.clearCart)

  return (
    <section>
      <h2>Cart</h2>
      <p>Items: {items.length}</p>
      <button onClick={clearCart}>Clear cart</button>
    </section>
  )
}
```

### Why this feels nice

- no provider wrapper
- no reducer boilerplate unless you want it
- component subscribes only to what it reads

---

## Zustand with Selectors

A powerful pattern in Zustand is subscribing to a **slice** of state.

Bad:

```jsx
const store = useCartStore()
```

This subscribes to everything in the store.

Better:

```jsx
const items = useCartStore(state => state.items)
const addItem = useCartStore(state => state.addItem)
```

Best when you need derived values:

```jsx
const total = useCartStore(state =>
  state.items.reduce((sum, item) => sum + item.price, 0)
)
```

### Full example

```jsx
import { create } from 'zustand'

export const useCartStore = create(set => ({
  items: [],

  addItem: item =>
    set(state => ({
      items: [...state.items, item],
    })),

  removeItem: id =>
    set(state => ({
      items: state.items.filter(item => item.id !== id),
    })),
}))
```

```jsx
import { useCartStore } from './stores/useCartStore'

export function CartTotals() {
  const itemCount = useCartStore(state => state.items.length)
  const total = useCartStore(state =>
    state.items.reduce((sum, item) => sum + item.price, 0)
  )

  return (
    <footer>
      <p>Total items: {itemCount}</p>
      <p>Total price: ${total.toFixed(2)}</p>
    </footer>
  )
}
```

Selectors help avoid unnecessary re-renders when unrelated store values change.

---

## Zustand with `immer`

When updates become nested, immutable update logic can feel noisy.

Install `immer` if your project does not already have it:

```bash
npm install immer
```

Then use Zustand's `immer` middleware:

```jsx
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

export const useWishlistStore = create(
  immer(set => ({
    items: [],

    addItem: item =>
      set(state => {
        state.items.push(item)
      }),

    toggleSaved: id =>
      set(state => {
        const existing = state.items.find(item => item.id === id)

        if (existing) {
          existing.saved = !existing.saved
        }
      }),
  }))
)
```

Without `immer`, you would write a lot more array/object copying. With `immer`, mutation-like syntax is converted into safe immutable updates.

### When to use `immer`

Use it when:

- state shape is nested
- updates are complex
- readability matters more than absolute minimalism

Skip it when:

- your state is small and flat
- simple spreads are already clear

---

## Zustand with `persist`

`persist` keeps state in `localStorage` so it survives page refreshes.

```jsx
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useAuthStore = create(
  persist(
    set => ({
      user: null,
      token: null,

      setUser: user => set({ user }),
      setToken: token => set({ token }),
      logout: () => set({ user: null, token: null }),
    }),
    {
      name: 'auth-store',
    }
  )
)
```

### Persisted favorites example

```jsx
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useFavoritesStore = create(
  persist(
    (set, get) => ({
      favorites: [],

      addFavorite: repo =>
        set(state => ({
          favorites: [...state.favorites, repo],
        })),

      removeFavorite: id =>
        set(state => ({
          favorites: state.favorites.filter(repo => repo.id !== id),
        })),

      toggleFavorite: repo => {
        const exists = get().favorites.some(item => item.id === repo.id)

        if (exists) {
          get().removeFavorite(repo.id)
          return
        }

        get().addFavorite(repo)
      },
    }),
    {
      name: 'favorite-repos',
    }
  )
)
```

Use it in a component:

```jsx
import { useFavoritesStore } from './stores/useFavoritesStore'

export function RepoCard({ repo }) {
  const favorites = useFavoritesStore(state => state.favorites)
  const toggleFavorite = useFavoritesStore(state => state.toggleFavorite)

  const isFavorite = favorites.some(item => item.id === repo.id)

  return (
    <article>
      <h3>{repo.name}</h3>
      <button onClick={() => toggleFavorite(repo)}>
        {isFavorite ? 'Remove favorite' : 'Save favorite'}
      </button>
    </article>
  )
}
```

### Persist caveats

- only store safe serializable data
- do not store sensitive tokens unless you fully understand the security trade-offs
- server-rendered apps may need hydration guards for persisted values

---

## Jotai: Atomic State

Jotai organizes state as small units called **atoms**.

Install it:

```bash
npm install jotai
```

### Basic example

```jsx
import { atom, useAtom } from 'jotai'

const countAtom = atom(0)
const doubledAtom = atom(get => get(countAtom) * 2)

function Counter() {
  const [count, setCount] = useAtom(countAtom)
  const [doubled] = useAtom(doubledAtom)

  return (
    <section>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <button onClick={() => setCount(value => value + 1)}>
        Increment
      </button>
    </section>
  )
}
```

### Forming a mental model

In Redux or Zustand, you usually think in terms of a store.  
In Jotai, you think in terms of many small reactive atoms.

That makes Jotai especially nice for:

- derived state
- composable state pieces
- large apps with many independent reactive values

### Async atom example

```jsx
import { atom, useAtom } from 'jotai'

const userIdAtom = atom(1)

const userAtom = atom(async get => {
  const userId = get(userIdAtom)
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/users/${userId}`
  )

  if (!response.ok) {
    throw new Error('Failed to fetch user')
  }

  return response.json()
})

export function UserProfile() {
  const [user] = useAtom(userAtom)

  return <h2>{user.name}</h2>
}
```

Jotai can pair especially well with Suspense-driven patterns.

---

## Redux Toolkit

Redux Toolkit (RTK) is the modern official way to write Redux.

Install it:

```bash
npm install @reduxjs/toolkit react-redux
```

### Why RTK still matters

Redux is no longer the default recommendation for every app, but it remains excellent for:

- large teams
- complex workflows
- apps needing explicit patterns
- rich DevTools and middleware

### Slice example

```jsx
// src/features/cart/cartSlice.js
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
  items: [],
}

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload)
    },
    removeItem(state, action) {
      state.items = state.items.filter(item => item.id !== action.payload)
    },
    clearCart(state) {
      state.items = []
    },
  },
})

export const { addItem, removeItem, clearCart } = cartSlice.actions
export default cartSlice.reducer
```

Configure the store:

```jsx
// src/app/store.js
import { configureStore } from '@reduxjs/toolkit'
import cartReducer from '../features/cart/cartSlice'

export const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
})
```

Provide it to React:

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { Provider } from 'react-redux'
import App from './App'
import { store } from './app/store'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
)
```

Read and dispatch:

```jsx
// src/features/cart/Cart.jsx
import { useDispatch, useSelector } from 'react-redux'
import { clearCart, removeItem } from './cartSlice'

export default function Cart() {
  const dispatch = useDispatch()
  const items = useSelector(state => state.cart.items)
  const total = useSelector(state =>
    state.cart.items.reduce((sum, item) => sum + item.price, 0)
  )

  return (
    <section>
      <h2>Cart</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button onClick={() => dispatch(removeItem(item.id))}>
              Remove
            </button>
          </li>
        ))}
      </ul>

      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={() => dispatch(clearCart())}>Clear cart</button>
    </section>
  )
}
```

### RTK Query

Redux Toolkit also includes **RTK Query**, a data-fetching layer. It is powerful, but if your app already uses TanStack Query, you usually do not need both.

---

## Comparing Zustand, Jotai, and Redux Toolkit

| | Zustand | Jotai | Redux Toolkit |
|--|--|--|--|
| Bundle size | ~1KB | ~3KB | ~15KB |
| Boilerplate | Minimal | Minimal | Medium |
| DevTools | ✅ | ✅ | ✅ excellent |
| Learning curve | Low | Low | Medium |
| Best for | Most apps | Fine-grained | Large teams |

### Deeper interpretation

- **Zustand** wins on speed of implementation.
- **Jotai** wins when your state graph is naturally compositional.
- **Redux Toolkit** wins when predictable structure and tooling matter more than minimal code.

---

## Server State vs Client State

One of the most common mistakes in React apps is putting **server data** into a global client store when it should live in a server-state library.

### Server state examples

- repository search results from GitHub API
- current user fetched from `/api/me`
- paginated products from your backend

These belong naturally in **TanStack Query** because it handles:

- caching
- refetching
- retries
- background updates
- stale-time logic

### Client state examples

- theme
- modal open state
- unsaved local filters
- cart drawer visibility
- persisted favorites

These belong naturally in Zustand, Jotai, Context, or Redux.

### Good combination

```jsx
const { data: repos, isLoading } = useQuery({
  queryKey: ['repos', topic],
  queryFn: () => fetchRepos(topic),
})

const favorites = useFavoritesStore(state => state.favorites)
const toggleFavorite = useFavoritesStore(state => state.toggleFavorite)
```

This is the ideal split:

- **TanStack Query** manages remote data
- **Zustand** manages local UI state

Do not duplicate fetched data into your client store unless you have a very specific reason.

---

## Recommended Decision Guide

Use this simple path:

### Use Context API when

- the state is small
- you already have a clear provider boundary
- you want zero dependencies

### Use Zustand when

- you want the best default choice
- the app has moderate shared UI state
- you want quick setup and low ceremony

### Use Jotai when

- your state is naturally atom-based
- derived state is everywhere
- you want highly granular subscriptions

### Use Redux Toolkit when

- many developers work on the same app
- explicit structure matters
- you need advanced middleware or long-term scaling conventions

---

## 💻 Exercises

### Level 1 – Theme Toggle with Zustand

Build a theme store:

1. Create `useThemeStore` with `theme` and `toggleTheme`.
2. Show the current theme in a header.
3. Add a button that switches between `"light"` and `"dark"`.
4. Change the page background and text color based on the store value.

**Stretch:** Persist the theme to `localStorage` using `persist`.

### Level 2 – Shopping Cart with Zustand + Persist

Rebuild a cart system:

1. Create a product list with at least 5 products.
2. Add `addItem`, `removeItem`, and `clearCart`.
3. Persist cart items with `persist`.
4. Show total item count and total price.
5. Use selectors instead of reading the whole store.

**Stretch:** Prevent duplicate items by increasing quantity instead.

### Level 3 – Rebuild Day 20 Cart with Redux Toolkit

Take your Day 20 shopping cart or reducer exercise and rebuild it with RTK:

1. Create a `cartSlice`.
2. Configure a Redux store.
3. Wrap the app in `Provider`.
4. Replace reducer dispatch calls with RTK actions.
5. Add at least one selector for derived totals.

Bonus challenge:

- compare the DX of `useReducer`, Zustand, and RTK
- write down when each felt best
