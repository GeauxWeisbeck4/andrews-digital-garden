---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Components
modified: 2024-10-01T13:48:58-04:00
description: Components in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
So far, we've only been working with a single component. Real Vue applications are typically created with nested components.

A parent component can render another component in its template as a child component. To use a child component, we need to first import it:

js

```
import ChildComp from './ChildComp.vue'
```

Then, we can use the component in the template as:

template

```
<ChildComp />
```

Now try it yourself - import the child component and render it in the template.

# Answer
`App.vue`
```vue
<script setup>
import ChildComp from './ChildComp.vue'
</script>

<template>
	<ChildComp />
</template>
```
`ChildComp.vue`
```vue
<template>
  <h2>A Child Component!</h2>
</template>
```
![[Pasted image 20241001134913.png]]