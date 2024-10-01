---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Slots
modified: 2024-10-01T13:56:35-04:00
description: Slots in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
In addition to passing data via props, the parent component can also pass down template fragments to the child via **slots**:

template

```
<ChildComp>
  This is some slot content!
</ChildComp>
```

In the child component, it can render the slot content from the parent using the `<slot>` element as outlet:

template

```
<!-- in child template -->
<slot/>
```

Content inside the `<slot>` outlet will be treated as "fallback" content: it will be displayed if the parent did not pass down any slot content:

template

```
<slot>Fallback content</slot>
```

Currently we are not passing any slot content to `<ChildComp>`, so you should see the fallback content. Let's provide some slot content to the child while making use of the parent's `msg` state.

# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const msg = ref('You are such a slot')
</script>

<template>
  <ChildComp>Message: {{ msg }}</ChildComp>
</template>
```
`ChildComp.vue`
```vue
<template>
  <slot>Fallback content</slot>
</template>
```
![[Pasted image 20241001135625.png]]