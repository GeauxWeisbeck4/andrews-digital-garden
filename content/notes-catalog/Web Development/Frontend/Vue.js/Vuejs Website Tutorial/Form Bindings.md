---
id: 01J94G885ZWPAGW1VJHPZ3Z19Z
title: Form Bindings
modified: 2024-10-01T13:26:41-04:00
description: Form bindings in Vue.js
tags:
  - vuejs
  - tutorial
  - web-development
  - programming
  - javascript
---
Using `v-bind` and `v-on` together, we can create two-way bindings on form input elements:

template

```
<input :value="text" @input="onInput">
```

js

```
function onInput(e) {
  // a v-on handler receives the native DOM event
  // as the argument.
  text.value = e.target.value
}
```

Try typing in the input box - you should see the text in `<p>` updating as you type.

To simplify two-way bindings, Vue provides a directive, `v-model`, which is essentially syntactic sugar for the above:

template

```
<input v-model="text">
```

`v-model` automatically syncs the `<input>`'s value with the bound state, so we no longer need to use an event handler for that.

`v-model` works not only on text inputs, but also on other input types such as checkboxes, radio buttons, and select dropdowns. We cover more details in [Guide - Form Bindings](https://vuejs.org/guide/essentials/forms.html).

Now, try to refactor the code to use `v-model` instead.

# Answer
`App.vue`
```vue
<script setup>
import { ref } from 'vue'

const text = ref('')
</script>

<template>
  <input v-model="text" placeholder="Type here">
  <p>{{ text }}</p>
</template>
```
![[Pasted image 20241001132659.png]]

# Resources
- [Guide - Form Bindings](https://vuejs.org/guide/essentials/forms.html)