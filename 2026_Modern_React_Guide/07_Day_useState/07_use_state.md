[<< Day 6](../06_Day_Props/06_props.md) | [Day 8 >>](../08_Day_Lists_and_Keys/08_lists_and_keys.md)

# Day 7 – useState

## Table of Contents

- [What is State?](#what-is-state)
- [The `useState` Hook](#the-usestate-hook)
- [Rules of Hooks](#rules-of-hooks)
- [State Updates are Asynchronous](#state-updates-are-asynchronous)
- [State with Different Types](#state-with-different-types)
- [Lazy Initialization](#lazy-initialization)
- [Lifting State Up](#lifting-state-up)
- [State vs Derived Values](#state-vs-derived-values)
- [Multiple State Variables](#multiple-state-variables)
- [When to Use State](#when-to-use-state)
- [Summary](#summary)
- [💻 Exercises](#-exercises)

## What is State?

State is data that can **change over time** and cause React to **re-render** a component.

You can experiment with today's examples inside the shared boilerplate:

```bash
cd boilerplate
npm install
npm run dev
```

Compare three kinds of values:

- **Props**: external, read-only inputs
- **State**: internal, changing data
- **Regular variables**: reset on every render and do not trigger UI updates

```js
const valueTypes = {
  props: 'external input',
  state: 'internal reactive data',
  variable: 'plain JavaScript value',
}
```

```jsx
function Counter() {
  let count = 0

  return (
    <button onClick={() => count++}>
      Count: {count}
    </button>
  )
}
```

The code above does not work as expected because changing a regular variable does not tell React to render again.

State does.

## The `useState` Hook

`useState` is the hook that gives a component memory.

```jsx
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  )
}
```

### What does `useState(0)` return?

It returns an array with two items:

1. the current value: `count`
2. the updater function: `setCount`

You almost always destructure them like this:

```jsx
const [value, setValue] = useState(initialValue)
```

## Rules of Hooks

Hooks are special React functions, and they follow special rules:

1. **Only call hooks at the top level**
2. **Only call hooks inside React function components or custom hooks**

### Good

```jsx
function Example() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### Bad

```jsx
function Example() {
  if (Math.random() > 0.5) {
    const [count, setCount] = useState(0) // ❌ never inside conditions
  }

  return <div>Hello</div>
}
```

Why? React tracks hooks by call order. If the order changes between renders, React no longer knows which state belongs to which hook call.

## State Updates are Asynchronous

Calling a setter schedules an update. It does not instantly change the variable in the current render.

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  const handleClick = () => {
    setCount(count + 1)
    console.log(count) // logs the old value
  }

  return <button onClick={handleClick}>Count: {count}</button>
}
```

When the next render happens, React provides the new value.

### Use the functional updater when next state depends on previous state

```jsx
setCount(prev => prev + 1) // safe
setCount(count + 1) // can become stale in closures
```

This matters when:

- multiple updates happen in one event
- updates are queued
- callbacks capture older values

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  const addThree = () => {
    setCount(prev => prev + 1)
    setCount(prev => prev + 1)
    setCount(prev => prev + 1)
  }

  return <button onClick={addThree}>Count: {count}</button>
}
```

## State with Different Types

### Boolean state

```jsx
function Menu() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <button onClick={() => setIsOpen(prev => !prev)}>
      {isOpen ? 'Close menu' : 'Open menu'}
    </button>
  )
}
```

### String state

```jsx
function NamePreview() {
  const [name, setName] = useState('')

  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} />
      <p>Preview: {name}</p>
    </div>
  )
}
```

### Number state

```jsx
function QuantityPicker() {
  const [quantity, setQuantity] = useState(1)

  return (
    <div>
      <button onClick={() => setQuantity(prev => prev - 1)}>-</button>
      <span>{quantity}</span>
      <button onClick={() => setQuantity(prev => prev + 1)}>+</button>
    </div>
  )
}
```

### Object state

Never replace just one field without copying the rest.

```jsx
function ProfileEditor() {
  const [profile, setProfile] = useState({
    name: '',
    email: '',
  })

  const updateName = event => {
    const newName = event.target.value

    setProfile(prev => ({
      ...prev,
      name: newName,
    }))
  }

  return <input value={profile.name} onChange={updateName} />
}
```

### Array state

Always create a new array.

```jsx
function TodoList() {
  const [items, setItems] = useState(['Read', 'Build', 'Ship'])

  const addItem = () => {
    setItems(prev => [...prev, 'Review'])
  }

  const removeItem = itemToRemove => {
    setItems(prev => prev.filter(item => item !== itemToRemove))
  }

  return (
    <div>
      <button onClick={addItem}>Add</button>
      <ul>
        {items.map(item => (
          <li key={item}>
            {item}
            <button onClick={() => removeItem(item)}>Remove</button>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## Lazy Initialization

If the initial value is expensive to compute, pass a function:

```jsx
import { useState } from 'react'

function expensiveComputation() {
  console.log('running expensive setup')
  return Array.from({ length: 1000 }, (_, i) => i)
}

function Example() {
  const [numbers] = useState(() => expensiveComputation())

  return <p>Total items: {numbers.length}</p>
}
```

React calls that initializer function only on the first render, not every render.

## Lifting State Up

If two sibling components need the same data, move that state to their nearest common parent.

### Shared color picker example

```jsx
import { useState } from 'react'

function ColorInput({ color, onColorChange }) {
  return (
    <input
      type="color"
      value={color}
      onChange={event => onColorChange(event.target.value)}
    />
  )
}

function ColorPreview({ color }) {
  return (
    <div
      style={{
        width: 120,
        height: 120,
        backgroundColor: color,
        border: '1px solid #ccc',
      }}
    />
  )
}

export default function App() {
  const [color, setColor] = useState('#ff6600')

  return (
    <div>
      <ColorInput color={color} onColorChange={setColor} />
      <ColorPreview color={color} />
    </div>
  )
}
```

Now both children stay in sync because the parent owns the shared state.

## State vs Derived Values

Do not store values in state if they can be calculated from other state or props.

### Bad

```jsx
const [items, setItems] = useState([])
const [count, setCount] = useState(0)
```

Now you have to keep both values synchronized.

### Good

```jsx
const [items, setItems] = useState([])
const count = items.length
```

Derived values should usually be computed during render.

Other examples of derived values:

- filtered lists
- totals
- labels like `"3 items left"`
- booleans such as `const isEmpty = items.length === 0`

## Multiple State Variables

Use separate state variables for independent concerns:

```jsx
const [name, setName] = useState('')
const [email, setEmail] = useState('')
const [isSubmitting, setIsSubmitting] = useState(false)
```

Group related state together when it naturally belongs together:

```jsx
const [form, setForm] = useState({
  name: '',
  email: '',
})
```

A useful rule: if values tend to update together, grouping them can make sense. If they change independently, separate state is often clearer.

## When to Use State

Use `useState` when:

- a value changes over time
- that change should update the UI
- the value is owned by the component

Do **not** use `useState` when:

- the value never changes
- the value is derived from other state or props
- the value belongs higher in the tree
- the value should live outside React entirely

## Summary

- State is component memory that triggers re-renders
- `useState` returns the current value and a setter
- Hooks must be called at the top level of React functions
- Setter calls are scheduled, not immediate
- Use functional updates when next state depends on previous state
- Objects and arrays must be updated immutably
- Lazy initialization avoids repeated expensive setup
- Shared state should be lifted to a common parent
- Derived values should not be duplicated in state

## 💻 Exercises

### Level 1 — Basic

Build a counter with:

- increment
- decrement
- reset

Bonus:

- disable decrement when the count is already `0`

### Level 2 — Intermediate

Build a todo list with:

- add item
- remove item
- toggle completed

Requirements:

- store todos as objects
- use immutable array updates
- show a derived count of incomplete items

### Level 3 — Challenge

Build a shopping cart UI with:

- product list
- add to cart
- increase quantity
- decrease quantity
- remove item

Requirements:

- store cart items in state
- derive total item count and total price during render
- use functional state updates throughout
