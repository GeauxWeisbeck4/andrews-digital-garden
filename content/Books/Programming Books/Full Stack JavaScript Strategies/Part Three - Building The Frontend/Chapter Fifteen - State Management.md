---
id: 01JH1NMV0DNKBX2M8NFX419JYW
modified: 2025-01-07T19:39:56-05:00
---
# Chapter 15. State Management

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 15th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

With the repo established and the app ready to be developed, it’s time to start building views based on the [Figma designs](https://www.figma.com/file/Be3O95Dt7bLIyzmNMc61yo/Example-Dashboard?type=design&node-id=1-119&mode=design&t=46kCq4F9DwoXjTtP-0) and your architecture diagram. You already have the header container mostly filled out, so you can see where you start to juggle multiple things like state, API calls, and conditional rendering.

Managing your state is going to determine how you handle dynamic data and values that change based on user actions. There are a few ways to approach state management, and most apps use a combination of the approaches to keep the code as simple as possible.

In this chapter, I’ll cover:

- Different ways to manage state and when you might use them
    
- The state lifecycle in React
    
- How to share state between components
    
- Prop drilling management
    

Understanding how state works, when it’s updated, and why you use different approaches is important. You need to have a deeper knowledge of this so that you can coach others on your team and you can write the most optimal code possible. When states aren’t managed correctly, that can lead to some strange side effects in the UI. So let’s start by diving into how component state works.

# How Component State Works

In React, there is one method that controls how things are initially rendered on the page: the `render` method, which is usually in the _main.tsx_ file. It looks like this:

```
ReactDOM
```

The argument in `createRoot` is the `domNode`, which is where you manage the Document Object Model (DOM) inside it. The `render` method is where you display a React component, typically the `<App />` component.

The `render` method alone won’t show any updates to the UI when the user interacts with it. State is how React handles rendering based on user interactions like button clicks, form events, and data changes. You can think of the state as a component’s memory, as described in the [official docs](https://react.dev/learn/state-a-components-memory). After the initial render, components rerender in response to state updates.

React uses a [virtual DOM](https://www.geeksforgeeks.org/reactjs-virtual-dom/) to try to make the rerenders as efficient as possible by updating only the affected parts of the page. So when your state updates, only the components with the state changes will rerender. For example, that’s what happens when a user clicks a button and an API call is made to fetch data that updates on the page or if a button is clicked and a modal pops up.

That’s where the different [React hooks](https://react.dev/reference/react/hooks) come into play. A _hook_ is a piece of code in the React framework that lets you work with the React lifecycle methods. Hooks are only usable in React functional components since they replaced the [original class methods](https://legacy.reactjs.org/docs/hooks-intro.html#motivation) that came in older versions of React. They are meant to simplify the process of sharing states between components and the process of maintaining a component throughout its lifecycle. A few are specifically used to handle state in your components as the project increases in complexity: `useState`, `useContext`, and `useReducer`.

## useState

This is the hook you’ll probably use the most in any React app. The `useState` hook provides the “memory” for an individual component. So when a user is entering values in a form, for example, `useState` stores those values until they change. Component state is isolated to a specific instance of a component, so you don’t have to worry about states accidentally overlapping. You might have multiple search bars on the screen for different parts of the page, such as one in the header and one in the footer, like in this small example:

```
import
```

Each `SearchBar` component has its own local state that is updated independently of the other. So any user actions in the first search bar won’t affect the way the second search bar works. The `Container` component doesn’t know anything that’s happening in either of the search bar components to render correctly because the states are completely isolated from it. If you needed to have a state that affects both search bars in some way, then you would raise the state from the search bar components to the higher-level `Container` component.

When you [raise the state](https://react.dev/learn/sharing-state-between-components) of a component, you take the state from a child component and move it to the parent component. In this case, that would mean you move the `useState` hook from the `SearchBar` component and have it in the `Container` component.

The `useState` hook works by giving you access to both a value and an update method for that value. That’s why you always see the state being declared in a format like this:

```
const
```

`userInfo` is the current value, and `setUserInfo` is the function you can use to update the current value; it triggers a rerender in React. `initialUserInfo` is the initial state value that gets used by the hook. The `useState` hook is great for whenever you have some state that’s specific only to a component and doesn’t need to be shared, such as forms, inputs, styled elements, and other small pieces of functionality that get reused a lot throughout the app. It can also be the starting point for testing out state in your app before you move on to a more complex state management tool.

## useReducer

Before you decide to use Redux, you should check out [the `useReducer` hook](https://react.dev/learn/extracting-state-logic-into-a-reducer) because it works similarly. This is React’s built-in way of creating a more structured representation of your state in components and using actions and dispatchers. You can have predefined ways to update the state instead of updating it all at once, like with `useState`. An example of this is when you need to update a state that has a lot of interrelationships. Maybe you have several filters on a page that change when other filters are used. This would be a good place to try the `useReducer` hook.

Once you have functionality like that, your app state is at a higher complexity—this is something you want to look out for. When you recognize that handling states in your components involves several state changes across multiple user events or you’re doing a lot of state management by passing props between components, it’s time to research how to make that more efficient and maintainable.

When you use the reducer, you’re able to manage a complex state as a single object instead of having multiple state variables. Many of the pros and cons of choosing between `useState` and `useReducer` come down to team preference. This is a great time to do a little demo to go over the differences and decide what to do together. You’ll find that a mix of `useReducer` and `useState` can be nice for handling more complex scenarios as the app grows.

## useContext

As the app grows in complexity and size, you will tend to raise the state higher and higher in the code. Over time, that leads to _prop drilling_, where you pass values through layers of components that don’t need them. That leads to inefficient rerendering, and the code can get messier. You can use the [`useContext` hook](https://react.dev/learn/passing-data-deeply-with-context) to extract your state and remove some prop drilling.

The `<ThemeProvider />` component is an example of context being used for providing styles for all the descendant components in the DOM tree. Anything that requests the values from the `<ThemeProvider />` component with the `useContext` hook will have direct access to them instead of having to pass them down the context tree.

It might be good to set up a context in the beginning if you know a certain value will get passed to deeply nested child components—although this is something that will likely be added over time because you can find real use cases for it when you need to refactor code. Understanding when to pivot to a different approach is one of those things that you’ll have to use your skills to bring up. When you find prop drilling happening, document the values being passed and why. Then take that back to the team and discuss what everyone thinks about making the switch to `useContext`. Larger apps will have many context providers at the top level of the app to manage values and state.

# MORE DETAILS ABOUT USECONTEXT

The `useContext` hook is a great tool, but there are some nuances to it that Ethan Brown was able to comment on. First, check out this example in [his GitHub repo](https://github.com/EthanRBrown/react-shared-state-demo). He also had a bit more to say about how this hook affects performance:

> When state is lifted to the top level in larger apps, it’s a common performance killer when people don’t understand what `useContext` does and doesn’t do for you. If you read through the example repo above, you’ll see that context does not do anything for performance. I’ve seen far too many instances where someone says, “We use this piece of state everywhere, and I’m tired of passing it down. So let’s switch to using context, and it will improve performance, too!”
> 
> The problem is that context doesn’t inherently do anything at all to improve performance and can even result in much worse performance if you’re not considering how often the state in question is changing. For example, something like theme, language (for i18n), light/dark mode, or “current user” are all pretty reasonable uses for context as they change seldom, and when they do, you expect them to substantially change the entire component tree. Another reasonable use is a significant application context switch. For example, if you’re building some kind of document editor, and users edit document X and work on that for a long time before switching to document Y, context makes sense.
> 
> On the other hand, an example of a poor use of context is a sort or filter setting on a table. Such a setting is likely to change frequently, and every time it changes, the entire component tree has to re-rerender. This would have significant negative performance consequences, which would outweigh any benefit that context brings.

## Knowing What App Level to Manage State In

When you have state that needs to be shared between components, then it’s time to lift that state higher in the app. For example, you might have two filter inputs that display on the same page, and you want to update the page only when both have been changed. To do that, you would remove the state from the individual filters to the page level and then toggle the state for both filter inputs there. That way, you have a single source of truth that determines when both filters have been updated.

As you run into props that are getting passed down over 10 levels, you might bring up using context with the team. Larger apps can pass more than 10 props down 20 levels, which gets unwieldy over time. So when you notice the depth of the props getting farther away from the parent component, stay aware of that over time. Just because props are being passed a few levels doesn’t mean it’s time to use context. When it becomes a notable hindrance to development, then it’s time to discuss it.

Take notes as you work on the code and add new features. It’s a great idea to track observations you have in the codebase. It doesn’t have to be a formal process or anything that gets shared with the team immediately. I usually keep a note with a bulleted list of potential refactors just so I remember where I saw room for improvements. During retrospective meetings or other team meetings, I’ll bring up some of those things and try to get them on the sprint as tech cleanup.

# Different Approaches to State Management

Before you move on to other tools, I highly encourage you to go through the [React docs](https://react.dev/learn/state-a-components-memory) and learn more details about how the hooks work to manage state. You might find out that all you need are the built-in state management hooks. If you’ve gotten to a point in your app where you need something more powerful than the built-in hooks, it’s time to evaluate other tools.

There are a few approaches you should keep in mind as you and the team choose the state management tools you’ll work with. Three approaches to state management are reducer, atom, and mutable.

The _reducer_ approach is the one that has a centralized source of truth. This is when you dispatch actions from all the components to the central source. Some strengths of this approach include having that central source and devtool support if you’re using [Redux](https://redux.js.org/) or [Zustand](https://docs.pmnd.rs/zustand/getting-started/introduction). Some challenges include a number of new terms and concepts to understand and they may not be the absolute fastest tools compared to the built-in hooks.

The _atom_ approach is where the state is split into smaller parts and can be managed through hooks. It’s like creating your state as shareable components because you can derive states from these smaller parts. Packages like [Recoil](https://recoiljs.org/) and [Jotai](https://jotai.org/) are commonly used in this approach. Some of the pros to managing state like this are that it integrates really well with React features and it uses a similar format to what you already do with the `useState` hook. The main drawback is that you may need to create a graph representation of state instead of a linear one, and that can confuse more junior devs on your team.

The last approach I’ll talk about is the _mutable_ approach. This is when you work with tools that have proxies under the hood to create mutable data sources that can be directly written to and read from. Some of the packages you can use for this include [MobX](https://mobx.js.org/README.html) and [Valtio](https://valtio.pmnd.rs/). Some cool things about this approach are how flexible it is, how your dependencies are automatically updated when the state changes, and how it helps decrease the number of rerenders with the proxy. A drawback is that you aren’t able to easily see when and how your data is being updated, making it harder to track changes through the code. The mixture of mutable and immutable data can also lead to unclarity in the code.

###### NOTE

Proxies are just objects or functions for other objects or functions. It’s like when you have an endpoint that handles the calls for your third-party services instead of calling them directly. With regard to state management, a proxy tracks changes to the original object and triggers listener functions when an object is updated. Check out the [MDN docs on proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to learn more about what’s built into JavaScript.

One of these approaches is typically used in addition to the built-in React state hooks. You’ll want to discuss this heavily with the team because the choices you all make now will affect the bundle size, the user and developer experience, and how scalable the code is that you’re working on.

A great way to make the choice of what approach to use is to figure out what the team’s expertise is and then make a few small prototypes using different packages. You could do a comparison of state management using Redux, Jotai, and Valtio with a simple demo app and do your own benchmark tests. This will show you how fast the app will perform, the difference in bundle size, and what it’s like to actually work with the package in the context of your app. Keep in mind that performance differences are complex and usually don’t show until the app is much larger.

Once you’ve gone through this process with the team, you can add more functionality to the `UserInfo.Container` component you started work on in [Chapter 14](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch14.html#building_the_react_app). For this project, you can assume it is of medium complexity, so you will use Valtio as the state management tool. While it’s an underdog, it can use existing devtools like [Redux Toolkit](https://redux-toolkit.js.org/), unlike MobX. It’s also compatible with Node and Next.js and doesn’t solely work in React.

###### NOTE

It’s important that you try out new things so that you can have a good toolbox to select from. It’s a good exercise for you to try approaches you’re unfamiliar with so that when you run into a relevant use case, you know where you can start. Don’t worry about becoming an expert on every tool and methodology out there. Just having some awareness of what’s possible will help you kick off conversations and spark ideas in other devs on the team.

# Setting Up the State Manager

This won’t be a tutorial on the details of how Valtio works, but it will show you how to implement it in your project. If you want to learn more about the inner workings behind the functions and variables we use, check out the [Valtio docs](https://valtio.pmnd.rs/docs/introduction/getting-started) for more code examples and explanations. The purpose of this section is to give you a hands-on example of how you would work with a new tool and integrate it into your app. You’ll need to install Valtio with the following command:

npm i valtio

Now you need to create a new file in _src/pages/UserInfo_ called _UserInfo.State.tsx._ This is where you’ll define some types, the [proxy](https://valtio.pmnd.rs/docs/api/basic/proxy) that Valtio will use to keep track of state variables, and the actions that will be used to update the state based on user interactions. In this new file, add the following code:

```
import
```

It’s important to note the `orderStore`. The `orderStore` is the actual proxy that stores the state as it’s updated across different components. So this will hold the values you would normally set in a component with the `useState` hook. With the state management functionality and variables in place, you need to use them in your components. Here’s a snippet from the _UserInfo.Container.tsx_ file:

```
…
```

This is where you need to import `useSnapshot` and `orderStore` so that you can work with the state in the component. The first thing to notice is the `orderSnap` variable because it’s how you’re able to access the current state in `orderStore`. When the page initially renders or rerenders from a state change, you’ll have access to all the latest state values by calling `useSnapshot` with `orderStore`.

In the `useEffect` hook, you can see the mixture of local state and state from the proxy. `orderSnap.orders` is how you update the state in the proxy; that state will come from some API call. Depending on the component and app, you might do a hybrid approach like this to manage state as efficiently as possible. This is where some of the art in tech comes in as well as your developer experiences. For example, you might choose to handle form state locally and other states with the proxy. It mostly depends on what you and the team agree on; just stay consistent with the approach.

The JSX of the component that gets rendered uses `orderSnap` to map all the orders that are currently held in the proxy state. Once you start adding more components to this page, such as filtering orders, you’ll start adding more actions to _UserInfo.State.tsx_ and use them throughout the app. This is a super-simple example of using a state management tool. You could do all this with the built-in React hooks, but hopefully, you get an idea of how this can grow with the app and expand to handle complex features and state that needs to be updated on a deeper level than what’s in the local component.

With the state extracted outside of any component, you can use it anywhere in the app. That’s the power of a state management tool and why they’re typically used on larger apps. Many times, you don’t need to lift the state outside the components until you have an established app and it becomes a need. Implementing a state management tool too early can slow development and add more complexity than needed.

# Conclusion

In this chapter, you dived into some of the details behind state management in React. Many of the built-in hooks will be more than enough to handle your application as it grows over time, but you can add a more powerful or versatile tool if the need arises. It’s important to talk through these decisions with your team because their expertise and opinions will drive the future development and maintainability of the app.

Something you can do to help facilitate these conversations is set up little demos where the team can see how different tools work. State management can get messy when you have multiple actions updating multiple states across several components, so getting everyone to weigh in on the direction you all will take is crucial. Remember, as you build the app and the product grows, you and the team will have to decide the best way to handle state going forward and make sure it’s well documented.