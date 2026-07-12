[<< Day 1](../01_Day_JavaScript_Refresher/01_javascript_refresher.md) | [Day 3 >>](../03_Day_Tooling_Vite_npm/03_tooling_vite_npm.md)

# Day 2 – Introduction to React

## Table of Contents
- [What Is React?](#what-is-react)
- [Why React?](#why-react)
- [React's Rendering Model](#reacts-rendering-model)
- [React 19 Highlights](#react-19-highlights)
- [React Without Build Tools](#react-without-build-tools)
- [React's Place in the Ecosystem](#reacts-place-in-the-ecosystem)
- [💻 Exercises](#-exercises)

## What Is React?

React is a **JavaScript library** for building user interfaces. It is maintained by **Meta** and used across the web to build interactive, component-driven apps.

React is not a full framework by itself. It focuses on the **view layer**:

- rendering UI
- updating UI when data changes
- composing UI from reusable components

### Declarative vs imperative programming

Imperative code tells the computer **how** to update the DOM step by step.

```js
const button = document.getElementById('toggle-btn')
const status = document.getElementById('status')

button.addEventListener('click', () => {
  if (status.textContent === 'Offline') {
    status.textContent = 'Online'
    status.style.color = 'green'
  } else {
    status.textContent = 'Offline'
    status.style.color = 'gray'
  }
})
```

React code describes **what** the UI should look like for a given state.

```jsx
import { useState } from 'react'

function StatusToggle() {
  const [online, setOnline] = useState(false)

  return (
    <div>
      <p style={{ color: online ? 'green' : 'gray' }}>
        {online ? 'Online' : 'Offline'}
      </p>
      <button onClick={() => setOnline((prev) => !prev)}>
        Toggle status
      </button>
    </div>
  )
}
```

You update state, and React updates the DOM.

---

## Why React?

### 1. Component-based architecture

A React app is made of components—small, reusable pieces of UI.

```jsx
function Header() {
  return <header>My App</header>
}

function Sidebar() {
  return <aside>Menu</aside>
}

function App() {
  return (
    <>
      <Header />
      <Sidebar />
    </>
  )
}
```

This makes apps easier to scale and maintain.

### 2. Declarative UI

You describe the UI for each state instead of manually synchronizing the DOM.

```jsx
function Message({ isLoggedIn }) {
  return <h2>{isLoggedIn ? 'Welcome back!' : 'Please sign in'}</h2>
}
```

### 3. Virtual DOM and efficient reconciliation

React computes changes in memory first, then updates the real DOM efficiently.

### 4. Large ecosystem

React has mature tooling and libraries:

- Vite
- Next.js
- React Router
- TanStack Query
- React Hook Form
- testing tools and UI libraries

### 5. Server Components in React 19

React now supports rendering some components on the server, reducing client-side JavaScript and improving performance for many app architectures.

---

## React's Rendering Model

### The Virtual DOM concept

The Virtual DOM is a lightweight JavaScript representation of the UI. When your component returns JSX, React turns it into an internal tree of elements.

```jsx
function App() {
  return <h1>Hello</h1>
}
```

Conceptually becomes something like:

```js
React.createElement('h1', null, 'Hello')
```

React compares the new virtual tree with the previous one and decides what to update in the real DOM.

### Reconciliation and Fiber

React's reconciliation engine is powered by **Fiber**. Fiber breaks rendering work into units so React can:

- prioritize updates
- pause and resume work
- support concurrent rendering features
- improve responsiveness during expensive UI updates

You do not manage Fiber directly, but React 19 features like Suspense and transitions build on it.

### When React re-renders a component

Common reasons:

1. its **state** changes
2. its **props** change
3. its parent re-renders
4. a context value it uses changes

Re-rendering means React calls the component function again. It does **not** necessarily mean the DOM is fully rebuilt.

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  console.log('Counter rendered')

  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

Every button click triggers another render of `Counter`.

---

## React 19 Highlights

React 19 continues React's shift toward better async workflows, improved forms, and server/client coordination.

### Actions with `useActionState`

`useActionState` helps manage async form or action workflows by linking an action function with state updates.

```jsx
import { useActionState } from 'react'

async function submitName(previousState, formData) {
  const name = formData.get('name')

  if (!name) {
    return { success: false, message: 'Name is required' }
  }

  await new Promise((resolve) => setTimeout(resolve, 800))

  return { success: true, message: `Saved ${name}` }
}

function NameForm() {
  const [state, formAction, pending] = useActionState(submitName, {
    success: false,
    message: '',
  })

  return (
    <form action={formAction}>
      <input name="name" placeholder="Enter your name" />
      <button disabled={pending}>
        {pending ? 'Saving...' : 'Save'}
      </button>
      <p>{state.message}</p>
    </form>
  )
}
```

### `useFormStatus`

`useFormStatus` is useful inside forms to detect pending submission state.

```jsx
import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending } = useFormStatus()

  return <button disabled={pending}>{pending ? 'Submitting...' : 'Submit'}</button>
}
```

### Async transitions

Transitions help mark updates as non-urgent so the UI stays responsive.

```jsx
import { startTransition, useState } from 'react'

function SearchBox() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])

  async function handleChange(event) {
    const nextQuery = event.target.value
    setQuery(nextQuery)

    const data = await fetch(`/api/search?q=${nextQuery}`).then((res) => res.json())

    startTransition(() => {
      setResults(data)
    })
  }

  return <input value={query} onChange={handleChange} />
}
```

### The new `use()` hook

`use()` lets components read promises or context in supported environments, especially around Suspense and server components.

```jsx
import { Suspense, use } from 'react'

const userPromise = fetch('/api/user').then((res) => res.json())

function UserName() {
  const user = use(userPromise)
  return <h2>{user.name}</h2>
}

function App() {
  return (
    <Suspense fallback={<p>Loading user...</p>}>
      <UserName />
    </Suspense>
  )
}
```

### Improved Suspense and streaming

Suspense lets you show fallback UI while data or code is loading.

```jsx
<Suspense fallback={<Spinner />}>
  <Profile />
</Suspense>
```

Streaming rendering sends parts of the UI to the browser as they are ready, which improves perceived performance.

### React Server Components overview

Server Components:

- render on the server
- can read server-side resources directly
- reduce the amount of JS sent to the client
- work especially well in frameworks like Next.js

They are not a replacement for all client components. Instead, they let you choose the right execution environment for each part of the UI.

### `ref` as a prop

React 19 makes `ref` usage more ergonomic, reducing the need for `forwardRef` in many cases.

Conceptually:

```jsx
function TextInput({ ref, ...props }) {
  return <input ref={ref} {...props} />
}
```

The exact experience depends on your tooling and framework integration, but the direction is clear: simpler ref plumbing.

---

## React Without Build Tools

Before Vite, JSX transforms, and bundlers, React can still run directly in the browser. This is useful for understanding the raw foundation.

### Minimal Hello World

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>React 19 CDN Demo</title>
  <script crossorigin src="https://unpkg.com/react@19/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@19/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    function App() {
      return <h1>Hello, React 19!</h1>
    }

    const root = ReactDOM.createRoot(document.getElementById('root'))
    root.render(<App />)
  </script>
</body>
</html>
```

### Simple counter example

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>React Counter</title>
  <script crossorigin src="https://unpkg.com/react@19/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@19/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    const { useState } = React

    function Counter() {
      const [count, setCount] = useState(0)

      return (
        <div>
          <h1>Count: {count}</h1>
          <button onClick={() => setCount(count + 1)}>Increment</button>
          <button onClick={() => setCount(count - 1)}>Decrement</button>
          <button onClick={() => setCount(0)}>Reset</button>
        </div>
      )
    }

    const root = ReactDOM.createRoot(document.getElementById('root'))
    root.render(<Counter />)
  </script>
</body>
</html>
```

### Why this setup is educational

It reveals that React fundamentally needs:

- the React library
- the DOM renderer
- a root DOM node
- component functions
- JSX transformed into JavaScript

In production apps, Vite handles the JSX transform and module system more efficiently than Babel standalone in the browser.

---

## React's Place in the Ecosystem

React is flexible, so you can use it in several ways.

### Plain React + Vite

Use when you want:

- a fast client-rendered SPA
- maximum control over tooling
- a good learning environment
- a lightweight setup

Great for this course.

### Next.js

Use when you want:

- file-based routing
- server rendering out of the box
- server components
- API routes and full-stack patterns
- optimized production defaults

### Remix

Use when you want:

- strong web-platform alignment
- nested routing
- form-first data workflows
- server-centric UX patterns

### React Native

Use when you want to build native mobile apps with React concepts.

React Native uses components and hooks, but renders native mobile UI instead of browser DOM elements.

---

## 💻 Exercises

### Level 1: Basic

Answer these in your own words:

1. Why is React called a library instead of a full framework?
2. What is the difference between declarative and imperative UI programming?
3. What are three reasons developers choose React?
4. What usually causes a component to re-render?
5. What problem does the Virtual DOM help solve?

### Level 2: Intermediate

Create the CDN example locally:

1. Make an `index.html` file.
2. Paste the React CDN setup from this lesson.
3. Change the `App` component so it renders:
   - a heading
   - a paragraph
   - a button
4. Add a click handler that shows `alert('React is working!')`.
5. Add a counter with `useState`.

### Level 3: Challenge

Build a small **Profile Card** using CDN React:

Requirements:

- render a card with name, role, bio, and avatar
- add a Follow button
- clicking the button toggles between `Follow` and `Following`
- show a status line such as `You are following this user`
- use at least one child component, such as `ProfileHeader` or `ProfileActions`

