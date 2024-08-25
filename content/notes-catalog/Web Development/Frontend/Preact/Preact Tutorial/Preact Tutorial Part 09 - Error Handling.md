---
id: 01J652EGB4HX3YXVRSCN3E6CGM
modified: 2024-08-25T11:42:40-04:00
title: Preact Tutorial Part 09 - Error Handling
description: How to handle errors in preact
tags:
  - preact
  - programming
  - web-development
  - typescript
  - javascript
---
# Error Handling

JavaScript is a flexible interpreted language, which means it's possible (and even easy) to encounter errors at runtime. Whether the result of an unexpected scenario or a mistake in code we've written, it's important to be able to monitor errors and implement some form of recovery or graceful error handling.

In Preact, the way we do this is to capture errors and save them as state. This lets a component intercept an unexpected or broken render and switch to rendering something different as a fallback.

### Turning errors into state

Two APIs are available for capturing errors and turning them into state: `componentDidCatch` and `getDerivedStateFromError`. They're functionally similar, and both are methods you can implement on a class component:

**componentDidCatch** gets passed an `Error` argument, and can decide what to do in response to that Error on a case-by-case basis. It can call `this.setState()` to render a fallback or alternative tree, which will "catch" the error and mark it as handled. Or, the method could simply log the error somewhere and allow it to continue unhandled (to crash).

**getDerivedStateFromError** is a static method that gets passed an `Error`, and returns a state update object, which is applied to the component via `setState()`. Since this method always produces a state change that results in its component being re-rendered, it always marks errors as handled.

The following example shows how to use either method to capture errors and show a graceful error message instead of crashing:

```jsx
import { Component } from 'preact'

class ErrorBoundary extends Component {
  state = { error: null }

  static getDerivedStateFromError(error) {
    return { error: error.message }
  }

  componentDidCatch(error) {
    console.error(error)
    this.setState({ error: error.message })
  }

  render() {
    if (this.state.error) {
      return <p>Oh no! We ran into an error: {this.state.error}</p>
    }
    return this.props.children
  }
}
```

The component above is a relatively common example of how error handling is implemented in Preact applications, often referred to as an _Error Boundary_.

### Nesting and error bubbling

Errors encountered when Preact is rendering your Virtual DOM tree "bubble up", much like DOM events. Starting from the component that encountered the error, each parent component in the tree is given an opportunity to handle the error.

As a result, Error Boundaries can be nested if implemented using `componentDidCatch`. When a component's `componentDidCatch()` method _doesn't_ call `setState()`, the error will continue to bubble up the Virtual DOM tree until it reaches an component with a `componentDidCatch` method that _does_ call `setState()`.

## Try it!

To test our error handling knowledge, let's add error handling to a simple App component. One of the components deep within App can throw an error in some scenario, and we want to catch this so we can show a friendly message telling the user that we've run into an unexpected error.

## Example
```jsx
import { render, Component } from 'preact';
import { useState } from 'preact/hooks';

function Clicker() {
  const [clicked, setClicked] = useState(false);

  if (clicked) {
    throw new Error('I am erroring');
  }

  return <button onClick={() => setClicked(true)}>Click Me</button>;
}

class App extends Component {
  state = { error: null };

  componentDidCatch(error) {
    this.setState({ error: error.message });
  }

  render() {
    const { error } = this.state;
    if (error) {
      return <p>Oh no! There was an error: {error}</p>
    }
    return <Clicker />;
  }
}

render(<App />, document.getElementById("app"));
```

- 🍅 (pomodoro::WORK) (duration:: 40m) (begin:: 2024-08-25 11:02) - (end:: 2024-08-25 11:42)