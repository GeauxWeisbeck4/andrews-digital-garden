---
id: 01J94M9MSH7TGYMM3JSPEWS619
modified: 2024-10-01T14:21:37-04:00
title: Chapter One - Introduction to Vue.js 3
description: Get the ins and outs of Vue.js 3
tags:
  - vuejs
  - books
  - programming
  - web-development
  - javascript
---
## Why use vue?
- It’s performant because it was built to be optimized from the ground up.
- It’s lightweight, tree-shakeable, and ships only the code that is needed to run your application. The minimal code (after being optimized in a build step) is about 16 KB in size.
- It’s highly scalable, using preferred patterns such as Single File Components and the Composition API, which makes it suitable for enterprise applications.
    
    **Single File Components** are part of the Vue.js philosophy where the template, script, and styling of a component are encapsulated in a single file, with the goal of improving the organization, readability, and maintainability of code.
    
    The **Composition API** allows better organization and reuse of code throughout the application, which makes code more modular and easy to maintain.

## My First Vue Component
- I'm so proud :)

```vue
<template>
    <div>My first <span>Vue.js</span> component!</div>
</template>

<style scoped>
div {
    color: #35495f;
    font-size: 1.6rem;
}
span {
    color: #41b883;
    font-weight: 700;
}
</style>

```