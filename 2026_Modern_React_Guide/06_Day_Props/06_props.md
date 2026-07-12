[<< Day 5](../05_Day_Functional_Components/05_functional_components.md) | [Day 7 >>](../07_Day_useState/07_use_state.md)

# Day 6 – Props

## Table of Contents

- [What are Props?](#what-are-props)
- [Passing Props](#passing-props)
- [Receiving Props](#receiving-props)
- [Default Props](#default-props)
- [The `children` Prop](#the-children-prop)
- [Rest Props and Prop Spreading](#rest-props-and-prop-spreading)
- [Passing Complex Data](#passing-complex-data)
- [Props Must Stay Immutable](#props-must-stay-immutable)
- [PropTypes (Optional)](#proptypes-optional)
- [Composition Over Inheritance](#composition-over-inheritance)
- [Prop Drilling](#prop-drilling)
- [Summary](#summary)
- [💻 Exercises](#-exercises)

## What are Props?

Props are **inputs to a component**. The name comes from **properties**.

If state is a component's internal memory, props are the values a parent sends down to a child. In modern React, props follow two important rules:

- **Read-only**: child components must not change them
- **One-way data flow**: data moves from parent to child

That one-way model is one of the reasons React apps stay predictable. A parent decides what data to pass, and a child decides how to display it.

Think of a component like a function:

```jsx
function greet(name) {
  return `Hello, ${name}`
}
```

React components work similarly:

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>
}
```

## Passing Props

You pass props as attributes in JSX:

```jsx
function App() {
  return <UserCard name="Alice" age={30} isAdmin={true} />
}
```

### JSX prop syntax rules

- **Strings** use quotes: `name="Alice"`
- **Numbers, booleans, objects, arrays, expressions, and variables** use curly braces: `age={30}`

Here is a more realistic example:

```jsx
function App() {
  const user = {
    name: 'Alice',
    age: 30,
    isAdmin: true,
  }

  return (
    <main>
      <UserCard name={user.name} age={user.age} isAdmin={user.isAdmin} />
    </main>
  )
}
```

## Receiving Props

A component receives props through its first parameter.

### Using the `props` object

```jsx
function UserCard(props) {
  return (
    <section>
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
      <p>{props.isAdmin ? 'Admin user' : 'Regular user'}</p>
    </section>
  )
}
```

### Destructuring props

Destructuring is the most common style because it is shorter and clearer:

```jsx
function UserCard({ name, age, isAdmin }) {
  return (
    <section>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>{isAdmin ? 'Admin user' : 'Regular user'}</p>
    </section>
  )
}
```

Use the `props` object when you want the whole thing. Use destructuring when you know the specific fields you need.

## Default Props

In modern React, use **default parameter values**, not `defaultProps`, for function components.

```jsx
function UserCard({ name = 'Anonymous', age = 0, isAdmin = false }) {
  return (
    <section>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>{isAdmin ? 'Admin user' : 'Regular user'}</p>
    </section>
  )
}
```

This helps when a prop is missing:

```jsx
function App() {
  return <UserCard />
}
```

Default values make components more resilient, but they should not hide real data bugs. Use them for sensible fallbacks, not for papering over broken logic.

## The `children` Prop

`children` is a special prop that represents the content nested inside a component's opening and closing tags.

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  )
}
```

Usage:

```jsx
function App() {
  return (
    <Card title="My Card">
      <p>Any content here</p>
      <button>Click me</button>
    </Card>
  )
}
```

Use `children` when you want to build:

- layout wrappers
- panels and cards
- modal shells
- reusable sections

It keeps components flexible because the parent controls the content while the child controls the frame.

## Rest Props and Prop Spreading

Sometimes a component should handle a few custom props and pass the rest to a real DOM element.

```jsx
function Button({ label, variant = 'primary', ...rest }) {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {label}
    </button>
  )
}
```

Usage:

```jsx
function App() {
  return (
    <Button
      label="Save"
      variant="secondary"
      disabled
      onClick={() => console.log('Saved')}
      type="button"
      aria-label="Save changes"
    />
  )
}
```

This pattern is great for reusable design-system components because it supports native button attributes without manually listing every possible prop.

## Passing Complex Data

Props are not limited to strings and numbers.

### Objects

```jsx
function UserProfile({ user }) {
  return (
    <article>
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </article>
  )
}
```

### Arrays

```jsx
function TagList({ tags }) {
  return (
    <ul>
      {tags.map(tag => (
        <li key={tag}>{tag}</li>
      ))}
    </ul>
  )
}
```

### Functions

Functions are commonly passed so a child can notify a parent about something:

```jsx
function UserCard({ name, onDelete }) {
  return <button onClick={onDelete}>Delete {name}</button>
}

function App() {
  const handleDelete = () => {
    console.log('Delete requested')
  }

  return <UserCard name="Alice" onDelete={handleDelete} />
}
```

This is still one-way data flow: the parent sends the function down, and the child calls it.

## Props Must Stay Immutable

Props should never be mutated.

### Bad

```jsx
function UserCard({ user }) {
  user.name = 'Changed' // ❌ never mutate props
  return <h2>{user.name}</h2>
}
```

### Good

```jsx
function UserCard({ user }) {
  return <h2>{user.name}</h2>
}
```

If data needs to change over time, that means you need **state**, which is tomorrow's topic.

Mutating props breaks React's mental model and often causes confusing bugs because the parent still "owns" that value.

## PropTypes (Optional)

If you're writing plain JavaScript, you can add runtime prop validation with `prop-types`.

```jsx
import PropTypes from 'prop-types'

function UserCard({ name, age, isAdmin }) {
  return (
    <section>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>{isAdmin ? 'Admin' : 'User'}</p>
    </section>
  )
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isAdmin: PropTypes.bool,
}
```

Install it with npm:

```bash
npm install prop-types
```

PropTypes are useful for learning and for JS-only projects, but **TypeScript is generally the better long-term option** because it catches issues before runtime.

## Composition Over Inheritance

React favors **composition**, not inheritance.

Instead of building a rigid hierarchy of base classes, you build small components and combine them.

Here is a flexible `Modal` built with composition:

```jsx
function Modal({ title, children, footer }) {
  return (
    <div className="modal-backdrop">
      <div className="modal">
        <header className="modal-header">
          <h2>{title}</h2>
        </header>

        <div className="modal-body">{children}</div>

        {footer && <footer className="modal-footer">{footer}</footer>}
      </div>
    </div>
  )
}
```

Usage:

```jsx
function App() {
  return (
    <Modal
      title="Delete file"
      footer={
        <>
          <button type="button">Cancel</button>
          <button type="button">Delete</button>
        </>
      }
    >
      <p>This action cannot be undone.</p>
    </Modal>
  )
}
```

The shell is reusable, and the content is injected through props. That is the React way.

## Prop Drilling

Sometimes a value must be passed through many layers:

```jsx
function App() {
  return <Layout theme="dark" />
}

function Layout({ theme }) {
  return <Sidebar theme={theme} />
}

function Sidebar({ theme }) {
  return <Nav theme={theme} />
}

function Nav({ theme }) {
  return <p>Current theme: {theme}</p>
}
```

This is called **prop drilling**. It is not always bad, but it becomes annoying when intermediate components do not actually care about the prop.

Later, on **Day 21**, you'll learn how **Context** can solve this problem for app-wide values like theme, auth, or locale.

## Summary

- Props are read-only inputs passed from parent to child
- React uses one-way data flow
- Destructuring makes prop access cleaner
- Use default parameter values for fallbacks
- `children` makes components flexible
- Rest props help reusable components pass DOM attributes through
- Props can contain objects, arrays, and functions
- Never mutate props
- React prefers composition over inheritance
- Prop drilling is a real scaling problem, and Context helps later

## 💻 Exercises

### Level 1 — Basic

Create a `ProfileCard` component that accepts:

- `name`
- `bio`
- `avatar`

Requirements:

- Render the avatar image, name, and bio
- Add a default bio fallback like `"No bio yet"`
- Render two `ProfileCard` instances from `App`

### Level 2 — Intermediate

Build a reusable `Button` component with:

- `variant` (`primary`, `secondary`, `danger`)
- `size` (`sm`, `md`, `lg`)
- `disabled`
- `children`

Requirements:

- Use default parameter values
- Use `...rest` so `onClick`, `type`, and `aria-*` props still work
- Render a small toolbar with several button combinations

### Level 3 — Challenge

Build a `DataTable` component that accepts:

- `columns`
- `rows`

Suggested shape:

```js
const columns = [
  { key: 'name', label: 'Name' },
  { key: 'role', label: 'Role' },
]

const rows = [
  { id: 1, name: 'Alice', role: 'Admin' },
  { id: 2, name: 'Bob', role: 'Editor' },
]
```

Requirements:

- Render table headers from `columns`
- Render each row dynamically
- Use stable keys
- Add support for an empty state message when `rows.length === 0`

