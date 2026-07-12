[<< Day 16](../16_Day_Styling_in_React/16_styling_in_react.md) | [Day 18 >>](../18_Day_Data_Fetching/18_data_fetching.md)

# Day 17 – React Router v7

## Table of Contents

- [What client-side routing means](#what-client-side-routing-means)
- [Installing React Router v7](#installing-react-router-v7)
- [Basic setup](#basic-setup)
- [Defining routes](#defining-routes)
- [Navigation](#navigation)
- [URL parameters](#url-parameters)
- [Query strings](#query-strings)
- [Nested routes and layouts](#nested-routes-and-layouts)
- [React Router v7 data APIs](#react-router-v7-data-apis)
- [Protected routes](#protected-routes)
- [`useLocation`](#uselocation)
- [Best practices](#best-practices)
- [💻 Exercises](#-exercises)

## What client-side routing means

In a traditional multi-page app, clicking a link asks the server for a completely new HTML page.

In a **single-page application (SPA)**, the browser loads the app shell once, and React updates the UI when the URL changes. That gives users:

- faster navigation
- smoother transitions
- less full-page flicker

Client-side routing means:

- the URL still changes
- browser history still works
- the page does **not** fully reload for internal navigation

React Router is the standard routing library for React apps.

## Installing React Router v7

Install the package:

```bash
npm install react-router
```

After installation, React Router provides components and hooks for matching routes, reading URL data, and navigating programmatically.

## Basic setup

Wrap your app with `BrowserRouter` in `main.jsx`.

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router'
import App from './App.jsx'
import './index.css'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
)
```

`BrowserRouter` listens to the browser's address bar and history stack.

## Defining routes

At the app level, define route-to-component mappings with `Routes` and `Route`.

```jsx
// src/App.jsx
import { Route, Routes } from 'react-router'

function Home() {
  return <h1>Home</h1>
}

function About() {
  return <h1>About</h1>
}

function UserProfile() {
  return <h1>User profile</h1>
}

function NotFound() {
  return <h1>404 - Page not found</h1>
}

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/users/:id" element={<UserProfile />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  )
}
```

Important route patterns:

- `/` → homepage
- `/about` → static route
- `/users/:id` → dynamic route parameter
- `*` → catch-all fallback

## Navigation

### `<Link>`

Use `Link` instead of `<a>` for internal navigation.

```jsx
import { Link } from 'react-router'

export default function Header() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
    </nav>
  )
}
```

`Link` updates the URL without a full page reload.

### `<NavLink>`

`NavLink` is useful for navigation bars because it can style the active link.

```jsx
import { NavLink } from 'react-router'

export default function Sidebar() {
  return (
    <nav>
      <NavLink to="/" className={({ isActive }) => (isActive ? 'active-link' : 'nav-link')}>
        Home
      </NavLink>
      <NavLink to="/about" className={({ isActive }) => (isActive ? 'active-link' : 'nav-link')}>
        About
      </NavLink>
    </nav>
  )
}
```

```css
.nav-link {
  color: #475569;
}

.active-link {
  color: #2563eb;
  font-weight: 700;
}
```

### `useNavigate`

`useNavigate` lets you change routes in code.

```jsx
import { useNavigate } from 'react-router'

export default function LoginSuccessButton() {
  const navigate = useNavigate()

  function handleContinue() {
    navigate('/dashboard')
  }

  return <button onClick={handleContinue}>Go to dashboard</button>
}
```

You can also navigate backward:

```jsx
navigate(-1)
```

That behaves like clicking the browser back button.

## URL parameters

Dynamic segments are read with `useParams()`.

```jsx
import { useParams } from 'react-router'

export default function UserProfile() {
  const { id } = useParams()

  return <h1>Viewing user {id}</h1>
}
```

If the URL is `/users/42`, then `id` will be `'42'`.

This is commonly used for:

- product pages
- user profiles
- blog posts

## Query strings

Query strings are useful for filters, search, sorting, and pagination.

```jsx
import { useSearchParams } from 'react-router'

export default function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams()
  const search = searchParams.get('search') || ''
  const page = Number(searchParams.get('page') || 1)

  function handleSearchChange(event) {
    const nextSearch = event.target.value

    setSearchParams({
      search: nextSearch,
      page: '1',
    })
  }

  return (
    <section>
      <input value={search} onChange={handleSearchChange} placeholder="Search articles" />
      <p>
        Search: <strong>{search || 'none'}</strong>
      </p>
      <p>Page: {page}</p>
    </section>
  )
}
```

For a URL like `?search=react&page=2`, you can read and update values without manually parsing strings.

## Nested routes and layouts

Large apps often share layout UI across multiple pages. Nested routes solve this cleanly.

```jsx
import { Outlet, Route, Routes } from 'react-router'

function DashboardLayout() {
  return (
    <div>
      <header>Dashboard Header</header>
      <nav>Sidebar links</nav>
      <main>
        <Outlet />
      </main>
    </div>
  )
}

function DashboardHome() {
  return <h2>Dashboard home</h2>
}

function Settings() {
  return <h2>Settings</h2>
}

function Profile() {
  return <h2>Profile</h2>
}

export default function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />
        <Route path="settings" element={<Settings />} />
        <Route path="profile" element={<Profile />} />
      </Route>
    </Routes>
  )
}
```

`<Outlet />` renders the matched child route inside the layout.

This pattern is excellent for:

- dashboards
- settings sections
- account areas

## React Router v7 data APIs

React Router also supports route-level data loading. This is especially useful when you want to fetch data before rendering a route.

```jsx
// src/routes/users.jsx
import { useLoaderData } from 'react-router'

export async function usersLoader() {
  const users = await fetch('/api/users').then(response => response.json())
  return { users }
}

export default function Users() {
  const { users } = useLoaderData()

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

```jsx
// route definition example
<Route path="/users" element={<Users />} loader={usersLoader} />
```

You can think of a **loader** as route-owned fetch logic.

React Router also supports **actions** for handling form submissions and mutations in framework-oriented routing setups.

Conceptually:

- **loader** → read data
- **action** → write data

## Protected routes

If a route should only be accessible to authenticated users, wrap it in a guard.

```jsx
import { Navigate } from 'react-router'

function useAuth() {
  return { isAuthenticated: false }
}

function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth()

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />
  }

  return children
}
```

Use it like this:

```jsx
<Route
  path="/admin"
  element={
    <ProtectedRoute>
      <AdminDashboard />
    </ProtectedRoute>
  }
/>
```

If the user is not logged in, they are redirected to `/login`.

## `useLocation`

`useLocation` gives you the current location object.

```jsx
import { useLocation, useNavigate } from 'react-router'

export default function CheckoutButton() {
  const navigate = useNavigate()
  const location = useLocation()

  function handleCheckout() {
    navigate('/login', {
      state: { from: location.pathname },
    })
  }

  return <button onClick={handleCheckout}>Checkout</button>
}
```

In the login page:

```jsx
import { useLocation } from 'react-router'

export default function LoginPage() {
  const location = useLocation()
  const from = location.state?.from || '/'

  return <p>After login, go back to: {from}</p>
}
```

This is useful for:

- redirect-after-login flows
- modal routing state
- analytics

## Best practices

- Use `Link` and `NavLink` for internal app links
- Keep route components focused on route-level concerns
- Use nested layouts to avoid duplicated page chrome
- Store filters and pagination in query strings when users should be able to share the URL
- Use loaders when route-level data ownership makes sense

## 💻 Exercises

### Level 1 — Three-page app

Build a mini app with:

- Home page
- About page
- Contact page
- a shared navigation bar using `Link`
- a `NotFound` route

### Level 2 — Dashboard with nested routes

Create a dashboard area with:

- `/dashboard`
- `/dashboard/settings`
- `/dashboard/profile`
- a layout containing a sidebar and `<Outlet />`

Use `NavLink` so the current page is highlighted.

### Level 3 — Blog with protected admin

Build a blog app with:

- `/posts`
- `/posts/:slug`
- `/admin`
- `/login`

Requirements:

- dynamic post pages using `useParams`
- search and pagination using `useSearchParams`
- a protected admin route using `Navigate`
- redirect back to the intended page after login using `useLocation`
