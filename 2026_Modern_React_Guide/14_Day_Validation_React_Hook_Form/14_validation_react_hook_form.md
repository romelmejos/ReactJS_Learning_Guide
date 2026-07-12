[<< Day 13](../13_Day_React19_Actions/13_react19_actions.md) | [Day 15 >>](../15_Day_Project_Folder_Structure/15_project_folder_structure.md)

# Day 14 – Validation with React Hook Form

## Table of Contents

- [Why use a form library](#why-use-a-form-library)
- [Installing React Hook Form](#installing-react-hook-form)
- [Basic React Hook Form usage](#basic-react-hook-form-usage)
- [Built-in validation rules](#built-in-validation-rules)
- [Understanding formState](#understanding-formstate)
- [watch for live form logic](#watch-for-live-form-logic)
- [setValue and reset](#setvalue-and-reset)
- [Installing Zod and resolvers](#installing-zod-and-resolvers)
- [Schema validation with Zod](#schema-validation-with-zod)
- [Advanced Zod patterns](#advanced-zod-patterns)
- [Controller for custom components](#controller-for-custom-components)
- [Performance benefits](#performance-benefits)
- [Complete example: signup form with RHF and Zod](#complete-example-signup-form-with-rhf-and-zod)
- [💻 Exercises](#-exercises)

## Why use a form library

Manual form state is great for learning, but large forms become repetitive fast:

- lots of `useState`
- lots of validation code
- lots of error wiring
- repeated submit-state handling

**React Hook Form (RHF)** is popular because it is:

- simple to adopt
- fast
- ergonomic
- based on uncontrolled inputs under the hood, which reduces unnecessary re-renders

That last point matters. RHF typically updates only the pieces that need to change instead of re-rendering the entire form on every keystroke.

## Installing React Hook Form

```bash
npm install react-hook-form
```

If you are starting from the shared boilerplate:

```bash
cp -r ../boilerplate ../day-14-rhf
cd ../day-14-rhf
npm install
npm install react-hook-form
npm run dev
```

## Basic React Hook Form usage

The `useForm` hook gives you the tools you need.

```jsx
import { useForm } from 'react-hook-form'

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm()

  const onSubmit = (data) => console.log(data)

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /\S+@\S+\.\S+/,
            message: 'Invalid email',
          },
        })}
      />

      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit">Login</button>
    </form>
  )
}
```

Notice what changed:

- no `value` prop
- no manual `onChange`
- `register()` connects the input to RHF

## Built-in validation rules

RHF supports common validation rules directly:

- `required`
- `min`
- `max`
- `minLength`
- `maxLength`
- `pattern`
- `validate`

Example:

```jsx
<input
  {...register('username', {
    required: 'Username is required',
    minLength: {
      value: 3,
      message: 'Username must be at least 3 characters',
    },
    maxLength: {
      value: 20,
      message: 'Username must be at most 20 characters',
    },
    validate: (value) =>
      !value.includes(' ') || 'Username cannot contain spaces',
  })}
/>
```

## Understanding formState

`formState` exposes important metadata:

- `errors`
- `isSubmitting`
- `isValid`
- `isDirty`
- `dirtyFields`
- `touchedFields`

```jsx
const {
  register,
  handleSubmit,
  formState: { errors, isSubmitting, isValid, isDirty, touchedFields },
} = useForm({
  mode: 'onChange',
})
```

These flags let you build smarter UI:

- disable submit until valid
- show save banners only after edits
- style fields differently after they are touched

## watch for live form logic

`watch` lets you observe field values as the user edits.

```jsx
import { useForm } from 'react-hook-form'

function PasswordForm() {
  const { register, watch } = useForm()
  const password = watch('password', '')

  return (
    <>
      <input type="password" {...register('password')} />
      <p>Password length: {password.length}</p>
    </>
  )
}
```

This is useful for:

- password strength meters
- conditional fields
- live previews

## setValue and reset

Use `setValue` to update a field programmatically, and `reset` to return the form to a fresh state.

```jsx
import { useForm } from 'react-hook-form'

function ProfileForm() {
  const { register, setValue, reset } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
    },
  })

  return (
    <form>
      <input {...register('firstName')} />
      <input {...register('lastName')} />

      <button
        type="button"
        onClick={() => setValue('firstName', 'Ada')}
      >
        Fill sample
      </button>

      <button
        type="button"
        onClick={() => reset()}
      >
        Reset
      </button>
    </form>
  )
}
```

## Installing Zod and resolvers

For more powerful validation, combine RHF with Zod.

```bash
npm install zod @hookform/resolvers
```

Zod lets you declare rules in one schema and reuse them.

## Schema validation with Zod

```jsx
import { useForm } from 'react-hook-form'
import { z } from 'zod'
import { zodResolver } from '@hookform/resolvers/zod'

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  age: z.number({ coerce: true }).min(18, 'Must be 18+'),
})

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(schema),
  })

  const onSubmit = (data) => console.log(data)

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <p>{errors.email.message}</p>}

      <input type="password" {...register('password')} />
      {errors.password && <p>{errors.password.message}</p>}

      <input type="number" {...register('age')} />
      {errors.age && <p>{errors.age.message}</p>}

      <button type="submit">Sign up</button>
    </form>
  )
}
```

The `coerce: true` option is important because browser number inputs still produce strings.

## Advanced Zod patterns

Zod scales well because it can express complex rules.

### Optional and nullable

```jsx
const schema = z.object({
  nickname: z.string().optional(),
  bio: z.string().nullable(),
})
```

### Cross-field validation with refine

```jsx
const signupSchema = z
  .object({
    password: z.string().min(8),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords do not match',
    path: ['confirmPassword'],
  })
```

### Transform values

```jsx
const schema = z.object({
  username: z.string().transform((value) => value.trim().toLowerCase()),
})
```

## Controller for custom components

Some UI libraries do not expose a normal `ref` or use custom value APIs. That is where `Controller` helps.

```jsx
import { Controller, useForm } from 'react-hook-form'

function ColorPicker({ value, onChange }) {
  return (
    <div className="color-row">
      {['red', 'green', 'blue'].map((color) => (
        <button
          key={color}
          type="button"
          className={value === color ? 'selected' : ''}
          onClick={() => onChange(color)}
        >
          {color}
        </button>
      ))}
    </div>
  )
}

function ThemeForm() {
  const { control, handleSubmit } = useForm({
    defaultValues: {
      theme: 'red',
    },
  })

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <Controller
        name="theme"
        control={control}
        render={({ field }) => (
          <ColorPicker value={field.value} onChange={field.onChange} />
        )}
      />

      <button type="submit">Save theme</button>
    </form>
  )
}
```

## Performance benefits

RHF is designed to minimize re-renders.

With a fully controlled form, every keystroke often causes the entire component to render again. RHF keeps more work closer to the input itself.

This matters more when:

- forms have many fields
- fields are nested deeply
- expensive UI updates sit beside the form

Use React DevTools to compare render frequency between manual controlled forms and RHF-powered forms.

```js
// Fewer unnecessary re-renders usually means better performance
// and a smoother typing experience in larger forms.
```

## Complete example: signup form with RHF and Zod

```jsx
import { useForm } from 'react-hook-form'
import { z } from 'zod'
import { zodResolver } from '@hookform/resolvers/zod'

const signupSchema = z
  .object({
    fullName: z.string().min(2, 'Full name must be at least 2 characters'),
    email: z.string().email('Enter a valid email'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
    confirmPassword: z.string(),
    age: z.number({ coerce: true }).min(18, 'You must be at least 18'),
    marketing: z.boolean().optional(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords do not match',
    path: ['confirmPassword'],
  })

export default function App() {
  const {
    register,
    handleSubmit,
    reset,
    watch,
    formState: { errors, isSubmitting, isValid, isDirty },
  } = useForm({
    resolver: zodResolver(signupSchema),
    mode: 'onChange',
    defaultValues: {
      fullName: '',
      email: '',
      password: '',
      confirmPassword: '',
      age: 18,
      marketing: false,
    },
  })

  const password = watch('password', '')

  async function onSubmit(data) {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    console.log('Submitted data:', data)
    reset()
  }

  return (
    <main className="page">
      <h1>Sign up</h1>

      <form className="form-card" onSubmit={handleSubmit(onSubmit)} noValidate>
        <label htmlFor="fullName">Full name</label>
        <input id="fullName" {...register('fullName')} />
        {errors.fullName && <p className="error">{errors.fullName.message}</p>}

        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <p className="error">{errors.email.message}</p>}

        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...register('password')} />
        <p className="hint">Password strength: {password.length}/8 characters</p>
        {errors.password && <p className="error">{errors.password.message}</p>}

        <label htmlFor="confirmPassword">Confirm password</label>
        <input
          id="confirmPassword"
          type="password"
          {...register('confirmPassword')}
        />
        {errors.confirmPassword && (
          <p className="error">{errors.confirmPassword.message}</p>
        )}

        <label htmlFor="age">Age</label>
        <input id="age" type="number" {...register('age')} />
        {errors.age && <p className="error">{errors.age.message}</p>}

        <label className="checkbox-row">
          <input type="checkbox" {...register('marketing')} />
          Send me product updates
        </label>

        <div className="actions">
          <button type="submit" disabled={!isDirty || !isValid || isSubmitting}>
            {isSubmitting ? 'Creating account...' : 'Create account'}
          </button>

          <button type="button" onClick={() => reset()}>
            Reset
          </button>
        </div>
      </form>
    </main>
  )
}
```

```css
.page {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
}

.form-card {
  display: grid;
  gap: 0.75rem;
}

.error {
  margin: -0.25rem 0 0;
  color: #b42318;
}

.hint {
  margin: -0.25rem 0 0;
  color: #475467;
  font-size: 0.95rem;
}

.checkbox-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.actions {
  display: flex;
  gap: 0.75rem;
}
```

## 💻 Exercises

### Level 1

Build a login form with RHF built-in validation.

Requirements:

- email and password fields
- email pattern validation
- password minimum length
- show errors below each field

### Level 2

Build a registration form with a Zod schema.

Requirements:

- full name, email, password, confirm password, age
- confirm password must match password
- age must be at least 18
- disable submit while invalid

### Level 3

Build a multi-step checkout form with RHF and Zod.

Requirements:

- shipping step
- billing step
- payment step
- preserve values between steps
- validate each step before allowing the next one

