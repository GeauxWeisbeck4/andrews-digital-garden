---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Attribute Bindings
modified: 2024-10-01T13:21:04-04:00
description: Attribute bindings in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
 [[Vuejs Website Tutorial]]  <- [[Declarative Rendering]]   


In Vue, mustaches are only used for text interpolation. To bind an attribute to a dynamic value, we use the `v-bind` directive:

template

```
<div v-bind:id="dynamicId"></div>
```

A **directive** is a special attribute that starts with the `v-` prefix. They are part of Vue's template syntax. Similar to text interpolations, directive values are JavaScript expressions that have access to the component's state. The full details of `v-bind` and directive syntax are discussed in [Guide - Template Syntax](https://vuejs.org/guide/essentials/template-syntax.html).

The part after the colon (`:id`) is the "argument" of the directive. Here, the element's `id` attribute will be synced with the `dynamicId` property from the component's state.

Because `v-bind` is used so frequently, it has a dedicated shorthand syntax:

template

```
<div :id="dynamicId"></div>
```

Now, try to add a dynamic `class` binding to the `<h1>`, using the `titleClass` ref as its value. If it's bound correctly, the text should turn red.

## Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'

const titleClass = ref('title')
</script>

<template>
  <h1 :class='titleClass'>Go Big Red!</h1> <!-- add dynamic class binding here -->
</template>

<style>
.title {
  color: red;
}
</style>
```
![[Pasted image 20241001131114.png]]'

# Resources
- [Reactivity Fundamentals | Vue.js](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)



-> [[Event Listeners]]