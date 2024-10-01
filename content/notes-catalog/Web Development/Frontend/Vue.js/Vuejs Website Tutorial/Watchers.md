---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Watchers
modified: 2024-10-01T13:46:59-04:00
description: Watchers in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
Sometimes we may need to perform "side effects" reactively - for example, logging a number to the console when it changes. We can achieve this with watchers:

js

```
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newCount) => {
  // yes, console.log() is a side effect
  console.log(`new count is: ${newCount}`)
})
```

`watch()` can directly watch a ref, and the callback gets fired whenever `count`'s value changes. `watch()` can also watch other types of data sources - more details are covered in [Guide - Watchers](https://vuejs.org/guide/essentials/watchers.html).

A more practical example than logging to the console would be fetching new data when an ID changes. The code we have is fetching todos data from a mock API on component mount. There is also a button that increments the todo ID that should be fetched. Try to implement a watcher that fetches a new todo when the button is clicked.

# Answer
`App.vue`
```vue
<script setup>
import { ref, watch } from 'vue'

const todoId = ref(1)
const todoData = ref(null)

async function fetchData() {
  todoData.value = null
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  todoData.value = await res.json()
}

fetchData()

watch(todoId, fetchData)
</script>

<template>
  <p>Todo id: {{ todoId }}</p>
  <button @click="todoId++" :disabled="!todoData">Fetch next todo</button>
  <p v-if="!todoData">Loading...</p>
  <pre v-else>{{ todoData }}</pre>
</template>
```
![[Pasted image 20241001134703.png]]