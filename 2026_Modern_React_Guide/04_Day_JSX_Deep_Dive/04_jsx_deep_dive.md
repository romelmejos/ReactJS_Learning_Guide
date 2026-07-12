[<< Day 3](../03_Day_Tooling_Vite_npm/03_tooling_vite_npm.md) | [Day 5 >>](../05_Day_Functional_Components/05_functional_components.md)

# Day 4 – JSX Deep Dive

## Table of Contents
- [What Is JSX?](#what-is-jsx)
- [JSX vs HTML Differences](#jsx-vs-html-differences)
- [JSX Expressions](#jsx-expressions)
- [JSX Must Have One Root](#jsx-must-have-one-root)
- [Fragments](#fragments)
- [Rendering Different Types](#rendering-different-types)
- [Conditional Rendering in JSX](#conditional-rendering-in-jsx)
- [Lists in JSX](#lists-in-jsx)
- [JSX Compilation](#jsx-compilation)
- [JSX Gotchas \& Best Practices](#jsx-gotchas--best-practices)
- [💻 Exercises](#-exercises)

## What Is JSX?

JSX stands for **JavaScript XML**. It lets you write UI that looks HTML-like inside JavaScript, but JSX is **not HTML**.

It is syntax sugar over `React.createElement()`.

```jsx
const element = <h1>Hello, JSX</h1>
```

Conceptually compiles to:

```js
const element = React.createElement('h1', null, 'Hello, JSX')
```

JSX makes component code easier to read because structure and logic can live close together.

### A slightly larger example

```jsx
function Welcome() {
  return (
    <section className="hero">
      <h1>Hello, React</h1>
      <p>JSX helps describe UI clearly.</p>
    </section>
  )
}
```

Compiles roughly to:

```js
function Welcome() {
  return React.createElement(
    'section',
    { className: 'hero' },
    React.createElement('h1', null, 'Hello, React'),
    React.createElement('p', null, 'JSX helps describe UI clearly.')
  )
}
```

You almost never write the compiled form manually, but understanding it makes JSX less mysterious.

---

## JSX vs HTML Differences

JSX looks like HTML, but it follows JavaScript rules and React conventions.

### `className` instead of `class`

```jsx
<div className="card">Profile</div>
```

Why? Because `class` is a JavaScript keyword and React uses `className` for the DOM attribute.

### `htmlFor` instead of `for`

```jsx
<label htmlFor="email">Email</label>
<input id="email" />
```

### Self-closing tags are required

```jsx
<img src="/avatar.png" alt="Avatar" />
<br />
<input type="text" />
```

### Event handlers use camelCase

```jsx
<button onClick={handleClick}>Click me</button>
<input onChange={handleChange} />
```

### `style` takes an object

```jsx
<p style={{ color: 'red', fontSize: '16px' }}>
  Styled text
</p>
```

JavaScript object keys are camelCased:

```jsx
<div style={{ backgroundColor: 'black', marginTop: '1rem' }} />
```

---

## JSX Expressions

Anything inside `{}` is a JavaScript **expression**.

```jsx
const name = 'Tina'
const age = 24

function App() {
  return (
    <div>
      <h1>Hello, {name}</h1>
      <p>Age: {age}</p>
      <p>Next year: {age + 1}</p>
    </div>
  )
}
```

You can use:

- strings
- numbers
- variables
- function calls
- array methods
- ternaries

```jsx
function formatName(name) {
  return name.toUpperCase()
}

<h2>{formatName('react')}</h2>
```

### Expressions are allowed, statements are not

This works:

```jsx
<p>{isLoggedIn ? 'Welcome' : 'Please sign in'}</p>
```

This does **not** work:

```jsx
// ❌ invalid inside JSX
<p>{if (isLoggedIn) 'Welcome'}</p>
```

If you need a statement like `if`, use it before the `return`.

```jsx
function StatusMessage({ isLoggedIn }) {
  if (isLoggedIn) {
    return <p>Welcome</p>
  }

  return <p>Please sign in</p>
}
```

---

## JSX Must Have One Root

A component must return a single root element.

This is invalid:

```jsx
function App() {
  return (
    <h1>Title</h1>
    <p>Description</p>
  )
}
```

Wrap the siblings:

```jsx
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Description</p>
    </div>
  )
}
```

Or use a fragment:

```jsx
function App() {
  return (
    <>
      <h1>Title</h1>
      <p>Description</p>
    </>
  )
}
```

---

## Fragments

Fragments let you group elements without adding extra DOM nodes.

### Short syntax

```jsx
<>
  <h1>Dashboard</h1>
  <p>Welcome back.</p>
</>
```

Use this when you just need a wrapper and do **not** need props.

### Explicit syntax

```jsx
<React.Fragment key={user.id}>
  <h3>{user.name}</h3>
  <p>{user.email}</p>
</React.Fragment>
```

Use `<React.Fragment>` when you need a prop like `key`.

Example inside a list:

```jsx
function UserRows({ users }) {
  return users.map((user) => (
    <React.Fragment key={user.id}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </React.Fragment>
  ))
}
```

---

## Rendering Different Types

React can render some JavaScript values directly.

### Strings and numbers

```jsx
<p>{'Hello'}</p>
<p>{42}</p>
```

### `null`, `undefined`, and `false`

These render nothing.

```jsx
<div>
  {false}
  {null}
  {undefined}
</div>
```

This behavior is useful for conditional rendering.

### Arrays

Arrays render each item.

```jsx
const tags = ['React', 'Vite', 'JSX']

function TagList() {
  return <div>{tags.map((tag) => <span key={tag}>{tag} </span>)}</div>
}
```

### Objects

Objects cannot be rendered directly.

```jsx
const user = { name: 'Mila' }

// ❌ Error
<p>{user}</p>
```

Instead, render a property or convert it:

```jsx
<p>{user.name}</p>
<pre>{JSON.stringify(user, null, 2)}</pre>
```

---

## Conditional Rendering in JSX

### Ternary operator

Use when you need one of two outputs.

```jsx
function AuthView({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <Dashboard /> : <Login />}
    </div>
  )
}
```

### Short-circuit `&&`

Use when you only need to render something if a condition is true.

```jsx
function AdminArea({ isAdmin }) {
  return <div>{isAdmin && <AdminPanel />}</div>
}
```

### Beware the `0` gotcha

```jsx
function NotificationCount({ count }) {
  return <div>{count && <p>You have notifications</p>}</div>
}
```

If `count` is `0`, React renders `0`.

Safer version:

```jsx
function NotificationCount({ count }) {
  return <div>{count > 0 && <p>You have notifications</p>}</div>
}
```

### Return `null` to render nothing

```jsx
function WarningBanner({ show }) {
  if (!show) {
    return null
  }

  return <p>Warning: Unsaved changes</p>
}
```

---

## Lists in JSX

Rendering lists is a core React skill.

```jsx
const items = [
  { id: 1, name: 'Notebook' },
  { id: 2, name: 'Pen' },
  { id: 3, name: 'Lamp' },
]

function ItemList() {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
```

### Why `key` matters

`key` helps React identify which list items changed, were added, or were removed.

Good keys:

- database IDs
- stable unique IDs

Avoid using array indexes as keys unless the list is static and never reordered.

Keys are covered in depth later, but start using them correctly now.

---

## JSX Compilation

JSX is not understood by browsers directly. It must be transformed by tooling such as:

- Babel
- esbuild
- SWC

### Example transform

JSX:

```jsx
function Button() {
  return <button className="primary">Save</button>
}
```

Conceptual output:

```js
function Button() {
  return React.createElement(
    'button',
    { className: 'primary' },
    'Save'
  )
}
```

### New JSX transform

Since React 17+, you usually do **not** need this in every file:

```jsx
import React from 'react'
```

Modern toolchains automatically inject the necessary JSX runtime support.

That is why current Vite React projects can contain:

```jsx
function App() {
  return <h1>Hello</h1>
}

export default App
```

without importing `React` manually.

---

## JSX Gotchas & Best Practices

### Adjacent JSX must be wrapped

```jsx
// ❌ invalid
<h1>Hello</h1>
<p>World</p>
```

Wrap with a fragment or container.

### JSX comments

Use comment syntax inside braces:

```jsx
function App() {
  return (
    <div>
      {/* This is a JSX comment */}
      <h1>Hello</h1>
    </div>
  )
}
```

### String entities

You can use the real character:

```jsx
<footer>© 2026 Modern React Guide</footer>
```

Or Unicode:

```jsx
<footer>{'\u00A9'} 2026 Modern React Guide</footer>
```

### Spreading props

```jsx
const props = {
  type: 'button',
  className: 'primary',
}

<Button {...props} />
```

This is useful, but do not overuse it when explicit props would be clearer.

### Keep JSX readable

Good:

```jsx
function ProductCard({ product }) {
  return (
    <article className="card">
      <h2>{product.name}</h2>
      <p>${product.price}</p>
    </article>
  )
}
```

Less readable:

```jsx
function ProductCard({ product }) {
  return <article className="card"><h2>{product.name}</h2><p>${product.price}</p></article>
}
```

Format JSX across multiple lines when structure matters.

---

## 💻 Exercises

### Level 1: Basic

Fix the following broken JSX:

```jsx
function App() {
  return (
    <h1 class="title">Hello</h1>
    <label for="email">Email</label>
    <input id="email">
  )
}
```

Tasks:

1. Make it valid JSX.
2. Add one root wrapper.
3. Replace invalid attributes.
4. Close tags properly.

### Level 2: Intermediate

Build a `ProfileCard` component that renders:

- profile photo
- full name
- role
- short bio
- a Follow button

Requirements:

- use `className`
- use an inline style object at least once
- use a ternary to display `Online` or `Offline`
- keep the component readable with proper indentation

### Level 3: Challenge

Build a dynamic list renderer:

1. Create an array of objects:

```js
const products = [
  { id: 1, name: 'Keyboard', price: 40, inStock: true },
  { id: 2, name: 'Monitor', price: 150, inStock: false },
  { id: 3, name: 'Mouse', price: 25, inStock: true },
]
```

2. Render the list with `.map()`.
3. Show `In stock` only when `inStock` is true.
4. Show a fallback paragraph if the array is empty.
5. Add a second section that renders only products over `$30`.

