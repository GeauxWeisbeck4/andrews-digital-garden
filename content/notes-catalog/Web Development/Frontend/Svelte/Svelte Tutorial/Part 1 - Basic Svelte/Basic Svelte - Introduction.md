---
id: 01J64WNTAE9CQVEP6CD4DC03J2
modified: 2024-08-26T20:15:34-04:00
title: Basic Svelte/Introduction
description: The very rudimentary basics of Svelte
tags:
  - svelte
  - programming
  - web-development
  - javascript
  - typescript
  - frameworks
---
# Basic Svelte/Introduction


## What is Svelte?

Svelte is a tool for building web applications. Like other user interface frameworks, it allows you to build your app _declaratively_ out of components that combine markup, styles and behaviours.

These components are _compiled_ into small, efficient JavaScript modules that eliminate overhead traditionally associated with UI frameworks.

You can build your entire app with Svelte (for example, using an application framework like [SvelteKit](https://kit.svelte.dev/), which this tutorial will cover), or you can add it incrementally to an existing codebase. You can also ship components as standalone packages that work anywhere.

## How to use this tutorial

> You'll need to have basic familiarity with HTML, CSS and JavaScript to understand Svelte.

This tutorial is split into four main parts:

- [Basic Svelte](https://learn.svelte.dev/tutorial/welcome-to-svelte) (you are here)
- [Advanced Svelte](https://learn.svelte.dev/tutorial/tweens)
- [Basic SvelteKit](https://learn.svelte.dev/tutorial/introducing-sveltekit)
- [Advanced SvelteKit](https://learn.svelte.dev/tutorial/optional-params)

Each section will present an exercise designed to illustrate a feature. Later exercises build on the knowledge gained in earlier ones, so it's recommended that you go from start to finish. If necessary, you can navigate via the menu above.

If you get stuck, you can click the `solve` button to the left of the editor. (The `solve` button is disabled on sections like this one that don't include an exercise.) Try not to rely on it too much; you will learn faster by figuring out where to put each suggested code block and manually typing it in to the editor.

[Next: Your first component](https://learn.svelte.dev/tutorial/your-first-component)

In Svelte, an application is composed from one or more _components_. A component is a reusable self-contained block of code that encapsulates HTML, CSS and JavaScript that belong together, written into a `.svelte` file. The `App.svelte` file, open in the code editor to the right, is a simple component.

## Adding data

A component that just renders some static markup isn't very interesting. Let's add some data.

First, add a script tag to your component and declare a `name` variable:

App.svelte

```
<script>
	let name = 'Svelte';
</script>

<h1>Hello world!</h1>
```

Then, we can refer to `name` in the markup:

App.svelte

```
<h1>Hello {name}!</h1>
```

Inside the curly braces, we can put any JavaScript we want. Try changing `name` to `name.toUpperCase()` for a shoutier greeting.

App.svelte

```
<h1>Hello {name.toUpperCase()}!</h1>
```

[Next: Dynamic attributes](https://learn.svelte.dev/tutorial/dynamic-attributes)