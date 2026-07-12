[<< Day 28](../28_Day_Testing/28_testing.md) | [Day 30 >>](../30_Day_Whats_Next/30_whats_next.md)

# Day 29 – Capstone Project: DevHub

## Table of Contents
- [Project Overview](#project-overview)
- [Feature List](#feature-list)
- [Tech Stack](#tech-stack)
- [Project Setup](#project-setup)
- [Folder Structure](#folder-structure)
- [Step 1: Set Up Routing](#step-1-set-up-routing)
- [Step 2: Add TanStack Query](#step-2-add-tanstack-query)
- [Step 3: GitHub API Integration](#step-3-github-api-integration)
- [Step 4: Build `RepoCard` and `RepoList`](#step-4-build-repocard-and-repolist)
- [Step 5: Add Search with React Hook Form + Zod](#step-5-add-search-with-react-hook-form--zod)
- [Step 6: Add Zustand Favorites with Persist](#step-6-add-zustand-favorites-with-persist)
- [Step 7: Build a Mock Login Form](#step-7-build-a-mock-login-form)
- [Step 8: Loading, Error, and Suspense States](#step-8-loading-error-and-suspense-states)
- [Step 9: Styling Approach](#step-9-styling-approach)
- [Step 10: Write Key Tests](#step-10-write-key-tests)
- [Stretch Goals](#stretch-goals)
- [Reflection Checklist](#reflection-checklist)
- [💻 Exercises](#-exercises)

## Project Overview

Welcome to the capstone.

You have spent 28 days learning:

- components
- props
- hooks
- routing
- forms
- async data
- Suspense
- global state
- testing

Today you combine them into one mini-application:

# **DevHub**

A developer resource aggregator where users can:

- browse GitHub repositories
- search by topic
- save favorites
- navigate multiple pages
- submit a mock login form

This is not a toy counter app. It is a realistic small product built from the same patterns used in modern React apps.

---

## Feature List

DevHub includes:

1. **Home page** – intro and featured repositories
2. **Search page** – search repositories by topic
3. **Favorites page** – saved repositories from Zustand + `localStorage`
4. **Login page** – mock authentication form with validation
5. **Shared UI** – cards, buttons, loading and error states
6. **Tests** – verify important components and behavior

---

## Tech Stack

- **Vite + React 19**
- **React Router v7** for routing
- **TanStack Query v5** for API data fetching
- **Zustand + persist** for favorites
- **React Hook Form + Zod** for forms and validation
- **CSS Modules or Tailwind** for styling
- **Vitest + RTL** for testing

This stack is an excellent 2026 baseline for modern SPA development.

---

## Project Setup

Create the project:

```bash
npm create vite@latest devhub -- --template react
cd devhub
npm install react-router @tanstack/react-query @tanstack/react-query-devtools zustand react-hook-form zod @hookform/resolvers axios
npm install -D vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom
```

### Recommended next steps

```bash
npm run dev
```

Then start organizing the project before writing features.

---

## Folder Structure

Use a feature-based structure:

```text
src/
├── components/      ← shared UI: Button, Card, Spinner, ErrorMessage
├── features/
│   ├── repos/       ← RepoList, RepoCard, useRepos hook, repos API
│   └── favorites/   ← FavoritesList, useFavoritesStore
├── pages/           ← Home, Search, Favorites, Login
├── lib/             ← queryClient, axiosInstance
├── hooks/           ← useDebounce
├── test/            ← setup.js
├── App.jsx
└── main.jsx
```

This structure keeps shared UI separate from domain features.

---

## Step 1: Set Up Routing

Install routing if needed:

```bash
npm install react-router
```

Create your pages:

```jsx
// src/App.jsx
import { Link, Outlet, createBrowserRouter, RouterProvider } from 'react-router'
import HomePage from './pages/HomePage'
import SearchPage from './pages/SearchPage'
import FavoritesPage from './pages/FavoritesPage'
import LoginPage from './pages/LoginPage'

function Layout() {
  return (
    <>
      <header>
        <nav>
          <Link to="/">Home</Link> |{' '}
          <Link to="/search">Search</Link> |{' '}
          <Link to="/favorites">Favorites</Link> |{' '}
          <Link to="/login">Login</Link>
        </nav>
      </header>

      <main>
        <Outlet />
      </main>
    </>
  )
}

const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      { index: true, element: <HomePage /> },
      { path: 'search', element: <SearchPage /> },
      { path: 'favorites', element: <FavoritesPage /> },
      { path: 'login', element: <LoginPage /> },
    ],
  },
])

export default function App() {
  return <RouterProvider router={router} />
}
```

Example pages:

```jsx
// src/pages/HomePage.jsx
export default function HomePage() {
  return (
    <section>
      <h1>DevHub</h1>
      <p>Discover and save useful developer repositories.</p>
    </section>
  )
}
```

```jsx
// src/pages/SearchPage.jsx
export default function SearchPage() {
  return <h1>Search Repositories</h1>
}
```

```jsx
// src/pages/FavoritesPage.jsx
export default function FavoritesPage() {
  return <h1>Your Favorites</h1>
}
```

```jsx
// src/pages/LoginPage.jsx
export default function LoginPage() {
  return <h1>Login</h1>
}
```

---

## Step 2: Add TanStack Query

Create a query client:

```jsx
// src/lib/queryClient.js
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
})
```

Wrap the app in `main.jsx`:

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import App from './App'
import { queryClient } from './lib/queryClient'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
)
```

Now the whole app can use `useQuery`.

---

## Step 3: GitHub API Integration

Create an Axios instance:

```jsx
// src/lib/axiosInstance.js
import axios from 'axios'

export const axiosInstance = axios.create({
  baseURL: 'https://api.github.com',
  headers: {
    Accept: 'application/vnd.github+json',
  },
})
```

Create an API helper:

```jsx
// src/features/repos/reposApi.js
import { axiosInstance } from '../../lib/axiosInstance'

export async function fetchReposByTopic(topic) {
  const normalizedTopic = topic?.trim() || 'react'

  const response = await axiosInstance.get('/search/repositories', {
    params: {
      q: `topic:${normalizedTopic}`,
      sort: 'stars',
      order: 'desc',
      per_page: 12,
    },
  })

  return response.data.items
}
```

Create a query hook:

```jsx
// src/features/repos/useRepos.js
import { useQuery } from '@tanstack/react-query'
import { fetchReposByTopic } from './reposApi'

export function useRepos(topic) {
  return useQuery({
    queryKey: ['repos', topic],
    queryFn: () => fetchReposByTopic(topic),
    enabled: Boolean(topic),
  })
}
```

### Why a custom hook?

It keeps page components cleaner and centralizes query behavior.

---

## Step 4: Build `RepoCard` and `RepoList`

Create a reusable card:

```jsx
// src/features/repos/RepoCard.jsx
import { useFavoritesStore } from '../favorites/useFavoritesStore'

export default function RepoCard({ repo }) {
  const favorites = useFavoritesStore(state => state.favorites)
  const toggleFavorite = useFavoritesStore(state => state.toggleFavorite)

  const isFavorite = favorites.some(item => item.id === repo.id)

  return (
    <article className="repo-card">
      <h3>
        <a href={repo.html_url} target="_blank" rel="noreferrer">
          {repo.name}
        </a>
      </h3>

      <p>{repo.description || 'No description available.'}</p>

      <div>
        <span>⭐ {repo.stargazers_count}</span>
        <span> · 🍴 {repo.forks_count}</span>
      </div>

      <button onClick={() => toggleFavorite(repo)}>
        {isFavorite ? 'Remove Favorite' : 'Save Favorite'}
      </button>
    </article>
  )
}
```

Create the list:

```jsx
// src/features/repos/RepoList.jsx
import RepoCard from './RepoCard'

export default function RepoList({ repos }) {
  if (!repos.length) {
    return <p>No repositories found.</p>
  }

  return (
    <section className="repo-grid">
      {repos.map(repo => (
        <RepoCard key={repo.id} repo={repo} />
      ))}
    </section>
  )
}
```

Use it on the home page:

```jsx
// src/pages/HomePage.jsx
import RepoList from '../features/repos/RepoList'
import { useRepos } from '../features/repos/useRepos'

export default function HomePage() {
  const { data = [], isLoading, isError, error } = useRepos('react')

  if (isLoading) return <p>Loading featured repositories...</p>
  if (isError) return <p>{error.message}</p>

  return (
    <section>
      <h1>DevHub</h1>
      <p>Discover and save useful developer repositories.</p>
      <RepoList repos={data.slice(0, 6)} />
    </section>
  )
}
```

---

## Step 5: Add Search with React Hook Form + Zod

Define validation:

```jsx
// src/features/repos/searchSchema.js
import { z } from 'zod'

export const searchSchema = z.object({
  topic: z
    .string()
    .trim()
    .min(2, 'Topic must be at least 2 characters'),
})
```

Build the form:

```jsx
// src/features/repos/SearchForm.jsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { searchSchema } from './searchSchema'

export default function SearchForm({ onSearch, defaultTopic = 'react' }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    resolver: zodResolver(searchSchema),
    defaultValues: {
      topic: defaultTopic,
    },
  })

  return (
    <form onSubmit={handleSubmit(onSearch)}>
      <label htmlFor="topic">Search by topic</label>
      <input id="topic" {...register('topic')} placeholder="e.g. react, vite, testing" />
      {errors.topic && <p role="alert">{errors.topic.message}</p>}

      <button type="submit" disabled={isSubmitting}>
        Search
      </button>
    </form>
  )
}
```

Use it in the page:

```jsx
// src/pages/SearchPage.jsx
import { useState } from 'react'
import SearchForm from '../features/repos/SearchForm'
import RepoList from '../features/repos/RepoList'
import { useRepos } from '../features/repos/useRepos'

export default function SearchPage() {
  const [topic, setTopic] = useState('react')
  const { data = [], isLoading, isError, error } = useRepos(topic)

  return (
    <section>
      <h1>Search Repositories</h1>

      <SearchForm onSearch={({ topic }) => setTopic(topic)} defaultTopic={topic} />

      {isLoading && <p>Loading repositories...</p>}
      {isError && <p>{error.message}</p>}
      {!isLoading && !isError && <RepoList repos={data} />}
    </section>
  )
}
```

---

## Step 6: Add Zustand Favorites with Persist

Create the store:

```jsx
// src/features/favorites/useFavoritesStore.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useFavoritesStore = create(
  persist(
    (set, get) => ({
      favorites: [],

      addFavorite: repo =>
        set(state => ({
          favorites: [...state.favorites, repo],
        })),

      removeFavorite: id =>
        set(state => ({
          favorites: state.favorites.filter(repo => repo.id !== id),
        })),

      toggleFavorite: repo => {
        const exists = get().favorites.some(item => item.id === repo.id)

        if (exists) {
          get().removeFavorite(repo.id)
          return
        }

        get().addFavorite(repo)
      },
    }),
    {
      name: 'devhub-favorites',
    }
  )
)
```

Favorites page:

```jsx
// src/pages/FavoritesPage.jsx
import RepoList from '../features/repos/RepoList'
import { useFavoritesStore } from '../features/favorites/useFavoritesStore'

export default function FavoritesPage() {
  const favorites = useFavoritesStore(state => state.favorites)

  return (
    <section>
      <h1>Your Favorites</h1>
      <RepoList repos={favorites} />
    </section>
  )
}
```

Now favorites survive refreshes.

---

## Step 7: Build a Mock Login Form

Create a schema:

```jsx
// src/pages/loginSchema.js
import { z } from 'zod'

export const loginSchema = z.object({
  email: z.string().email('Enter a valid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
})
```

Create the page:

```jsx
// src/pages/LoginPage.jsx
import { useState } from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { loginSchema } from './loginSchema'

export default function LoginPage() {
  const [submittedData, setSubmittedData] = useState(null)

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  })

  const onSubmit = async data => {
    await new Promise(resolve => setTimeout(resolve, 500))
    setSubmittedData(data)
  }

  return (
    <section>
      <h1>Login</h1>

      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label htmlFor="email">Email</label>
          <input id="email" type="email" {...register('email')} />
          {errors.email && <p role="alert">{errors.email.message}</p>}
        </div>

        <div>
          <label htmlFor="password">Password</label>
          <input id="password" type="password" {...register('password')} />
          {errors.password && <p role="alert">{errors.password.message}</p>}
        </div>

        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Logging in...' : 'Log in'}
        </button>
      </form>

      {submittedData && (
        <p>
          Welcome back, {submittedData.email}!
        </p>
      )}
    </section>
  )
}
```

This is mock auth, but it still teaches real validation and user experience patterns.

---

## Step 8: Loading, Error, and Suspense States

Every async app needs polished states.

### Loading UI

```jsx
// src/components/Spinner.jsx
export default function Spinner() {
  return <p>Loading...</p>
}
```

### Error UI

```jsx
// src/components/ErrorMessage.jsx
export default function ErrorMessage({ message = 'Something went wrong.' }) {
  return <p role="alert">{message}</p>
}
```

Use them:

```jsx
import ErrorMessage from '../components/ErrorMessage'
import Spinner from '../components/Spinner'

if (isLoading) return <Spinner />
if (isError) return <ErrorMessage message={error.message} />
```

### Suspense note

TanStack Query can integrate with Suspense, but you do not need to force it into this project. Manual loading and error states are perfectly fine for a capstone.

If you want to experiment:

```jsx
const { data } = useQuery({
  queryKey: ['repos', topic],
  queryFn: () => fetchReposByTopic(topic),
  suspense: true,
})
```

Then wrap the page area:

```jsx
import { Suspense } from 'react'
import Spinner from '../components/Spinner'

<Suspense fallback={<Spinner />}>
  <SearchResults />
</Suspense>
```

---

## Step 9: Styling Approach

You can choose CSS Modules or Tailwind. The important part is consistent organization.

### Example with CSS Modules

```css
/* src/features/repos/RepoCard.module.css */
.card {
  border: 1px solid #ddd;
  border-radius: 12px;
  padding: 1rem;
  background: white;
}

.actions {
  margin-top: 1rem;
}
```

```jsx
import styles from './RepoCard.module.css'

export default function RepoCard({ repo }) {
  return (
    <article className={styles.card}>
      <h3>{repo.name}</h3>
      <div className={styles.actions}>
        <button>Save Favorite</button>
      </div>
    </article>
  )
}
```

Styling is not the main lesson today, but the project should still feel intentional.

---

## Step 10: Write Key Tests

Start with the most important pieces.

### Example test for `RepoCard`

```jsx
// src/features/repos/RepoCard.test.jsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import RepoCard from './RepoCard'

const mockRepo = {
  id: 1,
  name: 'react',
  html_url: 'https://github.com/facebook/react',
  description: 'The library for web and native user interfaces',
  stargazers_count: 235000,
  forks_count: 48000,
}

it('renders repository details', () => {
  render(<RepoCard repo={mockRepo} />)

  expect(screen.getByText('react')).toBeInTheDocument()
  expect(
    screen.getByText(/the library for web and native user interfaces/i)
  ).toBeInTheDocument()
})

it('toggles favorite state when clicked', async () => {
  const user = userEvent.setup()

  render(<RepoCard repo={mockRepo} />)

  const button = screen.getByRole('button', { name: /save favorite/i })
  await user.click(button)

  expect(
    screen.getByRole('button', { name: /remove favorite/i })
  ).toBeInTheDocument()
})
```

### What else to test

- search form validation
- favorites page empty state
- login form validation and success flow
- repository loading and error states

---

## Stretch Goals

When the main app works, extend it:

- dark mode
- infinite scroll
- repository detail page
- sorting by stars or forks
- GitHub OAuth
- optimistic favorite toggling
- recent searches persisted to `localStorage`

Stretch goals are where the project becomes yours.

---

## Reflection Checklist

Before you finish, ask yourself:

- Did I use React Router correctly?
- Did I separate server state from client state?
- Did I validate forms with RHF + Zod?
- Did I persist favorites with Zustand?
- Did I handle loading and error states well?
- Did I write tests for important behaviors?
- Did I organize files clearly?
- Did I keep components reasonably small and focused?

This project should use concepts from nearly every part of the course.

---

## 💻 Exercises

### Level 1 – Complete Steps 1–4

Build:

1. routing
2. query client setup
3. GitHub API integration
4. repository cards and lists

Minimum success criteria:

- navigation works
- home page shows repositories
- code is organized into `features/repos`

### Level 2 – Complete Steps 5–8

Add:

1. validated search form
2. Zustand favorites
3. mock login form
4. polished loading/error states

Minimum success criteria:

- user can search by topic
- favorites persist after refresh
- login form validates and shows success UI

### Level 3 – Add Stretch Goals + Comprehensive Tests

Choose at least two stretch goals and write tests for:

- `RepoCard`
- the search form
- the login form
- favorites behavior

Bonus challenge:

Deploy the project and share it as part of your React portfolio.
