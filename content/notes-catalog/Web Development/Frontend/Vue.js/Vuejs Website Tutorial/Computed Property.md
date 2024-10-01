---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Computed Property
modified: 2024-10-01T13:42:35-04:00
description: Computed properties in Vue
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
Let's keep building on top of the todo list from the last step. Here, we've already added a toggle functionality to each todo. This is done by adding a `done` property to each todo object, and using `v-model` to bind it to a checkbox:

template

```
<li v-for="todo in todos">
  <input type="checkbox" v-model="todo.done">
  ...
</li>
```

The next improvement we can add is to be able to hide already completed todos. We already have a button that toggles the `hideCompleted` state. But how do we render different list items based on that state?

Introducing [`computed()`](https://vuejs.org/guide/essentials/computed.html). We can create a computed ref that computes its `.value` based on other reactive data sources:

js

```
import { ref, computed } from 'vue'

const hideCompleted = ref(false)
const todos = ref([
  /* ... */
])

const filteredTodos = computed(() => {
  // return filtered todos based on
  // `todos.value` & `hideCompleted.value`
})
```

diff

```
- <li v-for="todo in todos">
+ <li v-for="todo in filteredTodos">
```

A computed property tracks other reactive state used in its computation as dependencies. It caches the result and automatically updates it when its dependencies change.

Now, try to add the `filteredTodos` computed property and implement its computation logic! If implemented correctly, checking off a todo when hiding completed items should instantly hide it as well.

# Answer
`App.vue`
```vue
<script setup>
import { ref, computed } from 'vue'

let id = 0

const newTodo = ref('')
const hideCompleted = ref(false)
const todos = ref([
  { id: id++, text: 'Learn HTML', done: true },
  { id: id++, text: 'Learn JavaScript', done: true },
  { id: id++, text: 'Learn Vue', done: false }
])

const filteredTodos = computed(() => {
  return hideCompleted.value
    ? todos.value.filter((t) => !t.done)
    : todos.value
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value, done: false })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo" required placeholder="new todo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style>
.done {
  text-decoration: line-through;
}
</style>
```

Now you see me...
![[Pasted image 20241001134152.png]]

NOW YOU DON'T!
![[Pasted image 20241001134214.png]]

BUAHAHAHAHAAMUAHAHAHAHA!