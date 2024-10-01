---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Props
modified: 2024-10-01T13:51:11-04:00
description: Props in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
A child component can accept input from the parent via **props**. First, it needs to declare the props it accepts:

vue

```
<!-- ChildComp.vue -->
<script setup>
const props = defineProps({
  msg: String
})
</script>
```

Note `defineProps()` is a compile-time macro and doesn't need to be imported. Once declared, the `msg` prop can be used in the child component's template. It can also be accessed in JavaScript via the returned object of `defineProps()`.

The parent can pass the prop to the child just like attributes. To pass a dynamic value, we can also use the `v-bind` syntax:

template

```
<ChildComp :msg="greeting" />
```

Now try it yourself in the editor.

# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const greeting = ref('Hello from parent')
</script>

<template>
  <ChildComp :msg="greeting" />
</template>
```
`ChilComp.vue`
```vue
<script setup>
const props = defineProps({
  msg: String
})
</script>

<template>
  <h2>{{ msg || 'No props passed yet' }}</h2>
</template>
```
![[Pasted image 20241001135125.png]]