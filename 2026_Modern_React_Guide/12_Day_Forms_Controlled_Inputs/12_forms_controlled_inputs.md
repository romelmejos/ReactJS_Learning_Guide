[<< Day 11](../11_Day_Events/11_events.md) | [Day 13 >>](../13_Day_React19_Actions/13_react19_actions.md)

# Day 12 – Forms: Controlled Inputs

## Table of Contents

- [Why forms feel different in React](#why-forms-feel-different-in-react)
- [Controlled vs uncontrolled components](#controlled-vs-uncontrolled-components)
- [Basic controlled input](#basic-controlled-input)
- [Handling all common input types](#handling-all-common-input-types)
- [Managing many fields with one state object](#managing-many-fields-with-one-state-object)
- [Form submission](#form-submission)
- [Basic validation strategies](#basic-validation-strategies)
- [Controlled vs uncontrolled trade-offs](#controlled-vs-uncontrolled-trade-offs)
- [Form accessibility](#form-accessibility)
- [Complete example: contact form](#complete-example-contact-form)
- [💻 Exercises](#-exercises)

## Why forms feel different in React

Plain HTML forms can work on their own, but React gives you a better way to control values, show validation instantly, and derive UI from state.

Today’s lesson focuses on **controlled inputs**, the classic React approach where React state is the single source of truth.

```bash
cp -r ../boilerplate ../day-12-forms
cd ../day-12-forms
npm install
npm run dev
```

## Controlled vs uncontrolled components

### Controlled

In a controlled input:

- React owns the value
- every keystroke updates state
- the input reads from state again on the next render

This makes validation, previews, conditional UI, and submit handling straightforward.

### Uncontrolled

In an uncontrolled input:

- the DOM stores the current value
- React does not update state on every keystroke
- you usually read the value later with a `ref` or `FormData`

For most beginner-friendly React forms, **controlled is preferred** because it is easier to reason about.

## Basic controlled input

```jsx
import { useState } from 'react'

function NameInput() {
  const [name, setName] = useState('')

  return (
    <div>
      <label htmlFor="name">Name</label>
      <input
        id="name"
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <p>Hello, {name || 'stranger'}!</p>
    </div>
  )
}
```

This is the most important form pattern in React:

1. state stores the value
2. `value` reads from state
3. `onChange` writes back to state

## Handling all common input types

Different form elements expose different properties.

### Text-like inputs

Use `value` and `onChange`.

```jsx
function AccountFields() {
  const [form, setForm] = useState({
    email: '',
    password: '',
    age: 18,
    website: '',
    phone: '',
  })

  function handleChange(e) {
    const { name, value } = e.target
    setForm((prev) => ({ ...prev, [name]: value }))
  }

  return (
    <>
      <input name="email" type="email" value={form.email} onChange={handleChange} />
      <input name="password" type="password" value={form.password} onChange={handleChange} />
      <input name="age" type="number" value={form.age} onChange={handleChange} />
      <input name="website" type="url" value={form.website} onChange={handleChange} />
      <input name="phone" type="tel" value={form.phone} onChange={handleChange} />
    </>
  )
}
```

### `<textarea>`

In React, a textarea is controlled with `value`, not children.

```jsx
<textarea
  name="message"
  value={form.message}
  onChange={handleChange}
/>
```

### `<select>`

Put the selected value on the `select`, not on each option.

```jsx
<select name="role" value={form.role} onChange={handleChange}>
  <option value="">Select a role</option>
  <option value="developer">Developer</option>
  <option value="designer">Designer</option>
  <option value="manager">Manager</option>
</select>
```

### Checkbox

Checkboxes use `checked`, not `value`.

```jsx
function NewsletterField() {
  const [subscribed, setSubscribed] = useState(false)

  return (
    <label>
      <input
        type="checkbox"
        checked={subscribed}
        onChange={(e) => setSubscribed(e.target.checked)}
      />
      Subscribe to newsletter
    </label>
  )
}
```

### Radio group

Radio buttons share a `name` and compare against a selected value.

```jsx
function PlanSelector() {
  const [plan, setPlan] = useState('basic')

  return (
    <fieldset>
      <legend>Select a plan</legend>

      <label>
        <input
          type="radio"
          name="plan"
          value="basic"
          checked={plan === 'basic'}
          onChange={(e) => setPlan(e.target.value)}
        />
        Basic
      </label>

      <label>
        <input
          type="radio"
          name="plan"
          value="pro"
          checked={plan === 'pro'}
          onChange={(e) => setPlan(e.target.value)}
        />
        Pro
      </label>
    </fieldset>
  )
}
```

## Managing many fields with one state object

For larger forms, using one state object is often cleaner than one `useState` per field.

```jsx
import { useState } from 'react'

function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: '',
  })

  const handleChange = (e) => {
    const { name, value } = e.target
    setForm((prev) => ({ ...prev, [name]: value }))
  }

  return (
    <>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <textarea name="message" value={form.message} onChange={handleChange} />
    </>
  )
}
```

Why use the callback form of `setForm`?

- it safely reads the latest previous state
- it avoids stale updates
- it is the best default when the next state depends on the previous one

## Form submission

Usually, you stop the browser refresh and handle the data in JavaScript.

```jsx
function SignupForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
  })

  function handleChange(e) {
    const { name, value } = e.target
    setForm((prev) => ({ ...prev, [name]: value }))
  }

  function handleSubmit(e) {
    e.preventDefault()
    console.log(form)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <button type="submit">Create account</button>
    </form>
  )
}
```

## Basic validation strategies

There are three beginner-friendly strategies:

### 1. Validate on submit

Simple and common.

### 2. Validate on blur

Show errors after the user leaves a field.

### 3. Disable submit when invalid

Useful when the rules are clear and visible.

Here is a practical example:

```jsx
import { useState } from 'react'

const initialForm = {
  name: '',
  email: '',
  message: '',
}

export default function App() {
  const [form, setForm] = useState(initialForm)
  const [errors, setErrors] = useState({})

  function validate(values) {
    const nextErrors = {}

    if (!values.name.trim()) nextErrors.name = 'Name is required'
    if (!values.email.includes('@')) nextErrors.email = 'Enter a valid email'
    if (values.message.trim().length < 10) {
      nextErrors.message = 'Message must be at least 10 characters'
    }

    return nextErrors
  }

  function handleChange(e) {
    const { name, value } = e.target
    setForm((prev) => ({ ...prev, [name]: value }))
  }

  function handleBlur(e) {
    const { name } = e.target
    const nextErrors = validate(form)
    setErrors((prev) => ({ ...prev, [name]: nextErrors[name] }))
  }

  function handleSubmit(e) {
    e.preventDefault()
    const nextErrors = validate(form)
    setErrors(nextErrors)

    if (Object.keys(nextErrors).length > 0) return

    console.log('Submitted:', form)
    setForm(initialForm)
    setErrors({})
  }

  const isInvalid = Object.keys(validate(form)).length > 0

  return (
    <form className="contact-form" onSubmit={handleSubmit} noValidate>
      <label htmlFor="name">Name</label>
      <input
        id="name"
        name="name"
        value={form.name}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-invalid={Boolean(errors.name)}
        aria-describedby={errors.name ? 'name-error' : undefined}
        required
      />
      {errors.name && (
        <p id="name-error" className="error">
          {errors.name}
        </p>
      )}

      <label htmlFor="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-invalid={Boolean(errors.email)}
        aria-describedby={errors.email ? 'email-error' : undefined}
        required
      />
      {errors.email && (
        <p id="email-error" className="error">
          {errors.email}
        </p>
      )}

      <label htmlFor="message">Message</label>
      <textarea
        id="message"
        name="message"
        value={form.message}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-invalid={Boolean(errors.message)}
        aria-describedby={errors.message ? 'message-error' : undefined}
        required
      />
      {errors.message && (
        <p id="message-error" className="error">
          {errors.message}
        </p>
      )}

      <button disabled={isInvalid}>Send message</button>
    </form>
  )
}
```

```css
.contact-form {
  max-width: 520px;
  display: grid;
  gap: 0.75rem;
}

.error {
  margin: -0.25rem 0 0;
  color: #b42318;
  font-size: 0.95rem;
}

input[aria-invalid='true'],
textarea[aria-invalid='true'] {
  border-color: #d92d20;
}
```

## Controlled vs uncontrolled trade-offs

Controlled inputs are excellent when you need:

- live validation
- conditional rendering
- previews
- derived UI
- disabling submit based on state

Uncontrolled inputs are useful when:

- you want simpler low-level form handling
- you want fewer re-renders
- you are working with a form library like React Hook Form

Important special case:

- `<input type="file">` must always be uncontrolled

```jsx
function AvatarUpload() {
  function handleSubmit(event) {
    event.preventDefault()
    const formData = new FormData(event.currentTarget)
    const file = formData.get('avatar')
    console.log(file)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" name="avatar" />
      <button type="submit">Upload</button>
    </form>
  )
}
```

## Form accessibility

Accessible forms are easier for everyone to use.

Always remember:

- use `<label htmlFor="...">`
- give inputs matching `id` values
- connect error text with `aria-describedby`
- set `aria-invalid` when a field is invalid
- use semantic controls like `button`, `fieldset`, and `legend`
- add `required` where appropriate

```js
// Accessible forms are not "extra credit".
// They are part of writing correct UI.
```

## Complete example: contact form

This version uses several input types in one realistic form.

```jsx
import { useState } from 'react'

const initialForm = {
  fullName: '',
  email: '',
  topic: 'support',
  budget: '1000',
  subscribe: true,
  contactMethod: 'email',
  message: '',
}

export default function App() {
  const [form, setForm] = useState(initialForm)

  function handleChange(e) {
    const { name, value, type, checked } = e.target
    setForm((prev) => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value,
    }))
  }

  function handleSubmit(e) {
    e.preventDefault()
    console.log('Form submitted:', form)
  }

  return (
    <main className="page">
      <h1>Project Inquiry Form</h1>

      <form className="form-card" onSubmit={handleSubmit}>
        <label htmlFor="fullName">Full name</label>
        <input
          id="fullName"
          name="fullName"
          value={form.fullName}
          onChange={handleChange}
          required
        />

        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={form.email}
          onChange={handleChange}
          required
        />

        <label htmlFor="topic">Topic</label>
        <select id="topic" name="topic" value={form.topic} onChange={handleChange}>
          <option value="support">Support</option>
          <option value="consulting">Consulting</option>
          <option value="training">Training</option>
        </select>

        <label htmlFor="budget">Estimated budget</label>
        <input
          id="budget"
          name="budget"
          type="number"
          value={form.budget}
          onChange={handleChange}
          min="0"
        />

        <fieldset>
          <legend>Preferred contact method</legend>

          <label>
            <input
              type="radio"
              name="contactMethod"
              value="email"
              checked={form.contactMethod === 'email'}
              onChange={handleChange}
            />
            Email
          </label>

          <label>
            <input
              type="radio"
              name="contactMethod"
              value="phone"
              checked={form.contactMethod === 'phone'}
              onChange={handleChange}
            />
            Phone
          </label>
        </fieldset>

        <label>
          <input
            type="checkbox"
            name="subscribe"
            checked={form.subscribe}
            onChange={handleChange}
          />
          Subscribe to updates
        </label>

        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          rows="5"
          value={form.message}
          onChange={handleChange}
        />

        <button type="submit">Send inquiry</button>
      </form>

      <section className="preview-card">
        <h2>Live preview</h2>
        <pre>{JSON.stringify(form, null, 2)}</pre>
      </section>
    </main>
  )
}
```

## 💻 Exercises

### Level 1

Build a login form with email and password.

Requirements:

- both inputs must be controlled
- prevent page refresh on submit
- show the typed email below the form

### Level 2

Build a registration form with validation.

Requirements:

- full name, email, password, confirm password
- validate on blur and on submit
- show inline error messages
- disable submit when invalid

### Level 3

Build a multi-step form wizard.

Requirements:

- split the form into at least 3 steps
- preserve state between steps
- show previous and next buttons
- display a final review screen before submit

