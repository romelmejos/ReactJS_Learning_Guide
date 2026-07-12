[<< Day 15](../15_Day_Project_Folder_Structure/15_project_folder_structure.md) | [Day 17 >>](../17_Day_React_Router_v7/17_react_router_v7.md)

# Day 16 – Styling in React

## Table of Contents

- [Why styling strategy matters](#why-styling-strategy-matters)
- [Styling options overview](#styling-options-overview)
- [Inline styles](#inline-styles)
- [Global CSS](#global-css)
- [CSS Modules](#css-modules)
- [`clsx` and conditional classes](#clsx-and-conditional-classes)
- [Tailwind CSS v4 with Vite](#tailwind-css-v4-with-vite)
- [Tailwind v4 key changes](#tailwind-v4-key-changes)
- [CSS-in-JS brief overview](#css-in-js-brief-overview)
- [When to use each approach](#when-to-use-each-approach)
- [Dark mode patterns](#dark-mode-patterns)
- [💻 Exercises](#-exercises)

## Why styling strategy matters

React does not force one styling solution. That flexibility is powerful, but it also means teams can end up with inconsistent patterns if they do not pick a default.

In 2026, a strong default for React + Vite projects is:

- **CSS Modules** for component-scoped styling
- **Tailwind CSS v4** for fast product UI work
- **Global CSS** only for resets, tokens, and app-wide layout primitives

## Styling options overview

You can style React apps in several ways:

| Approach | Best for | Strengths | Weaknesses |
| --- | --- | --- | --- |
| Inline styles | Tiny one-off dynamic values | Easy, colocated with JSX | No `:hover`, no media queries, messy at scale |
| Global CSS | Resets, typography, app shell | Familiar, simple | Naming collisions, scoping issues |
| CSS Modules | Reusable components, design systems | Scoped by default, supports full CSS | Slightly more file management |
| Tailwind CSS | Product apps, fast UI building | Speed, consistency, responsive utilities | Long class strings, utility-first mindset |
| CSS-in-JS | Existing codebases, theme-heavy apps | Great component ergonomics | Runtime cost, extra dependency |

There is no universal winner. The right answer depends on team size, codebase age, and how reusable your UI needs to be.

## Inline styles

Inline styles are plain JavaScript objects passed to the `style` prop.

```jsx
export default function AlertMessage() {
  return (
    <div
      style={{
        color: 'red',
        fontSize: '16px',
        marginTop: '8px',
      }}
    >
      Something went wrong.
    </div>
  )
}
```

This works well for:

- quick experiments
- dynamic values like width or transform
- tiny style overrides

Example with dynamic styles:

```jsx
function ProgressBar({ value }) {
  return (
    <div style={{ backgroundColor: '#e5e7eb', borderRadius: '999px', padding: '2px' }}>
      <div
        style={{
          width: `${value}%`,
          backgroundColor: value > 70 ? '#16a34a' : '#2563eb',
          height: '12px',
          borderRadius: '999px',
          transition: 'width 200ms ease',
        }}
      />
    </div>
  )
}
```

### Limitations of inline styles

Inline styles are a poor default for real component styling because they do **not** support:

- pseudo-classes like `:hover`, `:focus`, `:disabled`
- media queries
- keyframe animations in a clean way
- reusable class composition

They also create style objects during rendering, which can become noisy and inefficient at scale.

**Rule of thumb:** use inline styles only for small, highly dynamic values. Do not build full component styling systems with them.

## Global CSS

Global CSS still matters. It is the best place for:

- CSS resets
- font declarations
- design tokens via CSS variables
- page-level layout helpers

In a Vite app, you usually import global CSS in `main.jsx`.

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

```css
/* src/index.css */
:root {
  font-family: Inter, system-ui, sans-serif;
  color: #111827;
  background: #f8fafc;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

a {
  color: inherit;
  text-decoration: none;
}
```

### The downside of global CSS

Global selectors can leak everywhere:

```css
.button {
  background: crimson;
}
```

If multiple files define `.button`, you now have a naming collision problem. This is why global CSS alone becomes risky as apps grow.

## CSS Modules

CSS Modules are one of the best defaults for React + Vite. They are built in, easy to understand, and scope class names automatically.

### Why CSS Modules are recommended

- scoped by default
- full CSS support, including `:hover` and media queries
- no runtime styling overhead
- easy to pair with reusable component libraries

Create a module file:

```css
/* src/components/Button/Button.module.css */
.button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 160ms ease, opacity 160ms ease;
}

.primary {
  background: #007bff;
  color: white;
}

.primary:hover {
  background: #005ecb;
}

.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@media (max-width: 640px) {
  .button {
    width: 100%;
  }
}
```

Use it in a component:

```jsx
// src/components/Button/Button.jsx
import styles from './Button.module.css'

export default function Button() {
  return <button className={`${styles.button} ${styles.primary}`}>Click</button>
}
```

Vite transforms the class names into unique generated names, so `button` in one module does not clash with `button` in another module.

### A more realistic CSS Module example

```css
/* src/components/ProfileCard/ProfileCard.module.css */
.card {
  padding: 1.25rem;
  border: 1px solid #dbe3f0;
  border-radius: 16px;
  background: white;
  box-shadow: 0 10px 30px rgb(15 23 42 / 0.08);
}

.name {
  margin: 0 0 0.25rem;
  font-size: 1.125rem;
}

.role {
  margin: 0;
  color: #475569;
}
```

```jsx
import styles from './ProfileCard.module.css'

export default function ProfileCard({ user }) {
  return (
    <article className={styles.card}>
      <h2 className={styles.name}>{user.name}</h2>
      <p className={styles.role}>{user.role}</p>
    </article>
  )
}
```

## `clsx` and conditional classes

As soon as components have variants like `primary`, `secondary`, `active`, or `disabled`, string concatenation becomes annoying.

Install `clsx`:

```bash
npm install clsx
```

```css
/* Button.module.css */
.button {
  padding: 0.75rem 1rem;
  border: none;
  border-radius: 0.5rem;
}

.active {
  outline: 3px solid #93c5fd;
}

.disabled {
  opacity: 0.5;
  pointer-events: none;
}
```

```jsx
import clsx from 'clsx'
import styles from './Button.module.css'

export default function Button({ isActive, disabled, children }) {
  return (
    <button
      className={clsx(styles.button, {
        [styles.active]: isActive,
        [styles.disabled]: disabled,
      })}
      disabled={disabled}
    >
      {children}
    </button>
  )
}
```

This is cleaner and easier to scale than manually building strings.

## Tailwind CSS v4 with Vite

Tailwind is a utility-first CSS framework that lets you build UI directly in JSX with utility classes.

For many product apps, it is the fastest way to ship polished interfaces.

### Install Tailwind v4

```bash
npm install tailwindcss @tailwindcss/vite
```

### Add the plugin to Vite

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

### Import Tailwind in your CSS entry file

```css
/* src/index.css */
@import "tailwindcss";
```

### Basic Tailwind usage

```jsx
export default function App() {
  return (
    <main className="min-h-screen bg-slate-50 p-8">
      <button className="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-md shadow-sm transition-colors">
        Save changes
      </button>
    </main>
  )
}
```

### A complete Tailwind card example

```jsx
export default function PricingCard() {
  return (
    <section className="max-w-sm rounded-2xl border border-slate-200 bg-white p-6 shadow-lg">
      <p className="text-sm font-semibold uppercase tracking-wide text-blue-600">Pro Plan</p>
      <h2 className="mt-2 text-3xl font-bold text-slate-900">$19</h2>
      <p className="mt-2 text-slate-600">Great for freelancers and small teams.</p>

      <ul className="mt-6 space-y-2 text-sm text-slate-700">
        <li>✔ Unlimited projects</li>
        <li>✔ Analytics dashboard</li>
        <li>✔ Email support</li>
      </ul>

      <button className="mt-6 w-full rounded-xl bg-slate-900 px-4 py-3 font-medium text-white hover:bg-slate-700">
        Start trial
      </button>
    </section>
  )
}
```

## Tailwind v4 key changes

Tailwind v4 changed the setup story significantly:

- **CSS-first configuration**: you do not need `tailwind.config.js` by default
- **`@theme` directive**: define tokens directly in CSS
- **better performance**: faster builds and a smoother dev experience

Example theme tokens:

```css
@import "tailwindcss";

@theme {
  --color-brand-500: #4f46e5;
  --radius-card: 1rem;
}
```

Then use those tokens via utilities generated from the theme system.

## CSS-in-JS brief overview

CSS-in-JS libraries such as **styled-components** and **emotion** let you write styles in JavaScript or TypeScript.

Example with `@emotion/css`:

```js
import { css } from '@emotion/css'

export const card = css({
  padding: '1rem',
  borderRadius: '12px',
  backgroundColor: 'white',
  boxShadow: '0 10px 20px rgba(15, 23, 42, 0.08)',
})
```

```jsx
import { card } from './styles'

export default function Card({ children }) {
  return <div className={card}>{children}</div>
}
```

Why teams liked CSS-in-JS:

- component-local styling
- great developer experience
- theming support

Why its popularity has declined:

- runtime CSS generation adds overhead
- Tailwind and CSS Modules solve many problems with less complexity

If you join an existing styled-components or emotion codebase, do not rewrite it just because newer patterns exist. Consistency is usually more important than trend chasing.

## When to use each approach

Use this decision guide:

- **Small project / prototype** → Tailwind CSS
- **Component library / design system** → CSS Modules
- **Existing styled-components codebase** → keep CSS-in-JS
- **Global app reset / typography** → global CSS
- **Complex component styling with inline styles** → avoid it

If you are unsure, start with this stack:

1. `index.css` for reset and design tokens
2. CSS Modules for shared components
3. Tailwind for app screens and product UI

## Dark mode patterns

There are two common approaches.

### 1. Tailwind `dark:` variant

```jsx
export default function Panel() {
  return (
    <section className="rounded-2xl bg-white p-6 text-slate-900 shadow dark:bg-slate-900 dark:text-slate-100">
      <h2 className="text-xl font-semibold">Settings</h2>
      <p className="mt-2 text-slate-600 dark:text-slate-300">Choose your preferences.</p>
    </section>
  )
}
```

Usually you toggle a `dark` class on the root element.

### 2. CSS custom properties

This works beautifully with CSS Modules and plain CSS.

```css
:root {
  --bg: #ffffff;
  --text: #0f172a;
  --surface: #f8fafc;
}

[data-theme='dark'] {
  --bg: #0f172a;
  --text: #f8fafc;
  --surface: #1e293b;
}

body {
  background: var(--bg);
  color: var(--text);
}
```

```jsx
import { useState } from 'react'

export default function ThemeToggle() {
  const [theme, setTheme] = useState('light')

  return (
    <div data-theme={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </div>
  )
}
```

This approach is excellent for themeable design systems because it separates theme values from component structure.

## 💻 Exercises

### Level 1 — CSS Modules button

Build a `Button` component with:

- a base `.button` class
- a `.primary` and `.secondary` variant
- a disabled state
- a hover style

Bonus: use `clsx` to switch variants with a `variant` prop.

### Level 2 — Tailwind card component

Create a reusable card with Tailwind that includes:

- title
- description
- badge
- action button
- hover elevation effect

Then make it responsive for mobile and desktop.

### Level 3 — Themeable design system

Build a mini design system using CSS custom properties:

- light and dark theme tokens
- button, card, and input components
- CSS Modules for component styling
- a theme toggle stored in local storage

Try to keep all colors driven by variables instead of hardcoded values inside components.
