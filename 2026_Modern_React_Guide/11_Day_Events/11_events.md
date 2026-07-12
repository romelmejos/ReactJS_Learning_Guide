[<< Day 10](../10_Day_useEffect/10_use_effect.md) | [Day 12 >>](../12_Day_Forms_Controlled_Inputs/12_forms_controlled_inputs.md)

# Day 11 – Events

## Table of Contents

- [Why events matter in React](#why-events-matter-in-react)
- [React Synthetic Events](#react-synthetic-events)
- [Attaching event handlers](#attaching-event-handlers)
- [The event object](#the-event-object)
- [Common event types](#common-event-types)
- [Preventing default behavior](#preventing-default-behavior)
- [Stopping propagation](#stopping-propagation)
- [Passing arguments to handlers](#passing-arguments-to-handlers)
- [Keyboard events and accessibility](#keyboard-events-and-accessibility)
- [Event delegation in React 19](#event-delegation-in-react-19)
- [Synthetic vs native events](#synthetic-vs-native-events)
- [Custom events pattern with callback props](#custom-events-pattern-with-callback-props)
- [Complete example: interactive task list](#complete-example-interactive-task-list)
- [💻 Exercises](#-exercises)

## Why events matter in React

React apps become interactive when your UI responds to user actions like clicks, typing, focus, submission, and keyboard shortcuts. In React, you usually describe **what should happen** when the user interacts with the interface, then connect that logic with event handlers.

For today, you can copy the Vite boilerplate and run it:

```bash
cp -r ../boilerplate ../day-11-events
cd ../day-11-events
npm install
npm run dev
```

## React Synthetic Events

React does not hand you the raw browser event directly in most component handlers. Instead, it wraps native DOM events in a cross-browser object called **`SyntheticEvent`**.

Why this is useful:

- Same API across browsers
- Familiar properties like `target`, `currentTarget`, and `type`
- Consistent behavior across React-supported environments

In modern React, you usually do not need to think much about the wrapper. Just know that React is normalizing the browser differences for you.

React 19 also continues React’s modern event system, where event delegation is attached to the **root container** instead of `document`. That makes React better isolated when multiple apps or non-React code share the same page.

## Attaching event handlers

In JSX, event names use **camelCase**.

```jsx
function App() {
  function handleClick() {
    console.log('Button clicked')
  }

  return <button onClick={handleClick}>Click me</button>
}
```

Pass the function reference, not the result of calling it:

```jsx
<button onClick={handleClick}>Correct</button>
<button onClick={handleClick()}>Incorrect</button>
```

`onClick={handleClick()}` runs immediately during rendering, which is usually a bug.

## The event object

When React calls your handler, it passes an event object.

```jsx
function EventInspector() {
  function handleClick(event) {
    console.log('target:', event.target)
    console.log('currentTarget:', event.currentTarget)
    console.log('type:', event.type)
  }

  return (
    <button className="inspect-button" onClick={handleClick}>
      Inspect event
    </button>
  )
}
```

Important properties:

- `event.target` → the element that triggered the event
- `event.currentTarget` → the element the handler is attached to
- `event.type` → the event name, such as `click` or `submit`

## Common event types

Here are the ones you will use often:

### Mouse events

- `onClick`
- `onDoubleClick`
- `onMouseEnter`
- `onMouseLeave`
- `onMouseMove`

### Keyboard events

- `onKeyDown`
- `onKeyUp`
- `onKeyPress` is deprecated; prefer `onKeyDown`

### Form events

- `onChange`
- `onSubmit`
- `onFocus`
- `onBlur`

### Clipboard events

- `onCopy`
- `onPaste`

### Touch events

React also supports touch events like `onTouchStart` and `onTouchEnd`. You will use them less often on desktop-first apps, but they matter for mobile interactions.

This demo shows several event types together:

```jsx
import { useState } from 'react'

export default function App() {
  const [message, setMessage] = useState('Interact with the card')

  return (
    <div className="event-demo">
      <input
        placeholder="Type here"
        onFocus={() => setMessage('Input focused')}
        onBlur={() => setMessage('Input blurred')}
        onKeyDown={(e) => setMessage(`Key down: ${e.key}`)}
        onChange={(e) => setMessage(`Value: ${e.target.value}`)}
      />

      <div
        className="hover-box"
        onMouseEnter={() => setMessage('Mouse entered')}
        onMouseLeave={() => setMessage('Mouse left')}
        onMouseMove={() => setMessage('Mouse moving')}
      >
        Hover over me
      </div>

      <p>{message}</p>
    </div>
  )
}
```

```css
.event-demo {
  display: grid;
  gap: 1rem;
  max-width: 420px;
}

.hover-box {
  padding: 1rem;
  border: 2px dashed #646cff;
  border-radius: 12px;
  background: #f5f7ff;
}
```

## Preventing default behavior

Some elements have built-in browser behavior:

- links navigate
- forms submit and reload the page
- checkboxes toggle

If you want to override that default behavior, call `event.preventDefault()`.

```jsx
function PreventDefaultExamples() {
  function handleSubmit(event) {
    event.preventDefault()
    console.log('Form submitted without page refresh')
  }

  return (
    <>
      <a href="#" onClick={(e) => e.preventDefault()}>
        Don&apos;t navigate
      </a>

      <form onSubmit={handleSubmit}>
        <input name="email" placeholder="Email" />
        <button type="submit">Submit</button>
      </form>
    </>
  )
}
```

## Stopping propagation

Events usually **bubble** upward through the DOM tree. If you click a button inside a card, the button’s click handler runs first, then the parent card’s click handler can run too.

Sometimes that is useful. Sometimes it is annoying.

Use `event.stopPropagation()` when you want to stop the bubbling behavior.

```jsx
function PropagationDemo() {
  function handleCardClick() {
    console.log('Card clicked')
  }

  function handleButtonClick(event) {
    event.stopPropagation()
    console.log('Inner button clicked only')
  }

  return (
    <div className="card" onClick={handleCardClick}>
      <p>Click anywhere in the card.</p>
      <button onClick={handleButtonClick}>Delete</button>
    </div>
  )
}
```

## Passing arguments to handlers

Often, a handler needs an ID or value.

Wrap the handler in an arrow function:

```jsx
function ItemList({ items }) {
  function handleDelete(id) {
    console.log('Delete item', id)
  }

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.label}
          <button onClick={() => handleDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  )
}
```

This is the standard pattern. If you later optimize a large list, `useCallback` can help keep handler references stable.

```js
// Good rule of thumb:
// First make it correct.
// Then optimize only if you measure a real performance problem.
```

## Keyboard events and accessibility

Native buttons already support keyboard interaction. If you build a custom clickable element like a `div`, you must restore keyboard access yourself.

```jsx
function AccessibleCardButton() {
  function handleClick() {
    console.log('Activated')
  }

  function handleKeyDown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault()
      handleClick()
    }
  }

  return (
    <div
      role="button"
      tabIndex={0}
      className="fake-button"
      onClick={handleClick}
      onKeyDown={handleKeyDown}
    >
      Open menu
    </div>
  )
}
```

Best practice: use a real `<button>` whenever possible. Build custom keyboard behavior only when you truly need a non-button element.

## Event delegation in React 19

React uses event delegation internally. Instead of attaching one event listener to every button, input, or list item, React listens higher up and figures out which component should respond.

What this means for you:

- You usually attach handlers directly in JSX
- You do **not** need to manually implement delegation for normal React components
- In React 19, the delegated listeners are associated with the React root, not `document`

That root-based model helps React coexist better with other scripts and multiple embedded React apps.

## Synthetic vs native events

Most of the time, React event props are enough. But some browser APIs live outside React’s event system, such as:

- `window` resize
- `document` visibility changes
- manual `keydown` listeners on `window`

For those, use `addEventListener` inside `useEffect` and remove the listener during cleanup.

```jsx
import { useEffect, useState } from 'react'

function WindowSize() {
  const [size, setSize] = useState({
    w: window.innerWidth,
    h: window.innerHeight,
  })

  useEffect(() => {
    const handleResize = () => {
      setSize({
        w: window.innerWidth,
        h: window.innerHeight,
      })
    }

    window.addEventListener('resize', handleResize)

    return () => {
      window.removeEventListener('resize', handleResize)
    }
  }, [])

  return (
    <p>
      Width: {size.w}, Height: {size.h}
    </p>
  )
}
```

## Custom events pattern with callback props

React components usually communicate from child to parent with **callback props**.

The parent passes a function down. The child calls it when something happens.

```jsx
function AddTaskForm({ onAddTask }) {
  function handleSubmit(event) {
    event.preventDefault()
    const formData = new FormData(event.currentTarget)
    const title = formData.get('title')?.trim()

    if (!title) return

    onAddTask(title)
    event.currentTarget.reset()
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" placeholder="New task" />
      <button type="submit">Add task</button>
    </form>
  )
}

function TaskApp() {
  function handleAddTask(title) {
    console.log('Parent received:', title)
  }

  return <AddTaskForm onAddTask={handleAddTask} />
}
```

This is not a browser custom event. It is the React pattern for child-to-parent communication.

## Complete example: interactive task list

This mini app combines clicks, keyboard access, submit handling, propagation control, and callback props.

```jsx
import { useState } from 'react'

function TaskInput({ onAddTask }) {
  function handleSubmit(event) {
    event.preventDefault()
    const formData = new FormData(event.currentTarget)
    const title = formData.get('title')?.trim()

    if (!title) return

    onAddTask(title)
    event.currentTarget.reset()
  }

  return (
    <form className="task-form" onSubmit={handleSubmit}>
      <input name="title" placeholder="Add a task" />
      <button type="submit">Add</button>
    </form>
  )
}

function TaskItem({ task, onToggle, onDelete }) {
  function handleKeyDown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault()
      onToggle(task.id)
    }
  }

  return (
    <li className="task-item" onClick={() => onToggle(task.id)}>
      <div
        role="button"
        tabIndex={0}
        className={`task-label ${task.done ? 'done' : ''}`}
        onKeyDown={handleKeyDown}
      >
        {task.title}
      </div>

      <button
        className="delete-button"
        onClick={(event) => {
          event.stopPropagation()
          onDelete(task.id)
        }}
      >
        Delete
      </button>
    </li>
  )
}

export default function App() {
  const [tasks, setTasks] = useState([
    { id: 1, title: 'Learn React events', done: false },
    { id: 2, title: 'Practice keyboard access', done: false },
  ])

  function addTask(title) {
    setTasks((prev) => [
      ...prev,
      { id: crypto.randomUUID(), title, done: false },
    ])
  }

  function toggleTask(id) {
    setTasks((prev) =>
      prev.map((task) =>
        task.id === id ? { ...task, done: !task.done } : task
      )
    )
  }

  function deleteTask(id) {
    setTasks((prev) => prev.filter((task) => task.id !== id))
  }

  return (
    <main className="app-shell">
      <h1>Event Playground</h1>
      <TaskInput onAddTask={addTask} />

      <ul className="task-list">
        {tasks.map((task) => (
          <TaskItem
            key={task.id}
            task={task}
            onToggle={toggleTask}
            onDelete={deleteTask}
          />
        ))}
      </ul>
    </main>
  )
}
```

```css
.app-shell {
  max-width: 640px;
  margin: 0 auto;
  padding: 2rem;
}

.task-form {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.task-form input {
  flex: 1;
}

.task-list {
  list-style: none;
  padding: 0;
  display: grid;
  gap: 0.75rem;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #d9ddf7;
  border-radius: 14px;
  background: white;
}

.task-label {
  cursor: pointer;
}

.task-label.done {
  text-decoration: line-through;
  opacity: 0.65;
}

.delete-button {
  background: #e11d48;
  color: white;
}
```

## 💻 Exercises

### Level 1

Build a button with a click counter.

Requirements:

- Show how many times the button was clicked
- Add a second button to reset the count
- Log `event.type` in the click handler

### Level 2

Build a keyboard-accessible dropdown menu.

Requirements:

- A button opens and closes the menu
- `Escape` closes the menu
- `Enter` and space should activate menu items
- Clicking outside should close the menu

### Level 3

Build a drag-and-drop style reorderable list using mouse events.

Requirements:

- Track the dragged item in state
- Use mouse events to update the current hover target
- Reorder the list on drop
- Add clear visual feedback while dragging

