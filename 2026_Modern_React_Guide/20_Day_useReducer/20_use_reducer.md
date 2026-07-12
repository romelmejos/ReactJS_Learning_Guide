[<< Day 19](../19_Day_TanStack_Query/19_tanstack_query.md) | [Day 21 >>](../21_Day_Context_API/21_context_api.md)

# Day 20 – useReducer

## Table of Contents

- [When `useState` is not enough](#when-usestate-is-not-enough)
- [The `useReducer` hook](#the-usereducer-hook)
- [The reducer pattern](#the-reducer-pattern)
- [Action objects](#action-objects)
- [`useReducer` vs `useState`](#usereducer-vs-usestate)
- [Extracting the reducer](#extracting-the-reducer)
- [Combining `useReducer` and `useContext`](#combining-usereducer-and-usecontext)
- [Immer for simpler reducers](#immer-for-simpler-reducers)
- [Lazy initialization](#lazy-initialization)
- [Testing reducers](#testing-reducers)
- [💻 Exercises](#-exercises)

## When `useState` is not enough

`useState` is perfect for simple local state.

Examples:

- a boolean modal flag
- a text input value
- a counter

But `useState` gets awkward when:

- state has several related fields
- multiple updates must stay in sync
- the next state depends on the previous state in more complex ways
- state transitions happen from many different event handlers

A reducer gives those transitions a central home.

## The `useReducer` hook

`useReducer` takes:

1. a reducer function
2. an initial state
3. optionally, a lazy initializer

```jsx
import { useReducer } from 'react'

const initialState = { count: 0, step: 1 }

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step }
    case 'decrement':
      return { ...state, count: state.count - state.step }
    case 'setStep':
      return { ...state, step: action.payload }
    case 'reset':
      return initialState
    default:
      throw new Error(`Unknown action: ${action.type}`)
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <>
      <p>
        Count: {state.count} (step: {state.step})
      </p>

      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>

      <input
        type="number"
        value={state.step}
        onChange={event =>
          dispatch({
            type: 'setStep',
            payload: Number(event.target.value),
          })
        }
      />

      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </>
  )
}
```

The key idea: components **dispatch actions**, and the reducer decides how state changes.

## The reducer pattern

A reducer is a pure function:

```js
(state, action) => newState
```

That means:

- no side effects
- no API calls
- no random values inside the reducer
- no mutating the existing state object

Bad:

```js
state.count++
return state
```

Good:

```js
return { ...state, count: state.count + 1 }
```

Reducers are easier to test because they are just functions.

## Action objects

Actions describe **what happened**.

Common shape:

```js
{ type: 'increment' }
{ type: 'setStep', payload: 5 }
```

You will often see constants used for action names:

```js
export const ACTIONS = {
  ADD_TODO: 'addTodo',
  TOGGLE_TODO: 'toggleTodo',
  REMOVE_TODO: 'removeTodo',
}
```

Then use them in the reducer:

```js
switch (action.type) {
  case ACTIONS.ADD_TODO:
    return {
      ...state,
      todos: [...state.todos, action.payload],
    }
  default:
    return state
}
```

This reduces typo-related bugs in bigger reducers.

## `useReducer` vs `useState`

Choose `useState` when:

- state is simple
- updates are isolated
- readability is better with direct setters

Choose `useReducer` when:

- state is an object with related sub-values
- many actions update the same state
- state transitions are easier to reason about centrally
- you want logic that is easy to unit test

### Example comparison

`useState` can become scattered:

```jsx
const [count, setCount] = useState(0)
const [step, setStep] = useState(1)
const [history, setHistory] = useState([])
```

Now imagine every button must update all three correctly. A reducer often becomes clearer than multiple setters.

## Extracting the reducer

Move reducers outside components or into separate files when they grow.

```js
// src/features/counter/reducer.js
export const initialState = { count: 0, step: 1 }

export function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step }
    case 'decrement':
      return { ...state, count: state.count - state.step }
    case 'setStep':
      return { ...state, step: action.payload }
    default:
      throw new Error(`Unknown action: ${action.type}`)
  }
}
```

```jsx
// src/features/counter/Counter.jsx
import { useReducer } from 'react'
import { counterReducer, initialState } from './reducer'

export default function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState)

  return <button onClick={() => dispatch({ type: 'increment' })}>{state.count}</button>
}
```

This separation improves testability and reuse.

## Combining `useReducer` and `useContext`

This is a powerful pattern for medium-sized apps. It gives you a lightweight global state solution without bringing in Redux immediately.

### Shopping cart example

```jsx
// src/cart/CartContext.jsx
import { createContext, useContext, useReducer } from 'react'

const CartStateContext = createContext(null)
const CartDispatchContext = createContext(null)

const initialState = {
  items: [],
}

function cartReducer(state, action) {
  switch (action.type) {
    case 'addItem': {
      const existingItem = state.items.find(item => item.id === action.payload.id)

      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        }
      }

      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      }
    }

    case 'removeItem':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload),
      }

    case 'clearCart':
      return initialState

    default:
      throw new Error(`Unknown action: ${action.type}`)
  }
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState)

  return (
    <CartStateContext.Provider value={state}>
      <CartDispatchContext.Provider value={dispatch}>{children}</CartDispatchContext.Provider>
    </CartStateContext.Provider>
  )
}

export function useCartState() {
  const context = useContext(CartStateContext)

  if (!context) {
    throw new Error('useCartState must be used inside CartProvider')
  }

  return context
}

export function useCartDispatch() {
  const context = useContext(CartDispatchContext)

  if (!context) {
    throw new Error('useCartDispatch must be used inside CartProvider')
  }

  return context
}
```

Use it in the app:

```jsx
import { CartProvider, useCartDispatch, useCartState } from './cart/CartContext'

function Product({ product }) {
  const dispatch = useCartDispatch()

  return (
    <button onClick={() => dispatch({ type: 'addItem', payload: product })}>
      Add {product.name}
    </button>
  )
}

function CartSummary() {
  const { items } = useCartState()
  const totalItems = items.reduce((sum, item) => sum + item.quantity, 0)

  return <p>Items in cart: {totalItems}</p>
}

export default function App() {
  const product = { id: 1, name: 'Keyboard' }

  return (
    <CartProvider>
      <CartSummary />
      <Product product={product} />
    </CartProvider>
  )
}
```

This pattern is sometimes called a **mini-Redux** architecture.

## Immer for simpler reducers

Immutable updates with nested objects can become verbose. Immer lets you write updates in a mutation-like style while preserving immutability.

Install it:

```bash
npm install immer
```

```js
import { produce } from 'immer'

const initialState = {
  count: 0,
  history: [],
}

function reducer(state, action) {
  return produce(state, draft => {
    if (action.type === 'increment') {
      draft.count += 1
      draft.history.push(draft.count)
    }

    if (action.type === 'reset') {
      draft.count = 0
      draft.history = []
    }
  })
}
```

Immer is optional, but it becomes especially helpful for deep state trees.

## Lazy initialization

Sometimes initial state is expensive to compute or depends on props or storage.

Use the third argument to `useReducer`:

```jsx
import { useReducer } from 'react'

function init(initialCount) {
  return {
    count: initialCount,
    step: 1,
  }
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step }
    default:
      return state
  }
}

export default function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init)

  return <button onClick={() => dispatch({ type: 'increment' })}>{state.count}</button>
}
```

This is useful when reading from:

- `localStorage`
- URL params
- expensive calculations

## Testing reducers

Reducers are plain functions, which makes them perfect for unit tests.

Example with Vitest:

```js
// src/features/counter/reducer.test.js
import { describe, expect, it } from 'vitest'
import { counterReducer, initialState } from './reducer'

describe('counterReducer', () => {
  it('increments by the current step', () => {
    const state = { count: 0, step: 2 }
    const nextState = counterReducer(state, { type: 'increment' })

    expect(nextState).toEqual({ count: 2, step: 2 })
  })

  it('sets the step', () => {
    const nextState = counterReducer(initialState, {
      type: 'setStep',
      payload: 5,
    })

    expect(nextState.step).toBe(5)
  })
})
```

You do not need a rendered component to test state transitions. Just pass in a state and an action, then assert the result.

## 💻 Exercises

### Level 1 — Convert a counter

Take a `useState` counter and refactor it to `useReducer`.

Requirements:

- increment
- decrement
- reset
- configurable step

### Level 2 — Todo list reducer

Build a todo app with actions for:

- add todo
- toggle todo
- remove todo
- clear completed

Try to keep all state logic inside the reducer instead of scattering it across handlers.

### Level 3 — Shopping cart with context

Build a cart system using `useReducer` + `useContext`.

Include:

- add item
- remove item
- update quantity
- clear cart
- derived total price

Bonus:

- persist cart data to `localStorage`
- extract the reducer into a separate file
- write a few Vitest reducer tests
