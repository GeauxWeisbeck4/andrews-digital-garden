---
id: 01J651A0105DT845NH5F79AAYF
title: Preact Tutorial Part 05 - Refs
modified: 2024-08-25T11:15:29-04:00
description: How to use refs in preact
tags:
  - preact
  - web-development
  - programming
  - javascript
  - typescript
---
# Refs

As we learned in the first chapter, the DOM provides an imperative API, which lets us make changes by calling functions on elements. One example where we might need to access the imperative DOM API from a Preact component would be to automatically move focus to an input element.

The `autoFocus` prop (or `autofocus` attribute) can be used to focus an input the first time it is rendered, however there are situations where we want to move focus to an input at a specific time, or in response to a specific event.

For these cases where we need to interact directly with DOM elements, we can use a feature called "refs". A ref is a plain JavaScript object with a `current` property that point to any value. JavaScript objects are passed by reference, which means that any function with access to a ref object can get or set its value using the `current` property. Preact does not track changes to ref objects, so they can be used to store information during rendering, which can then be accessed later by any function with access to the ref object.

We can see what direct usage of the ref feature looks like without rendering anything:

```js
import { createRef } from 'preact'

// create a ref:
const ref = createRef('initial value')
// { current: 'initial value' }

// read a ref's current value:
ref.current === 'initial value'

// update a ref's current value:
ref.current = 'new value'

// pass refs around:
console.log(ref) // { current: 'new value' }
```

What makes refs useful in Preact is that a ref object can be passed to a Virtual DOM element during rendering, and Preact will set the ref's value (its `current` property) to the corresponding HTML element. Once set, we can use the ref's current value to access and modify the HTML element:

```jsx
import { createRef } from 'preact';

// create a ref:
const input = createRef()

// pass the ref as a prop on a Virtual DOM element:
render(<input ref={input} />, document.body)

// access the associated DOM element:
input.current // an HTML <input> element
input.current.focus() // focus the input!
```

Using `createRef()` globally isn't recommended, since multiple renders will overwrite the ref's current value. Instead, it's best to store refs as class properties:

```jsx
import { createRef, Component } from 'preact';

export default class App extends Component {
  input = createRef()

  // this function runs after <App> is rendered
  componentDidMount() {
    // access the associated DOM element:
    this.input.current.focus();
  }

  render() {
    return <input ref={this.input} />
  }
}
```

For function components, a `useRef()` hook provides a convenient way to create a ref and access that same ref on subsequent renders. The following example also shows how to use the `useEffect()` hook to invoke a callback after our component is rendered, in which our ref's current value will then be set to the HTML input element:

```jsx
import { useRef, useEffect } from 'preact/hooks';

export default function App() {
  // create or retrieve our ref:  (hook slot 0)
  const input = useRef()

  // the callback here will run after <App> is rendered:
  useEffect(() => {
    // access the associated DOM element:
    input.current.focus()
  }, [])

  return <input ref={input} />
}
```

Remember, refs aren't limited to storing only DOM elements. They can be used to store information between renders of a component without setting state that would cause additional rendering. We'll see some uses for that in a later chapter.

## Try it!

Now let's put this to practice by creating a button that, when clicked, focuses an input field by accessing it using a ref.

```jsx
import { render } from 'preact';
import { useRef } from 'preact/hooks';

function App() {
  const input = useRef()

  function onClick() {
    input.current.focus()
  }

  return (
    <div>
      <input ref={input} defaultValue="Hello Momma!" />
      <button onClick={onClick}>Focus input</button>
    </div>
  );
}

render(<App />, document.getElementById("app"));
```