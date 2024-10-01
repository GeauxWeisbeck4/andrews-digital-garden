---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: List Rendering
modified: 2024-10-01T13:35:46-04:00
description: How to render a list in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
We can use the `v-for` directive to render a list of elements based on a source array:

template

```
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

Here `todo` is a local variable representing the array element currently being iterated on. It's only accessible on or inside the `v-for` element, similar to a function scope.

Notice how we are also giving each todo object a unique `id`, and binding it as the [special `key` attribute](https://vuejs.org/api/built-in-special-attributes.html#key) for each `<li>`. The `key` allows Vue to accurately move each `<li>` to match the position of its corresponding object in the array.

There are two ways to update the list:

1. Call [mutating methods](https://stackoverflow.com/questions/9009879/which-javascript-array-functions-are-mutating) on the source array:
    
    js
    
    ```
    todos.value.push(newTodo)
    ```
    
2. Replace the array with a new one:
    
    js
    
    ```
    todos.value = todos.value.filter(/* ... */)
    ```
    

Here we have a simple todo list - try to implement the logic for `addTodo()` and `removeTodo()` methods to make it work!

More details on `v-for`: [Guide - List Rendering](https://vuejs.org/guide/essentials/list.html)


# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'

// give each todo a unique id
let id = 0

const newTodo = ref('')
const todos = ref([
  { id: id++, text: 'Learn HTML' },
  { id: id++, text: 'Learn JavaScript' },
  { id: id++, text: 'Learn Vue' }
])

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value })
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
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.text }}
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
</template>
```
![[Pasted image 20241001133535.png]]