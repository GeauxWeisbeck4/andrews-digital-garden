---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Emits
modified: 2024-10-01T13:53:31-04:00
description: Emits in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
In addition to receiving props, a child component can also emit events to the parent:

vue

```
<script setup>
// declare emitted events
const emit = defineEmits(['response'])

// emit with argument
emit('response', 'hello from child')
</script>
```

The first argument to `emit()` is the event name. Any additional arguments are passed on to the event listener.

The parent can listen to child-emitted events using `v-on` - here the handler receives the extra argument from the child emit call and assigns it to local state:

template

```
<ChildComp @response="(msg) => childMsg = msg" />
```

Now try it yourself in the editor.

# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const childMsg = ref('No child msg yet')
</script>

<template>
  <ChildComp @response="(msg) => childMsg = msg" />
  <p>{{ childMsg }}</p>
</template>
```
`ChildComp.vue`
```vue
<script setup>
const emit = defineEmits(['response'])

emit('response', 'hello from child')
</script>

<template>
  <h2>Child component</h2>
</template>
```