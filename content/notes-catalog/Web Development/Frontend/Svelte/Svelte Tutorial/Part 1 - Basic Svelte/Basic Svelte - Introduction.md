---
id: 01J64WNTAE9CQVEP6CD4DC03J2
modified: 2024-08-27T02:00:59-04:00
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
## Answer
```javascript
<script>
	let name = 'Svelte';
</script>
<h1>Hello {name.toUpperCase()}!</h1>

```
[Next: Dynamic attributes](https://learn.svelte.dev/tutorial/dynamic-attributes)

Just like you can use curly braces to control text, you can use them to control element attributes.

Our image is missing a `src` — let's add one:

App.svelte

```
<img src={src} />
```

That's better. But if you hover over the `<img>` in the editor, Svelte is giving us a warning:

> A11y: <img> element should have an alt attribute

When building web apps, it's important to make sure that they're _accessible_ to the broadest possible userbase, including people with (for example) impaired vision or motion, or people without powerful hardware or good internet connections. Accessibility (shortened to a11y) isn't always easy to get right, but Svelte will help by warning you if you write inaccessible markup.

In this case, we're missing the `alt` attribute that describes the image for people using screenreaders, or people with slow or flaky internet connections that can't download the image. Let's add one:

App.svelte

```
<img src={src} alt="A man dances." />
```

We can use curly braces _inside_ attributes. Try changing it to `"{name} dances."` — remember to declare a `name` variable in the `<script>` block.

## Shorthand attributes

It's not uncommon to have an attribute where the name and value are the same, like `src={src}`. Svelte gives us a convenient shorthand for these cases:

App.svelte

```
<img {src} alt="{name} dances." />
```
## Answer
```javascript
<script>
	let src = '/image.gif';
	let name = "A man"
</script>

<img {src} alt="{name} dances" />

```
[Next: Styling](https://learn.svelte.dev/tutorial/styling)

Just like in HTML, you can add a `<style>` tag to your component. Let's add some styles to the `<p>` element:

App.svelte

```
<p>This is a paragraph.</p>

<style>
	p {
		color: goldenrod;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>
```

Importantly, these rules are _scoped to the component_. You won't accidentally change the style of `<p>` elements elsewhere in your app, as we'll see in the next step.
## Answer
```javascript
<p>This is a paragraph.</p>

<style>
	p {
		color: goldenrod;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>

```
[Next: Nested components](https://learn.svelte.dev/tutorial/nested-components)

It would be impractical to put your entire app in a single component. Instead, we can import components from other files and include them in our markup.

Add a `<script>` tag to the top of `App.svelte` that imports `Nested.svelte`...

App.svelte

```
<script>
	import Nested from './Nested.svelte';
</script>
```

...and include a `<Nested />` component:

App.svelte

```
<p>This is a paragraph.</p>
<Nested />
```

Notice that even though `Nested.svelte` has a `<p>` element, the styles from `App.svelte` don't leak in.

> Component names are always capitalised, to distinguish them from HTML elements.
## Answer
`App.svelte`
```javascript
<script>
	import Nested from './Nested.svelte';
</script>

<p>This is a paragraph.</p>
<Nested />

<style>
	p {
		color: goldenrod;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>

```
`Nested.svelte`
```javascript
<p>This is another paragraph.</p>
```
[Next: HTML tags](https://learn.svelte.dev/tutorial/html-tags)

Ordinarily, strings are inserted as plain text, meaning that characters like `<` and `>` have no special meaning.

But sometimes you need to render HTML directly into a component. For example, the words you're reading right now exist in a markdown file that gets included on this page as a blob of HTML.

In Svelte, you do this with the special `{@html ...}` tag:

App.svelte

```
<p>{@html string}</p>
```

> **Warning!** Svelte doesn't perform any sanitization of the expression inside `{@html ...}` before it gets inserted into the DOM. This isn't an issue if the content is something you trust like an article you wrote yourself. However if it's some untrusted user content, e.g. a comment on an article, then it's critical that you manually escape it, otherwise you risk exposing your users to [Cross-Site Scripting](https://owasp.org/www-community/attacks/xss/) (XSS) attacks
## Answer
```javascript
<script>
	let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<p>{@html string}</p>
```

[Next: Reactivity](https://learn.svelte.dev/tutorial/reactive-assignments)

At the heart of Svelte is a powerful system of _reactivity_ for keeping the DOM in sync with your application state — for example, in response to an event.

To demonstrate it, we first need to wire up an event handler (we'll learn more about these [later](https://learn.svelte.dev/tutorial/dom-events)):

App.svelte

```
<button on:click={increment}>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>
```

Inside the `increment` function, all we need to do is change the value of `count`:

App.svelte

```
function increment() {
	count += 1;
}
```

Svelte 'instruments' this assignment with some code that tells it the DOM will need to be updated.

[Next: Declarations](https://learn.svelte.dev/tutorial/reactive-declarations)