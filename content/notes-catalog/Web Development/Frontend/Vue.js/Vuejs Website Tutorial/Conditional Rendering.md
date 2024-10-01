---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Conditional Rendering
modified: 2024-10-01T13:30:31-04:00
description: Conditional rendering in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
We can use the `v-if` directive to conditionally render an element:

template

```
<h1 v-if="awesome">Vue is awesome!</h1>
```

This `<h1>` will be rendered only if the value of `awesome` is [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy). If `awesome` changes to a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value, it will be removed from the DOM.

We can also use `v-else` and `v-else-if` to denote other branches of the condition:

template

```
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

Currently, the demo is showing both `<h1>`s at the same time, and the button does nothing. Try to add `v-if` and `v-else` directives to them, and implement the `toggle()` method so that we can use the button to toggle between them.

More details on `v-if`: [Guide - Conditional Rendering](https://vuejs.org/guide/essentials/conditional.html)
# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'

const awesome = ref(true)

function toggle() {
  awesome.value = !awesome.value
}
</script>

<template>
  <button @click="toggle">Toggle</button>
  <h1 v-if="awesome">Vue is awesome!</h1>
  <h1 v-else>Oh no 😢</h1>
</template>
```
![[Pasted image 20241001133034.png]]