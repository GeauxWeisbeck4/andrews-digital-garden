---
id: 01J94N2BB2DEYPYSMFNQ1B54ZQ
modified: 2024-10-01T16:06:12-04:00
title: Chapter Two - Creating a Todo List App
description: Creating a Todo list app using vue.js
---
- # Topics 
	- -Using the CLI tool to create a custom environment for an app
	- Vue.js reactivity concept
	- Styling with CSS
	- First glance at Vue.js DevTools
- # Project Requirements
	- We will make sure that you see a list of items
	- Each item will have a checkbox
	- The list will be sorted by unchecked items first and checked items second
	- The status of an item should be preserved by the browser on future visits

# Building the App
- ## Step One: Create the `AppHeader` component
	- `AppHeader.vue`
```vue
<template>
    <header>
        <h1><span class="icon" aria-hidden="true">✅</span></h1>
    </header>
</template>
<style scoped>
header {
    border-bottom: #333 1px solid;
    background-color: #fff;
}
header::after {
    content: "";
    display: block;
    height: 1px;
    box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.75);
}
h1 {
    font-size: 2rem;
}
h1 .icon {
    font-size: 1rem;
    vertical-align: middle;
}
</style>

```
- ## Step Two: Create ListItem Component
	- `ListItem.vue`
```vue
<script lang="ts" setup>defineProps<{
    isChecked?: boolean | false
}>()
</script>
<template>
    <label :class="{ 'checked': isChecked }">
        <input type="checkbox" :checked="isChecked" />
        <slot></slot>
    </label>
</template>
<style scoped>
label {
    cursor: pointer;
}
.checked {
    text-decoration: line-through;
}
</style>

```
- We define the props that will be passed to the component. Props are the properties that can be controlled from outside of the component. These are usually values that are being processed by the component and determine the unique characteristics of the component in that state. In rare cases, you could pass down a function as well.
- By using the **defineProps** method, we’re using the Vue.js API to declare our props properly.
- The second bit is the way that the component should render HTML to the virtual DOM. Vue.js uses a syntax that is HTML-based. You can read more about it here: [https://vuejs.org/guide/essentials/template-syntax.html#template-syntax](https://vuejs.org/guide/essentials/template-syntax.html#template-syntax).
- ## Step Three: Creating the List
	- We can now start generating the list. This component holds the info and use it to render all the individual items.
	- `TodoList.vue`
```vue
<script lang="ts" setup>
import ListItem from './ListItem.vue'
</script>
<template>
    <ul>
        <ListItem :is-checked="false">
            This is slotted content
        </ListItem>
    </ul>
</template>

```
- `App.vue`
```vue
<script setup lang="ts">
import AppHeader from './components/AppHeader.vue';
import TodoList from './components/TodoList.vue';
</script>

<template>
  <AppHeader />
  <TodoList />
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
```
- ## Step Four - Making a list
	- Let’s look at the **script** block first. Apart from importing the **ListItem** component, we’re defining **type** for **Item**, which consists of the **title** property as a string and the optional property **checked** as a Boolean. TypeScript lets us define a **Type** alias, which our IDE can plug into when interacting with **Type**.
	- In this example, when accessing the properties on **item** in the **ListItem** template, the IDE already recognizes the **title** and optional **checked** properties:
```vue
<script lang="ts" setup>
import ListItem from './ListItem.vue'

type Item = {
    title: string,
    checked?: boolean
}

const listItems: Item[] = [
    { title: 'Create todo list app', checked: true },
    { title: 'Predict the weather app', checked: false },
    { title: 'Play some tunes', checked: true },
    { title: 'Let\'s get cooking', checked: false },
  { title: 'Pump some iron', checked: false },
  { title: 'Track my expenses', checked: false },
  { title: 'Organize a game night', checked: false },
  { title: 'Learn a new language', checked: false },
  { title: 'Publish my work' }
]
</script>
<template>
    <ul>
        <li
          :key='key'
          v-for='(item, key) in listItems'
          >
            <ListItem :is-checked="false">
              This is slotted content
            </ListItem>
        </li>
    </ul>
</template>
<style scoped>
ul {
    list-style: none;
}
li {
    margin: 0.4rem 0;
}
</style>
```
- When constructing the **ListItems** array, we assign **Type** as an array of that type using the **[]** symbols. We immediately fill the **ListItems** array with a list of items. This means that TypeScript could also infer the types, but it is a better practice to explicitly set the types where possible.
- In the template, we are creating an unordered list element and use the **v-for** directive to iterate over the items in the array:
- The **v-for** directive is used to loop over collections and repeat the template that marks the collection. For every item, the current value is assigned to the first argument (**item**) and optionally provides an index of the collection as the second argument (**key**).
- ## Step Five - Reactivity Explained
	- If you have opened the app and clicked an item, you can toggle the checkbox, but on refreshing the page, nothing happens. Also, if you’ve looked closely at the CSS of the **<ListItem />** component, you may have noticed that strikethrough styling should be applied on a checked item. This is only the case on the first item.
	- The toggling of the checkbox is, in fact, native browser behavior and doesn’t signify anything in the context of the state of the Todo list!
	- We need to wire the changes in the UI to the state of the application:
```vue
const listItems: Ref<Item[]> = ref([
    { title: 'Create todo list app', checked: true },
    { title: 'Predict the weather app', checked: false },
    { title: 'Play some tunes', checked: true },
    { title: 'Let\'s get cooking', checked: false },
    { title: 'Pump some iron', checked: false },
    { title: 'Track my expenses', checked: false },
    { title: 'Organize a game night', checked: false },
    { title: 'Learn a new language', checked: false },
    { title: 'Publish my work' }
])
```
Note that capitalized **Ref** is used to type the value and lowercase **ref** is used as a wrapper of the array of items. If we now want to access the values in the script block, we need to access them by using **listItems.value**.

Now that the **listItems** are reactive, the virtual DOM will respond automatically to changes on the variables. We can add a method that changes an item so that it will be reflected in the user interface.
Let’s add the following function to the **script** block:
```vue
<script lang="ts" setup>
const updateItem = (item: Item): void => {
    const updatedItem = findItemInList(item)
    toggleItemChecked(updatedItem)
}
const findItemInList = (item: Item): Item | undefined => {
    return listItems.value.find(
        (itemInList: Item) => itemInList.title === item.title
    )
}
const toggleItemChecked = (item: Item): void => {
    item.checked = !item.checked
}
</script>
```
Adopting Robert C Martin’s Clean Code philosophy, I split the instruction into separate functions with their own clear intent. When calling **updateItem** with **item** as an argument, it will try to find it in the **itemList** and toggle the **checked** property on the object.

We can see that TypeScript is guiding us to a slightly better solution: because **findItemInList** could return an **undefined** value and **toggleItemChecked** expects a parameter, the argument of calling the **toggleItemChecked** function gets a squiggly line.

## Step Six: Sorting the List
We’re now perfectly capable of showing variables in the template. There are cases, however, when you have a need for more advanced expressions, such as, in our example, the requirement of sorting the list. For variables that have no side effects and include reactive data, you can use the Vue.js **computed** function.

Typically, you would use **computed** for filtering data, format expressions, displaying calculations, or Boolean conditionals. Let’s apply it to sort the list with completed items at the bottom.

First, we’ll import **computed** to the **TodoList** component. We can add it to the import where we also import the **ref** function: