[<< Day 14](../14_Day_Validation_React_Hook_Form/14_validation_react_hook_form.md) | [Day 16 >>](../16_Day_Styling_in_React/16_styling_in_react.md)

# Day 15 – Project Folder Structure

## Table of Contents

- [Why folder structure matters](#why-folder-structure-matters)
- [The two main approaches](#the-two-main-approaches)
- [A recommended hybrid structure](#a-recommended-hybrid-structure)
- [Barrel exports with indexjs](#barrel-exports-with-indexjs)
- [Vite path aliases](#vite-path-aliases)
- [The component folder pattern](#the-component-folder-pattern)
- [Naming conventions](#naming-conventions)
- [When to split and when not to split](#when-to-split-and-when-not-to-split)
- [Environment variables in Vite](#environment-variables-in-vite)
- [Scaling up](#scaling-up)
- [Complete example: reorganizing a small app](#complete-example-reorganizing-a-small-app)
- [💻 Exercises](#-exercises)

## Why folder structure matters

As projects grow, a random collection of files becomes painful:

- code is hard to find
- imports become messy
- teams duplicate logic
- onboarding takes longer

A folder structure is not about perfection. It is about making a codebase easier to understand and change.

There is no single universal structure, but some patterns scale much better than others.

```bash
cp -r ../boilerplate ../day-15-structure
cd ../day-15-structure
npm install
npm run dev
```

## The two main approaches

### 1. Layer-based (type-based)

Files are grouped by what they are:

- `components/`
- `hooks/`
- `utils/`
- `services/`
- `types/`

This is simple for small projects.

### 2. Feature-based (domain-based)

Files are grouped by what part of the product they belong to:

- `features/auth/`
- `features/dashboard/`
- `features/settings/`

This works well when the app has many product areas.

## A recommended hybrid structure

For most React apps, a hybrid structure is a great default:

```text
src/
├── assets/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.module.css
│   │   └── Button.test.jsx
│   └── index.js
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── api.js
│   └── dashboard/
├── hooks/
├── lib/
├── pages/
├── store/
├── types/
├── utils/
├── App.jsx
└── main.jsx
```

How to think about it:

- `components/` → shared reusable UI
- `features/` → app-specific business areas
- `hooks/` → shared custom hooks
- `lib/` → setup for tools like Axios, Query Client, or Firebase
- `pages/` → route-level screens
- `store/` → global state
- `utils/` → pure helper functions

## Barrel exports with index.js

Barrel exports create a single entry point for imports.

```js
// src/components/index.js
export { default as Button } from './Button/Button'
export { default as Modal } from './Modal/Modal'
export { default as Card } from './Card/Card'
```

Usage:

```jsx
import { Button, Modal, Card } from '@/components'
```

Benefits:

- fewer long relative imports
- clearer public API for shared modules
- easier refactoring later

Be careful not to overdo barrels in very large apps if they create circular dependencies.

## Vite path aliases

Path aliases make imports cleaner than `../../../`.

```js
// vite.config.js
import path from 'path'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

Now you can write:

```jsx
import Header from '@/components/Header/Header'
import { formatDate } from '@/utils/formatDate'
```

## The component folder pattern

When a component has related files, colocate them.

```text
components/
└── UserCard/
    ├── UserCard.jsx
    ├── UserCard.module.css
    ├── UserCard.test.jsx
    └── index.js
```

This keeps everything about the component together.

Example:

```jsx
// src/components/UserCard/UserCard.jsx
import styles from './UserCard.module.css'

export default function UserCard({ user }) {
  return (
    <article className={styles.card}>
      <h3>{user.name}</h3>
      <p>{user.role}</p>
    </article>
  )
}
```

```css
.card {
  padding: 1rem;
  border: 1px solid #d9def8;
  border-radius: 12px;
  background: white;
}
```

## Naming conventions

Use predictable naming:

- PascalCase for component files: `UserCard.jsx`
- camelCase for hooks: `useAuth.js`
- camelCase for utilities: `formatDate.js`
- PascalCase for type files: `User.types.ts`

This consistency helps people scan the codebase quickly.

## When to split and when not to split

Not every component deserves its own folder.

Use a folder when a component has:

- styles
- tests
- stories
- helper files

A single file is fine when the component is small and standalone.

Bad habit to avoid:

- creating folders just because it feels “enterprise”

Good habit:

- introduce structure only when it reduces confusion

## Environment variables in Vite

Vite environment variables have a special rule:

- only variables prefixed with `VITE_` are exposed to browser code

Example files:

```bash
# .env
VITE_API_URL=https://api.example.com

# .env.local
VITE_ANALYTICS_KEY=dev-only-key
```

Use them in React code like this:

```jsx
const apiUrl = import.meta.env.VITE_API_URL

export default function App() {
  return <p>API URL: {apiUrl}</p>
}
```

General convention:

- `.env` may be committed for safe defaults
- `.env.local` should usually stay uncommitted

## Scaling up

When should you lean harder into feature-based structure?

Usually when the app grows to roughly:

- more than 10 meaningful product areas
- more than 50 components
- multiple developers working in parallel

At that point, `features/` often becomes the main home of app logic.

In even larger organizations, teams may move to:

- package-based architecture
- workspaces
- monorepos

You do not need that on day one.

```js
// Start with a structure the team can understand today.
// Evolve it when the pain becomes real.
```

## Complete example: reorganizing a small app

Imagine you start with a flat `src/`:

```text
src/
├── App.jsx
├── Header.jsx
├── Dashboard.jsx
├── LoginForm.jsx
├── useAuth.js
├── api.js
├── formatCurrency.js
└── chart.png
```

A better version:

```text
src/
├── assets/
│   └── chart.png
├── components/
│   └── Header/
│       ├── Header.jsx
│       └── index.js
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   └── LoginForm.jsx
│   │   ├── hooks/
│   │   │   └── useAuth.js
│   │   └── api.js
│   └── dashboard/
│       └── components/
│           └── Dashboard.jsx
├── utils/
│   └── formatCurrency.js
├── App.jsx
└── main.jsx
```

Here is how the imports might look after the refactor:

```jsx
import Header from '@/components/Header'
import LoginForm from '@/features/auth/components/LoginForm'
import Dashboard from '@/features/dashboard/components/Dashboard'
import { formatCurrency } from '@/utils/formatCurrency'

export default function App() {
  const totalRevenue = 24500

  return (
    <main>
      <Header />
      <LoginForm />
      <Dashboard />
      <p>Revenue: {formatCurrency(totalRevenue)}</p>
    </main>
  )
}
```

Utility example:

```js
// src/utils/formatCurrency.js
export function formatCurrency(amount) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}
```

One possible `Header` component:

```jsx
// src/components/Header/Header.jsx
export default function Header() {
  return (
    <header className="site-header">
      <h1>Analytics Dashboard</h1>
    </header>
  )
}
```

```css
.site-header {
  padding: 1rem 0;
  border-bottom: 1px solid #e4e7ec;
  margin-bottom: 1.5rem;
}
```

## 💻 Exercises

### Level 1

Reorganize a flat collection of files into the recommended structure.

Requirements:

- create `components`, `features`, `utils`, and `assets`
- move shared UI out of feature folders
- keep feature-specific logic inside feature folders

### Level 2

Set up barrel exports and path aliases.

Requirements:

- create at least one `index.js` barrel file
- configure `@` to point to `src`
- replace long relative imports with alias imports

### Level 3

Design the folder structure for a social media app.

Requirements:

- include auth, feed, profile, notifications, and messaging features
- show where shared components belong
- include a place for API clients, state, and utilities
- explain why you grouped files that way
