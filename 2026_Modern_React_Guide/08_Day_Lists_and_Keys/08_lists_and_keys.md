[<< Day 7](../07_Day_useState/07_use_state.md) | [Day 9 >>](../09_Day_Conditional_Rendering/09_conditional_rendering.md)

# Day 8 – Rendering Lists and Keys

## Table of Contents

- [Rendering Lists with `.map()`](#rendering-lists-with-map)
- [Why the `key` Prop Matters](#why-the-key-prop-matters)
- [What to Use as Keys](#what-to-use-as-keys)
- [Fragments with Keys](#fragments-with-keys)
- [Filtering Lists](#filtering-lists)
- [Sorting Lists Immutably](#sorting-lists-immutably)
- [Nested Lists](#nested-lists)
- [Key Extraction Pattern](#key-extraction-pattern)
- [Performance Note](#performance-note)
- [Common Patterns](#common-patterns)
- [Summary](#summary)
- [💻 Exercises](#-exercises)

## Rendering Lists with `.map()`

In React, you usually render repeated UI by turning an array into JSX with `.map()`.

To follow along in Vite:

```bash
cd boilerplate
npm install
npm run dev
```

Here's the plain JavaScript data that will drive the UI:

```js
const fruits = ['Apple', 'Banana', 'Cherry']
```

```jsx
const fruits = ['Apple', 'Banana', 'Cherry']

function FruitList() {
  return (
    <ul>
      {fruits.map(fruit => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  )
}
```

The important idea is simple:

- start with data
- transform each item into JSX
- return the list inside the parent element

Real apps usually map objects instead of strings:

```jsx
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Carol' },
]

function UserList() {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

The raw data often looks like this before JSX gets involved:

```js
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Carol' },
]
```

## Why the `key` Prop Matters

Keys help React identify which list items changed, were added, or were removed.

When React compares one render to the next, it uses keys during reconciliation. Stable keys let React preserve the correct DOM nodes and component instances.

Without proper keys:

- items may appear to jump around
- input state can attach to the wrong row
- React may do extra work

Keys must be:

- **stable**
- **unique among siblings**
- **predictable**

## What to Use as Keys

### ✅ Best: unique IDs from your data

```jsx
items.map(item => <Row key={item.id} item={item} />)
```

### ⚠️ Acceptable: unique strings

```jsx
items.map(item => <Tag key={item.name} label={item.name} />)
```

Only use this if the string is actually unique among siblings.

### ❌ Usually avoid: array indexes

```jsx
items.map((item, index) => <Row key={index} item={item} />)
```

Indexes are only acceptable for **static lists that never reorder, insert, or delete items**.

### ❌ Never use `Math.random()`

```jsx
items.map(item => <Row key={Math.random()} item={item} />)
```

That creates a new key on every render, which forces React to unmount and remount every item.

## Fragments with Keys

Sometimes one list item needs to return multiple siblings.

Use a keyed `React.Fragment`:

```jsx
import React from 'react'

function DefinitionList({ items }) {
  return (
    <dl>
      {items.map(item => (
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  )
}
```

The short fragment syntax `<>...</>` cannot accept a key, so use `React.Fragment` when you need one.

## Filtering Lists

You can filter before mapping:

```jsx
const activeItems = items.filter(item => !item.completed)
```

```jsx
function ActiveTaskList({ items }) {
  const activeItems = items.filter(item => !item.completed)

  return (
    <ul>
      {activeItems.map(item => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  )
}
```

This keeps render logic readable and prevents clutter inside JSX.

## Sorting Lists Immutably

Sorting mutates arrays, so copy first.

```jsx
const sorted = [...items].toSorted((a, b) => a.name.localeCompare(b.name))
```

For older environments:

```jsx
const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name))
```

Example:

```jsx
function SortedUsers({ users }) {
  const sortedUsers = [...users].toSorted((a, b) =>
    a.name.localeCompare(b.name)
  )

  return (
    <ul>
      {sortedUsers.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

Never do this in render:

```jsx
users.sort((a, b) => a.name.localeCompare(b.name)) // ❌ mutates original array
```

## Nested Lists

Lists can contain other lists.

```jsx
const categories = [
  {
    id: 'frontend',
    name: 'Frontend',
    tools: ['React', 'Vite', 'Vitest'],
  },
  {
    id: 'backend',
    name: 'Backend',
    tools: ['Node.js', 'Express'],
  },
]

function CategoryList() {
  return (
    <div>
      {categories.map(category => (
        <section key={category.id}>
          <h2>{category.name}</h2>
          <ul>
            {category.tools.map(tool => (
              <li key={tool}>{tool}</li>
            ))}
          </ul>
        </section>
      ))}
    </div>
  )
}
```

Keys are only scoped to siblings. A key does not need to be globally unique across the entire app.

## Key Extraction Pattern

You can build reusable list abstractions with a render prop.

```jsx
function List({ items, renderItem, getKey }) {
  return (
    <ul>
      {items.map(item => (
        <li key={getKey(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  )
}
```

Usage:

```jsx
const products = [
  { id: 'p1', name: 'Keyboard', price: 99 },
  { id: 'p2', name: 'Mouse', price: 49 },
]

function App() {
  return (
    <List
      items={products}
      getKey={product => product.id}
      renderItem={product => (
        <span>
          {product.name} - ${product.price}
        </span>
      )}
    />
  )
}
```

This pattern separates:

- the iteration logic
- the key logic
- the rendering logic

## Performance Note

React's reconciliation depends heavily on stable keys.

If a key changes, React treats that item as a completely different component:

- the old item unmounts
- the new item mounts
- local state is lost

That is why stable IDs are so important for inputs, animations, editable tables, and drag-and-drop interfaces.

## Common Patterns

### List of cards

```jsx
function PostGrid({ posts }) {
  return (
    <div className="grid">
      {posts.map(post => (
        <article key={post.id} className="card">
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  )
}
```

### Table rows

```jsx
function OrdersTable({ orders }) {
  return (
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        {orders.map(order => (
          <tr key={order.id}>
            <td>{order.id}</td>
            <td>{order.status}</td>
          </tr>
        ))}
      </tbody>
    </table>
  )
}
```

### Dynamic navigation

```jsx
function NavMenu({ items }) {
  return (
    <nav>
      <ul>
        {items.map(item => (
          <li key={item.href}>
            <a href={item.href}>{item.label}</a>
          </li>
        ))}
      </ul>
    </nav>
  )
}
```

## Summary

- Use `.map()` to turn arrays into JSX
- Every rendered list item needs a stable key
- Prefer real IDs from data
- Avoid indexes for dynamic lists
- Filter and sort before mapping
- Copy arrays before sorting
- Keys only need to be unique among siblings
- Stable keys improve correctness and performance

## 💻 Exercises

### Level 1 — Basic

Render a list of countries from an array.

Requirements:

- show country name and capital
- use a stable key
- display at least five countries

### Level 2 — Intermediate

Build a filterable and sortable list of products.

Requirements:

- filter by search text
- sort by name
- sort by price
- never mutate the original array

### Level 3 — Challenge

Build a kanban-style board with:

- multiple columns
- cards inside each column
- stable keys everywhere

Requirements:

- cards should look draggable even if you do not implement drag-and-drop yet
- filter cards by a search box
- sort cards by priority within each column
