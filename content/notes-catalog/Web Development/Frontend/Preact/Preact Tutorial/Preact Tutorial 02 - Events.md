---
id: 01J64Z2TDHA0Q4CS89Z83CFVD4
title: Preact Tutorial 02 - Events
modified: 2024-08-25T10:41:11-04:00
description: How to create events in Preact
tags:
  - preact
  - programming
  - web-development
  - javascript
  - typescript
---
# Events

Events are how we make applications interactive, responding to inputs like keyboard and mouse, and reacting to changes like an image loading. Events work the same way in Preact as they do in the DOM – any event type or behavior you might find on [MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events) can be used in Preact. As an example, here's how event handlers are typically registered using the imperative DOM API:

```js
function clicked() {
  console.log('clicked')
}
const myButton = document.getElementById('my-button')
myButton.addEventListener('click', clicked)
```

Where Preact differs from the DOM API is how event handlers are registered. In Preact, event handlers are registered declaratively as props on an element, just like `style` and `class`. In general, any prop that has a name beginning with "on" is an event handler. The value of an event handler prop is the handler function to be called when that event occurs.

For example, we can listen for the "click" event on a button by adding an `onClick` prop with our handler function as its value:

```jsx
function clicked() {
  console.log('clicked')
}
<button onClick={clicked}>
```

Event handler names are case-sensitive, like all prop names. However, Preact will detect when you're registering a standard event type on an Element (click, change, touchmove, etc), and uses the correct case behind the scenes. That's why `<button onClick={..}>` works even though the event is `"click"` (lower case).

---

## Try it!

To complete this chapter, try adding your own click handler to the JSX for the button element on the right. In your handler, log a message using `console.log()` like we did above.

Once your code runs, click the button to call your event handler and move to the next chapter.

## Example
```jsx
import { render } from "preact";

function App() {
  const clicked = () => {
    console.log('hi')
  }

  return (
    <div>
      <p class="count">Count:</p>
      <button onClick={clicked}>Click Me!</button>
    </div>
  )
}

render(<App />, document.getElementById("app"));
```

