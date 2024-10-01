---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Declarative Rendering
modified: 2024-10-01T13:18:40-04:00
description: The main feature of Vue.js - declarative rendering
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
# Declarative Rendering

What you see in the editor is a Vue Single-File Component (SFC). An SFC is a reusable self-contained block of code that encapsulates HTML, CSS and JavaScript that belong together, written inside a `.vue` file.

The core feature of Vue is **declarative rendering**: using a template syntax that extends HTML, we can describe how the HTML should look based on JavaScript state. When the state changes, the HTML updates automatically.

State that can trigger updates when changed is considered **reactive**. We can declare reactive state using Vue's `reactive()` API. Objects created from `reactive()` are JavaScript [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) that work just like normal objects:

js

```
import { reactive } from 'vue'

const counter = reactive({
  count: 0
})

console.log(counter.count) // 0
counter.count++
```

`reactive()` only works on objects (including arrays and built-in types like `Map` and `Set`). `ref()`, on the other hand, can take any value type and create an object that exposes the inner value under a `.value` property:

js

```
import { ref } from 'vue'

const message = ref('Hello World!')

console.log(message.value) // "Hello World!"
message.value = 'Changed'
```

Details on `reactive()` and `ref()` are discussed in [Guide - Reactivity Fundamentals](https://vuejs.org/guide/essentials/reactivity-fundamentals.html).

Reactive state declared in the component's `<script setup>` block can be used directly in the template. This is how we can render dynamic text based on the value of the `counter` object and `message` ref, using mustaches syntax:

template

```
<h1>{{ message }}</h1>
<p>Count is: {{ counter.count }}</p>
```

Notice how we did not need to use `.value` when accessing the `message` ref in templates: it is automatically unwrapped for more succinct usage.

The content inside the mustaches is not limited to just identifiers or paths - we can use any valid JavaScript expression:

template

```
<h1>{{ message.split('').reverse().join('') }}</h1>
```

Now, try to create some reactive state yourself, and use it to render dynamic text content for the `<h1>` in the template.

## Answer
```vue
<script setup>
import { reactive, ref } from 'vue'

const counter = reactive({ count: 0 })
const message = ref('Hello World!')
</script>

<template>
  <h1>{{ message }}</h1>
  <p>Count is: {{ counter.count }}</p>
</template>
```
![[Pasted image 20241001130520.png]]



# Resources
- [Template Syntax | Vue.js](https://vuejs.org/guide/essentials/template-syntax.html)
- 