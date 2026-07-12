[<< Day 21](../21_Day_Context_API/21_context_api.md) | [Day 23 >>](../23_Day_useRef_DOM_Access/23_use_ref.md)

# Day 22 – Custom Hooks

## Table of Contents

- [What are custom hooks](#what-are-custom-hooks)
- [Why build custom hooks](#why-build-custom-hooks)
- [Building useFetch](#building-usefetch)
- [Building useLocalStorage](#building-uselocalstorage)
- [Building useDebounce](#building-usedebounce)
- [Building useWindowSize](#building-usewindowsize)
- [Building useToggle](#building-usetoggle)
- [Rules of custom hooks](#rules-of-custom-hooks)
- [Sharing custom hooks](#sharing-custom-hooks)
- [Testing custom hooks](#testing-custom-hooks)
- [Complete example: reusable hooks folder](#complete-example-reusable-hooks-folder)
- [💻 Exercises](#-exercises)

## What are custom hooks

Custom hooks are regular JavaScript functions whose names start with `use` and which call other React hooks.

They let you extract **reusable stateful logic**.

Important idea:

- components share **UI**
- custom hooks share **logic**

They do **not** create shared state automatically.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine)

  useEffect(() => {
    const goOnline = () => setIsOnline(true)
    const goOffline = () => setIsOnline(false)

    window.addEventListener('online', goOnline)
    window.addEventListener('offline', goOffline)

    return () => {
      window.removeEventListener('online', goOnline)
      window.removeEventListener('offline', goOffline)
    }
  }, [])

  return isOnline
}
```

Every component calling `useOnlineStatus()` gets its own stateful logic instance.

## Why build custom hooks

Custom hooks are useful because they improve:

- **DRYness**: avoid copy-pasting state and effects
- **encapsulation**: hide messy implementation details
- **testability**: test logic separately from UI markup
- **separation of concerns**: keep components focused on rendering

Without a custom hook, several components may repeat the same effect, cleanup, and loading logic. With a hook, you write it once and reuse it.

## Building useFetch

This is a classic beginner-friendly hook.

```jsx
import { useEffect, useState } from 'react'

export function useFetch(url) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    const controller = new AbortController()

    setLoading(true)
    setError(null)

    fetch(url, { signal: controller.signal })
      .then((response) => {
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }

        return response.json()
      })
      .then((json) => {
        setData(json)
        setLoading(false)
      })
      .catch((err) => {
        if (err.name !== 'AbortError') {
          setError(err.message)
          setLoading(false)
        }
      })

    return () => controller.abort()
  }, [url])

  return { data, loading, error }
}
```

Usage:

```jsx
import { useFetch } from './hooks/useFetch'

export default function UsersList() {
  const {
    data: users,
    loading,
    error,
  } = useFetch('https://jsonplaceholder.typicode.com/users')

  if (loading) return <p>Loading users...</p>
  if (error) return <p>Error: {error}</p>

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

This pattern is a good learning tool, though in larger apps you will often prefer TanStack Query for caching, retries, and stale data management.

## Building useLocalStorage

This hook syncs state with the browser's local storage.

```jsx
import { useState } from 'react'

export function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })

  const setValue = (value) => {
    const valueToStore =
      value instanceof Function ? value(storedValue) : value

    setStoredValue(valueToStore)
    window.localStorage.setItem(key, JSON.stringify(valueToStore))
  }

  return [storedValue, setValue]
}
```

Usage:

```jsx
import { useLocalStorage } from './hooks/useLocalStorage'

export default function SavedTheme() {
  const [theme, setTheme] = useLocalStorage('theme', 'light')

  return (
    <button
      onClick={() => setTheme((current) => (current === 'light' ? 'dark' : 'light'))}
    >
      Current theme: {theme}
    </button>
  )
}
```

Good use cases:

- saved theme
- dismissed banners
- drafts
- simple user preferences

## Building useDebounce

Debouncing waits until a value stops changing for a short time.

```jsx
import { useEffect, useState } from 'react'

export function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}
```

Usage in search:

```jsx
import { useEffect, useState } from 'react'
import { useDebounce } from './hooks/useDebounce'

export default function SearchBox() {
  const [searchTerm, setSearchTerm] = useState('')
  const debouncedSearch = useDebounce(searchTerm, 500)

  useEffect(() => {
    if (!debouncedSearch.trim()) return

    console.log('Fetch results for:', debouncedSearch)
  }, [debouncedSearch])

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  )
}
```

This improves UX and reduces wasted requests.

## Building useWindowSize

This hook listens for browser resize events.

```jsx
import { useEffect, useState } from 'react'

export function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  })

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      })
    }

    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])

  return size
}
```

Usage:

```jsx
import { useWindowSize } from './hooks/useWindowSize'

function ViewportInfo() {
  const { width, height } = useWindowSize()

  return (
    <p>
      Viewport: {width}px × {height}px
    </p>
  )
}
```

Note: in server-rendered apps, guard direct `window` access. In this course's Vite client-side setup, this version is fine.

## Building useToggle

This hook is tiny but useful.

```jsx
import { useCallback, useState } from 'react'

export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue((current) => !current), [])

  return [value, toggle]
}
```

Usage:

```jsx
import { useToggle } from './hooks/useToggle'

export default function ModalExample() {
  const [isOpen, toggleOpen] = useToggle(false)

  return (
    <>
      <button onClick={toggleOpen}>
        {isOpen ? 'Close modal' : 'Open modal'}
      </button>

      {isOpen && <div className="modal">Hello from the modal</div>}
    </>
  )
}
```

## Rules of custom hooks

Custom hooks follow the same rules as React hooks:

1. name must start with `use`
2. call hooks only at the top level
3. do not call them conditionally
4. call them only from React components or other hooks

Bad:

```jsx
if (isOpen) {
  const value = useToggle()
}
```

Good:

```jsx
const [isOpen, toggleOpen] = useToggle(false)
```

Then conditionally render based on the returned value.

## Sharing custom hooks

A common project structure is:

```text
src/
├── hooks/
│   ├── useFetch.js
│   ├── useLocalStorage.js
│   ├── useDebounce.js
│   ├── useWindowSize.js
│   ├── useToggle.js
│   └── index.js
```

Barrel export:

```js
// src/hooks/index.js
export { useFetch } from './useFetch'
export { useLocalStorage } from './useLocalStorage'
export { useDebounce } from './useDebounce'
export { useWindowSize } from './useWindowSize'
export { useToggle } from './useToggle'
```

Importing becomes cleaner:

```jsx
import { useDebounce, useToggle } from '@/hooks'
```

## Testing custom hooks

Custom hooks are easier to test than tangled component logic.

```jsx
import { act, renderHook } from '@testing-library/react'
import { useToggle } from '../hooks/useToggle'

it('should toggle', () => {
  const { result } = renderHook(() => useToggle(false))

  expect(result.current[0]).toBe(false)

  act(() => result.current[1]())

  expect(result.current[0]).toBe(true)
})
```

A typed hook example in TypeScript:

```ts
import { useState } from 'react'

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState<number>(initialValue)

  const increment = () => setCount((current) => current + 1)
  const decrement = () => setCount((current) => current - 1)
  const reset = () => setCount(initialValue)

  return { count, increment, decrement, reset }
}
```

## Complete example: reusable hooks folder

```jsx
// src/hooks/useCounter.js
import { useState } from 'react'

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  const increment = () => setCount((current) => current + 1)
  const decrement = () => setCount((current) => current - 1)
  const reset = () => setCount(initialValue)

  return { count, increment, decrement, reset }
}
```

```jsx
// src/components/CounterCard.jsx
import { useCounter } from '../hooks/useCounter'

export default function CounterCard() {
  const { count, increment, decrement, reset } = useCounter(10)

  return (
    <section>
      <h2>Count: {count}</h2>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>Reset</button>
    </section>
  )
}
```

The component stays focused on rendering. The hook owns the state logic.

## 💻 Exercises

### Level 1 – Build useCounter

- Create `useCounter(initialValue)`
- Return `count`, `increment`, `decrement`, and `reset`
- Use it in a counter card component

### Level 2 – Build useAsync

- Create `useAsync(asyncFn)`
- Return `execute`, `loading`, `error`, and `value`
- Use it to load user data when a button is clicked

### Level 3 – Build useIntersectionObserver

- Create a hook that watches an element entering the viewport
- Return a ref and an `isVisible` boolean
- Use it for lazy loading images or infinite scroll
