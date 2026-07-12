[<< Day 20](../20_Day_useReducer/20_use_reducer.md) | [Day 22 >>](../22_Day_Custom_Hooks/22_custom_hooks.md)

# Day 21 – Context API

## Table of Contents

- [The problem Context solves](#the-problem-context-solves)
- [Creating context](#creating-context)
- [Providing context](#providing-context)
- [Consuming context](#consuming-context)
- [Real-world pattern: Auth Context](#real-world-pattern-auth-context)
- [Context plus useReducer](#context-plus-usereducer)
- [Performance pitfalls](#performance-pitfalls)
- [Splitting contexts](#splitting-contexts)
- [Context vs props vs global state](#context-vs-props-vs-global-state)
- [Multiple contexts](#multiple-contexts)
- [Complete example: theme system](#complete-example-theme-system)
- [💻 Exercises](#-exercises)

## The problem Context solves

Context solves **prop drilling**: passing the same value through many components that do not actually use it.

```jsx
function App() {
  const theme = 'dark'

  return <Layout theme={theme} />
}

function Layout({ theme }) {
  return <Sidebar theme={theme} />
}

function Sidebar({ theme }) {
  return <ThemedButton theme={theme} />
}

function ThemedButton({ theme }) {
  return <button className={`btn-${theme}`}>Click</button>
}
```

`Layout` and `Sidebar` are only forwarding props. Context lets the button read the shared value directly.

Use Context when data is needed by many descendants:

- theme
- authenticated user
- locale
- feature flags

## Creating context

Create a context with `createContext`.

```jsx
import { createContext } from 'react'

export const ThemeContext = createContext('light')
```

The default value is used only when there is no matching provider above the component tree.

## Providing context

Wrap the part of the app that needs access.

```jsx
import { ThemeContext } from './ThemeContext'

export default function Root() {
  return (
    <ThemeContext.Provider value="dark">
      <App />
    </ThemeContext.Provider>
  )
}
```

Any descendant of `App` can now read `"dark"`.

## Consuming context

Use `useContext` inside a function component or custom hook.

```jsx
import { useContext } from 'react'
import { ThemeContext } from './ThemeContext'

function ThemedButton() {
  const theme = useContext(ThemeContext)

  return <button className={`btn-${theme}`}>Click</button>
}
```

This is cleaner than threading `theme` through every layer.

## Real-world pattern: Auth Context

Context shines when several pages need the same app-level state.

```jsx
// src/contexts/AuthContext.jsx
import { createContext, useContext, useMemo, useState } from 'react'

async function apiLogin(credentials) {
  await new Promise((resolve) => setTimeout(resolve, 600))

  return {
    id: 1,
    name: credentials.email.split('@')[0],
    email: credentials.email,
  }
}

const AuthContext = createContext(null)

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null)

  const login = async (credentials) => {
    const authenticatedUser = await apiLogin(credentials)
    setUser(authenticatedUser)
  }

  const logout = () => setUser(null)

  const value = useMemo(
    () => ({ user, login, logout, isAuthenticated: Boolean(user) }),
    [user]
  )

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>
}

export function useAuth() {
  const ctx = useContext(AuthContext)

  if (!ctx) {
    throw new Error('useAuth must be used within AuthProvider')
  }

  return ctx
}
```

Usage:

```jsx
import { useState } from 'react'
import { useAuth } from './contexts/AuthContext'

function LoginPanel() {
  const { user, login, logout, isAuthenticated } = useAuth()
  const [email, setEmail] = useState('ada@example.com')

  const handleLogin = async () => {
    await login({ email, password: 'secret123' })
  }

  return (
    <section>
      <h2>Account</h2>

      {isAuthenticated ? (
        <>
          <p>Welcome, {user.name}</p>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <>
          <input value={email} onChange={(e) => setEmail(e.target.value)} />
          <button onClick={handleLogin}>Login</button>
        </>
      )}
    </section>
  )
}
```

## Context plus useReducer

This is the classic **mini-Redux** pattern:

- `useReducer` manages state transitions
- Context exposes `state` and `dispatch`

```jsx
import {
  createContext,
  useContext,
  useMemo,
  useReducer,
} from 'react'

const ThemeStateContext = createContext(null)
const ThemeDispatchContext = createContext(null)

function themeReducer(state, action) {
  switch (action.type) {
    case 'toggle':
      return {
        ...state,
        mode: state.mode === 'light' ? 'dark' : 'light',
      }
    default:
      throw new Error(`Unknown action: ${action.type}`)
  }
}

export function ThemeProvider({ children }) {
  const [state, dispatch] = useReducer(themeReducer, { mode: 'light' })
  const value = useMemo(() => state, [state])

  return (
    <ThemeStateContext.Provider value={value}>
      <ThemeDispatchContext.Provider value={dispatch}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  )
}

export function useThemeState() {
  const context = useContext(ThemeStateContext)
  if (!context) throw new Error('useThemeState must be used within ThemeProvider')
  return context
}

export function useThemeDispatch() {
  const context = useContext(ThemeDispatchContext)
  if (!context) throw new Error('useThemeDispatch must be used within ThemeProvider')
  return context
}

function ThemeToggle() {
  const { mode } = useThemeState()
  const dispatch = useThemeDispatch()

  return (
    <button onClick={() => dispatch({ type: 'toggle' })}>
      Current theme: {mode}
    </button>
  )
}
```

Why split state and dispatch into separate contexts? `dispatch` never changes, so components that only dispatch actions avoid unnecessary re-renders.

## Performance pitfalls

Context is convenient, but there is an important cost:

1. every consumer re-renders when the context value changes
2. object values create new references unless memoized
3. giant “everything context” objects make the whole app noisier

Bad:

```jsx
function AuthProvider({ children }) {
  const [user, setUser] = useState(null)

  const login = async () => {}
  const logout = () => setUser(null)

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}
```

That object is recreated on every render.

Better:

```jsx
import { useMemo, useState } from 'react'

function AuthProvider({ children }) {
  const [user, setUser] = useState(null)

  const login = async () => {}
  const logout = () => setUser(null)

  const value = useMemo(() => ({ user, login, logout }), [user])

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>
}
```

Memoization is not magic, but it helps avoid needless updates caused by fresh object references.

## Splitting contexts

Do not put unrelated data in one mega-context.

Split by **responsibility** and **update frequency**:

- `ThemeContext` for theme
- `AuthContext` for user session
- `ConfigContext` for app config
- `CartContext` for cart state

Example:

```jsx
<ConfigProvider>
  <AuthProvider>
    <ThemeProvider>
      <App />
    </ThemeProvider>
  </AuthProvider>
</ConfigProvider>
```

If configuration almost never changes but cart contents change constantly, keeping them separate prevents many pointless re-renders.

## Context vs props vs global state

| Tool | Best for | Avoid when |
| --- | --- | --- |
| Props | parent-to-child data flow, local composition | many intermediate components just pass values along |
| Context | shared app-level values used by many descendants | highly dynamic state used everywhere every second |
| Global state library | complex cross-feature state, async caching, devtools needs | the state is small and can stay local or in Context |

Practical guideline:

- start with props
- move to Context when prop drilling appears
- reach for a state library only when complexity really grows

## Multiple contexts

Nesting providers is normal.

```jsx
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>{children}</CartProvider>
      </AuthProvider>
    </ThemeProvider>
  )
}
```

Then use it at the app root:

```jsx
import { createRoot } from 'react-dom/client'
import App from './App'
import { AppProviders } from './providers/AppProviders'

createRoot(document.getElementById('root')).render(
  <AppProviders>
    <App />
  </AppProviders>
)
```

This custom wrapper keeps `main.jsx` tidy.

## Complete example: theme system

```jsx
// src/contexts/ThemeContext.jsx
import { createContext, useContext, useMemo, useState } from 'react'

const ThemeContext = createContext(null)

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light')

  const toggleTheme = () => {
    setTheme((current) => (current === 'light' ? 'dark' : 'light'))
  }

  const value = useMemo(() => ({ theme, toggleTheme }), [theme])

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
}

export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) throw new Error('useTheme must be used within ThemeProvider')
  return context
}
```

```jsx
// src/App.jsx
import { ThemeProvider, useTheme } from './contexts/ThemeContext'

function Toolbar() {
  const { theme, toggleTheme } = useTheme()

  return (
    <header className={`toolbar toolbar-${theme}`}>
      <p>Theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle theme</button>
    </header>
  )
}

function Page() {
  const { theme } = useTheme()

  return (
    <main className={`page page-${theme}`}>
      <h1>Context makes shared UI state easy to reach</h1>
      <p>No prop drilling required.</p>
    </main>
  )
}

export default function App() {
  return (
    <ThemeProvider>
      <Toolbar />
      <Page />
    </ThemeProvider>
  )
}
```

## 💻 Exercises

### Level 1 – Theme toggle with context

- Create `ThemeContext`
- Wrap your app with `ThemeProvider`
- Build a toggle button and a page that changes styles based on the current theme

### Level 2 – Auth context with login/logout

- Create `AuthProvider` with `user`, `login`, and `logout`
- Show different UI for authenticated vs logged-out users
- Add a loading state during login

### Level 3 – Shopping cart with context + useReducer

- Create a reducer with `addItem`, `removeItem`, and `clearCart`
- Expose state and dispatch through Context
- Build a product list, a cart sidebar, and a total price display
