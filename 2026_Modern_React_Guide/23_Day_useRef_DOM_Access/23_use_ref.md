[<< Day 22](../22_Day_Custom_Hooks/22_custom_hooks.md) | [Day 24 >>](../24_Day_Performance_Optimization/24_performance_optimization.md)

# Day 23 – useRef and DOM Access

## Table of Contents

- [What is useRef](#what-is-useref)
- [Accessing DOM nodes](#accessing-dom-nodes)
- [Common DOM ref use cases](#common-dom-ref-use-cases)
- [Refs as mutable values](#refs-as-mutable-values)
- [Ref vs state](#ref-vs-state)
- [Refs in React 19](#refs-in-react-19)
- [useImperativeHandle](#useimperativehandle)
- [Callback refs](#callback-refs)
- [When not to use refs](#when-not-to-use-refs)
- [Complete example: media controller](#complete-example-media-controller)
- [💻 Exercises](#-exercises)

## What is useRef

`useRef` returns a mutable object:

```jsx
const ref = useRef(initialValue)
```

That object has one property:

```js
{
  current: initialValue
}
```

Two main use cases:

1. store a reference to a DOM node
2. store a mutable value that persists across renders without causing re-renders

Changing `ref.current` does **not** trigger a component update.

## Accessing DOM nodes

The most common ref example is focusing an input after mount.

```jsx
import { useEffect, useRef } from 'react'

function FocusInput() {
  const inputRef = useRef(null)

  useEffect(() => {
    inputRef.current.focus()
  }, [])

  return <input ref={inputRef} type="text" placeholder="Type here..." />
}
```

React assigns the actual input element to `inputRef.current` after rendering.

## Common DOM ref use cases

Refs are useful for imperative browser actions:

- auto-focus on mount
- scroll to an element
- measure an element
- control media playback
- integrate non-React libraries
- trigger animations

```jsx
function ScrollToDetails() {
  const detailsRef = useRef(null)

  return (
    <>
      <button onClick={() => detailsRef.current.scrollIntoView({ behavior: 'smooth' })}>
        Jump to details
      </button>

      <div style={{ height: '100vh' }} />

      <section ref={detailsRef}>
        <h2>Details</h2>
      </section>
    </>
  )
}
```

Measuring size:

```jsx
import { useEffect, useRef, useState } from 'react'

function MeasuredCard() {
  const cardRef = useRef(null)
  const [width, setWidth] = useState(0)

  useEffect(() => {
    const rect = cardRef.current.getBoundingClientRect()
    setWidth(Math.round(rect.width))
  }, [])

  return (
    <article ref={cardRef}>
      <p>Measured width: {width}px</p>
    </article>
  )
}
```

## Refs as mutable values

Refs can also store values that survive re-renders.

### Tracking a previous value

```jsx
import { useEffect, useRef, useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  const prevCountRef = useRef(0)

  useEffect(() => {
    prevCountRef.current = count
  }, [count])

  return (
    <>
      <p>Now: {count}</p>
      <p>Before: {prevCountRef.current}</p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </>
  )
}
```

### Storing interval IDs

```jsx
import { useEffect, useRef, useState } from 'react'

function Stopwatch() {
  const [seconds, setSeconds] = useState(0)
  const intervalRef = useRef(null)

  const start = () => {
    if (intervalRef.current) return

    intervalRef.current = setInterval(() => {
      setSeconds((current) => current + 1)
    }, 1000)
  }

  const stop = () => {
    clearInterval(intervalRef.current)
    intervalRef.current = null
  }

  useEffect(() => stop, [])

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  )
}
```

This is perfect for values that matter to logic but do not need to be rendered on each change.

## Ref vs state

| Ref | State |
| --- | --- |
| mutable | immutable update pattern |
| changing it does not re-render | changing it re-renders |
| great for DOM nodes and mutable values | great for values that affect UI |

Rule of thumb:

- use **state** when the screen must update
- use **ref** when the value should persist but should not trigger rendering

Bad ref usage:

```jsx
const countRef = useRef(0)
countRef.current += 1
return <p>{countRef.current}</p>
```

The UI will not update reliably because changing a ref does not cause a render.

## Refs in React 19

Older React patterns often used `forwardRef`. In React 19, refs can be received as a regular prop in function components.

```jsx
function FancyInput({ ref, ...props }) {
  return <input ref={ref} className="fancy-input" {...props} />
}

export default function App() {
  const myRef = useRef(null)

  return <FancyInput ref={myRef} placeholder="React 19 ref prop" />
}
```

This simplifies several component APIs. You may still see `forwardRef` in older tutorials and older codebases.

## useImperativeHandle

Sometimes a parent should not get the raw DOM node. Instead, it should get a limited public API.

```jsx
import { useImperativeHandle, useRef } from 'react'

function VideoPlayer({ ref, src }) {
  const internalRef = useRef(null)

  useImperativeHandle(ref, () => ({
    play: () => internalRef.current.play(),
    pause: () => internalRef.current.pause(),
    reset: () => {
      internalRef.current.pause()
      internalRef.current.currentTime = 0
    },
  }))

  return <video ref={internalRef} src={src} width="420" controls />
}
```

Parent usage:

```jsx
import { useImperativeHandle, useRef } from 'react'

function LessonPlayer() {
  const playerRef = useRef(null)

  return (
    <>
      <VideoPlayer ref={playerRef} src="/intro.mp4" />
      <button onClick={() => playerRef.current.play()}>Play</button>
      <button onClick={() => playerRef.current.pause()}>Pause</button>
      <button onClick={() => playerRef.current.reset()}>Reset</button>
    </>
  )
}
```

## Callback refs

A ref can also be a function.

```jsx
function Example() {
  const handleRef = (node) => {
    if (node) {
      console.log('Mounted node:', node)
    } else {
      console.log('Unmounted')
    }
  }

  return <div ref={handleRef}>Watch the console</div>
}
```

Callback refs are handy when:

- you need custom mount/unmount behavior
- you want to avoid storing the node in a ref object
- you integrate with measurement or observer APIs

## When not to use refs

Refs are an escape hatch, not the default tool.

Avoid refs when:

- you really need state
- you can solve the problem declaratively
- you are trying to manually force React-like behavior

Prefer:

- state for UI data
- props for data flow
- effects for syncing with external systems

Use refs for imperative browser tasks or mutable non-visual values.

## Complete example: media controller

```jsx
import { useRef } from 'react'

function AudioPlayer({ ref, src }) {
  const audioRef = useRef(null)

  useImperativeHandle(ref, () => ({
    play() {
      audioRef.current.play()
    },
    pause() {
      audioRef.current.pause()
    },
    jumpTo(seconds) {
      audioRef.current.currentTime = seconds
    },
  }))

  return <audio ref={audioRef} src={src} />
}

export default function PodcastControls() {
  const playerRef = useRef(null)

  return (
    <section>
      <AudioPlayer ref={playerRef} src="/podcast.mp3" />
      <button onClick={() => playerRef.current.play()}>Play</button>
      <button onClick={() => playerRef.current.pause()}>Pause</button>
      <button onClick={() => playerRef.current.jumpTo(30)}>Jump to 0:30</button>
    </section>
  )
}
```

This is a strong example of refs used well: imperative control of a browser media element.

## 💻 Exercises

### Level 1 – Autofocus and video controls

- Build an input that focuses itself on mount
- Create a video player with Play and Pause buttons using a ref

### Level 2 – Infinite scroll with IntersectionObserver

- Create a `loadMoreRef`
- Observe when the sentinel enters the viewport
- Load the next page of items when it becomes visible

### Level 3 – Drag-and-drop sortable list

- Use refs to track positions while dragging
- Reorder items based on pointer movement
- Keep rendered order in state, but use refs for live drag measurements
