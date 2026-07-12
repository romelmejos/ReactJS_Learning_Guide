[<< Day 12](../12_Day_Forms_Controlled_Inputs/12_forms_controlled_inputs.md) | [Day 14 >>](../14_Day_Validation_React_Hook_Form/14_validation_react_hook_form.md)

# Day 13 – React 19 Actions

## Table of Contents

- [Why React 19 Actions matter](#why-react-19-actions-matter)
- [What are React 19 Actions](#what-are-react-19-actions)
- [The action prop on form](#the-action-prop-on-form)
- [useActionState](#useactionstate)
- [useFormStatus](#useformstatus)
- [useOptimistic](#useoptimistic)
- [Progressive enhancement](#progressive-enhancement)
- [useActionState vs useState plus useEffect](#useactionstate-vs-usestate-plus-useeffect)
- [Error handling in Actions](#error-handling-in-actions)
- [Complete example: comment form with optimistic UI](#complete-example-comment-form-with-optimistic-ui)
- [💻 Exercises](#-exercises)

## Why React 19 Actions matter

Before React 19, form workflows often required a mix of:

- `useState` for values
- `useState` for loading
- `useState` for errors
- `useEffect` for side effects after submission

That worked, but it was noisy. React 19 introduces **Actions**, which make form submissions feel more like a first-class React feature.

```bash
cp -r ../boilerplate ../day-13-actions
cd ../day-13-actions
npm install
npm run dev
```

## What are React 19 Actions

An Action is a function React can call during a form submission or related async flow. React helps manage:

- pending state
- error state
- optimistic UI
- form-driven async updates

The biggest idea is simple:

- instead of manually wiring everything to `onSubmit`
- you can give the form an `action`

## The action prop on form

The `action` prop can take an async function. React will pass a `FormData` object to it.

```jsx
async function createUser(formData) {
  const name = formData.get('name')
  await saveToDatabase({ name })
}

function UserForm() {
  return (
    <form action={createUser}>
      <input name="name" />
      <button type="submit">Create</button>
    </form>
  )
}
```

This is a major mental shift:

- browser form semantics still exist
- React now participates more deeply in the submission lifecycle

Here is a runnable local example:

```jsx
function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

async function createUser(formData) {
  const name = formData.get('name')
  await wait(1000)
  console.log(`Saved user: ${name}`)
}

export default function App() {
  return (
    <form action={createUser}>
      <input name="name" placeholder="Enter a name" />
      <button type="submit">Create</button>
    </form>
  )
}
```

```js
export function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}
```

## useActionState

`useActionState` lets you keep submission state together with the action result.

```jsx
import { useActionState } from 'react'

async function submitAction(prevState, formData) {
  const name = formData.get('name')

  if (!name) {
    return { error: 'Name is required' }
  }

  await saveUser({ name })
  return { success: true }
}

function UserForm() {
  const [state, formAction, isPending] = useActionState(submitAction, null)

  return (
    <form action={formAction}>
      <input name="name" required />
      {state?.error && <p className="error">{state.error}</p>}
      {state?.success && <p className="success">User created!</p>}
      <button disabled={isPending}>
        {isPending ? 'Saving...' : 'Create User'}
      </button>
    </form>
  )
}
```

The action receives:

1. `prevState`
2. `formData`

That means you can treat form submission like a reducer-style state transition.

Here is a complete version:

```jsx
import { useActionState } from 'react'

function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

async function submitAction(prevState, formData) {
  const name = formData.get('name')?.trim()

  if (!name) {
    return { error: 'Name is required', success: false }
  }

  await wait(1200)

  return {
    error: null,
    success: true,
    savedName: name,
  }
}

export default function App() {
  const [state, formAction, isPending] = useActionState(submitAction, null)

  return (
    <main className="page">
      <h1>Create User</h1>

      <form className="card" action={formAction}>
        <label htmlFor="name">Name</label>
        <input id="name" name="name" placeholder="Ava" />

        {state?.error && <p className="error">{state.error}</p>}
        {state?.success && (
          <p className="success">Created user: {state.savedName}</p>
        )}

        <button disabled={isPending}>
          {isPending ? 'Saving...' : 'Create User'}
        </button>
      </form>
    </main>
  )
}
```

## useFormStatus

`useFormStatus` lets nested components know whether the current form is submitting.

It must be used inside a form tree.

```jsx
import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  )
}
```

This is especially helpful when your submit button is extracted into its own component.

```jsx
import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" className="submit-button" disabled={pending}>
      {pending ? 'Sending...' : 'Send Message'}
    </button>
  )
}

async function sendMessage(formData) {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  console.log(formData.get('message'))
}

export default function App() {
  return (
    <form action={sendMessage}>
      <textarea name="message" placeholder="Write a message" />
      <SubmitButton />
    </form>
  )
}
```

## useOptimistic

`useOptimistic` is for optimistic UI. You temporarily show the future result before the server or async task fully completes.

```jsx
import { useOptimistic } from 'react'

const [optimisticMessages, addOptimisticMessage] = useOptimistic(
  messages,
  (state, newMessage) => [...state, { ...newMessage, sending: true }]
)
```

Why this feels good:

- the app responds immediately
- users do not feel blocked by network delays
- the UI can later confirm or rollback

## Progressive enhancement

One of the most exciting parts of Actions is that they align well with normal HTML forms.

In frameworks like Next.js or Remix, forms can often still work in a more basic way without client-side JavaScript, especially when wired to server actions or route actions.

That is called **progressive enhancement**:

- the basic HTML behavior works first
- JavaScript upgrades the experience

This is different from older patterns that required JavaScript for every form interaction.

## useActionState vs useState plus useEffect

Use `useActionState` when:

- your async flow is primarily form submission
- you want pending, success, and error state close to the form
- the action result naturally becomes the next UI state

Use manual `useState` when:

- the flow is not form-driven
- you need custom orchestration across multiple interactions
- the logic does not fit the action pattern well

```js
// Rule of thumb:
// If the user submits a form and you want the result of that submission
// to directly drive the UI, Actions are often the cleanest choice.
```

## Error handling in Actions

There are two common approaches:

### Return error state

Great for expected validation errors.

```jsx
async function submitAction(prevState, formData) {
  const email = formData.get('email')

  if (!email.includes('@')) {
    return { error: 'Please enter a valid email' }
  }

  return { success: true }
}
```

### Throw an error

Better for unexpected failures, especially when an Error Boundary should handle them.

```jsx
async function submitAction(prevState, formData) {
  const response = await fetch('/api/save', { method: 'POST', body: formData })

  if (!response.ok) {
    throw new Error('Server failed to save the form')
  }

  return { success: true }
}
```

Think of it like this:

- expected user mistakes → return error state
- unexpected app failures → throw

## Complete example: comment form with optimistic UI

This example combines `useActionState`, `useFormStatus`, and `useOptimistic`.

```jsx
import { useActionState, useOptimistic, useState } from 'react'
import { useFormStatus } from 'react-dom'

function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

function SubmitButton() {
  const { pending } = useFormStatus()

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Posting...' : 'Post comment'}
    </button>
  )
}

export default function App() {
  const [comments, setComments] = useState([
    { id: 1, text: 'React 19 feels much cleaner for forms.', sending: false },
  ])

  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (state, newComment) => [...state, { ...newComment, sending: true }]
  )

  async function submitComment(prevState, formData) {
    const text = formData.get('text')?.trim()

    if (!text) {
      return { error: 'Comment cannot be empty' }
    }

    const optimisticComment = {
      id: crypto.randomUUID(),
      text,
    }

    addOptimisticComment(optimisticComment)
    await wait(1200)

    setComments((prev) => [
      ...prev,
      { ...optimisticComment, sending: false },
    ])

    return { error: null, success: true }
  }

  const [state, formAction] = useActionState(submitComment, {
    error: null,
    success: false,
  })

  return (
    <main className="page">
      <h1>Comments</h1>

      <form className="comment-form" action={formAction}>
        <label htmlFor="text">Add comment</label>
        <textarea id="text" name="text" rows="4" placeholder="Share your thoughts" />

        {state.error && <p className="error">{state.error}</p>}

        <SubmitButton />
      </form>

      <ul className="comment-list">
        {optimisticComments.map((comment) => (
          <li key={comment.id} className="comment-card">
            <p>{comment.text}</p>
            {comment.sending && <small>Sending...</small>}
          </li>
        ))}
      </ul>
    </main>
  )
}
```

```css
.page {
  max-width: 680px;
  margin: 0 auto;
  padding: 2rem;
}

.comment-form,
.comment-list {
  display: grid;
  gap: 1rem;
}

.comment-card {
  padding: 1rem;
  border: 1px solid #d9def8;
  border-radius: 14px;
  background: #fff;
}

.error {
  color: #b42318;
}
```

## 💻 Exercises

### Level 1

Convert a controlled form to use the `action` prop.

Requirements:

- remove `onSubmit`
- read values from `FormData`
- keep the UI behavior the same

### Level 2

Build a form with `useActionState` and pending state.

Requirements:

- validate at least 2 fields
- show success and error messages
- disable the submit button while pending

### Level 3

Implement optimistic updates for a comment system.

Requirements:

- add the new comment immediately
- mark optimistic items as sending
- replace them with confirmed items after the async call completes
- handle at least one failure scenario
