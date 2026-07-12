[<< Day 9](../09_Day_Conditional_Rendering/09_conditional_rendering.md) | [Day 11 >>](../11_Day_Events/11_events.md)

# Day 10 – useEffect

## Table of Contents

- [What is a Side Effect?](#what-is-a-side-effect)
- [The `useEffect` Hook](#the-useeffect-hook)
- [The Dependency Array](#the-dependency-array)
- [Cleanup Functions](#cleanup-functions)
- [Common Patterns](#common-patterns)
- [StrictMode Double-Invocation](#strictmode-double-invocation)
- [The Stale Closure Problem](#the-stale-closure-problem)
- [What Not to Put in `useEffect`](#what-not-to-put-in-useeffect)
- [`useEffect` vs Event Handlers](#useeffect-vs-event-handlers)
- [React 19 Note](#react-19-note)
- [Summary](#summary)
- [💻 Exercises](#-exercises)

## What is a Side Effect?

A side effect is anything your component does that reaches **outside** React's pure render process.

To try today's effects in the Vite starter:

```bash
cd boilerplate
npm install
npm run dev
```

Examples:

- fetching data
- starting a timer
- subscribing to events
- updating `document.title`
- writing to `localStorage`
- logging to an analytics service

```js
const sideEffects = [
  'fetch data',
  'start timer',
  'subscribe to events',
  'update browser APIs',
]
```

Rendering should stay pure: given the same props and state, it should return the same JSX. Side effects are the extra work that happens **after render**.

## The `useEffect` Hook

`useEffect` is React's hook for running side effects after the component renders.

```jsx
import { useEffect } from 'react'

useEffect(() => {
  // side effect code
  return () => {
    // cleanup (optional)
  }
}, [dependencies])
```

Basic example:

```jsx
import { useEffect, useState } from 'react'

function PageCounter() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    document.title = `Count: ${count}`
  }, [count])

  return <button onClick={() => setCount(prev => prev + 1)}>{count}</button>
}
```

When `count` changes, React re-renders the component and then runs the effect again.

## The Dependency Array

The dependency array tells React **when** the effect should run.

### 1. Empty array: run once after mount

```jsx
useEffect(() => {
  console.log('Mounted once')
}, [])
```

### 2. Specific dependencies: run after mount and when they change

```jsx
useEffect(() => {
  console.log('Query changed:', query)
}, [query])
```

### 3. No dependency array: run after every render

```jsx
useEffect(() => {
  console.log('Runs after every render')
})
```

This third form is rarely what you want because it is easy to cause unnecessary work or effect loops.

## Cleanup Functions

An effect may return a cleanup function. React runs it:

- before the effect runs again
- when the component unmounts

This is essential for timers, subscriptions, listeners, and in-flight requests.

### Timer cleanup

```jsx
import { useEffect, useState } from 'react'

function Clock() {
  const [seconds, setSeconds] = useState(0)

  useEffect(() => {
    const id = setInterval(() => {
      setSeconds(prev => prev + 1)
    }, 1000)

    return () => clearInterval(id)
  }, [])

  return <p>Elapsed: {seconds}s</p>
}
```

### Fetch cleanup with `AbortController`

```jsx
import { useEffect, useState } from 'react'

function Users() {
  const [data, setData] = useState([])
  const [error, setError] = useState(null)

  useEffect(() => {
    const controller = new AbortController()

    fetch('https://jsonplaceholder.typicode.com/users', {
      signal: controller.signal,
    })
      .then(response => response.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err)
        }
      })

    return () => controller.abort()
  }, [])

  if (error) return <p>Failed to load users.</p>

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

## Common Patterns

### Data fetching

Use an effect when a component must synchronize with an external data source.

```jsx
import { useEffect, useState } from 'react'

function Posts() {
  const [posts, setPosts] = useState([])
  const [status, setStatus] = useState('loading')

  useEffect(() => {
    let ignore = false

    async function loadPosts() {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts')
        const result = await response.json()

        if (!ignore) {
          setPosts(result.slice(0, 5))
          setStatus('success')
        }
      } catch {
        if (!ignore) {
          setStatus('error')
        }
      }
    }

    loadPosts()

    return () => {
      ignore = true
    }
  }, [])

  if (status === 'loading') return <p>Loading...</p>
  if (status === 'error') return <p>Failed to load posts.</p>

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### Document title

```jsx
useEffect(() => {
  document.title = `Cart (${itemCount})`
}, [itemCount])
```

### Event listeners

```jsx
import { useEffect } from 'react'

function WindowWidthLogger() {
  useEffect(() => {
    function handleResize() {
      console.log(window.innerWidth)
    }

    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])

  return <p>Resize the window and check the console.</p>
}
```

### WebSocket or EventSource setup

```jsx
useEffect(() => {
  const socket = new WebSocket('wss://example.com/socket')

  socket.addEventListener('message', event => {
    console.log(event.data)
  })

  return () => socket.close()
}, [])
```

### Timers

```jsx
useEffect(() => {
  const id = setTimeout(() => {
    console.log('Debounced action')
  }, 500)

  return () => clearTimeout(id)
}, [query])
```

## StrictMode Double-Invocation

In React 19 development mode, StrictMode intentionally mounts, unmounts, and remounts components to help you catch effect bugs.

That means an effect may appear to run twice in development.

This is not a production bug. It is a safety check.

Your effects must be **idempotent**:

- they should tolerate running more than once
- cleanup should fully undo setup

If an effect breaks in StrictMode, it usually means the cleanup logic is incomplete.

## The Stale Closure Problem

Effects capture values from the render in which they were created. If you leave a reactive value out of the dependency array, the effect may keep using an old value.

```jsx
useEffect(() => {
  console.log(searchTerm)
}, []) // ❌ stale value if searchTerm changes later
```

Correct version:

```jsx
useEffect(() => {
  console.log(searchTerm)
}, [searchTerm])
```

This is why ESLint's `react-hooks/exhaustive-deps` rule matters so much. It warns when your effect depends on values that are not listed.

## What Not to Put in `useEffect`

Not every piece of logic belongs in an effect.

### Do not use effects for pure render transformations

```jsx
const visibleItems = items.filter(item => item.visible)
```

That should be computed during render, not stored with an effect.

### Do not mirror state from other state unless absolutely necessary

```jsx
// usually unnecessary
useEffect(() => {
  setFullName(`${firstName} ${lastName}`)
}, [firstName, lastName])
```

Instead:

```jsx
const fullName = `${firstName} ${lastName}`
```

Effects are for syncing with **external systems**, not for managing ordinary render logic.

## `useEffect` vs Event Handlers

Ask this question:

**Did this happen because the user did something right now?**

If yes, use an **event handler**.

```jsx
function SaveButton() {
  const handleClick = () => {
    console.log('Saving...')
  }

  return <button onClick={handleClick}>Save</button>
}
```

Ask a different question:

**Does this component need to stay synchronized with something outside React?**

If yes, use **`useEffect`**.

```jsx
useEffect(() => {
  document.title = title
}, [title])
```

This distinction prevents overusing effects.

## React 19 Note

React 19's compiler work can reduce the need for some manual memoization patterns, but it does **not** remove the need to understand effects.

You still need `useEffect` whenever your component must synchronize with:

- the network
- browser APIs
- timers
- subscriptions
- external systems

Understanding effects is still foundational React knowledge.

## Summary

- Effects run after render
- Use them for side effects, not pure calculations
- The dependency array controls when they rerun
- Cleanup functions prevent leaks and stale subscriptions
- StrictMode exposes cleanup bugs in development
- Missing dependencies can cause stale closures
- Prefer event handlers for user-triggered actions

## 💻 Exercises

### Level 1 — Basic

Fetch a list from a public API and display it.

Requirements:

- show loading text first
- render the list on success
- show an error message on failure

### Level 2 — Intermediate

Build a live search with debounced API calls.

Requirements:

- store the search term in state
- wait a short time before fetching
- cancel or ignore outdated requests
- clean up timers properly

### Level 3 — Challenge

Build a real-time counter using `setInterval`.

Requirements:

- start counting when the component mounts
- stop counting when it unmounts
- add pause and resume buttons
- ensure cleanup works correctly in StrictMode
