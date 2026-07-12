[<< readMe](../readMe.md) | [Day 2 >>](../02_Day_Introduction_to_React/02_introduction_to_react.md)

# Day 1 – JavaScript Refresher

## Table of Contents
- [Why JavaScript Fundamentals Matter for React](#why-javascript-fundamentals-matter-for-react)
- [Variables \& Scoping](#variables--scoping)
- [Destructuring](#destructuring)
- [Spread \& Rest Operators](#spread--rest-operators)
- [Template Literals](#template-literals)
- [Arrow Functions](#arrow-functions)
- [Optional Chaining](#optional-chaining)
- [Nullish Coalescing](#nullish-coalescing)
- [Array Methods for React](#array-methods-for-react)
- [Objects](#objects)
- [Promises \& asyncawait](#promises--asyncawait)
- [Fetch API](#fetch-api)
- [ES Modules](#es-modules)
- [ES2023–2024 Features You Should Know](#es20232024-features-you-should-know)
- [💻 Exercises](#-exercises)

## Why JavaScript Fundamentals Matter for React

React is "just JavaScript" plus a component model. If you are comfortable with modern JavaScript, React code becomes much easier to read and write:

- props are often destructured
- state updates usually rely on object and array spread
- UI lists are rendered with `.map()`
- data fetching uses `async`/`await`
- conditional rendering often uses `?.`, `??`, and ternaries

React 19 codebases are also deeply modern: hooks-first, ESM-first, and usually built with Vite. That means ES2024-friendly habits are not optional—they are the baseline.

---

## Variables & Scoping

Use `const` by default, `let` when reassignment is needed, and avoid `var`.

### `const`

`const` prevents reassignment of the variable binding.

```js
const appName = 'React Guide'
// appName = 'Other' // ❌ TypeError
```

For objects and arrays, `const` does **not** make the value immutable. It only prevents rebinding.

```js
const user = { name: 'Ava' }
user.name = 'Lina' // ✅ allowed
```

### `let`

Use `let` when a variable must change.

```js
let count = 0
count += 1
```

### Why `var` is avoided

`var` is function-scoped, can be redeclared, and causes confusing bugs because it does not respect block scope.

```js
if (true) {
  var message = 'hello'
}

console.log(message) // 'hello'
```

Compare that with `let`:

```js
if (true) {
  let message = 'hello'
}

// console.log(message) // ❌ ReferenceError
```

### Block scope

`let` and `const` only exist inside the nearest block:

```js
{
  const theme = 'dark'
  console.log(theme) // 'dark'
}

// console.log(theme) // ❌ theme is not defined
```

### Temporal Dead Zone (TDZ)

Variables declared with `let` and `const` exist in a "temporal dead zone" from the start of the block until the declaration is reached.

```js
// console.log(score) // ❌ Cannot access 'score' before initialization
let score = 10
```

This is good: it catches bugs early.

---

## Destructuring

Destructuring is everywhere in React, especially with props, hook returns, and array/object transformations.

### Array destructuring

```js
const [a, b] = [1, 2]
console.log(a, b) // 1 2
```

Skip values when needed:

```js
const [first, , third] = ['React', 'Vue', 'Svelte']
console.log(first, third) // React Svelte
```

Use rest to capture the remaining items:

```js
const numbers = [10, 20, 30, 40]
const [first, ...rest] = numbers

console.log(first) // 10
console.log(rest)  // [20, 30, 40]
```

This pattern feels very natural if you have used React hooks:

```js
const [count, setCount] = useState(0)
```

### Object destructuring

```js
const person = { name: 'Maya', age: 27 }
const { name, age } = person
```

Rename a property while destructuring:

```js
const { name: fullName } = person
console.log(fullName) // Maya
```

Provide default values:

```js
const user = { name: 'Ravi' }
const { name, role = 'Guest' } = user

console.log(role) // Guest
```

Nested destructuring:

```js
const profile = {
  id: 1,
  contact: {
    email: 'hello@example.com',
    address: {
      city: 'Lagos',
    },
  },
}

const {
  contact: {
    email,
    address: { city },
  },
} = profile

console.log(email, city) // hello@example.com Lagos
```

### Function parameter destructuring

Very common in React components:

```jsx
function UserCard({ name, role = 'Member', avatarUrl }) {
  return (
    <article>
      <img src={avatarUrl} alt={name} />
      <h2>{name}</h2>
      <p>{role}</p>
    </article>
  )
}
```

You can also destructure nested props:

```jsx
function Address({ user: { name, address: { city } } }) {
  return <p>{name} lives in {city}</p>
}
```

---

## Spread & Rest Operators

They look the same (`...`) but do different things depending on context.

### Spread with arrays

Merge arrays:

```js
const frontend = ['HTML', 'CSS']
const javascript = ['React', 'Vite']

const skills = [...frontend, ...javascript]
console.log(skills) // ['HTML', 'CSS', 'React', 'Vite']
```

Clone arrays:

```js
const original = [1, 2, 3]
const copy = [...original]
```

### Spread with objects

Merge objects:

```js
const baseUser = { name: 'Noah', role: 'User' }
const adminUser = { ...baseUser, role: 'Admin' }

console.log(adminUser) // { name: 'Noah', role: 'Admin' }
```

This pattern is crucial for React state updates because React expects immutable updates.

```js
const state = {
  user: { name: 'Zara', online: false },
}

const nextState = {
  ...state,
  user: {
    ...state.user,
    online: true,
  },
}
```

### Rest in functions

```js
function sum(...nums) {
  return nums.reduce((total, num) => total + num, 0)
}

console.log(sum(1, 2, 3, 4)) // 10
```

### Rest in destructuring

```js
const config = {
  apiUrl: '/api',
  retries: 3,
  timeout: 5000,
}

const { apiUrl, ...options } = config

console.log(apiUrl)  // /api
console.log(options) // { retries: 3, timeout: 5000 }
```

---

## Template Literals

Template literals use backticks and allow interpolation and multiline strings.

```js
const name = 'Amina'
const greeting = `Hello, ${name}!`
console.log(greeting)
```

Multiline strings:

```js
const message = `
Welcome to React 19.
We write components with modern JavaScript.
`
```

Briefly: tagged templates let you process template literal parts with a function.

```js
function highlight(strings, value) {
  return `${strings[0]}**${value}**${strings[1]}`
}

const result = highlight`Hello ${'React'}!`
console.log(result) // Hello **React**!
```

---

## Arrow Functions

Arrow functions are compact and very common in React.

### Basic syntax

```js
const double = (n) => {
  return n * 2
}
```

### Implicit return

If the function body is a single expression, the value is returned automatically.

```js
const double = (n) => n * 2
const getName = (user) => user.name
```

Return an object by wrapping it in parentheses:

```js
const createUser = (name) => ({ name, online: true })
```

### `this` behavior

Arrow functions do **not** create their own `this`. They capture `this` from the surrounding scope.

```js
const counter = {
  count: 0,
  start() {
    setInterval(() => {
      this.count += 1
      console.log(this.count)
    }, 1000)
  },
}
```

A regular function inside `setInterval` would have a different `this` unless explicitly bound.

In React, arrow functions are often used in callbacks:

```jsx
const items = ['React', 'Vite', 'Hooks']
const list = items.map((item) => <li key={item}>{item}</li>)
```

---

## Optional Chaining

Optional chaining (`?.`) safely accesses deeply nested values.

```js
const user = {
  profile: {
    address: {
      city: 'Accra',
    },
  },
}

console.log(user?.profile?.address?.city) // Accra
console.log(user?.settings?.theme)        // undefined
```

Method calls:

```js
const logger = {
  info(message) {
    console.log(message)
  },
}

logger?.info?.('Loaded successfully')
```

Array access:

```js
const colors = ['red', 'green']
console.log(colors?.[0]) // red
console.log(colors?.[5]) // undefined
```

This is useful when rendering API data that may not be ready yet.

```jsx
<p>{user?.profile?.name ?? 'Loading...'}</p>
```

---

## Nullish Coalescing

`??` only falls back when the left side is `null` or `undefined`.

```js
const username = null
console.log(username ?? 'Guest') // Guest
```

### Difference from `||`

`||` treats `0`, `''`, and `false` as falsy.

```js
console.log(0 || 100)   // 100
console.log(0 ?? 100)   // 0

console.log('' || 'N/A') // N/A
console.log('' ?? 'N/A') // ''
```

That matters a lot in forms and UI rendering.

```js
const quantity = 0
const safeQuantity = quantity ?? 1 // keeps 0
```

### Nullish assignment

```js
let theme = null
theme ??= 'light'

console.log(theme) // light
```

---

## Array Methods for React

React relies heavily on non-mutating array operations.

### `.map()`

Transforms each item into a new value.

```js
const prices = [10, 20, 30]
const formatted = prices.map((price) => `$${price}`)
console.log(formatted) // ['$10', '$20', '$30']
```

React example:

```jsx
const users = [
  { id: 1, name: 'Ada' },
  { id: 2, name: 'Lin' },
]

function UserList() {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### `.filter()`

Returns items that pass a test.

```js
const products = [
  { id: 1, inStock: true },
  { id: 2, inStock: false },
]

const available = products.filter((product) => product.inStock)
```

### `.reduce()`

Accumulates a value.

```js
const cart = [
  { price: 25, qty: 2 },
  { price: 10, qty: 1 },
]

const total = cart.reduce((sum, item) => sum + item.price * item.qty, 0)
console.log(total) // 60
```

### `.find()` and `.findIndex()`

```js
const users = [
  { id: 1, name: 'Mia' },
  { id: 2, name: 'Kai' },
]

const match = users.find((user) => user.id === 2)
const index = users.findIndex((user) => user.id === 2)
```

### `.some()` and `.every()`

```js
const scores = [85, 92, 78]

const hasLowScore = scores.some((score) => score < 80)
const allPassed = scores.every((score) => score >= 50)
```

### `.flat()` and `.flatMap()`

```js
const nested = [[1, 2], [3, 4]]
console.log(nested.flat()) // [1, 2, 3, 4]
```

```js
const sentences = ['learn react', 'ship apps']
const words = sentences.flatMap((sentence) => sentence.split(' '))
console.log(words) // ['learn', 'react', 'ship', 'apps']
```

### Chaining methods

This style is very common in data-driven UIs.

```js
const data = [
  { id: 1, name: 'react', stars: 5, active: true },
  { id: 2, name: 'vue', stars: 4, active: false },
  { id: 3, name: 'svelte', stars: 5, active: true },
]

const result = data
  .filter((item) => item.active)
  .map((item) => ({ ...item, name: item.name.toUpperCase() }))
  .sort((a, b) => a.name.localeCompare(b.name))

console.log(result)
```

Important rule: prefer non-mutating methods when preparing data for React rendering.

---

## Objects

### Shorthand properties

```js
const name = 'Luca'
const age = 30

const user = { name, age }
// same as { name: name, age: age }
```

### Computed properties

```js
const key = 'theme'
const settings = {
  [key]: 'dark',
}
```

Useful when updating state dynamically:

```js
const field = 'email'
const value = 'user@example.com'

const nextForm = {
  ...form,
  [field]: value,
}
```

### `Object.keys`, `Object.values`, `Object.entries`

```js
const stats = { likes: 10, shares: 2, comments: 4 }

console.log(Object.keys(stats))   // ['likes', 'shares', 'comments']
console.log(Object.values(stats)) // [10, 2, 4]
console.log(Object.entries(stats))
```

Render object entries:

```jsx
function StatsList({ stats }) {
  return (
    <ul>
      {Object.entries(stats).map(([label, value]) => (
        <li key={label}>
          {label}: {value}
        </li>
      ))}
    </ul>
  )
}
```

### `Object.assign`

```js
const merged = Object.assign({}, obj1, obj2)
```

It works, but object spread is usually shorter and easier to read:

```js
const merged = { ...obj1, ...obj2 }
```

---

## Promises & async/await

Async code is essential for React apps that load remote data.

### Creating a Promise

```js
const delay = (ms) =>
  new Promise((resolve) => {
    setTimeout(resolve, ms)
  })
```

With rejection:

```js
const fetchUser = (id) =>
  new Promise((resolve, reject) => {
    if (!id) {
      reject(new Error('User ID is required'))
      return
    }

    resolve({ id, name: 'Tariq' })
  })
```

### `.then()`, `.catch()`, `.finally()`

```js
fetchUser(1)
  .then((user) => {
    console.log('User:', user)
  })
  .catch((error) => {
    console.error('Failed:', error.message)
  })
  .finally(() => {
    console.log('Done')
  })
```

### `async` / `await`

```js
async function loadUser() {
  const user = await fetchUser(1)
  console.log(user)
}
```

### Error handling with `try` / `catch`

```js
async function loadUserSafely(id) {
  try {
    const user = await fetchUser(id)
    return user
  } catch (error) {
    console.error('Could not load user:', error.message)
    return null
  }
}
```

### `Promise.all`

Runs promises in parallel and fails fast if any reject.

```js
const [user, posts] = await Promise.all([
  fetch('/api/user').then((res) => res.json()),
  fetch('/api/posts').then((res) => res.json()),
])
```

### `Promise.allSettled`

Useful when you want all results, even if some fail.

```js
const results = await Promise.allSettled([
  Promise.resolve('ok'),
  Promise.reject(new Error('bad')),
])

console.log(results)
```

### `Promise.race`

Returns the first settled promise.

```js
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Timed out')), 3000)
)

const result = await Promise.race([
  fetch('/api/data'),
  timeout,
])
```

---

## Fetch API

`fetch()` returns a promise for a `Response`.

```js
const response = await fetch('https://jsonplaceholder.typicode.com/users')
const users = await response.json()
console.log(users)
```

### Important: fetch does not throw on HTTP 4xx/5xx

This surprises many beginners.

```js
async function getJson(url) {
  const response = await fetch(url)

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`)
  }

  return response.json()
}
```

### Practical example

```js
async function loadPosts() {
  try {
    const posts = await getJson('https://jsonplaceholder.typicode.com/posts')
    return posts.slice(0, 5)
  } catch (error) {
    console.error('Could not load posts:', error.message)
    return []
  }
}
```

In React, this logic often lives inside `useEffect`, server actions, or data-layer utilities.

---

## ES Modules

Modern React projects use ES modules.

### Named exports

```js
export const apiUrl = '/api'
export function formatPrice(value) {
  return `$${value}`
}
```

Import them by exact name:

```js
import { apiUrl, formatPrice } from './utils.js'
```

### Default export

```js
export default function Button() {
  return 'button'
}
```

Import it with any name:

```js
import Button from './Button.js'
```

### Namespace import

```js
import * as mathUtils from './math.js'

console.log(mathUtils.add(2, 3))
```

### Dynamic import

Loads a module on demand.

```js
const module = await import('./charts.js')
module.renderChart()
```

This is useful for lazy loading and code splitting.

---

## ES2023–2024 Features You Should Know

These newer APIs are especially helpful because they avoid mutation.

### `toSorted()`

Unlike `.sort()`, it returns a new array.

```js
const numbers = [3, 1, 2]
const sorted = numbers.toSorted((a, b) => a - b)

console.log(sorted)  // [1, 2, 3]
console.log(numbers) // [3, 1, 2]
```

### `toReversed()`

```js
const letters = ['a', 'b', 'c']
const reversed = letters.toReversed()
```

### `toSpliced()`

Non-mutating version of `.splice()`.

```js
const items = ['a', 'b', 'c']
const updated = items.toSpliced(1, 1, 'x')

console.log(updated) // ['a', 'x', 'c']
console.log(items)   // ['a', 'b', 'c']
```

### `Array.prototype.with()`

Replace one item immutably.

```js
const list = ['React', 'Vue', 'Svelte']
const updated = list.with(1, 'Solid')
```

### `Object.groupBy()`

Great for grouping API data before rendering.

```js
const products = [
  { name: 'Laptop', category: 'tech' },
  { name: 'Phone', category: 'tech' },
  { name: 'Notebook', category: 'stationery' },
]

const grouped = Object.groupBy(products, (product) => product.category)
console.log(grouped)
```

### `structuredClone()`

Deep clones supported data structures.

```js
const original = {
  user: { name: 'Nina' },
  tags: ['react', 'vite'],
}

const copy = structuredClone(original)
copy.user.name = 'Eva'

console.log(original.user.name) // Nina
console.log(copy.user.name)     // Eva
```

Use it carefully: React state updates are usually better handled with targeted immutable updates rather than cloning everything blindly.

---

## 💻 Exercises

### Level 1: Basic

1. Create an array `['React', 'Vite', 'npm']` and:
   - destructure the first value
   - skip the second
   - collect the rest
2. Create an object `{ name: 'Ayo', role: 'Student' }` and:
   - rename `name` to `fullName`
   - give `role` a default value anyway
   - add a new property using spread
3. Write a function `formatUser({ name, city = 'Unknown' })` that returns a template literal string.

### Level 2: Intermediate

Given this data:

```js
const products = [
  { id: 1, name: 'Keyboard', price: 40, inStock: true },
  { id: 2, name: 'Mouse', price: 20, inStock: false },
  { id: 3, name: 'Monitor', price: 150, inStock: true },
  { id: 4, name: 'USB Cable', price: 10, inStock: true },
]
```

Do the following:

1. Get only in-stock products.
2. Convert them into strings like `'Keyboard - $40'`.
3. Find the first product that costs more than 100.
4. Calculate the total price of all in-stock products.
5. Sort the products by price **without mutating the original array**.

### Level 3: Challenge

Build a small async data transformation pipeline:

1. Create an `async function loadUsers()`.
2. Inside it, fetch users from `https://jsonplaceholder.typicode.com/users`.
3. Throw an error if `response.ok` is false.
4. Transform the data into this shape:

```js
[
  {
    id: 1,
    name: 'Leanne Graham',
    city: 'Gwenborough',
    company: 'Romaguera-Crona',
  },
]
```

5. Group the users by city using `Object.groupBy()`.
6. Create a second result that contains only company names, sorted alphabetically with `toSorted()`.
7. Wrap everything in `try`/`catch` and return a fallback object if the request fails.
