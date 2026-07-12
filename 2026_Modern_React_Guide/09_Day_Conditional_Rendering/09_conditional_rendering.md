[<< Day 8](../08_Day_Lists_and_Keys/08_lists_and_keys.md) | [Day 10 >>](../10_Day_useEffect/10_use_effect.md)

# Day 9 – Conditional Rendering

## Table of Contents

- [Introduction](#introduction)
- [`if` / `else` Outside JSX](#if--else-outside-jsx)
- [Ternary Operator Inside JSX](#ternary-operator-inside-jsx)
- [Short-circuit AND (`&&`)](#short-circuit-and-)
- [Short-circuit OR (`||`)](#short-circuit-or-)
- [Nullish Coalescing (`??`)](#nullish-coalescing-)
- [Rendering `null`](#rendering-null)
- [Conditional CSS Classes](#conditional-css-classes)
- [Switch-like Rendering](#switch-like-rendering)
- [Real-world Patterns](#real-world-patterns)
- [Summary](#summary)
- [💻 Exercises](#-exercises)

## Introduction

React does not have a special template language for conditions. You use regular JavaScript to decide what should appear in the UI.

Run the lesson examples in the shared project:

```bash
cd boilerplate
npm install
npm run dev
```

Conditional rendering means:

- show something
- hide something
- swap one UI branch for another

Most UI is conditional:

- logged in vs logged out
- loading vs success vs error
- admin vs regular user
- empty list vs populated list

```js
const statuses = ['idle', 'loading', 'success', 'error']
```

## `if` / `else` Outside JSX

Use `if` when the branches are large or early returns make the component easier to read.

```jsx
function Alert({ type, message }) {
  if (!message) return null
  if (type === 'error') return <div className="error">{message}</div>
  return <div className="info">{message}</div>
}
```

Another example:

```jsx
function AuthPanel({ isLoggedIn }) {
  if (isLoggedIn) {
    return <Dashboard />
  }

  return <Login />
}
```

This style works well when:

- there are multiple exits
- the branches return very different trees
- readability matters more than brevity

## Ternary Operator Inside JSX

Use a ternary when you need an inline "if/else" inside markup.

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

Example:

```jsx
function Header({ isLoggedIn, username }) {
  return (
    <header>
      <h1>{isLoggedIn ? `Welcome back, ${username}` : 'Please sign in'}</h1>
    </header>
  )
}
```

Ternaries are best when both branches are short. If they become deeply nested, move the logic outside JSX.

## Short-circuit AND (`&&`)

Use `&&` when you want to show something or show nothing.

```jsx
{hasError && <ErrorMessage />}
```

Example:

```jsx
function CartSummary({ itemCount }) {
  return (
    <div>
      <h2>Your Cart</h2>
      {itemCount > 0 && <p>You have {itemCount} items.</p>}
    </div>
  )
}
```

### The zero gotcha

This can surprise beginners:

```jsx
{count && <List />}
```

If `count` is `0`, React renders `0`.

Safer versions:

```jsx
{count > 0 && <List />}
{!!count && <List />}
```

## Short-circuit OR (`||`)

Use `||` for a simple fallback when any falsy value should trigger it.

```jsx
{username || 'Guest'}
```

Example:

```jsx
function Welcome({ username }) {
  return <p>Hello, {username || 'Guest'}</p>
}
```

Be careful: `''`, `0`, and `false` are all falsy, so `||` treats them as missing.

## Nullish Coalescing (`??`)

Use `??` when you only want the fallback for `null` or `undefined`.

```jsx
{profile.bio ?? 'No bio provided'}
```

Example:

```jsx
function Bio({ profile }) {
  return <p>{profile.bio ?? 'No bio provided'}</p>
}
```

This is often better than `||` when empty strings or zero are valid values.

## Rendering `null`

A component can return `null` to render nothing.

```jsx
function Banner({ message }) {
  if (!message) return null

  return <div className="banner">{message}</div>
}
```

Important detail: the component still exists in React's tree. It can still run hooks and lifecycle-like behavior. It just produces no DOM nodes for that render.

## Conditional CSS Classes

Sometimes the markup stays the same, but styling changes.

```jsx
<button className={`btn ${isActive ? 'btn-active' : 'btn-inactive'}`}>
  Save
</button>
```

You can also use a tiny utility helper:

```jsx
const cn = (...classes) => classes.filter(Boolean).join(' ')
```

```jsx
function SaveButton({ isActive, isDisabled }) {
  return (
    <button
      className={cn(
        'btn',
        isActive && 'btn-active',
        isDisabled && 'btn-disabled'
      )}
      disabled={isDisabled}
    >
      Save
    </button>
  )
}
```

This pattern keeps class logic readable as styling rules grow.

## Switch-like Rendering

When many states exist, a lookup object or `switch` can be cleaner than nested ternaries.

```jsx
function StatusView({ status }) {
  const statusComponents = {
    loading: <Spinner />,
    error: <ErrorPage />,
    success: <Content />,
  }

  return statusComponents[status] ?? <NotFound />
}
```

Equivalent `switch` version:

```jsx
function StatusView({ status }) {
  switch (status) {
    case 'loading':
      return <Spinner />
    case 'error':
      return <ErrorPage />
    case 'success':
      return <Content />
    default:
      return <NotFound />
  }
}
```

## Real-world Patterns

### Loading / Error / Success

```jsx
function UserPanel({ status, user, error }) {
  if (status === 'loading') {
    return <p>Loading user...</p>
  }

  if (status === 'error') {
    return <p>Something went wrong: {error.message}</p>
  }

  if (status === 'success') {
    return <h2>{user.name}</h2>
  }

  return null
}
```

### Feature flags

```jsx
function Sidebar({ flags }) {
  return (
    <nav>
      <a href="/dashboard">Dashboard</a>
      {flags.newReports && <a href="/reports">Reports</a>}
    </nav>
  )
}
```

### Role-based rendering

```jsx
function AdminTools({ role }) {
  return (
    <div>
      <p>Common settings</p>
      {role === 'admin' ? <button>Delete User</button> : null}
    </div>
  )
}
```

These patterns appear everywhere in production React apps.

## Summary

- Conditional rendering is just JavaScript
- Use `if` for larger branches and early returns
- Use ternaries for short inline alternatives
- Use `&&` to show or hide UI
- Use `||` for broad falsy fallbacks
- Use `??` for null/undefined-only fallbacks
- Return `null` when nothing should render
- Conditional class names are often cleaner than conditional markup

## 💻 Exercises

### Level 1 — Basic

Create a component that renders different messages based on a `status` prop.

Statuses:

- `success`
- `warning`
- `error`

### Level 2 — Intermediate

Build a data display component with:

- loading state
- error state
- success state
- empty state

Requirements:

- use clear conditional branches
- avoid nested ternaries if they hurt readability

### Level 3 — Challenge

Build a permissions-based navigation menu.

Requirements:

- all users see common links
- editors see editor links
- admins see admin links
- premium users see premium links
- hide empty sections cleanly
