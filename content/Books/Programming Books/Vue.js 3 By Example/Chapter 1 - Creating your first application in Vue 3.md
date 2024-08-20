---
id: 01J5S0ZKXEEQWF9Y0F725HGV0F
modified: 2024-08-20T19:13:06-04:00
---
# _Chapter 1_: Creating Your First Application in Vue 3

**Vue 3** is the latest version of the popular Vue.js framework. It is focused on improving developer experience and speed. It is a component-based framework that lets us create modular, testable apps with ease. It includes concepts that are common to other frameworks such as props, transitions, event handling, templates, directives, data binding, and more. The main goal of this chapter is to get you started with developing your first Vue app. This chapter is focused on how to create components.

In this chapter, we will look at how to use Vue 3 to create simple apps from scratch. We will start by building the most basic apps and then move on to building more complex solutions in the next few chapters.

The major topics we will cover are as follows:

- Understanding Vue as a framework
- Setting up the Vue project
- Vue 3 core features – components and built-in directives
- Debugging with Vue.js Devtools

# Technical requirements

The code for this chapter is located at [https://github.com/PacktPublishing/-Vue.js-3-By-Example/tree/master/Chapter01](https://github.com/PacktPublishing/-Vue.js-3-By-Example/tree/master/Chapter01).

# Understanding Vue as a framework

As we mentioned in the introduction, there are concepts in Vue that are available from other frameworks. Directives manipulate the **Document Object Model** (**DOM**) just like in Angular.js and Angular. Templates render data like we do with Angular. It also has its own special syntax for data binding and adding directives.

Angular and React both have props that pass data between components. We can also loop through array and object entries to display items from lists. Also, like Angular, we can add plugins to a Vue project to extend its functionality.

Concepts that are exclusive to Vue.js include computed properties, which are component properties that are derived from other properties. Also, Vue components have watchers that let us watch for reactive data changes. Reactive data is data that is watched by Vue.js and actions are done automatically when reactive data changes.

As reactive data changes, other parts of a component and other components that reference those values are all updated automatically. This is the magic of Vue. It is one of the reasons that we can do so much with so little code. It takes care of the task of watching for data changes for us, so that we don't have to do that.

Another unique feature of Vue 3 is that we can add the framework and its libraries with script tags. This means that if we have a legacy frontend, we can still use Vue 3 and its libraries to enhance legacy frontends. Also, we don't need to add build tools to build our app. This is a great feature that isn't available with most other popular frameworks.

There's also the popular Vue Router library for routing, and the Vuex library for state management. They have all been updated to be compatible with Vue 3, so we can use them safely. This way, we don't have to worry about which router and state management library to use as we do with other frameworks such as React. Angular comes with its own routes, but no standard state management library has been designated.

# Setting up the Vue project with the Vue CLI and script tag

There are several ways to create Vue projects or to add script tags to our existing frontends. For prototyping or learning purposes, we can add the latest version of Vue 3 by adding the following code:
