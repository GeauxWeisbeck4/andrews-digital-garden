---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Event Listeners
modified: 2024-10-01T13:23:27-04:00
description: How to use event listeners in Vue
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
We can listen to DOM events using the `v-on` directive:

template

```
<button v-on:click="increment">{{ count }}</button>
```

Due to its frequent use, `v-on` also has a shorthand syntax:

template

```
<button @click="increment">{{ count }}</button>
```

Here, `increment` is referencing a function declared in `<script setup>`:

vue

```
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  // update component state
  count.value++
}
</script>
```

Inside the function, we can update the component state by mutating refs.

Event handlers can also use inline expressions, and can simplify common tasks with modifiers. These details are covered in [Guide - Event Handling](https://vuejs.org/guide/essentials/event-handling.html).

Now, try to implement the `increment` function yourself and bind it to the button using `v-on`.

# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```
![[Pasted image 20241001132338.png]]

# Resources
- [Event Handling | Vue.js](https://vuejs.org/guide/essentials/event-handling.html)