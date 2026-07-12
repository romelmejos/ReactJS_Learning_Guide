[<< Day 25](../25_Day_Suspense_Concurrent/25_suspense_concurrent.md) | [Day 27 >>](../27_Day_Global_State_Management/27_global_state_management.md)

# Day 26 – `use()` Hook & Server Components

## Table of Contents
- [Why Day 26 Matters](#why-day-26-matters)
- [The `use()` Hook](#the-use-hook)
- [`use()` with Promises](#use-with-promises)
- [`use()` with Context](#use-with-context)
- [`use()` vs `useEffect` + `useState` for Data](#use-vs-useeffect--usestate-for-data)
- [React Server Components (RSC) Introduction](#react-server-components-rsc-introduction)
- [The `"use client"` Directive](#the-use-client-directive)
- [Server Component vs Client Component](#server-component-vs-client-component)
- [RSC in Practice with Vite](#rsc-in-practice-with-vite)
- [Server Actions](#server-actions)
- [Plain React vs Next.js](#plain-react-vs-nextjs)
- [Putting It Together](#putting-it-together)
- [💻 Exercises](#-exercises)

## Why Day 26 Matters

React 19 introduced a major idea shift:

- data can be **read during render**
- Suspense can handle waiting states
- some components can run only on the **server**
- not every React component needs to ship JavaScript to the browser

This lesson introduces two related ideas:

1. the new `use()` API
2. React Server Components and the frameworks that support them

Even if you build plain Vite apps today, understanding these patterns is essential because they shape modern React architecture in 2026.

---

## The `use()` Hook

`use()` is a new React 19 API that can read:

- a **Promise**
- a **Context**

```jsx
import { use } from 'react'
```

Unlike most hooks, `use()` is special:

- it can be called conditionally
- it can be called in loops
- when given a Promise, it integrates with Suspense

That makes it different from hooks like `useState`, `useEffect`, or `useContext`, which must follow the normal Rules of Hooks.

### Mental model

Think of `use()` as:  
**“Read this value now, and if it is not ready, suspend rendering until it is.”**

---

## `use()` with Promises

The most exciting use case is reading a Promise directly during render.

```jsx
import { use, Suspense } from 'react'

function UserCard({ userPromise }) {
  const user = use(userPromise)

  return (
    <article className="card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </article>
  )
}

function Spinner() {
  return <p>Loading user...</p>
}

function fetchUser(id) {
  return fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
    .then(response => {
      if (!response.ok) {
        throw new Error('Failed to load user')
      }

      return response.json()
    })
}

export default function App() {
  const userPromise = fetchUser(1)

  return (
    <Suspense fallback={<Spinner />}>
      <UserCard userPromise={userPromise} />
    </Suspense>
  )
}
```

### Important rule: create the Promise outside the reading component

This matters:

```jsx
function App() {
  const userPromise = fetchUser(1)
  return <UserCard userPromise={userPromise} />
}
```

This is usually okay because the Promise is created in a parent and passed down.

Avoid starting a fresh Promise every render inside the component that reads it:

```jsx
function UserCard() {
  const user = use(fetchUser(1)) // not ideal
  return <div>{user.name}</div>
}
```

That pattern can cause repeated work because rendering may restart.

### `use()` is not `await`

`await` can only be used inside an `async` function:

```js
async function loadUser() {
  const user = await fetchUser(1)
  return user
}
```

`use()` works inside a component render:

```jsx
function UserCard({ userPromise, showDetails }) {
  const user = use(userPromise)

  if (showDetails) {
    return <pre>{JSON.stringify(user, null, 2)}</pre>
  }

  return <div>{user.name}</div>
}
```

It can also be conditional:

```jsx
import { use } from 'react'

function Dashboard({ shouldLoadProfile, profilePromise }) {
  if (!shouldLoadProfile) {
    return <p>Profile loading skipped.</p>
  }

  const profile = use(profilePromise)
  return <h2>{profile.name}</h2>
}
```

That would be illegal with normal hooks.

### Promise + Suspense + Error Boundary flow

- if the Promise is pending, the component suspends
- React shows the nearest `<Suspense fallback={...}>`
- if the Promise resolves, React renders with the value
- if the Promise rejects, the error goes to an Error Boundary

```jsx
import { Suspense, use } from 'react'

function ProductDetails({ productPromise }) {
  const product = use(productPromise)
  return <h2>{product.title}</h2>
}

function ProductSection({ productPromise }) {
  return (
    <Suspense fallback={<p>Loading product...</p>}>
      <ProductDetails productPromise={productPromise} />
    </Suspense>
  )
}
```

---

## `use()` with Context

`use()` can also read context values:

```jsx
import { createContext, use, useState } from 'react'

const ThemeContext = createContext('light')

function ThemeLabel({ showTheme }) {
  if (!showTheme) {
    return <p>Theme hidden</p>
  }

  const theme = use(ThemeContext)
  return <p>Current theme: {theme}</p>
}

export default function App() {
  const [showTheme, setShowTheme] = useState(true)

  return (
    <ThemeContext value="dark">
      <button onClick={() => setShowTheme(value => !value)}>
        Toggle theme label
      </button>

      <ThemeLabel showTheme={showTheme} />
    </ThemeContext>
  )
}
```

This is conceptually similar to `useContext(ThemeContext)`, but `use()` can be called conditionally.

### Why this matters

Normally, this is invalid:

```jsx
if (showTheme) {
  const theme = useContext(ThemeContext) // ❌ normal hooks cannot be conditional
}
```

But with `use()`:

```jsx
if (showTheme) {
  const theme = use(ThemeContext) // ✅ allowed
}
```

Use this power carefully. “Allowed” does not always mean “the clearest design.” Prefer readable component structure first.

---

## `use()` vs `useEffect` + `useState` for Data

For years, many React apps fetched data like this:

```jsx
import { useEffect, useState } from 'react'

function UserCard({ userId }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    let cancelled = false

    async function loadUser() {
      try {
        setLoading(true)
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`
        )

        if (!response.ok) {
          throw new Error('Failed to fetch user')
        }

        const data = await response.json()

        if (!cancelled) {
          setUser(data)
          setError(null)
        }
      } catch (err) {
        if (!cancelled) {
          setError(err)
        }
      } finally {
        if (!cancelled) {
          setLoading(false)
        }
      }
    }

    loadUser()

    return () => {
      cancelled = true
    }
  }, [userId])

  if (loading) return <p>Loading...</p>
  if (error) return <p>{error.message}</p>

  return <div>{user.name}</div>
}
```

This works, but it is verbose:

- local loading state
- local error state
- cleanup logic
- extra render after mount

With `use()` and Suspense:

```jsx
import { Suspense, use } from 'react'

function UserCard({ userPromise }) {
  const user = use(userPromise)
  return <div>{user.name}</div>
}

function App() {
  const userPromise = fetch(
    'https://jsonplaceholder.typicode.com/users/1'
  ).then(response => response.json())

  return (
    <Suspense fallback={<p>Loading...</p>}>
      <UserCard userPromise={userPromise} />
    </Suspense>
  )
}
```

### What improves

- no manual loading flag
- less effect code
- Suspense handles pending UI
- render reads the data directly

### Important nuance

For real client-side apps, **TanStack Query** is still the most common solution for data fetching in plain React + Vite projects. `use()` is powerful, but framework support and caching strategy matter.

Use the right tool:

- **plain Vite SPA** → TanStack Query is often best
- **RSC framework** like Next.js App Router → `use()` and server-first data patterns become more natural

---

## React Server Components (RSC) Introduction

React Server Components split your UI into two categories.

### 1. Server Components

Server Components:

- run on the server
- do **not** ship their component code to the browser
- can read databases, files, secrets, and server-only APIs
- can `await` directly
- reduce client bundle size

Example:

```jsx
// app/repos/page.jsx
export default async function ReposPage() {
  const response = await fetch('https://api.github.com/orgs/facebook/repos', {
    headers: {
      Accept: 'application/vnd.github+json',
    },
  })

  const repos = await response.json()

  return (
    <main>
      <h1>Popular React Repositories</h1>
      <ul>
        {repos.slice(0, 5).map(repo => (
          <li key={repo.id}>{repo.name}</li>
        ))}
      </ul>
    </main>
  )
}
```

### 2. Client Components

Client Components:

- support interactivity
- can use state and effects
- can attach event handlers
- are included in the browser JavaScript bundle

Example:

```jsx
'use client'

import { useState } from 'react'

export function LikeButton() {
  const [liked, setLiked] = useState(false)

  return (
    <button onClick={() => setLiked(value => !value)}>
      {liked ? 'Liked' : 'Like'}
    </button>
  )
}
```

### Why RSC is a big deal

Historically, every React component ended up becoming client JavaScript. With RSC, only the interactive pieces need client code.

That means:

- smaller bundles
- better performance
- simpler server data access
- fewer “fetch in effect after mount” patterns

---

## The `"use client"` Directive

`"use client"` marks a file as a Client Component boundary.

```jsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  )
}
```

### Rules

- it must be at the **top of the file**
- it applies to that file's exports
- you only add it where interactivity is needed

### Common misconception

You do **not** put `"use client"` everywhere in a Next.js App Router project. Doing so removes many benefits of Server Components.

Good pattern:

- keep page/layout/data-heavy shells as server components
- isolate interactive islands in small client components

Example:

```jsx
// app/dashboard/page.jsx
import { Counter } from './Counter'

export default async function DashboardPage() {
  const stats = await getDashboardStats()

  return (
    <main>
      <h1>Dashboard</h1>
      <p>Total users: {stats.users}</p>
      <Counter />
    </main>
  )
}
```

```jsx
// app/dashboard/Counter.jsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Local clicks: {count}
    </button>
  )
}
```

---

## Server Component vs Client Component

|  | Server Component | Client Component |
|--|--|--|
| Runs on | Server | Browser (and server for SSR) |
| Can use hooks | ❌ | ✅ |
| Can use event handlers | ❌ | ✅ |
| Can access DB/FS | ✅ | ❌ |
| Bundle size | 0 | Added to JS bundle |
| Directive | none (default) | `"use client"` |

### Practical reading of the table

- If you need `useState`, it must be a Client Component.
- If you need a database query, that belongs naturally in a Server Component.
- If you only render static or fetched content, prefer a Server Component when your framework supports it.

---

## RSC in Practice with Vite

Here is the key reality:

### Plain Vite does not support RSC natively

If you create an app with:

```bash
npm create vite@latest my-app -- --template react
```

you get a standard client-side React app. That app supports React 19 features like hooks, Actions, and Suspense patterns, but **not a full React Server Components runtime**.

### Frameworks that support RSC well

- **Next.js 15** (App Router)
- **Remix**
- **Waku**

So for learning RSC, you usually study framework examples rather than raw Vite examples.

### What this means for you

- use Vite for classic SPA learning and client-side React development
- use Next.js or Remix when you need server components, server actions, and full-stack routing

---

## Server Actions

Server Actions let forms and mutations call server-side logic directly in supported frameworks.

Example in Next.js:

```jsx
// app/posts/actions.js
'use server'

import { db } from '@/lib/db'

export async function createPost(formData) {
  const title = formData.get('title')

  if (!title || title.trim().length < 3) {
    throw new Error('Title must be at least 3 characters long')
  }

  await db.post.create({
    data: {
      title: title.trim(),
    },
  })
}
```

Then use that action directly in a form:

```jsx
// app/posts/NewPostForm.jsx
import { createPost } from './actions'

export default function NewPostForm() {
  return (
    <form action={createPost}>
      <label htmlFor="title">Post title</label>
      <input id="title" name="title" />
      <button type="submit">Create post</button>
    </form>
  )
}
```

### Why this feels different

Traditional SPA flow:

1. handle submit in client JavaScript
2. call `fetch('/api/posts', { method: 'POST' })`
3. parse response
4. update UI manually

Server Action flow:

1. submit form
2. framework calls server function directly
3. server mutates data
4. framework revalidates or refreshes affected UI

### Another example with client interactivity

```jsx
// app/todos/actions.js
'use server'

import { db } from '@/lib/db'

export async function addTodo(formData) {
  const text = formData.get('text')

  if (!text) {
    throw new Error('Todo text is required')
  }

  await db.todo.create({
    data: {
      text,
      completed: false,
    },
  })
}
```

```jsx
// app/todos/page.jsx
import { addTodo } from './actions'

export default async function TodosPage() {
  const todos = await db.todo.findMany({
    orderBy: { id: 'desc' },
  })

  return (
    <main>
      <h1>Todos</h1>

      <form action={addTodo}>
        <input name="text" placeholder="Add a todo" />
        <button type="submit">Save</button>
      </form>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </main>
  )
}
```

---

## Plain React vs Next.js

There is no universal winner. Choose based on app needs.

| Need | Plain React + Vite | Next.js / Remix |
|--|--|--|
| Fast SPA development | ✅ excellent | ✅ possible |
| SEO-heavy marketing pages | ⚠️ limited without SSR | ✅ strong |
| Server Components | ❌ | ✅ |
| Server Actions | ❌ | ✅ |
| Simple static dashboard | ✅ great fit | ✅ also works |
| Full-stack app with server rendering | ⚠️ more manual setup | ✅ strong default |
| Tight control over client-only architecture | ✅ | ✅ |

### Choose plain React + Vite when

- you are building an SPA
- SEO is not critical
- you want a simpler toolchain
- APIs already exist separately
- your team wants client-only deployment

### Choose Next.js when

- SEO matters
- you want file-based routing plus layouts
- you want RSC and Server Actions
- you need full-stack React in one project
- you want server-first data fetching

### Easy rule of thumb

- **internal dashboards, admin tools, client-heavy apps** → Vite is often enough
- **content-heavy, commerce, SEO, full-stack product apps** → Next.js is often the better choice

---

## Putting It Together

Here is a realistic split in a Next.js App Router app:

```jsx
// app/repos/page.jsx
import RepoFilter from './RepoFilter'

async function getRepos() {
  const response = await fetch('https://api.github.com/search/repositories?q=topic:react', {
    headers: {
      Accept: 'application/vnd.github+json',
    },
    cache: 'no-store',
  })

  if (!response.ok) {
    throw new Error('Failed to fetch repositories')
  }

  const data = await response.json()
  return data.items.slice(0, 10)
}

export default async function ReposPage() {
  const repos = await getRepos()

  return (
    <main>
      <h1>React Repositories</h1>
      <RepoFilter repos={repos} />
    </main>
  )
}
```

```jsx
// app/repos/RepoFilter.jsx
'use client'

import { useMemo, useState } from 'react'

export default function RepoFilter({ repos }) {
  const [query, setQuery] = useState('')

  const filteredRepos = useMemo(() => {
    const normalized = query.toLowerCase().trim()

    if (!normalized) {
      return repos
    }

    return repos.filter(repo =>
      repo.name.toLowerCase().includes(normalized)
    )
  }, [query, repos])

  return (
    <section>
      <input
        value={query}
        onChange={event => setQuery(event.target.value)}
        placeholder="Filter repositories"
      />

      <ul>
        {filteredRepos.map(repo => (
          <li key={repo.id}>
            <a href={repo.html_url} target="_blank" rel="noreferrer">
              {repo.name}
            </a>
          </li>
        ))}
      </ul>
    </section>
  )
}
```

This is a good RSC mental model:

- **server** fetches the data
- **client** handles local interactivity

---

## 💻 Exercises

### Level 1 – Read a Promise with `use()`

Build a small profile card:

1. Create a `fetchUser(id)` function that returns a Promise.
2. In `App`, create `const userPromise = fetchUser(1)`.
3. Pass that Promise to `<UserCard userPromise={userPromise} />`.
4. Wrap it in `<Suspense fallback={<p>Loading profile...</p>}>`.
5. Render the user's name, email, and company.

**Stretch:** Add an Error Boundary and simulate a rejected Promise.

### Level 2 – Minimal Next.js App Router Demo

Create a fresh App Router project:

```bash
npx create-next-app@latest my-rsc-demo
cd my-rsc-demo
npm run dev
```

Then:

1. Make `app/page.jsx` an async Server Component.
2. Fetch a list of posts from `https://jsonplaceholder.typicode.com/posts`.
3. Show the first 5 post titles.
4. Add a separate `"use client"` component with a button that toggles details on and off.

Goal: clearly separate server-rendered content from client interactivity.

### Level 3 – Full Server + Client Split

Build a mini dashboard with:

- a **Server Component** that reads data from a database or mock server function
- a **Client Component** that filters, sorts, or favorites those results
- a form using a **Server Action** to create a new item

Requirements:

1. The server component must fetch data without `useEffect`.
2. The client component must use `useState`.
3. The mutation must happen in a `'use server'` action.
4. Re-render the list after submission.

If you are not ready for a real DB, simulate one with a server-only array or a small Prisma/SQLite setup in Next.js.
