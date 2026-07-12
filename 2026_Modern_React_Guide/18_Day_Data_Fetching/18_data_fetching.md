[<< Day 17](../17_Day_React_Router_v7/17_react_router_v7.md) | [Day 19 >>](../19_Day_TanStack_Query/19_tanstack_query.md)

# Day 18 – Data Fetching

## Table of Contents

- [Why data fetching feels hard](#why-data-fetching-feels-hard)
- [The baseline pattern: fetch plus useEffect](#the-baseline-pattern-fetch-plus-useeffect)
- [Why fetch does not throw on 4xx or 5xx](#why-fetch-does-not-throw-on-4xx-or-5xx)
- [Axios vs fetch](#axios-vs-fetch)
- [Using Axios](#using-axios)
- [Creating an Axios instance](#creating-an-axios-instance)
- [Race conditions](#race-conditions)
- [Loading, error, and success state](#loading-error-and-success-state)
- [Parallel requests](#parallel-requests)
- [Why this does not scale well](#why-this-does-not-scale-well)
- [💻 Exercises](#-exercises)

## Why data fetching feels hard

Fetching data sounds simple: request data, show data.

In real apps, it becomes harder because you must manage:

- loading states
- error states
- retries
- cancellation
- race conditions
- stale data
- caching

If you do everything manually, components can quickly become cluttered.

Today you will learn the **baseline** approach first. Tomorrow, you will see how TanStack Query improves the experience.

## The baseline pattern: fetch plus useEffect

The most common manual pattern is:

1. create state for data
2. create state for loading
3. create state for error
4. fetch inside `useEffect`
5. clean up in-flight requests

```jsx
import { useEffect, useState } from 'react'

function Spinner() {
  return <p>Loading...</p>
}

function ErrorMessage({ message }) {
  return <p style={{ color: 'crimson' }}>Error: {message}</p>
}

export default function UserList() {
  const [users, setUsers] = useState([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    const controller = new AbortController()

    fetch('https://jsonplaceholder.typicode.com/users', {
      signal: controller.signal,
    })
      .then(response => {
        if (!response.ok) {
          throw new Error('Network error')
        }

        return response.json()
      })
      .then(data => {
        setUsers(data)
        setLoading(false)
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err.message)
          setLoading(false)
        }
      })

    return () => controller.abort()
  }, [])

  if (loading) return <Spinner />
  if (error) return <ErrorMessage message={error} />

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

This pattern is valid, and you should understand it even if you later use a library.

## Why fetch does not throw on 4xx or 5xx

This surprises many developers:

```js
const response = await fetch('/api/users')
```

`fetch` only rejects for **network-level failures** such as:

- offline connection
- DNS failure
- request abortion

It does **not** automatically reject when the server returns:

- `404`
- `401`
- `500`

So you must check `response.ok` yourself:

```js
const response = await fetch('/api/users')

if (!response.ok) {
  throw new Error(`Request failed with status ${response.status}`)
}

const data = await response.json()
```

This is one reason many teams prefer Axios.

## Axios vs fetch

Both are useful. They solve similar problems, but their ergonomics differ.

| Feature | Fetch | Axios |
| --- | --- | --- |
| Built into browser | Yes | No |
| Installation required | No | Yes |
| Automatic JSON parsing | No | Yes |
| Throws on 4xx/5xx | No | Yes |
| Interceptors | No | Yes |
| Better defaults for APIs | Minimal | Strong |

### Install Axios

```bash
npm install axios
```

### When fetch is enough

Use fetch when:

- you want zero dependencies
- your API usage is simple
- you only have a few endpoints

### When Axios shines

Use Axios when:

- you need auth headers everywhere
- you want interceptors
- you want simpler request code
- your app talks to APIs heavily

## Using Axios

Axios automatically parses JSON and throws for non-2xx status codes.

```jsx
import { useEffect, useState } from 'react'
import axios from 'axios'

export default function PostList() {
  const [posts, setPosts] = useState([])
  const [error, setError] = useState(null)

  useEffect(() => {
    async function loadPosts() {
      try {
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/posts')
        setPosts(data.slice(0, 5))
      } catch (err) {
        setError(err.message)
      }
    }

    loadPosts()
  }, [])

  if (error) return <p>Error: {error}</p>

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

This is shorter than the fetch version because Axios handles some defaults for you.

## Creating an Axios instance

In real apps, do not repeat `baseURL`, headers, and timeout options everywhere.

```js
// src/lib/axios.js
import axios from 'axios'

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
})

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')

  if (token) {
    config.headers.Authorization = 'Bearer ' + token
  }

  return config
})
```

Then use the shared instance:

```js
import { api } from '../lib/axios'

export async function getUsers() {
  const { data } = await api.get('/users')
  return data
}
```

Interceptors are powerful for:

- auth tokens
- logging
- response normalization
- automatic refresh flows

## Race conditions

A race condition happens when two requests are in flight and the older one finishes last, overwriting newer data.

Example: a search box sends requests for:

- `r`
- `re`
- `rea`
- `react`

If the `r` request returns after `react`, your UI may show outdated results.

### Fix 1: AbortController

```jsx
import { useEffect, useState } from 'react'

export default function SearchUsers({ query }) {
  const [results, setResults] = useState([])

  useEffect(() => {
    if (!query) {
      setResults([])
      return
    }

    const controller = new AbortController()

    async function search() {
      const response = await fetch(`/api/users?search=${encodeURIComponent(query)}`, {
        signal: controller.signal,
      })

      if (!response.ok) {
        throw new Error('Search failed')
      }

      const data = await response.json()
      setResults(data)
    }

    search().catch(error => {
      if (error.name !== 'AbortError') {
        console.error(error)
      }
    })

    return () => controller.abort()
  }, [query])

  return (
    <ul>
      {results.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Fix 2: Cancelled flag

If a library does not support `AbortController`, you can ignore outdated results:

```jsx
useEffect(() => {
  let cancelled = false

  async function loadData() {
    const response = await fetch('/api/items')
    const data = await response.json()

    if (!cancelled) {
      setItems(data)
    }
  }

  loadData()

  return () => {
    cancelled = true
  }
}, [])
```

Aborting is usually better because it stops wasted work.

## Loading, error, and success state

Nearly every fetching component has three core states:

1. **loading** → request in progress
2. **error** → request failed
3. **success** → data is available

A clear rendering pattern helps:

```jsx
if (loading) return <Spinner />
if (error) return <ErrorMessage message={error} />
return <DataView data={data} />
```

You can also model the state more explicitly:

```js
const [status, setStatus] = useState('idle')
```

Possible values:

- `idle`
- `loading`
- `success`
- `error`

This can be easier to reason about than juggling multiple booleans.

## Parallel requests

If two requests are independent, do not wait for them one by one.

```jsx
import { useEffect, useState } from 'react'

export default function Dashboard() {
  const [data, setData] = useState({ users: [], posts: [] })

  useEffect(() => {
    async function loadDashboard() {
      const [usersResponse, postsResponse] = await Promise.all([
        fetch('/api/users'),
        fetch('/api/posts'),
      ])

      if (!usersResponse.ok || !postsResponse.ok) {
        throw new Error('Failed to load dashboard data')
      }

      const [users, posts] = await Promise.all([
        usersResponse.json(),
        postsResponse.json(),
      ])

      setData({ users, posts })
    }

    loadDashboard().catch(console.error)
  }, [])

  return (
    <section>
      <h2>Users: {data.users.length}</h2>
      <h2>Posts: {data.posts.length}</h2>
    </section>
  )
}
```

`Promise.all` is ideal when requests do not depend on each other.

## Why this does not scale well

Manual fetching works, but it starts to hurt as apps grow:

- no built-in caching
- no automatic request deduplication
- no background refetching
- repeated loading/error logic everywhere
- lots of boilerplate

This is exactly why libraries like **TanStack Query** exist.

Tomorrow you will learn how to keep your fetching logic while offloading caching, refetching, and synchronization to a dedicated tool.

## 💻 Exercises

### Level 1 — Fetch posts

Use `fetch` and `useEffect` to load posts from:

- `https://jsonplaceholder.typicode.com/posts`

Show:

- loading state
- error state
- list of post titles

### Level 2 — Paginated list

Build a paginated users or posts view with:

- current page state
- Previous button
- Next button
- disabled buttons at boundaries

Store page in React state or in the query string if you want an extra challenge.

### Level 3 — Search as you type

Build a search UI with:

- controlled input
- debouncing
- `AbortController`
- loading and error states
- empty-state message

Goal: prevent stale requests from overwriting newer search results.
