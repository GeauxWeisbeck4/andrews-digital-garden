---
id: 01JH1NPA2Q9NM035CZ5855202W
modified: 2025-01-07T19:40:46-05:00
---
# Chapter 17. Custom Styles

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 17th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

You already have designs to work with to get the initial app built, but this will go through multiple iterations as you build out features and the Product team gets feedback from users. You will be working with the Design team constantly to ensure that the styling of the app remains consistent as new features are added and new designers are brought onto their team.

Fonts, sizes, spacing, colors, and custom components like steppers, modals, and containers are going to be reused throughout the app. Sometimes the Design team will introduce inconsistencies because there is a lot to manage. As this happens, make sure to mention it immediately so that you all can decide which is the best design for both the users and the devs. If you notice a design will decrease accessibility, mention that, too.

In this chapter, I’ll cover:

- Accessibility
    
- Consistency in designs
    
- Custom themes
    
- Responsive design
    

You’ll need to use your experience to know when designs are becoming too complex in the codebase and work with the Design team to find a good balance. You already have the structure set up for custom theming, so now you’ll expand on that with the organization style guide. Let’s start by covering accessibility because that should be built into the designs from the beginning.

# Accessibility

The reason I’m bringing up accessibility first is because it can be an afterthought when you are trying to get a minimum viable product (MVP) ready and you and the team are churning out features as quickly as possible. Accessibility isn’t a secondary part of the app design. It’s just as important, if not more important, than concerns like mobile design. Not only does accessibility give fair access to information for those who have disabilities, but it’s also a legal requirement under the [Americans with Disabilities Act (ADA)](https://www.ada.gov/resources/web-guidance/).

While MUI has many accessibility features, it’s your responsibility to ensure you use things like aria labels, alt text on images, semantic HTML for screen readers, and keyboard commands for navigating the page. Here are some example code snippets of these:

```
<!-- aria label example -->
```

As you build out forms, make sure you have clear instructions and you give useful feedback with error and success messages. While you should highlight any poor color contrasts you notice, the Design team should be aware of that and manage it. Make sure that static content is easy for users to find with their screen readers because it usually has important info they shouldn’t miss.

# USE SEMANTIC ELEMENTS MORE THAN <DIV> TAGS

If you notice that your components are made of a bunch of `<div>` tags, update them to use semantic elements like `<section>`, `<article>`, `<aside>`, or [any of the others](https://developer.mozilla.org/en-US/docs/Web/HTML/Element). Developers have the habit of reaching for one or two elements and none of the others. Since you’re going to have to implement custom styles anyways, using different elements isn’t going to affect the way the page renders visually.

Each of the HTML elements has a meaning to accessible devices, and they will help give users an exponentially better experience. So when you type a `<div>` element, see how easy it would be for you to navigate the app with a screen reader and keyboard buttons alone. You can try it out with a tool like [Polypane](https://polypane.app/docs/) or [Responsively](https://responsively.app/). This doesn’t mean that `<div>` elements are bad, but you should consider if there’s a better element before using them.

## Making an Accessible Form

It’s important that forms are accessible because they allow users to take action. If someone can’t use or understand a form, they won’t be able to handle critical tasks like submitting payments, updating personal information, or requesting services. Don’t forget about making your informational content accessible as well or else a differently abled user may not even get to your forms.

You’re going to create a reusable, styled form in the app, and it will be added to the user info page for the first search bar with accessibility in mind. You can assume you’ve talked this over with the team and decided to go with [React Hook Form](https://www.react-hook-form.com/get-started) as the package that will be used throughout the app. Install it with this command:

npm install react-hook-form

Once it’s installed, create a new component under _src/elements_ called _SearchBar.tsx_. This will be used for all the search bars in the app. Here’s the code for the search bar:

```
import
```

###### TIP

Don’t forget to create the _index.tsx_ and _SearchBar.test.tsx_ files! I won’t explicitly say that each time, but the pattern for new components will always have these files in addition to the core component file.

MUI combined with React Hook Form will give you forms that are easy to style, have accessibility out of the box, and allow you to handle your fields with a lightweight API. You’ll have to read through the MUI docs to fully understand the props being passed to the [`<Input />`](https://mui.com/material-ui/api/input/) [component](https://mui.com/material-ui/api/input/), but a few things need to be called out in terms of accessibility.

All of the [`aria-labels`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label) are there to help users on screen readers keep track of where they are on the page. Not every element on the page needs an `aria-label`, so use it only on interactive elements. Most of the elements should be intuitive enough with the visible text, and that typically translates well to assistive devices. Something else to note is that there isn’t a Submit button on the search bar. This is per the design and also helps with accessibility because the default behavior for a form is to submit when the Enter key is pressed.

The `type=“search”` prop is one way to introduce semantic HTML with your component because it calls out that this element is going to be used to perform a search. Another thing doing a lot of work here is the [`inputProps`](https://mui.com/material-ui/api/input/#input-prop-inputProps) prop. This is one way MUI lets you handle errors in your forms.

Errors are an essential part of accessibility because they help a user figure out where they need to fix something and how they can do that. This input requires a value for submission, and there’s a minimum number of characters. If a user enters an invalid value, they will get a friendly alert next to the input with the problem. [Chapter 18](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#frontend_error_handling) will focus on all the different errors that happen on the frontend and how you can account for them.

## Checking Your Accessibility Implementations

Once you are ready to check how well your accessibility work has been implemented, the first place you can turn to is the browser devtools. Chrome in particular has some good devtools for evaluating how accessible your app is. You can do a quick audit by checking how things are grouped and any labels or functionality attached to them, as shown in [Figure 17-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch17.html#accessibility_tab_in_chrome_devtools). You can check the order that users will tab through as they navigate the page with a keyboard only.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1701.png)

###### Figure 17-1. Accessibility tab in Chrome DevTools

You can also run an audit using [Lighthouse](https://developer.chrome.com/docs/lighthouse/overview) to get more insights into how you can improve your accessibility, as shown in [Figure 17-2](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch17.html#lighthouse_accessibility_audit). Lighthouse is an open source tool that audits your web app for performance, accessibility, and a number of other metrics. If you aren’t sure of what to improve, this can give you a great list of things to do. Run your findings by the Product and Design teams because there may be legal implications that they deal with in the background. This won’t necessarily tell you the exact components that need to be improved, but it’ll give you a general idea of what to look for.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1702.png)

###### Figure 17-2. Lighthouse accessibility audit

Another tool I’ve used on enterprise apps is [axe-core](https://www.npmjs.com/package/axe-core). This is a great lightweight tool to add an extra layer of accessibility testing. You can install it in your app and add it as part of your testing during development. It will help you and the team find commonly missed accessibility rules. It’s a valuable tool when you want to add more automated testing to your deployment pipeline, which we’ll go over in [Chapter 24](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch24.html#integration_testing). Here’s an example of how you can add it to your code:

```
axe
```

This will show all accessibility issues in the console when you run the app. You can configure `axe` to run only for specific [Web Content Accessibility Guidelines (WCAG) rules](https://github.com/dequelabs/axe-core/blob/develop/doc/API.md) or to target specific components of the app if you need to focus on a complex area. This can be used to help you with compliance audits before new releases.

## More Accessibility Considerations

Most apps will be used by people who speak languages other than English, so it’s important to keep them in mind. One great tool for doing this type of localization is [i18n](https://www.i18next.com/overview/getting-started). You can create files for each language that you need translations in. If a user toggles the app to another language, all the text will be updated immediately. You will have to keep the design in mind as you add translations because some languages require more space for the text, so that can cause variations in spacing.

# Consistent Designs

After you’ve addressed accessibility for the app to meet the current designs and specs, there will be continuous iterations on the designs. The hard part comes when the designs get scattered across different platforms like Figma, Miro, or even screenshots. It becomes difficult to keep track of the current versions of the designs. Differences in common components start to slip in, and the next thing you know, there are four designs for the same modal.

While the management and versioning of designs often falls on the Design team, you are the one who has to meet the requirements for new features. When you start noticing inconsistencies, you have to bring them up. It doesn’t matter if it’s as “small” as a change in padding on buttons or the colors being used in certain parts of the app. Changes like these can influence the way you and the team work on the app.

Something else you want to cover within the dev team is how you will implement styles and make sure that you document them. There will be a lot of opinions on how the code should be written. Some devs consider inline styles to be harder to maintain and feel that every element should be some sort of styled component. Others will disagree and say that CSS classes should be used to define styles. Then there are others who will want to have files for styles on the component level. None of these are wrong, and usually a combination of strategies will be implemented.

The main thing is to keep your codebase consistent. When you’re doing PR reviews, call out anything that goes against the convention everyone agreed on. It’s OK for changes to happen as the app grows and the team changes, but it should be understood and documented when something deviates from the norm. Over time, the code will become a form of version control for the designs, so you want to make sure that everyone is able to understand and explain where things are changing.

# Custom Themes

You’ve already implemented custom theming with MUI in the _theme.tsx,_ and you’ll keep this file updated with any changes that affect the MUI components you use. Buttons are a good example of a commonly customized component. You’ll have the same buttons throughout the app, and they will need to align with the company branding. In the styles for this app, several buttons have the branded green color. This is where you can update _theme.tsx_ to something like this:

```
…
```

MUI has good support for custom theming, but it’s not without issues. You can run into some conflicts with MUI. This happens with any component library that you choose, but there inevitably comes a time where your designs will get tricky to implement. You’ll run into some situations where you’ll have something in your custom themes and will still need inline styles for a specific page.

This leads to a mixture of CSS rules in your theme and in local components, making it hard to figure out where to add new styles. So anytime the Design team comes to you with global changes that already have custom styles, just ask about it. These usually create more work than Design anticipates and can unexpectedly change other parts of the design.

It’s important to remember that there isn’t a perfect component library or styling solution. There are other options like [Tailwind CSS](https://tailwindcss.com/docs/installation) or [CSS Modules](https://github.com/css-modules/css-modules/blob/master/README.md), but implementing styles is more of an art than a science. Every developer who works on the frontend will have an opinion, and the Design team may also have opinions on what they think will work best. You even run into situations where a new dev on the team isn’t aware of existing components.

Tools like [Storybook](https://storybook.js.org/docs/get-started/why-storybook), [Ladle](https://ladle.dev/), or [React Cosmos](https://github.com/react-cosmos/react-cosmos) help you with documentation so that finding existing components becomes easier. This will also help keep the Design team up to date with components that have been implemented. And these tools are good for prototyping components, especially when one component has several variations. You can think of this like the developer-design bridge for shared components, and it can be like a brand book for the overall organization.

# Responsive Design

Mobile-first design is a common concept, but it doesn’t always happen first. Sometimes you will be working on apps that only have desktop designs and you have to add responsiveness later. Before you jump into the design work for desktop only, double-check with Product and Design if there should be a mobile version, too.

There are a few ways to handle responsive implementation later in a project, but it’s going to require a large amount of refactoring no matter how you try to get around it. If you’re using a component library, like we are with MUI here, then you do get some responsiveness out of the box. The rest will depend on the layout you develop. One way to handle that is with _breakpoints_, where you switch to styles or components that have been developed for a subset of devices like tablets and phones. Here’s an example of how you might implement that with the `Header` component:

```
const
```

Using [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries) like this is a common approach to building responsiveness. It generally works well regardless of the other style implementations you have, and you can add these at any point in the project. Or you might create separate components for the mobile view because the functionality is arranged differently enough that it’s easier to read and maintain code this way. You and the team could even go with custom layout structures and create stylesheets because the page changes so drastically.

# CREATE YOUR COMPONENTS WITH RESPONSIVENESS IN MIND

I’ve worked on apps where the company truly believed it wouldn’t need a mobile or responsive version and that everyone would _have_ to use the product on a desktop. Responsive versions of those apps were never created. While it’s rare, this does still happen.

More often than not, though, many apps reach a point where they need to be responsive. Remember, this covers more than just mobile and desktop. Your users could have the window at less than full-screen size, they could be using any number of smaller screen sizes, and their resolution could be different. You should also consider [adaptive design](https://www.invisionapp.com/defined/adaptive-design) where you have fixed layouts for specific device types.

If you can, create your components with this in mind. It doesn’t have to be perfect if you aren’t styling according to designs. But if you have some responsiveness in mind from the beginning, that will make everything easier for development later.

Be aware of UX with responsive designs. You might have parts of the app that slide in and out on the page, or you might hide content completely on smaller views. This can make content inaccessible to users, or the product may become unintuitive to work with, especially when you consider going from cursor-based interactions to touchscreen interactions. Things like hover states aren’t as easy for a user to figure out on a touchscreen device.

Images are something else you have to be careful with in responsive design. You may want to stretch or shrink an image to fit into a certain box, within a reasonable range. The aspect ratio should be maintained across all screen sizes to keep the image as clear as possible. With raster images like PNGs and JPGs, your images could get grainy when they’re stretched. When you have images like graphics or illustrations, you can use Scalable Vector Graphics (SVG) to keep image quality across all device sizes. Here’s an example of how you might implement a responsive image with [`srcset`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#srcset):

```
<
```

Make sure you pay attention to browser support for the CSS you use to implement designs with a tool like [CanIUse](http://caniuse.com/). Not every browser version will support the latest CSS features, and some features aren’t backward compatible. You’ll need to work with Product to decide which browsers to support and which versions of those browsers you should support. Otherwise, you can end up with some very messy CSS styles just to support Internet Explorer 11 for a handful of users.

Also consider working with [Web Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components). These are custom elements you can create with the JavaScript API. These elements are part of the shadow DOM, which keeps the styles and functionality of the elements separate from the main DOM so that you don’t have to worry about them clashing with each other. This is a newer option that’s growing in adaptation across JavaScript frameworks, so it’s something that could come up in the discussions about the best way to handle responsive design.

# Conclusion

In this chapter, you dived into some of the concepts behind implementing designs in your project. Of all the things on the frontend, this is probably the one that brings in the most opinions. There really isn’t a standard way to implement designs because every team has its own expertise and ideas for what is maintainable. As the app grows, the organization expands, and new members of the Design team are added, even the core themes can change drastically.

The main thing here is to strive for consistency in user and developer experience. Whether you decide to use a prebuilt component library, implement styles based on class names, or do some combination of everything, as long as it doesn’t interfere with the UX, you’ve reached the goal. And as long as the team can figure out where styles are coming from and how they are connected, then the DX is manageable. Just keep the communication flowing among your team, the Product team, and the Design team to avoid confusion as much as possible.