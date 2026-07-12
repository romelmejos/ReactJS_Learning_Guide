[<< Day 4](../04_Day_JSX_Deep_Dive/04_jsx_deep_dive.md) | [Day 6 >>](../06_Day_Props/06_props.md)

# Day 5 – Functional Components

## Table of Contents
- [What Is a Component?](#what-is-a-component)
- [Your First Functional Component](#your-first-functional-component)
- [Naming Conventions](#naming-conventions)
- [Composing Components](#composing-components)
- [Default vs Named Exports](#default-vs-named-exports)
- [Component File Organization](#component-file-organization)
- [Rendering Components](#rendering-components)
- [Component Return Rules](#component-return-rules)
- [Component Trees](#component-trees)
- [Moving Beyond Static Components](#moving-beyond-static-components)
- [💻 Exercises](#-exercises)

## What Is a Component?

A React component is a **function that returns JSX**. Components are the fundamental building blocks of a React app.

Instead of writing one huge file for the whole UI, you split the UI into smaller pieces:

- header
- sidebar
- button
- card
- form

Each component has one job and can be reused.

```jsx
function WelcomeMessage() {
  return <h1>Welcome to React</h1>
}
```

When React sees `<WelcomeMessage />`, it calls the function and renders the returned JSX.

---

## Your First Functional Component

Classic function form:

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>
}

export default Greeting
```

Arrow function form:

```jsx
const Greeting = () => <h1>Hello, World!</h1>

export default Greeting
```

Both are valid. Teams usually choose one style and stay consistent.

### A slightly richer example

```jsx
function WelcomeBanner() {
  return (
    <section className="banner">
      <h1>Welcome to Day 5</h1>
      <p>Today we are learning functional components.</p>
    </section>
  )
}

export default WelcomeBanner
```

---

## Naming Conventions

React components must start with an **uppercase** letter.

### Correct

```jsx
function Button() {
  return <button>Click</button>
}
```

### Incorrect

```jsx
function button() {
  return <button>Click</button>
}
```

Lowercase tags are treated as native HTML elements, not custom components.

### Use PascalCase

For multi-word names, use PascalCase:

- `UserCard`
- `BlogPost`
- `SiteHeader`

### Match file name to component name

Good examples:

- `Greeting.jsx` → `Greeting`
- `UserProfile.jsx` → `UserProfile`

This makes projects easier to navigate.

---

## Composing Components

Components can render other components. This is called composition.

```jsx
function Header() {
  return <header><h1>My Blog</h1></header>
}

function Main() {
  return (
    <main>
      <p>Latest articles appear here.</p>
    </main>
  )
}

function Footer() {
  return <footer>© 2026 My Blog</footer>
}

function App() {
  return (
    <>
      <Header />
      <Main />
      <Footer />
    </>
  )
}

export default App
```

### Why composition matters

It helps you:

- separate concerns
- reuse UI
- make files smaller
- test parts independently

### Example: a page built from pieces

```jsx
function Logo() {
  return <h1>DevDaily</h1>
}

function Navigation() {
  return (
    <nav>
      <a href="/">Home</a>
      <a href="/articles">Articles</a>
      <a href="/about">About</a>
    </nav>
  )
}

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
    </header>
  )
}
```

---

## Default vs Named Exports

You will use both constantly in React projects.

### Default export

Use one default export per file.

```jsx
function Header() {
  return <header>Site Header</header>
}

export default Header
```

Import it with any name:

```jsx
import Header from './Header.jsx'
```

### Named exports

Use named exports when a file exposes multiple things.

```jsx
export function Card() {
  return <div>Card</div>
}

export function CardTitle() {
  return <h2>Title</h2>
}
```

Import them with matching names:

```jsx
import { Card, CardTitle } from './Card.jsx'
```

Or alias them:

```jsx
import { Card as ProductCard } from './Card.jsx'
```

### When to use each

Common convention:

- **default export** for the main component in a file
- **named exports** for helpers, hooks, utilities, or secondary components

Example:

```jsx
export function formatDate(date) {
  return new Date(date).toLocaleDateString()
}

function ArticleCard() {
  return <article>...</article>
}

export default ArticleCard
```

---

## Component File Organization

As your app grows, structure matters.

### Common guidelines

- one main component per file
- colocate related styles
- colocate related tests
- group reusable UI in a `components/` folder

Example structure:

```text
src/
├── components/
│   ├── Header/
│   │   ├── Header.jsx
│   │   ├── Header.css
│   │   └── Header.test.jsx
│   ├── Footer/
│   │   ├── Footer.jsx
│   │   └── Footer.css
│   └── Button/
│       ├── Button.jsx
│       └── Button.css
├── pages/
│   └── HomePage.jsx
└── main.jsx
```

### Small-project alternative

For small apps, a flatter structure is fine:

```text
src/
├── App.jsx
├── Header.jsx
├── Main.jsx
├── Footer.jsx
└── index.css
```

Start simple, then organize further when the app grows.

---

## Rendering Components

### Rendering at the root with `createRoot().render()`

In Vite React apps, rendering starts in `main.jsx`.

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

### Rendering components inside other components

```jsx
import Greeting from './Greeting.jsx'

function App() {
  return (
    <main>
      <Greeting />
    </main>
  )
}
```

### Self-closing vs wrapping syntax

These are equivalent when there are no children:

```jsx
<Greeting />
<Greeting></Greeting>
```

The self-closing form is cleaner and more common.

---

## Component Return Rules

A component must return something React can render.

### Valid returns

JSX:

```jsx
function Title() {
  return <h1>Hello</h1>
}
```

`null`:

```jsx
function HiddenMessage({ show }) {
  if (!show) {
    return null
  }

  return <p>Visible now</p>
}
```

A primitive:

```jsx
function CountLabel() {
  return 5
}
```

This works, but returning JSX is much more typical for top-level components.

### Early returns

Early returns keep components readable.

```jsx
function UserPanel({ user }) {
  if (!user) {
    return <p>Loading user...</p>
  }

  return <h2>{user.name}</h2>
}
```

---

## Component Trees

React UIs form a tree of parent-child relationships.

Example:

```jsx
function Avatar() {
  return <img src="/avatar.png" alt="User avatar" />
}

function UserInfo() {
  return (
    <section>
      <Avatar />
      <h2>Maria</h2>
      <p>Frontend Developer</p>
    </section>
  )
}

function Sidebar() {
  return <aside>Links</aside>
}

function App() {
  return (
    <div>
      <UserInfo />
      <Sidebar />
    </div>
  )
}
```

Tree view:

```text
App
├── UserInfo
│   └── Avatar
└── Sidebar
```

Understanding the tree helps you reason about:

- data flow
- re-renders
- component ownership
- where props and state should live

---

## Moving Beyond Static Components

So far, these components are static—they always render the same content.

The next big step is making components dynamic.

### Props (coming next)

Props let components receive data from parents.

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>
}
```

### State (coming soon)

State lets components remember data and react to user interaction.

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

This is where React becomes truly interactive.

---

## 💻 Exercises

### Level 1: Basic

Create these three components:

1. `Header` that returns a site title
2. `WelcomeMessage` that returns a short paragraph
3. `Footer` that returns a copyright line

Then render all three inside `App`.

### Level 2: Intermediate

Build a **Blog Post Card** using sub-components:

- `PostCard`
- `PostTitle`
- `PostMeta`
- `PostExcerpt`

Requirements:

- compose them together in one parent component
- export the main component as default
- keep each component name in PascalCase
- render a realistic article preview

### Level 3: Challenge

Build a full page layout using components:

- `Header`
- `Nav`
- `Main`
- `Sidebar`
- `Footer`

Requirements:

1. Create one file per main component.
2. Render them in a tree from `App`.
3. Add realistic placeholder content.
4. Use fragments where appropriate.
5. Draw the component tree in a comment or separate note for yourself before coding.

