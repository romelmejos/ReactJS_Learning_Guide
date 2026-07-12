[<< Day 18](../18_Day_Data_Fetching/18_data_fetching.md) | [Day 20 >>](../20_Day_useReducer/20_use_reducer.md)

# Day 19 – TanStack Query

## Table of Contents

- [What TanStack Query is](#what-tanstack-query-is)
- [Installation](#installation)
- [Setup](#setup)
- [`useQuery`](#usequery)
- [Query keys](#query-keys)
- [Stale time and cache lifetime](#stale-time-and-cache-lifetime)
- [Refetching](#refetching)
- [`useMutation`](#usemutation)
- [Optimistic updates](#optimistic-updates)
- [Dependent queries](#dependent-queries)
- [Infinite queries](#infinite-queries)
- [Prefetching](#prefetching)
- [When to use TanStack Query](#when-to-use-tanstack-query)
- [💻 Exercises](#-exercises)

## What TanStack Query is

TanStack Query is a server-state library for React.

It is **not** a replacement for `fetch` or Axios. Instead, it sits on top of them and manages the difficult parts of remote data:

- caching
- background refetching
- stale-while-revalidate behavior
- loading and error state tracking
- request deduplication
- mutations and invalidation

Think of it like this:

- `fetch` / Axios → how the request happens
- TanStack Query → how the app manages that data over time

## Installation

Install the main package:

```bash
npm install @tanstack/react-query
```

For development, add the devtools:

```bash
npm install @tanstack/react-query-devtools
```

## Setup

Create a `QueryClient`, then wrap your app with `QueryClientProvider`.

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import App from './App.jsx'

const queryClient = new QueryClient()

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
)
```

Once the provider is in place, query hooks work anywhere below it.

## `useQuery`

`useQuery` is the main hook for reading remote data.

```jsx
import { useQuery } from '@tanstack/react-query'

function Spinner() {
  return <p>Loading...</p>
}

export default function UserList() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users')

      if (!response.ok) {
        throw new Error('Failed to load users')
      }

      return response.json()
    },
  })

  if (isLoading) return <Spinner />
  if (isError) return <p>Error: {error.message}</p>

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

This removes a lot of manual `useEffect` boilerplate.

## Query keys

Every query needs a **query key**. This is how TanStack Query identifies cached data.

Examples:

```js
['users']
['users', userId]
['users', { filter: 'active' }]
```

Guidelines:

- use arrays, not plain strings
- include every value that affects the result
- keep keys stable and predictable

Example with a user detail query:

```jsx
function UserProfile({ userId }) {
  const { data } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
  })

  return <h2>{data.name}</h2>
}
```

If `userId` changes, the key changes, and TanStack Query knows it should fetch new data.

## Stale time and cache lifetime

Two important options:

- `staleTime` → how long data is considered fresh
- `gcTime` → how long unused cached data stays in memory before garbage collection

```jsx
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  staleTime: 1000 * 60 * 5,
  gcTime: 1000 * 60 * 30,
})
```

Meaning:

- for 5 minutes, the data is fresh
- after no components use it, keep it cached for 30 minutes

This is a major improvement over manual fetching because your app can revisit screens without immediately refetching.

## Refetching

TanStack Query makes refetch behavior configurable.

```jsx
const { data } = useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  refetchOnWindowFocus: true,
  refetchInterval: 30000,
})
```

Useful options:

- `refetchOnWindowFocus` → refresh when the tab becomes active again
- `refetchInterval` → polling
- manual invalidation via `queryClient.invalidateQueries`

Manual invalidation example:

```jsx
import { useQueryClient } from '@tanstack/react-query'

function RefreshUsersButton() {
  const queryClient = useQueryClient()

  return (
    <button onClick={() => queryClient.invalidateQueries({ queryKey: ['users'] })}>
      Refresh users
    </button>
  )
}
```

This tells TanStack Query that the cached users data should be considered stale and refetched.

## `useMutation`

Queries are for reading. Mutations are for creating, updating, and deleting.

```jsx
import axios from 'axios'
import { useMutation, useQueryClient } from '@tanstack/react-query'

export default function CreateUser() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: newUser => axios.post('/api/users', newUser).then(response => response.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
  })

  return (
    <button onClick={() => mutation.mutate({ name: 'Alice' })}>
      {mutation.isPending ? 'Creating...' : 'Create User'}
    </button>
  )
}
```

Common mutation states:

- `isPending`
- `isError`
- `isSuccess`

## Optimistic updates

Optimistic updates make the UI feel instant by updating the cache **before** the server confirms success.

```jsx
import axios from 'axios'
import { useMutation, useQueryClient } from '@tanstack/react-query'

function AddUserButton() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: newUser => axios.post('/api/users', newUser).then(response => response.data),
    onMutate: async newUser => {
      await queryClient.cancelQueries({ queryKey: ['users'] })

      const previousUsers = queryClient.getQueryData(['users']) || []

      queryClient.setQueryData(['users'], old => [...(old || []), newUser])

      return { previousUsers }
    },
    onError: (_error, _newUser, context) => {
      queryClient.setQueryData(['users'], context.previousUsers)
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
  })

  return <button onClick={() => mutation.mutate({ id: Date.now(), name: 'Optimistic Alice' })}>Add user</button>
}
```

This pattern gives users snappy feedback while still recovering if the server fails.

## Dependent queries

Sometimes a query should only run when another value exists.

```jsx
import { useQuery } from '@tanstack/react-query'

function UserProjects({ userId }) {
  const { data, isLoading } = useQuery({
    queryKey: ['projects', userId],
    queryFn: () => fetch(`/api/users/${userId}/projects`).then(r => r.json()),
    enabled: !!userId,
  })

  if (!userId) return <p>Select a user first.</p>
  if (isLoading) return <p>Loading projects...</p>

  return (
    <ul>
      {data.map(project => (
        <li key={project.id}>{project.name}</li>
      ))}
    </ul>
  )
}
```

`enabled: !!userId` prevents the query from running until `userId` is available.

## Infinite queries

For pagination and infinite scroll, use `useInfiniteQuery`.

```jsx
import { useInfiniteQuery } from '@tanstack/react-query'

export default function Feed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    initialPageParam: 1,
    queryFn: async ({ pageParam }) => {
      const response = await fetch(`/api/feed?page=${pageParam}`)

      if (!response.ok) {
        throw new Error('Failed to load feed')
      }

      return response.json()
    },
    getNextPageParam: lastPage => lastPage.nextPage ?? undefined,
  })

  return (
    <section>
      {data?.pages.flatMap(page =>
        page.items.map(item => <p key={item.id}>{item.title}</p>)
      )}

      <button onClick={() => fetchNextPage()} disabled={!hasNextPage || isFetchingNextPage}>
        {isFetchingNextPage ? 'Loading more...' : hasNextPage ? 'Load more' : 'No more items'}
      </button>
    </section>
  )
}
```

This is much easier than manually tracking multiple pages in component state.

## Prefetching

Prefetching loads data before the user navigates to a page.

```jsx
import { useQueryClient } from '@tanstack/react-query'
import { Link } from 'react-router'

function UserLink({ user }) {
  const queryClient = useQueryClient()

  async function handleMouseEnter() {
    await queryClient.prefetchQuery({
      queryKey: ['users', user.id],
      queryFn: () => fetch(`/api/users/${user.id}`).then(r => r.json()),
    })
  }

  return (
    <Link to={`/users/${user.id}`} onMouseEnter={handleMouseEnter}>
      {user.name}
    </Link>
  )
}
```

When the user clicks, the data may already be in cache, making the next screen feel instant.

## When to use TanStack Query

TanStack Query is a great fit when your app:

- reads server data in many places
- needs caching
- needs background synchronization
- performs frequent mutations
- benefits from retries and refetching

It is less necessary for:

- tiny apps with almost no remote data
- one-off demo components

Still, for most real production React apps, TanStack Query is one of the highest-value libraries you can add.

## 💻 Exercises

### Level 1 — Replace `useEffect` with `useQuery`

Take a component that fetches data manually and refactor it to:

- use `useQuery`
- show `isLoading`
- show `isError`
- render the result

### Level 2 — CRUD with mutation and invalidation

Build a user or todo feature with:

- list query
- create mutation
- update mutation
- delete mutation
- cache invalidation after writes

Bonus: add a loading label on mutation buttons.

### Level 3 — Infinite scroll

Build an infinite feed with:

- `useInfiniteQuery`
- a next-page cursor or page number
- load-more button or intersection observer
- empty state
- error state

Bonus: prefetch the next page before the user clicks.
