[<< Day 27](../27_Day_Global_State_Management/27_global_state_management.md) | [Day 29 >>](../29_Day_Capstone_Project/29_capstone_project.md)

# Day 28 – Testing React Apps

## Table of Contents
- [Why Test React Apps](#why-test-react-apps)
- [The Testing Pyramid](#the-testing-pyramid)
- [Vitest Setup](#vitest-setup)
- [Testing Library Philosophy](#testing-library-philosophy)
- [Your First Test](#your-first-test)
- [Querying Elements](#querying-elements)
- [User Interactions with `userEvent`](#user-interactions-with-userevent)
- [Testing Async Behavior](#testing-async-behavior)
- [Mocking API Calls](#mocking-api-calls)
- [Testing Custom Hooks](#testing-custom-hooks)
- [Running Tests](#running-tests)
- [Coverage Goals](#coverage-goals)
- [💻 Exercises](#-exercises)

## Why Test React Apps

Testing is not about chasing a coverage number. It is about confidence.

Good tests help you:

- refactor without fear
- catch regressions early
- document intended behavior
- verify important user flows
- reduce “works on my machine” mistakes

In React, tests are especially useful because UI behavior depends on many moving parts:

- props
- state
- async data
- forms
- routing
- user interaction

If you can describe expected behavior, you can usually test it.

---

## The Testing Pyramid

The classic testing pyramid looks like this:

1. **Unit tests** – test small pieces in isolation
2. **Integration tests** – test multiple pieces working together
3. **E2E tests** – test the whole app in a real browser

### In modern React, integration tests give the most value

That is the core Testing Library philosophy:

> Test the app the way a user experiences it.

Examples:

- render a form
- type into fields
- click submit
- assert validation messages and success state

This is often more valuable than testing tiny implementation details.

### A healthy balance

- **Unit tests** for utility functions and critical logic
- **Integration tests** for components, forms, and page flows
- **E2E tests** for checkout, login, or other high-value journeys

---

## Vitest Setup

Vitest is the natural test runner for Vite projects.

Install the core testing tools:

```bash
npm install -D vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom
```

Add test configuration in `vite.config.js`:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.js',
  },
})
```

Create `src/test/setup.js`:

```js
import '@testing-library/jest-dom'
```

### Suggested script setup

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

### Why `jsdom`?

React components interact with the DOM, so tests need a browser-like environment. `jsdom` simulates enough of the browser for component testing.

---

## Testing Library Philosophy

React Testing Library (RTL) encourages behavior-first testing.

### Prefer testing what the user sees

Good:

- visible text
- buttons by accessible name
- inputs by label
- alerts and status messages by role

Avoid:

- checking internal component state
- testing implementation-only details
- querying by CSS classes unless absolutely necessary

### Example

Instead of this:

```js
expect(container.querySelector('.btn-primary')).toBeTruthy()
```

prefer this:

```js
expect(screen.getByRole('button', { name: /save/i })).toBeInTheDocument()
```

The second test is more resilient and closer to actual user behavior.

---

## Your First Test

Suppose you have a component:

```jsx
// src/components/Greeting.jsx
export default function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>
}
```

Test it like this:

```jsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import Greeting from '../Greeting'

describe('Greeting', () => {
  it('renders the user name', () => {
    render(<Greeting name="Alice" />)

    expect(screen.getByText(/hello, alice/i)).toBeInTheDocument()
  })
})
```

### What is happening here

- `render()` mounts the component into the test DOM
- `screen` gives you user-facing queries
- `expect(...).toBeInTheDocument()` comes from `jest-dom`

---

## Querying Elements

Testing Library provides query families with different behavior.

### `getBy*`

Use when the element **must exist immediately**.

```jsx
screen.getByRole('button', { name: /submit/i })
```

If it is missing, the test fails right away.

### `queryBy*`

Use when the element **may be absent**.

```jsx
expect(screen.queryByText(/error/i)).not.toBeInTheDocument()
```

This returns `null` instead of throwing.

### `findBy*`

Use when the element appears **asynchronously**.

```jsx
const user = await screen.findByText('Alice Smith')
expect(user).toBeInTheDocument()
```

### Query priority order

Prefer queries in this order:

1. `ByRole`
2. `ByLabelText`
3. `ByPlaceholderText`
4. `ByText`
5. `ByDisplayValue`
6. `ByTestId` as a last resort

### Examples

```jsx
screen.getByRole('button', { name: /log in/i })
screen.getByLabelText(/email/i)
screen.getByText(/welcome back/i)
screen.getByTestId('spinner')
```

---

## User Interactions with `userEvent`

Use `userEvent` for realistic interactions like clicking, typing, and tabbing.

```jsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Counter from './Counter'

it('increments counter on click', async () => {
  const user = userEvent.setup()

  render(<Counter />)

  await user.click(screen.getByRole('button', { name: /increment/i }))

  expect(screen.getByText('1')).toBeInTheDocument()
})
```

### Why not fireEvent for everything?

`fireEvent` is lower-level. `userEvent` better simulates what a real user does:

- keyboard typing
- focus changes
- click behavior

### Form interaction example

```jsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import LoginForm from './LoginForm'

it('submits email and password', async () => {
  const user = userEvent.setup()
  const handleSubmit = vi.fn()

  render(<LoginForm onSubmit={handleSubmit} />)

  await user.type(screen.getByLabelText(/email/i), 'alice@example.com')
  await user.type(screen.getByLabelText(/password/i), 'secret123')
  await user.click(screen.getByRole('button', { name: /log in/i }))

  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'alice@example.com',
    password: 'secret123',
  })
})
```

---

## Testing Async Behavior

Async UI is everywhere:

- loading spinners
- fetched data
- delayed validation
- mutation success messages

### Example component

```jsx
import { useEffect, useState } from 'react'

export default function UserList() {
  const [users, setUsers] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function loadUsers() {
      const response = await fetch('/api/users')
      const data = await response.json()
      setUsers(data)
      setLoading(false)
    }

    loadUsers()
  }, [])

  if (loading) {
    return <p>Loading...</p>
  }

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Test

```jsx
import { render, screen } from '@testing-library/react'
import UserList from './UserList'

it('loads and displays users', async () => {
  render(<UserList />)

  expect(screen.getByText(/loading/i)).toBeInTheDocument()

  const user = await screen.findByText('Alice Smith')

  expect(user).toBeInTheDocument()
})
```

### When to use `waitFor`

Use `waitFor` when you need to wait for a condition rather than a single element query.

```jsx
import { waitFor } from '@testing-library/react'

await waitFor(() => {
  expect(mockFn).toHaveBeenCalledTimes(1)
})
```

---

## Mocking API Calls

There are two common levels of mocking:

1. **module mocking** with `vi.mock`
2. **network mocking** with MSW

### `vi.mock`

Use it when you want to replace a module dependency.

```jsx
import { render, screen } from '@testing-library/react'
import { vi } from 'vitest'
import UserProfile from './UserProfile'

vi.mock('../api/users', () => ({
  getUserById: vi.fn().mockResolvedValue({
    id: 1,
    name: 'Alice',
  }),
}))
```

### MSW (Mock Service Worker)

For realistic API tests, MSW is the gold standard.

Install it:

```bash
npm install -D msw
```

Create handlers:

```js
// src/mocks/handlers.js
import { http, HttpResponse } from 'msw'

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'Alice Smith' },
      { id: 2, name: 'Sam Lee' },
    ])
  }),
]
```

Create the test server:

```js
// src/mocks/server.js
import { setupServer } from 'msw/node'
import { handlers } from './handlers'

export const server = setupServer(...handlers)
```

Wire it into test setup:

```js
// src/test/setup.js
import '@testing-library/jest-dom'
import { afterAll, afterEach, beforeAll } from 'vitest'
import { server } from '../mocks/server'

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

Now test the component:

```jsx
import { render, screen } from '@testing-library/react'
import UserList from './UserList'

it('renders users from the API', async () => {
  render(<UserList />)

  expect(await screen.findByText('Alice Smith')).toBeInTheDocument()
  expect(await screen.findByText('Sam Lee')).toBeInTheDocument()
})
```

### Overriding a handler in one test

```jsx
import { http, HttpResponse } from 'msw'
import { server } from '../mocks/server'

it('shows an error message when the API fails', async () => {
  server.use(
    http.get('/api/users', () => {
      return new HttpResponse(null, { status: 500 })
    })
  )

  render(<UserList />)

  expect(await screen.findByText(/failed to load users/i)).toBeInTheDocument()
})
```

---

## Testing Custom Hooks

Sometimes the most important logic lives in hooks.

Use `renderHook` to test them directly.

```jsx
import { renderHook, act } from '@testing-library/react'
import { useState } from 'react'

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  const increment = () => setCount(value => value + 1)
  const decrement = () => setCount(value => value - 1)
  const reset = () => setCount(initialValue)

  return { count, increment, decrement, reset }
}

it('useCounter increments', () => {
  const { result } = renderHook(() => useCounter(0))

  act(() => {
    result.current.increment()
  })

  expect(result.current.count).toBe(1)
})
```

### Hook with async behavior

```jsx
import { renderHook, waitFor } from '@testing-library/react'

function useUserName() {
  const [name, setName] = useState('')

  useEffect(() => {
    fetch('/api/me')
      .then(response => response.json())
      .then(data => setName(data.name))
  }, [])

  return name
}

it('loads the user name', async () => {
  const { result } = renderHook(() => useUserName())

  await waitFor(() => {
    expect(result.current).toBe('Alice Smith')
  })
})
```

If a hook depends on context providers, create a wrapper and pass it to `renderHook`.

---

## Running Tests

Useful commands:

```bash
npm test
npm run coverage
```

### Common workflow

- run `npm test` in watch mode while building features
- use `npm run coverage` before opening a PR
- keep flaky tests at zero

### Naming convention

Common patterns:

- `Component.test.jsx`
- `hook.test.js`
- `feature.spec.ts`

Be consistent across the project.

---

## Coverage Goals

Coverage is a tool, not the goal itself.

### Good priorities

Focus on:

- business logic
- validation rules
- condition-heavy UI
- critical flows like login, checkout, saving data

### Bad goal

Trying to hit 100% with trivial assertions like:

- “component renders”
- “line 8 was executed”

### Better goal

Ask:

- does this test protect a meaningful behavior?
- would it catch a bug I care about?
- is the component easy to refactor with these tests in place?

---

## 💻 Exercises

### Level 1 – Test a Button Component

Create tests for a reusable `Button` component.

Your tests should cover:

1. rendering button text
2. disabled state
3. click handler firing
4. variant prop such as `primary` or `secondary`

**Stretch:** Assert the accessible name and disabled behavior with `toBeDisabled()`.

### Level 2 – Test a Form with Validation

Build or reuse a login form with:

- email field
- password field
- validation messages

Write tests that verify:

1. empty form shows validation errors
2. invalid email is rejected
3. valid input submits successfully
4. submit button can be clicked only when appropriate, if your UI supports that

**Stretch:** Use `userEvent.tab()` to test keyboard navigation and focus order.

### Level 3 – Test a Data-Fetching Component with MSW

Create a component that fetches `/api/users`.

Write tests for:

1. loading state
2. successful response
3. error response
4. empty state

Use MSW handlers for each scenario.  
Bonus: test a retry button that refetches after failure.
