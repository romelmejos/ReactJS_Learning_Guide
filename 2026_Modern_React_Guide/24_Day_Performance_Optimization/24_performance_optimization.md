[<< Day 23](../23_Day_useRef_DOM_Access/23_use_ref.md) | [Day 25 >>](../25_Day_Suspense_Concurrent/25_suspense_concurrent.md)

# Day 24 – Performance Optimization

## Table of Contents

- [When to optimize](#when-to-optimize)
- [Understanding re-renders](#understanding-re-renders)
- [Reactmemo](#reactmemo)
- [useMemo](#usememo)
- [useCallback](#usecallback)
- [The React Compiler](#the-react-compiler)
- [Code splitting with Reactlazy and Suspense](#code-splitting-with-reactlazy-and-suspense)
- [useTransition](#usetransition)
- [useDeferredValue](#usedeferredvalue)
- [List virtualization](#list-virtualization)
- [Profiling with React DevTools](#profiling-with-react-devtools)
- [Key lessons](#key-lessons)
- [💻 Exercises](#-exercises)

## When to optimize

Do **not** optimize just because you see a re-render.

Optimize when:

- the UI feels slow
- profiling shows expensive commits
- user input becomes laggy
- large lists or expensive calculations block rendering

React is already fast. Premature optimization often adds complexity without helping.

Your workflow should be:

1. observe a problem
2. measure it
3. identify the cause
4. optimize the bottleneck
5. re-measure

## Understanding re-renders

A component re-renders when:

- its state changes
- its parent re-renders
- the context it subscribes to changes
- its props change

Re-renders are not automatically bad. A cheap component can re-render many times without trouble.

The real question is:

> Is this render expensive enough to hurt the user experience?

## React.memo

`React.memo` skips re-rendering a component if its props are shallowly equal to the previous render.

```jsx
import React from 'react'

const ExpensiveList = React.memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList rendered')

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
})
```

Useful when:

- child rendering is expensive
- props stay stable often
- parent re-renders frequently

Not useful when:

- props are always new object or array references
- child rendering is cheap

Bad pairing:

```jsx
<ExpensiveList items={[...items]} />
```

That creates a fresh array every render, defeating `memo`.

## useMemo

`useMemo` memoizes the result of a calculation.

```jsx
import { useMemo, useState } from 'react'

function ProductTable({ items }) {
  const [sortOrder, setSortOrder] = useState('asc')

  const sortedItems = useMemo(() => {
    const copy = [...items]

    copy.sort((a, b) => {
      return sortOrder === 'asc'
        ? a.name.localeCompare(b.name)
        : b.name.localeCompare(a.name)
    })

    return copy
  }, [items, sortOrder])

  return (
    <>
      <button
        onClick={() =>
          setSortOrder((current) => (current === 'asc' ? 'desc' : 'asc'))
        }
      >
        Sort: {sortOrder}
      </button>

      <ul>
        {sortedItems.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </>
  )
}
```

Use `useMemo` for expensive pure computations, not for every tiny operation.

Good candidates:

- sorting large arrays
- filtering thousands of records
- heavy derived calculations

Poor candidates:

- `count + 1`
- short string concatenation

## useCallback

`useCallback` memoizes a function reference.

```jsx
import { useCallback, useState } from 'react'

function TodoApp() {
  const [items, setItems] = useState([
    { id: 1, label: 'Learn React 19' },
    { id: 2, label: 'Practice hooks' },
  ])

  const handleDelete = useCallback((id) => {
    setItems((current) => current.filter((item) => item.id !== id))
  }, [])

  return (
    <ul>
      {items.map((item) => (
        <TodoRow key={item.id} item={item} onDelete={handleDelete} />
      ))}
    </ul>
  )
}

const TodoRow = React.memo(function TodoRow({ item, onDelete }) {
  return (
    <li>
      {item.label}
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </li>
  )
})
```

Use it mainly when:

- passing callbacks to `React.memo` children
- a callback is in an effect dependency list

Do not wrap every function “just in case”.

## The React Compiler

React 19 ships with an opt-in compiler that can automatically memoize many values and functions. You may hear it called **React Forget** or **the React Compiler**.

That means:

- fewer manual `useMemo` calls
- fewer manual `useCallback` calls
- less hand-written memo boilerplate

In a Vite app, the setup depends on the current React compiler plugin version. The general pattern is:

```bash
npm install -D babel-plugin-react-compiler
```

Then enable the plugin in your build config.

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
})
```

Always check the latest React compiler docs because setup may evolve. The key lesson is conceptual: in React 19, **manual memoization matters less than before**.

## Code splitting with React.lazy and Suspense

Load code only when the user needs it.

```jsx
import { Suspense } from 'react'
import { Route, Routes } from 'react-router-dom'

const Dashboard = React.lazy(() => import('./pages/Dashboard'))
const Settings = React.lazy(() => import('./pages/Settings'))

function Spinner() {
  return <p>Loading page...</p>
}

export default function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  )
}
```

This improves initial load time, especially for larger apps.

## useTransition

`useTransition` marks updates as non-urgent so the UI can stay responsive.

```jsx
import { useMemo, useState, useTransition } from 'react'

const hugeList = Array.from({ length: 5000 }, (_, i) => `Item ${i + 1}`)

export default function SearchDemo() {
  const [input, setInput] = useState('')
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()

  const filteredItems = useMemo(() => {
    return hugeList.filter((item) =>
      item.toLowerCase().includes(query.toLowerCase())
    )
  }, [query])

  const handleChange = (event) => {
    const value = event.target.value
    setInput(value)

    startTransition(() => {
      setQuery(value)
    })
  }

  return (
    <>
      <input value={input} onChange={handleChange} placeholder="Search items..." />
      {isPending && <p>Updating results...</p>}
      <p>Results: {filteredItems.length}</p>
    </>
  )
}
```

Urgent update: typing into the input.  
Non-urgent update: recalculating heavy search results.

## useDeferredValue

`useDeferredValue` lets React delay a value used by expensive rendering.

```jsx
import { useDeferredValue, useMemo, useState } from 'react'

const products = Array.from({ length: 4000 }, (_, i) => ({
  id: i,
  name: `Product ${i + 1}`,
}))

export default function DeferredSearch() {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query)

  const filteredProducts = useMemo(() => {
    return products.filter((product) =>
      product.name.toLowerCase().includes(deferredQuery.toLowerCase())
    )
  }, [deferredQuery])

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <p>Showing results for: {deferredQuery || 'all products'}</p>
      <ul>
        {filteredProducts.slice(0, 20).map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </>
  )
}
```

This is useful when you have the value itself but not direct control over how it was set.

## List virtualization

Rendering thousands of DOM nodes is expensive. Virtualization renders only the visible window.

Popular libraries:

- `@tanstack/react-virtual`
- `react-window`

Example with `react-window`:

```jsx
import { FixedSizeList as List } from 'react-window'

const items = Array.from({ length: 10000 }, (_, i) => `Row ${i + 1}`)

function Row({ index, style }) {
  return <div style={style}>{items[index]}</div>
}

export default function VirtualizedList() {
  return (
    <List height={400} itemCount={items.length} itemSize={36} width={320}>
      {Row}
    </List>
  )
}
```

If a list contains 10,000 rows, virtualization often helps far more than memoizing each row.

## Profiling with React DevTools

React DevTools Profiler helps you measure real performance costs.

Typical process:

1. open React DevTools
2. switch to the **Profiler**
3. click **Record**
4. interact with the app
5. stop recording
6. inspect flame graphs and commit timings

Look for:

- components with unexpectedly high render durations
- repeated renders caused by prop changes
- context-driven render chains

Optimization without profiling is mostly guessing.

## Key lessons

- measure first
- most re-renders are fine
- `React.memo` helps only when props stay stable
- `useMemo` is for expensive calculations
- `useCallback` is mostly for stable callback references
- React 19's compiler reduces the need for manual memoization
- virtualization beats micro-optimizations for very large lists

## 💻 Exercises

### Level 1 – Profile a slow list

- Build a long list with an intentionally expensive child
- Use React DevTools Profiler to find the hotspot
- Write down which component is slow and why

### Level 2 – Memoize and lazy-load

- Memoize an expensive sort with `useMemo`
- Wrap a child row component with `React.memo`
- Lazy-load a route using `React.lazy` and `Suspense`

### Level 3 – Virtualize 10,000 items

- Install a virtualization library
- Render a list of 10,000 rows
- Compare scrolling and render performance before and after virtualization
