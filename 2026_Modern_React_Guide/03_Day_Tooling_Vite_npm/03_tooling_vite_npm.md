[<< Day 2](../02_Day_Introduction_to_React/02_introduction_to_react.md) | [Day 4 >>](../04_Day_JSX_Deep_Dive/04_jsx_deep_dive.md)

# Day 3 – Tooling: Vite and npm

## Table of Contents
- [Why Vite Replaced Create React App](#why-vite-replaced-create-react-app)
- [Node.js \& npm Setup](#nodejs--npm-setup)
- [Scaffolding with Vite](#scaffolding-with-vite)
- [Project Structure Deep-Dive](#project-structure-deep-dive)
- [npm Scripts](#npm-scripts)
- [Understanding packagejson](#understanding-packagejson)
- [ESLint + Prettier Setup](#eslint--prettier-setup)
- [Vite Configuration](#vite-configuration)
- [Environment Variables](#environment-variables)
- [💻 Exercises](#-exercises)

## Why Vite Replaced Create React App

For years, Create React App (CRA) was the default starting point for React beginners. In modern React development, Vite is the preferred option.

### Why CRA fell out of favor

- CRA is effectively unmaintained
- it relies on webpack-based tooling with slower startup
- configuration is more opaque
- cold starts and rebuilds feel heavy in modern workflows

### Why Vite won

Vite improves the developer experience dramatically:

- uses **native ES modules** during development
- provides **near-instant server startup**
- delivers **fast Hot Module Replacement (HMR)**
- uses **esbuild** for speed and **Rollup** for production builds
- works beautifully with React, TypeScript, and modern ESM workflows

### In simple terms

CRA bundled almost everything up front.

Vite serves source files on demand during development and only performs a full optimized bundle for production.

That is why it feels much faster.

---

## Node.js & npm Setup

React with Vite requires Node.js and npm.

### Recommended versions for this guide

- **Node.js 20+**
- **npm 10+**

Check your installed versions:

```bash
node -v
npm -v
```

Example output:

```bash
v22.3.0
10.8.1
```

### What is npm?

npm stands for **Node Package Manager**. It is used to:

- install dependencies
- run project scripts
- manage package metadata
- publish and consume JavaScript packages

When you run:

```bash
npm install
```

npm reads `package.json`, installs dependencies, and creates `node_modules/`.

---

## Scaffolding with Vite

Create a new React app with:

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

### Step-by-step explanation

#### 1. `npm create vite@latest my-app -- --template react`

- `npm create` runs a project scaffolding package
- `vite@latest` selects the latest Vite scaffolder
- `my-app` is the new folder name
- `--template react` chooses the React template

#### 2. `cd my-app`

Move into the generated project directory.

#### 3. `npm install`

Installs dependencies listed in `package.json`.

Typical packages include:

- `react`
- `react-dom`
- `vite`
- `@vitejs/plugin-react`
- `eslint`

#### 4. `npm run dev`

Starts the development server with HMR.

You will usually see output like:

```bash
VITE v6.x.x  ready in 200 ms

➜  Local:   http://localhost:5173/
```

Open the URL in your browser to see your app.

---

## Project Structure Deep-Dive

Typical Vite React project:

```text
my-app/
├── index.html
├── vite.config.js
├── eslint.config.js
├── package.json
├── public/
└── src/
    ├── main.jsx
    ├── App.jsx
    ├── App.css
    └── index.css
```

### `index.html`

This is the app's HTML entry point. Unlike CRA, it lives at the project root.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

Important difference: the browser loads `src/main.jsx` directly in development.

### `vite.config.js`

Controls Vite behavior, plugins, aliases, and more.

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

### `eslint.config.js`

Defines linting rules. New Vite templates typically use ESLint's flat config format.

### `package.json`

Contains:

- project metadata
- dependencies
- scripts
- module type settings

### `public/`

Files here are served as-is and are **not** processed by Vite.

Use it for:

- favicons
- static images
- manifest files

If a file is in `public/logo.png`, it is available at `/logo.png`.

### `src/main.jsx`

The real JavaScript entry point.

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

This file mounts your React app into the DOM.

### `src/App.jsx`

Your root component.

```jsx
function App() {
  return <h1>Hello from Vite + React</h1>
}

export default App
```

### `src/App.css` and `src/index.css`

- `index.css` usually holds global styles
- `App.css` often styles the main example component

---

## npm Scripts

Open `package.json` and you will typically see scripts like:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint ."
  }
}
```

### `npm run dev`

Starts the development server with HMR.

### `npm run build`

Creates an optimized production build in the `dist/` folder.

```bash
npm run build
```

After a successful build, the app is ready to deploy.

### `npm run preview`

Serves the production build locally so you can test what will actually ship.

```bash
npm run preview
```

### `npm run lint`

Runs ESLint across the project.

```bash
npm run lint
```

This helps catch bugs, bad patterns, and inconsistent code.

---

## Understanding `package.json`

Here is a simplified example:

```json
{
  "name": "my-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.0",
    "eslint": "^9.0.0",
    "vite": "^6.0.0"
  }
}
```

### `dependencies`

Packages required to run your app in production.

Examples:

- `react`
- `react-dom`

### `devDependencies`

Packages only needed during development or build time.

Examples:

- `vite`
- `eslint`
- `prettier`

### `scripts`

Reusable terminal commands you run with `npm run`.

### `type: "module"`

This tells Node.js to interpret `.js` files as ES modules by default.

That matches the ESM style used by Vite.

---

## ESLint + Prettier Setup

Vite already gives you ESLint. Prettier is usually added separately for formatting.

### Install Prettier

```bash
npm install -D prettier eslint-config-prettier
```

### Create `.prettierrc`

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2
}
```

### Why `eslint-config-prettier`?

It disables ESLint rules that conflict with Prettier formatting.

### Example ESLint integration idea

Depending on your ESLint setup, you may extend or include Prettier compatibility so formatting and linting do not fight each other.

### VS Code settings for format-on-save

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

### Recommended workflow

Use:

- ESLint for code quality
- Prettier for formatting

They solve different problems.

---

## Vite Configuration

Vite config is usually small, but powerful.

### Basic config

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

### Add an alias

```js
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

Then import like this:

```jsx
import Button from '@/components/Button.jsx'
```

### Environment variables with `import.meta.env`

Vite exposes environment variables via `import.meta.env`.

```js
console.log(import.meta.env.VITE_API_URL)
console.log(import.meta.env.MODE)
console.log(import.meta.env.DEV)
console.log(import.meta.env.PROD)
```

---

## Environment Variables

Vite supports several `.env` files.

### Common files

- `.env`
- `.env.local`
- `.env.development`
- `.env.production`

### Critical rule: `VITE_` prefix

Only variables prefixed with `VITE_` are exposed to client-side code.

Example `.env`:

```env
VITE_API_URL=https://api.example.com
VITE_APP_NAME=Modern React Guide
```

Use them in your code:

```js
const apiUrl = import.meta.env.VITE_API_URL
```

### Important security note

Never put secrets meant to stay private into client-side environment variables. If the browser can access it, users can access it too.

### `.env.local`

Useful for machine-specific values that should not be shared.

Many teams add `.env.local` to `.gitignore`.

---

## 💻 Exercises

### Level 1: Basic

1. Create a new Vite React app named `my-react-day3`.
2. Run `npm install`.
3. Start the development server with `npm run dev`.
4. Open the app in your browser.
5. Change the text in `App.jsx` and confirm HMR updates the page instantly.

### Level 2: Intermediate

1. Install Prettier and `eslint-config-prettier`.
2. Create a `.prettierrc` file using this config:

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2
}
```

3. Configure your editor for format-on-save.
4. Intentionally make formatting messy in `App.jsx`.
5. Save the file and verify Prettier reformats it.

### Level 3: Challenge

1. Add a path alias `@` that points to `src/`.
2. Create:
   - `src/components/Header.jsx`
   - `src/components/Footer.jsx`
3. Import them using the alias:

```jsx
import Header from '@/components/Header.jsx'
import Footer from '@/components/Footer.jsx'
```

4. Create a `.env` file with:

```env
VITE_APP_TITLE=30 Days of React
```

5. Display `import.meta.env.VITE_APP_TITLE` in your app.
6. Run `npm run build` and make sure the app still builds correctly.

