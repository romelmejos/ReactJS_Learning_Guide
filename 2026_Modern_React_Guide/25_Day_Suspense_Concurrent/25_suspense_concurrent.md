[<< Day 24](../24_Day_Performance_Optimization/24_performance_optimization.md) | [Day 26 >>](../26_Day_use_Hook_Server_Components/26_use_hook_server_components.md)

# Day 25 – Suspense and Concurrent React

## Table of Contents

- [What is Concurrent React](#what-is-concurrent-react)
- [Suspense for lazy loading](#suspense-for-lazy-loading)
- [Suspense for data fetching](#suspense-for-data-fetching)
- [useTransition](#usetransition)
- [useDeferredValue](#usedeferredvalue)
- [useTransition vs useDeferredValue](#usetransition-vs-usedeferredvalue)
- [Nested Suspense boundaries](#nested-suspense-boundaries)
- [Suspense with error boundaries](#suspense-with-error-boundaries)
- [Streaming SSR brief](#streaming-ssr-brief)
- [startViewTransition integration in React 19](#startviewtransition-integration-in-react-19)
- [Complete example: concurrent dashboard](#complete-example-concurrent-dashboard)
- [💻 Exercises](#-exercises)

## What is Concurrent React

Concurrent React is not a separate API you “turn on” for each component. It is the rendering model that lets React:

- interrupt a render
- pause work
- resume later
- abandon outdated work

That means React can keep urgent interactions responsive while lower-priority rendering happens in the background.

This is why typing can stay smooth even if the page is also recalculating a large filtered list.

Concurrent rendering has been available by default since React 18. React 19 continues building on that model.

## Suspense for lazy loading

`Suspense` is the loading boundary for async UI work.

The most common use is code splitting.

```jsx
import React, { Suspense } from 'react'

const Chart = React.lazy(() => import('./Chart'))

function Dashboard() {
  return (
    <Suspense fallback={<p>Loading chart...</p>}>
      <Chart />
    </Suspense>
  )
}
```

While the component bundle loads, React shows the fallback.

This is excellent for:

- dashboards
- admin routes
- modals with heavy editors
- rarely visited pages

## Suspense for data fetching

Suspense also works for data fetching when your library supports it.

With TanStack Query v5:

```jsx
import { Suspense } from 'react'
import { useSuspenseQuery } from '@tanstack/react-query'

async function fetchUsers() {
  const response = await fetch('https://jsonplaceholder.typicode.com/users')
  if (!response.ok) throw new Error('Failed to load users')
  return response.json()
}

function UsersList() {
  const { data } = useSuspenseQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  })

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}

export default function UsersPage() {
  return (
    <Suspense fallback={<p>Loading users...</p>}>
      <UsersList />
    </Suspense>
  )
}
```

The key idea is that the component can “suspend” until data is ready, and the nearest `Suspense` boundary handles the loading UI.

## useTransition

Use `useTransition` when you control the state update and want to mark part of it as non-urgent.

```jsx
import { useMemo, useState, useTransition } from 'react'

const items = Array.from({ length: 8000 }, (_, i) => ({
  id: i,
  name: `Product ${i + 1}`,
}))

export default function SearchPage() {
  const [inputValue, setInputValue] = useState('')
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()

  const filteredItems = useMemo(() => {
    return items.filter((item) =>
      item.name.toLowerCase().includes(query.toLowerCase())
    )
  }, [query])

  const handleSearch = (event) => {
    const value = event.target.value

    setInputValue(value)

    startTransition(() => {
      setQuery(value)
    })
  }

  return (
    <section>
      <input value={inputValue} onChange={handleSearch} placeholder="Search products" />
      {isPending && <p>Updating results...</p>}
      <p>{filteredItems.length} matches</p>
    </section>
  )
}
```

The input remains urgent. The heavy list update becomes interruptible.

## useDeferredValue

Use `useDeferredValue` when you have a value and want React to lag that value slightly for expensive rendering.

```jsx
import { useDeferredValue, useMemo, useState } from 'react'

function SearchResults({ items }) {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query)

  const filteredList = useMemo(() => {
    return items.filter((item) =>
      item.name.toLowerCase().includes(deferredQuery.toLowerCase())
    )
  }, [deferredQuery, items])

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <p>Searching for: {deferredQuery || 'everything'}</p>
      <ul>
        {filteredList.slice(0, 15).map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </>
  )
}
```

The displayed results may lag slightly behind typing, which is often acceptable if it keeps the UI responsive.

## useTransition vs useDeferredValue

| Tool | Use when |
| --- | --- |
| `useTransition` | you control the setter and want to wrap non-urgent updates |
| `useDeferredValue` | you already have a value and want a deferred version of it |

Rule of thumb:

- `useTransition` for **actions**
- `useDeferredValue` for **values**

## Nested Suspense boundaries

You can place smaller boundaries around slower parts of a page.

```jsx
<Suspense fallback={<PageSkeleton />}>
  <DashboardLayout>
    <Suspense fallback={<ChartSkeleton />}>
      <SalesChart />
    </Suspense>

    <Suspense fallback={<TableSkeleton />}>
      <OrdersTable />
    </Suspense>
  </DashboardLayout>
</Suspense>
```

Benefits:

- the page shell can show quickly
- fast sections can appear without waiting for slow sections
- loading states feel more precise

This is usually better than one giant fallback covering the whole page.

## Suspense with error boundaries

`Suspense` handles loading. It does **not** handle errors. Pair it with an error boundary.

One popular hooks-friendly approach is `react-error-boundary`.

```bash
npm install react-error-boundary
```

```jsx
import { Suspense } from 'react'
import { ErrorBoundary } from 'react-error-boundary'

function ErrorFallback() {
  return <p>Could not load reports.</p>
}

function AsyncSection() {
  return <ReportsPanel />
}

export default function ReportsPage() {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<p>Loading reports...</p>}>
        <AsyncSection />
      </Suspense>
    </ErrorBoundary>
  )
}
```

In real apps the architecture stays the same:

- error boundary for failures
- suspense boundary for loading

## Streaming SSR brief

Suspense is also important on the server.

With streaming SSR, React can:

1. send the page shell immediately
2. stream slower sections later
3. progressively reveal suspended content

This is especially useful in frameworks like Next.js and Remix. For this Vite-focused course, you mainly need the concept: Concurrent React makes progressive rendering possible.

## startViewTransition integration in React 19

React 19 can integrate with the browser View Transitions API so route or state changes can animate more smoothly.

Conceptually:

- the browser captures the old view
- React renders the new view
- the browser animates between them

A simplified browser API example looks like this:

```js
document.startViewTransition(() => {
  setPage('details')
})
```

Frameworks and routers may wrap this for you. The key takeaway is that React 19 is becoming better at coordinating rich UI transitions with browser-native primitives.

## Complete example: concurrent dashboard

```jsx
import React, {
  Suspense,
  useDeferredValue,
  useMemo,
  useState,
  useTransition,
} from 'react'

const AnalyticsPanel = React.lazy(() => import('./AnalyticsPanel'))

const reports = Array.from({ length: 5000 }, (_, i) => ({
  id: i,
  title: `Report ${i + 1}`,
}))

function ReportSearch() {
  const [input, setInput] = useState('')
  const [query, setQuery] = useState('')
  const [isPending, startTransition] = useTransition()
  const deferredQuery = useDeferredValue(query)

  const filteredReports = useMemo(() => {
    return reports.filter((report) =>
      report.title.toLowerCase().includes(deferredQuery.toLowerCase())
    )
  }, [deferredQuery])

  const handleChange = (event) => {
    const value = event.target.value
    setInput(value)

    startTransition(() => {
      setQuery(value)
    })
  }

  return (
    <section>
      <input value={input} onChange={handleChange} placeholder="Search reports..." />
      {isPending && <p>Refreshing results...</p>}
      <ul>
        {filteredReports.slice(0, 20).map((report) => (
          <li key={report.id}>{report.title}</li>
        ))}
      </ul>
    </section>
  )
}

export default function DashboardPage() {
  return (
    <>
      <h1>Dashboard</h1>

      <Suspense fallback={<p>Loading analytics...</p>}>
        <AnalyticsPanel />
      </Suspense>

      <ReportSearch />
    </>
  )
}
```

This combines:

- lazy loading with `Suspense`
- non-urgent updates with `useTransition`
- deferred rendering with `useDeferredValue`

## 💻 Exercises

### Level 1 – Lazy-load a heavy component

- Move a large component into its own file
- Load it with `React.lazy`
- Wrap it with `Suspense` and a fallback UI

### Level 2 – Responsive search with useTransition

- Build a search input over a large list
- Keep the input responsive with `useTransition`
- Show a pending indicator while results update

### Level 3 – Dashboard with multiple Suspense boundaries

- Create a dashboard with chart, table, and activity feed sections
- Wrap each section in its own `Suspense` boundary
- Add an error boundary around the whole async area
