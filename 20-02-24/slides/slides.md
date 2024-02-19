---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
transition: slide-left
title: 20-02-24
mdc: true
---

<style global>
  .slidev-layout ol {
    padding-left: 0.5rem;
  }

  .slidev-code {
    --slidev-code-font-size: 8px;
    --slidev-code-line-height: 10px;

    --prism-font-size: 0.5rem;
    --prism-line-height: 0.5rem;
  }
</style>

# An introduction to React

The dominant JavaScript framework

---
transition: fade-out
---

# What is React?

A JavaScript library for building user interfaces.

React, created and maintained by Facebook, is a JavaScript library for building user interfaces.
React makes use of the Virtual DOM and ASTs ("Abstract Syntax Tree") in an attempt to target which areas of the view need to be updated based on a state change.

This allows developers to create interactive UIs by using JSX (HTML-like syntax "JavaScript XML") to manage state and lifecycle events in segmented snippets of code called "Components".

---

# Components

There are two types of React Component: Class Component and Function Component.

When creating a React Component in the 2010s ("Classic" React era) developers would have to create a "Class Component" which extended a base Component class from React. This provided access to a series of methods and properties which a developer could use to dynamically change what was rendered on the screen based on the Component's lifecycle.

In 2019 React released the concept of "Hooks" which completely changed the landscape and how developers would think about Components – this was coined "Function Components". Instead of extending anything, a Function Component calls hooks (which are normal JavaScript functions) that trigger during different stages of the Component's lifecycle.

Overall Function Components allow for a more concise and modular code structure making it easier to manage component logic and reuse code.

---
layout: center
class: text-center
transition: fade-out
---

# Component Examples

Let's look at some code examples

---
layout: two-cols
---

# Class Component

Let's set up a basic component to fetch data for a specific user identifier.

1. Extend `React.Component`
2. Provide `props` as first argument
3. Component set up with `this.state`
4. `componentDidMount`: On mount, fetch user data, begin interval
5. `componentDidUnmount`: On unmount clean up interval
6. `componentDidUpdate`: On prop update, fetch user data
7. `render`: Render user data

::right::

```js {all|1,3|4-5,10|7-9|9,12-18,28-35|9,20-22|24-35|37-39|all}
import React from 'react'

export class ClassComponent extends React.Component {
  constructor (props) {
    super(props)

    this.state = { name: null }

    this.interval = React.createRef()
  }

  componentDidMount () {
    this.fetchData(this.props.userId)

    this.interval.current = setInterval(() => (
      this.fetchData(this.props.userId)
    ), 5_000)
  }

  componentDidUnmount () {
    clearInterval(this.interval.current)
  }

  componentDidUpdate (prevProps, prevState, snapshot) {
    if (this.props.userId !== prevProps.userId) {
      this.fetchData(this.props.userId);
    }
  }

  async fetchData(userId) {
    const res = await fetch(`/user/${userId}`)
    const { name } = await res.json()

    this.setState({ name })
  }

  render () {
    return <p>{this.state.name}</p>
  }
}
```

---
layout: two-cols
---

# Function Component

Now let's try and set up fetching user data as a function component.

1. Initialize a normal JavaScript function
2. Provide `props` as first argument
3. Comoponent set up with `useState`
4. `useEffect`: On mount, fetch user data, begin interval
5. `useEffect`: On unmount clean up interval
6. `useEffect`: On prop update, fetch user data
7. Render user data

::right::

```js {all|3|1,4|1,5,7-12,14-19,24|1,14,21-23,24|1,14,24|26-28|all}
import React from 'react'

export function FunctionComponent (props) {
  const [state, setState] = React.useState({ name: null })
  const interval = React.useRef(null)

  const fetchData = async (userId) => {
    const res = await fetch(`/user/${userId}`)
    const { name } = await res.json()

    setState({ name })
  }

  React.useEffect(() => {
    fetchData(props.userId)

    interval.current = setInterval(() => (
      fetchData(props.userId)
    ), 5_000)

    return () => {
      clearInterval(interval.current)
    }
  }, [props.userId])

  return (
    <p>{state.name}</p>
  )
}
```

---
layout: two-cols
---

# Using Components

It's almost like using HTML!

In the example to the right we have our top level ("Page") rendering a Profile component providing the user identifier of `john.doe`. The Page doesn't need to care about what Profile is going to do with `userId`.

A component can render another component, at the top level of any website there needs to be pages that render smaller functional areas of the page.

Attributes on a component ("Props") generally should always use camel case, this is blatant when trying to add a DOM event handler such as `click` to a `button` – it must be called `onClick`.

Components are given a special prop `children` which can be used to pass through content. We'll see this in use later.

::right::

```js {all|1-8|1-2,9-17|18-23|18-19,25-33|all}
// ~> /components/profile.jsx
export function Profile ({ userId }) {
  // ~> Do something with the user identifier (e.g. load profile data)

  return (
    <p>{userId}</p>
  )
}

// ~> /pages/profile.jsx
// ~> <p>john.doe</p>
export default function Page () {
  return (
    <Profile userId="john.doe" />
  )
}

// ~> /components/wrapper.jsx
export function Wrapper ({ children }) {
  return (
    <div>{children}</div>
  )
}

// ~> /pages/profile-b.jsx
// ~> <div><p>Jane Foo</p></div>
export default function Page () {
  return (
    <Wrapper>
      <p>Jane Foo</p>
    </Wrapper>
  )
}
```

---
layout: center
class: text-center
transition: fade-out
---

React Components: Questions?

---
layout: center
class: text-center
transition: fade-out
---

# Hooks

Now we know how to use components, let's take a look at hooks

---
layout: two-cols
---

# Built-in Hooks

React offers a series of different hooks which can be used to build your own custom hooks.

- `useState`: keep track of a variable's value across renders [updating causes a re-render]
- `useRef`: keep track of a variable across renders [not causing a re-render]
- `useEffect`: do something on load or when a dependency changes
- `useMemo`: cache a value of an expensive operation
- `useCallback`: cache a function definition

::right::

```js {all|1,4,27|1,5,14,17,28-30|1,13-23|1,4,7|1,9-11,13,14,19}
import React from 'react'

export function MyComponent {
  const [count, setCount] = React.useState(0)
  const interval = React.useRef(null)

  const doubled = React.useMemo(() => count * 2, [count])

  const increment = React.useCallback(() => {
    setCount(count => count + 1)
  }, [setState])

  React.useEffect(() => {
    interval.current = setInterval(increment, 1_000)

    return () => {
      clearInterval(interval.current)
    }
  }, [increment])

  React.useEffect(() => {
    console.debug('MyComponent:count %s', count)
  }, [count])

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => clearInterval(interval.current)}>
        Stop incrementing count
      </button>
    </>
  )
}
```

---
layout: two-cols
---

# useFetchUserData

We've moved the logic of loading a user's data out into a separate file, which exports a function `useFetchUserData`.

A general rule of thumb when programming is to limit code duplication, hooks allow us to pull out logic into reuseable JavaScript functions that can be used within Function Components.

Now the logic is separate we can include our custom hook in many components without copying across the logic.

::right::

```js {all|1,3-28|4,27|3-4,30-37|3-4,39-46|all}
import React from 'react'

// ~> /hooks/use-fetch-user-data.js
export function useFetchUserData (userId) {
  const [state, setState] = React.useState({ name: null })
  const interval = React.useRef(null)

  const fetchData = async (userId) => {
    const res = await fetch(`/user/${userId}`)
    const { name } = await res.json()

    setState({ name })
  }

  React.useEffect(() => {
    fetchData(userId)

    interval.current = setInterval(() => (
      fetchData(userId)
    ), 5_000)

    return () => {
      clearInterval(interval.current)
    }
  }, [userId])

  return state
}

// ~> /components/current-user.jsx
export function CurrentUser (props) {
  const state = useFetchUserData(props.userId)

  return (
    <p>{state.name}</p>
  )
}

// ~> /components/settings.jsx
export function Settings (props) {
  const state = useFetchUserData(props.userId)

  return (
    <p>{state.name}</p>
  )
}
```

---
layout: two-cols
---

# useUser

Whenever we want to use `useFetchUserData` we must re-fetch the user's information wherever we use it.

`CurrentUser` and `Settings` each make a request to our `/user/:id` endpoint – this could get expensive, jittery, and non-performant.

How can we solve this? `React.createContext` and `React.useContext`.

::right::

```js {all|1,3-14|1,3-4,16-21|3,6-14,23-35|all}
import React from 'react'

// ~> /providers/current-user.jsx
export const CurrentUserContext = React.createContext(null)

export function CurrentUserProvider ({ userId, children }) {
  const user = useFetchUserData(userId)

  return (
    <CurrentUserContext.Provider value={user}>
      {children}
    </CurrentUserContext.Provider>
  )
}

// ~> /hooks/use-current-user.js
export function useCurrentUser () {
  const user = React.useContext(CurrentUserContext)

  return user
}

// ~> /pages/settings.jsx
export default function Page () {
  return (
    <CurrentUserProvider userId="john.doe">
      <nav>
        <CurrentUser />
      </nav>
      <main>
        <Settings />
      </main>
    </CurrentUserProvider>
  )
}
```

---
layout: center
class: text-center
transition: fade-out
---

React Hooks: Questions?

---
layout: center
class: text-center
transition: fade-out
---

# Coding challenge

If we've got time, let's work through a challenge together

---
layout: center
class: text-center
transition: fade-out
---

Questions?
