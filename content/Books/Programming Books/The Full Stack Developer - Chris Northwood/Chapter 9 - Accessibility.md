---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 9 - Accessibility
modified: 2024-10-26T16:37:45-04:00
tags:
  - full-stack
  - books
---
# 9. Accessibility

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_9_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

Accessibility seems mysterious to many, but the basic concept behind it is the same as usability—someone who visits a web page should be able to read its content easily, and those who open a web app should be able to use it. An accessible web site is one that is also usable to everyone who may try to access it. In fact, many accessibility features are used optionally by able-bodied people—keyboard navigation can be quicker than a mouse for filling in forms, and subtitles in video can be useful even if you’re not hard of hearing.

Accessibility covers a large range of topics, and like much of software development, it rarely exists in a vacuum. Accessibility is often used to talk about assistive technologies (AT), like screen readers; or larger font size; or considerations specifically for people who have disabilities; but it’s more than that. Compare building a web site to constructing a building, for example—all new buildings must be accessible. A particularly visible aspect of this may be ramps for wheelchair users, but it goes beyond that. Signs may have braille alternatives, and may have large, high-contrast text for those who have trouble seeing. Elevators will announce their floors audibly, and there won’t be any strobes or flickering lights that may “look good” but risk triggering epilepsy. Many of these small details aren’t just details added at the end by an accessibility specialist. Can you imagine a situation where a building was designed and built, and then before opening a specialist came in and demanded the installation of ramps and replacing all the signs in the building? When web sites are built that way, it’s easy to see how accessibility can be dismissed as an additional cost and get dropped in a budget-constrained project. This is especially true if there are unidentified fundamental issues with the accessibility of your site; leaving it to the end can either result in only patching superficial issues, or a large amount of rework.

Designing and building a web site to be accessible from the start takes surprisingly little extra effort compared to adding it later. Accessibility can be thought of as a cross-functional requirement, in that it’s something that should be built into every feature or story that you build, rather than a standalone feature to be added in at the end of a project. Indeed, as a software engineer, you have an ethical and legal obligation not to exclude classes of people from your application based on factors they cannot control.

Fortunately, accessibility is not a magic dust that is applied, but actually a fairly small set of principles supported by good APIs that are available to all web developers. By default, all pages are accessible—we build in complexity by adding styles and rich scripting that can introduce inaccessibility. There are guidelines for checking accessibility of pages, but like many automated tools, these are general and may miss the context of your specific application. Being compliant with the guidelines is good, but not enough to ensure your site is actually accessible.

## Accessible from the Start

Accessibility starts with the design of a web site. If the text is too small, or if there is poor color contrast (for example, a light gray on white), then it can be hard for able-bodied people in full health to use, let alone those who may have sight difficulties. Building these kinds of design elements to be accessible has the benefit of helping everyone. Even those with perfect sight who may be fine with something that a partially sighted person struggles with can benefit if a design is reworked to be readable to all audience.

This type of accessibility is more than just helping out those with specific disabilities. It is natural for eyesight to deteriorate with age, and even a person with normal sight who is older may struggle with something a younger person finds acceptable. This problem is often confounded by the fact that development teams are often relatively young, as we remain a young industry in general, so these issues are often missed.

Going further, the lines of accessibility, usability, and user experience can start to blur. For example, people who have dyslexia or learning difficulties can find dense text hard to follow. The appropriate use of headings, which should serve to suggest the hierarchy of information, can make a page usable by these audiences, as well as benefiting other users by making it easy to scan a page to find specific content.

This kind of accessible development is done in the design stage, often before a developer is involved. However, the end result of a development process is always everyone’s responsibility, and catching accessibility issues early can make them much easier to correct. When working with a designer to produce a wireframe or specification to implement, a critical eye looking for accessibility issues can be very helpful.

## Working with Assistive Technologies

When the implementation of a new feature or story starts, the developer’s role often revolves around ensuring web pages remain compatible with assistive technologies, which could explain why AT and accessibility are often conflated. However, facilitating this compatibility is not purely the job of the developer. Especially when implementing web apps, particularly complex UX patterns may need specific alternative implementations and interaction methods, which could require further collaboration with an interaction designer to define how those interactions work in these constrained environments. But like addressing other accessibility issues, this can benefit all users too.

When British law was changed to mandate dropped curbs at pedestrian crossings, this was mostly done to benefit wheelchair users. However, it was appreciated by other unintended users too, such as parents with prams, or skateboarders, where the change in level between the road and the pavement was previously a challenge to overcome. Adding keyboard shortcuts to a web app that relies largely on clicking buttons and mouse movement can benefit power users, where the shortcuts can be learned to speed up common tasks.

You won’t be surprised to learn that progressive enhancement is an effective way of making web pages work well with assistive technologies. By starting with the base of a simple page and then layering on additional enhancements, you always maintain that accessible base, and all you then need to do is ensure that you don’t break the accessibility of the base with these enhancements. The other benefit to HTML being accessible by default is that if you use HTML in the way it was designed to be used, then those enhancements you bring in will also be accessible with little extra work.

The main thing to be aware of if you’re building highly customized interaction mechanisms (and this is often the case when you’re building a web app) is that you need to use some way of indicating to assistive technology what the underlying meaning of those elements are. A technology called ARIA can be used to do this, and ARIA is used by adding attributes to HTML to indicate what the roles of these custom elements are.

The reason tools like ARIA are useful is that many assistive technologies, especially those that support non-sighted users, work by evaluating the structure of a page. By default, these tools will understand what the default elements of HTML mean.

From a purely visual perspective, it can be tempting to make everything a <div> and apply styles appropriately. This can resolve issues with browser default styling clashing with a button or a heading tag, for example, but is actually an anti-pattern known as div soup (or tag soup), as all the information is now completely unstructured and hard for a tool to discern.

Semantic HTML is the name given to the technique of using HTML as it was intended to be used—headings should be marked up as h1, h2, h3, etc., and paragraphs as p tags. This should be common sense, but can be tricky to get right. For example, if you have something that visually looks like a button, and takes you to another page, it might be tempting to use a <button> tag to get the visual style of a button. This can cause issues, as people who navigate links with a keyboard (as well as search engines that might follow them) will now not be able to see a <button> as a link. This works both ways. If you have something that looks like inline text that triggers an action when pressed (but doesn’t actually navigate to a full page), that should actually be a <button> styled to look like a link. This is especially true when progressive enhancement is being applied, so that the <a> tag will correctly do a full page reload, even if it may be progressively enhanced to use AJAX to bring in a partial update.

There’s another type of tool that analyzes the structure rather than the visual rendering of a page: search bots. An accessible web site is often also “search engine optimized” (SEO). Although there are SEO techniques that involve more than just using semantic HTML, Google in particular will reward well-structured HTML pages.

However, a warning must be applied here. Unlike web browsers, AT tools are often proprietary and paid for, meaning there’s little incentive to upgrade, which would involve spending more money. For this reason, the understanding of brand new tags and semantics lags far behind the technology will support. For example, the defined semantics of a <section> were to reset any heading levels thus far, but support of this in screen readers is patchy at best. Although having a <h1> at the top of each <section> may appear to be fully compliant with the spec, it’s not backwards compatible with HTML4, and some screen reading software will interpret it incorrectly. This kind of use is now officially discouraged, but be wary when dealing with these kinds of structural elements, especially new ones that pop up.

There are also cases where a plain HTML element (even when styled) is unsuitable. Perhaps there is none that fully satisfies what you’re trying to do, so you’re building a component from scratch using buttons and divs. The Web Accessibility Initiative’s Accessible Rich Internet Applications specification (usually referred to as WAI-ARIA or just ARIA) can be a useful tool in this regard.

ARIA allows you to add hints that indicate what the semantics and structure of these elements are. Take, for example, a tabbed view on a web page. It is common to implement this with a list followed by a series of divs that are selectively made visible based on which list item is selected. By itself, this structure doesn’t give enough information in the HTML to do the right thing, and CSS is often used to style this to look like the familiar tab structure a user might expect. Similar to the way we use CSS to style the list to look like tabs so that the user knows they’re tabs, ARIA allows us to give the list a “role” so non-visual users also know their tabs. In the case of tabs, the list would be given a role of tablist by adding an HTML5 attribute of role, e.g., <ul class="tabs-list" role="tablist">. You would also give the link to each tab a role of tab and the divs containing the actual content a role of tabpanel.

The final missing piece is to describe the relationship between the divs and the appropriate item in the list. aria-describedby is another attribute that can be helpful here, and is used by adding to the content divs to reference the list item that describes this content. The downside here is that it refers to the other element by ID, and using the id attribute in HTML is often frowned upon (as it can limit reusability and cause issues with specificity in CSS), but careful choice of IDs for accessibility purposes can avoid any reusability issues and maintain manageable.

It’s worth noting that overusing ARIA can sometimes introduce more problems than it can solve. You can make an element perfectly accessible and sensible without having to give everything ARIA attributes. ARIA is most helpful when used to fill the gaps and add additional hinting when standard HTML structure fails.

There are many types of ARIA attributes that can be used to add semantic meaning and structure—many more than can be described here. Fortunately, there is a dedicated group of accessibility experts within the web development community who have published many guides and documentation on these. The most important thing to do is including testing with AT as you develop your web site, and then try to find an appropriate pattern to use to correct any accessibility issues you find.

## Dealing with Interactive UI

Although ARIA and semantic HTML will go a long way for content-style web sites, web apps are characterized by having a much richer type of interactivity. For assistive technologies, this can present a problem, as it’s no longer a case of simply analyzing the structure of the page and then navigating through it, because the structure is dynamic and changes.

The situation is not as bad as it sounds—AT tools do interrogate the DOM dynamically, so any changes are reflected in them. The main issues are around notifying users when an action or other important event has happened (this is often done visually, such as with a validation error below a form field), making sure that you can trigger all the interactions you need to using tools such as a keyboard, and ensuring that if the content of the DOM changes, the user doesn’t lose their place.

To deal with dynamic changes to content, we can add additional attributes to our content to indicate which elements are dynamic and give context to screen readers to help users navigate them. There are two attributes here that are of use: the role attribute and aria-live. role is preferred, sometimes along with aria-live for compatibility reasons, as it gives a better idea of context: aria-live simply says “this element will update,” whereas the role says what type of information is being presented.

The following roles are useful to indicate content areas which can change:

- role="alert"
    
- role="status"
    
- role="log"
    
- role="timer"
    
- role="progressbar"
    

alert is probably the most common role, and is read immediately when the page loads, an element that contains it is added to the DOM, or the attribute is added to an existing element (for example, highlighting instructions that may have been missed while completing a form). Its most common use is for error messages—for example, during form validation, or after being logged out due to inactivity.

Many screen readers allow users to check the current status of a page, so role="status" may be appropriate for loading indicators, or perhaps features like Google Docs’ “Saving... / Saved” indicator. For a loading bar, the progressbar role might be more appropriate. This role by itself is not enough, and should be combined with more descriptive attributes aria-valuemin, aria-valuemax, and aria-valuenow. aria-valuetext is also available for giving more detailed information about what is happening at each stage in a multi-step process.

The log role is used for streaming content, such as chat rooms (or even logs in a developer tool), where any added content is read out at an appropriate time.

The final role to discuss is timer. Most screen readers will not read content annotated with this role when it changes unless explicitly asked to, as it is assumed it will change fairly frequently and, as its name suggests, this is most appropriate for things like displaying the current time, timers, or countdowns.

aria-live has three possible values: off, polite, and assertive. off indicates that the region is not a live one, whereas polite means the screen reader will announce the updates when it next pauses. assertive will interrupt what is currently being read out to the user, so should be used sparingly. aria-live can be combined with the attributes aria-atomic and aria-relevant to control whether or not the whole element is re-spoken on change, or which parts of it are the most relevant.

All of these should be combined with the aria-describedby and other roles described previously to give an adequate level of coverage for these dynamic elements should screen readers fail to understand them by themselves.

Some HTML elements have an implicit ARIA role assigned to them—for example, the <progress> element will behave as if it has role="progressbar" set without anything further being done, as well as the other aria attributes that can be derived from the regular HTML attributes.

A common interaction mechanism for moving through web pages is to use the tab key to access elements such as form fields and buttons. There’s a good chance you use this while filling in form fields yourself. When an element has been tabbed to, it is said to have focus. This can be thought of analogous to hovering over an element with a mouse, and it allows AT tools to indicate where a user is on a page. If you’re using standard HTML components, then most interactive ones can be tabbed to and interacted with directly without any further work. However, if you’re using elements such as divs, then you can use the tabindex attribute to indicate that the element should be able to be selected with the tab key.

The tabindex attribute is given a numeric value to indicate where it should appear when the document is being tabbed through. You should always set this to 0 to make an element be able to be tabbed to, which simply means that the order should be as it would naturally appear in the document (conversely, you can hide an element that is normally accessible by setting the tabindex to -1). Other numbers are possible, but this can introduce subtle bugs and dependencies with other fields (for example, if you add an element at the top of the page, you’d have to increase the tabindex of everything that comes after it, as they must be sequential). It is thought of as an anti-pattern if explicit tabindexes (i.e., not 0) are needed, as this often means that your page structure does not match the visual flow, and AT tools can get very confused if this is the case. There should be a strong link between the structure of your HTML and the visual flow of the page and the information.

You should also ensure that you listen to the appropriate events in JavaScript. Most important are the focus and blur events, which you should use if at any point you’re responding to mouse events like onmouseover. By listening to these events, you’re ensuring that, for users who navigate with a keyboard, the components respond in the same way as those who navigate with a mouse. This type of keyboard focus has equivalent pseudo-selectors in CSS too, like :hover and :focus. Of course, with the rise of touchscreen devices, relying on mouse hover and similar events is decreasing in popularity. Instead, listening for explicit touch or click events is now a common interaction mode, but fortunately selecting an item with the keyboard still fires the click event, which can simplify the logic.

The final aspect to consider, especially for screen readers, is making sure it is clear what a button does. Links such as “click here” give very little indication to the user what the link actually does without the visual context of what surrounds it. “Submit” is not a very good name either, so it is often useful to make a form’s submit button explicitly name the action it will complete (such as “Add Product”). This is especially true when there are multiple buttons with the same name on the page, perhaps for different forms, because a screen reader may not have a sense of location on the page, so it can be unclear which form the button is referring to.

Icons are popular to use in buttons, but even for users who browse without AT, these can be confusing without context. As a result, it is very common to have an icon and a textual description of the action side by side for all users, which then makes the icon a decorative element that is not read out for screen readers. A common way of making it a purely decorative element is to use a CSS pseudo-element with a background-image element to bring the logo in, or an img tag with an empty alt tag to denote that it is purely decorative.

In the event that you decide an icon by itself is enough, then you must make a textual alternative available. Be careful when you do this! A lot of research has been done into icons and has found that, with the exception of a few very common ones used in a familiar context, icons can be confusing to users. Take, for example, the cross symbol. If this appears in the top right corner of a popup, then this is commonly taken to mean close the popup. But using the same cross icon as a button in the list item might feasibly mean “delete this item from the list,” or perhaps “disable,” or “discard changes,” so even the subset of universally recognized icons are only identifyable in an appropriate context.

### Having Text only for Screen Readers

There might be other use cases where you want to make a textual alternative available for screen readers only. For example, sometimes you want the contents of a h1 tag to be the logo or name of your site, but using an img tag can have negative consequences for SEO, as often the alt tag will be ignored, making it look like you have an empty h1. A common technique to avoid this is to specify your h1 like normal, but with a nested span inside:

<h1 class="site-title">

  <span class="site-title__inner">My Site</span>

</h1>

CSS is then used to hide the inner span and a background-image used on the h1 to insert the logo image. However, the way in which you go around hiding the text can result in it being hidden from screen readers too, if care is not taken. A naive approach might be to give the inner span a display: none;, but this also hides the element from AT tools and CSS-aware search engines (which include Google). The traditional approach is to give the h1 element a CSS attribute of text-indent: -9999px; which will push the text out of the viewport, but still show iton the screen, so screen readers (and bots) will still find it. Note that if the inner span bothers you, you can achieve the same result using CSS pseudo-elements.

The text-indent method is not without its flaws. The first is an impact on performance, as it causes the browser to render the text off-page, increasing the area the browser has to render. Some people also jokingly refer to the “10,000px-pocalypse,” where once the average screen size is over 10,000px, the hidden text will start to reappear. More realistically, if your site has any horizontally scrolling components (for example, a slideshow/carousel), then this technique can fail.

Instead of the text-indent method, a more modern alternative has sprung up, which instead uses a clipping rectangle and resizes the container to hide the text from view while keeping it in its present location in the DOM, avoiding the performance hit. This can be implemented using the following snippet of CSS:

    .sr-only {

        clip: rect(0, 0, 0, 0);

        clip-path: inset(50%);

        height: 1px;

        overflow: hidden;

        padding: 0;

        position: absolute;

        white-space: nowrap;

        width: 1px;

    }

Unless you need to support very old browsers, this is a better method to use, and many CSS frameworks will provide this as a utility class (often called .sr-only for “screen-reader only.”)

If both the 1px box and the text-indent methods feel like hacks to you, that’s because they are, so overuse of these methods can be indicative of accessibility issues in your site. Be mindful when using them.

Screen readers and other AT tools use the concept of focus to keep track of the current position of a user who is navigating a page. Focus can be thought of as analogous to scrolling through a long web page, where the current position in the scroll indicates where a user currently is.

It is common for interactive elements to make changes to the DOM in response to user events, such as adding a new item to a list, or opening/collapsing trays. The techniques used to notify users of messages and contents will work in that case, but less well when the change is not actually an announcement or message.

For users of these tools, doing an action such as toggling open a list or drawer seems the same as clicking a button where the effect happens offscreen—it is not immediately obvious to the user what has just happened. Fortunately, we can manipulate focus using JavaScript to move users to the modified area of the page, so their flow is not broken. The focus() method on an HTMLElement allows us to change the focus.

We also need to consider cases where the opposite has happened—an item has disappeared from the page. Often, there is little to do here beyond a success announcement to ensure that the user knows an action has been successful, but there are cases where it is not so simple. When the element that is currently in focus is removed or hidden from the DOM, then the browser is said to have lost focus, and resets it to the start of the page. This can be a very jarring experience for screen readers.

Take, for example a drawer that has a “close” button inside. When close is selected, then the drawer is hidden from the DOM, including the close button that was inside it. This causes the focus to be lost. The same technique of deliberately setting focus can be used here to avoid that jar. For example, you may want to set focus back to the open button, or instead to the next element in the list, depending on what you’re trying to achieve.

The final thing to consider is something known as a keyboard trap. A keyboard trap occurs when a user cannot leave an element using the keyboard and gets “stuck.” This can be both a blessing and a curse. One way this could happen is if you re-map the tab button to some other purpose, or if your app has a bug that constantly resets focus to the same element. This is very frustrating for the user, as it keeps them from fully exploring the page. Conversely, a keyboard trap can be useful if you have a modal dialog, as you can trap the keyboard user into that dialog, similar to the way you might block UI interaction with the page.

Most controls will behave in an accessible way by default, but if you want to implement your own custom behavior, the above gives an overview of how to start considering accessibility. However, it is far from comprehensive; there are many guides online that provide the patterns you should subscribe to, with ARIA specifications considered the most definitive and widely respected versions.

## Testing for Accessibility

Of course, compliance with the ARIA spec is not enough to make a web page accessible. You should also test how your web site actually works and performs with AT tools.

Testing for accessibility is similar to testing a web site normally. Instead of simply using a browser to navigate the site, a screen reader or similar tool should also be used, so interactions are executed in the same way a real user would. And, as a developer would always do a sanity check of a feature on their machine before handing it over to a QA, it is your responsibility to do sanity checks for screen readers too. Like normal web browsers, AT tools have varying levels of support for standards, as well as per-tool quirks and bugs. These will often require workarounds and shims in order to give the user an acceptable experience.

As a web developer, you should learn how to use a screen reader to complete basic actions to navigate your web pages or app. Fortunately, there are many introductory articles on how to do this. If you are using macOS, then this is a built-in feature of the operating system known as VoiceOver. Pressing ⌘ + F5 will toggle this feature on and off, and you can then use a keyboard to move around your web browser. On Windows, the applications NVDA and JAWS are popular.

Once you are familiar with a screen reader, you can then do some ad hoc sanity testing on accessibility/usability before a QA might do a fuller test. The simplest thing to do is run through and make sure your page makes sense when read out loud; that if you use your keyboard to navigate to an action, it’s obvious what that action will do; and that performing an action ensures the screen reader matches any actions that a user of a regular browser will see.

Once you are sure that your content is readable and actions are properly marked up to behave correctly, the final step is to check for “traps” when navigating with a keyboard. Especially if you are manipulating focus, or showing/hiding things on keyboard focus, it can be possible to get into a loop with keyboard focus where you cannot escape or go any further to access other parts of the document.

Beyond these sanity checks, there are specific sets of guidelines for testing accessibility. These guidelines by themselves are not sufficient to ensure a web site is accessible. Web Content Accessibility Guidelines (WCAG) is the most common one, and there are automated tools that can check compliance with these guidelines. However, like usability testing in general, these tools have not evolved to the point where they can replace a human, as they do not understand the context of your application. These checkers will almost certainly miss some issues and misdiagnose others, so cannot be used as an alternative to manual checking.

A standards checker can tell you that your web site has well-formed HTML, but it won’t tell you if your design intent will correctly render in all the user’s browsers. Accessibility tools have the same problem. What they can do is check that any programmatic hooks are in place, but you should use these alongside actually doing testing using accessibility tools like screen readers to ensure full compliance. Pa11y, released by _Nature_, is one such tool you can run to get a report on how compliant you are with various specifications.

One good side effect of building an accessible web site, with good programmatic hooks for screen readers, is that it actually makes the general automated testing of your web site or web app much easier. Automation tools like Selenium can control a browser in a very similar way as browser accessibility tools (they often go above and beyond this too, such as with JavaScript hooks), but if you have a component that is hard to hook into for tests with Selenium, there’s a good chance that the component is also not compatible with AT either.

## Avoiding Common Mistakes

It can sometimes be surprisingly easy to accidentally break accessibility. The final section in this chapter looks at some of those common mistakes and how to avoid them.

### Hover and Focus Styling

When using the CSS pseudo-selector :hover to style an element while a mouse is hovering over it, you should almost always use the :focus pseudo-selector as well, so that keyboard users get the same behavior to indicate when something is highlighted.

### outline: 0

Quite often, a bug will be reported along the lines of “an ugly ring appears outside the element when it is clicked on.” This is the outline ring that indicates which element has keyboard focus, and outline: 0 will hide it, sometimes making for more pleasing aesthetics. However, this can make a site completely unusable for a keyboard user, as they cannot see where the site has focus. The “ugly ring” appears on many different web sites, so users are often used to seeing it. Getting rid of it in the name of aesthetics is often unnecessary! If you absolutely must get rid of the outline, then make sure that some other way of indicating focus is included, but bear in mind that users are familiar with the outline ring, so another way of indicating focus may not be as effective the default outline ring.

### The Order of Headings

It is easy to fall into the trap of making the structure of your HTML match the visual structure, but headings are an important navigational aid. In particular, heading levels should be used in order, rather than skipping or re-using levels to match a visual style. Going straight to a h4 when the previous heading element was a h2 can confuse screen reader navigation. Another common anti-pattern is when a card starts with an image:

<div class="card">

  <img src="/contents.jpg" alt="Photograph of John">

  <h3>John</h3>

  <p>John is the marketing director of FooBar Inc.</p>

</div>

From the heading structure alone, it is not obvious that the image actually belongs under the heading of “John.” Instead, the <h3> should be the first thing in the div and the image below that. A padding-top in CSS can then be used to make space for the image, with the image being absolutely positioned at the top of the card to make the visual style correct. This might seem counterintuitive, but for users of AT who navigate using heading levels, it’s the only way to correctly indicate the structure of the document.

### **Multiple** **h1** **s**

The HTML5 spec previously specified that using some of the new semantic elements, like <section>, reset the meaning of the heading hierarchy, but screen readers have been slow to adopt these semantics. For consistency it’s best to ignore this and assume a global hierarchy for the heading structure. In particular, there should only be one h1 per page, as this is often used to jump to the main body of a page.

### Skip Links

One common problem with web page design is that the first elements on the page can be navigation bars, logos, etc., which is a lot of cruft for an AT tool user to navigate through to get to the actual content. Although skipping to an h1 can be a way to get around this, another way is to use a pattern known as a “skip link.” A skip link is an anchor link that appears close to the top of a page and allows a user to skip to the main content of the page. Quite often, you will want to make this anchor invisible to non-keyboard users for aesthetic reasons, and a common pattern to solve this is only use a :focus pseudo-selector in the CSS to hide it off screen until it has keyboard focus.

### Buttons vs. Anchors

When creating dynamic elements, it can be tempting to use <a> tags and add click event handlers to it in JavaScript. This can be jarring for users of AT tools, as the usual semantics of an <a> tag take you somewhere else (a different part of the same document, or another document altogether), as opposed to a <button>, which is used to trigger an action.

When implementing actions, using the right HTML element can help screen readers interpret the page as well as suggesting the correct semantics, although styling a button can seem like more work than styling an a tag. The use of <a href="#"> is an anti-pattern to be avoided, as the href indicates that a link goes nowhere, which suggests that the link is actually an action, and should be a button.

### The Correct Use of an **alt** Attribute

The use of alt attributes on images is a rare accessibility requirement that is hard-coded into the HTML spec, and that a standard validator will check for. Despite this (or because of this), inappropriate use of the alt attribute is common.

alt attributes should only be used when an image adds content (not just aesthetics) to a page, and as the name suggests, it should be a _textual alternative_, not simply a description of the image. If an image exists only to add some styling information to a page, then it is often best to actually include it as a background image in CSS, rather than as an <img> tag. Although alt attributes are compulsory, it is okay to leave them empty to indicate that the image is purely for styling reasons. It is particularly egregious when an alt attribute simply repeats a heading, as all this does is cause duplicate text to be read out in a screen reader.

### Icon Fonts

A common optimization for icons is to deliver a set of icons to the browser as an icon font, using Unicode or other characters to indicate custom characters, which are then typed into the document. However, a screen reader does not know how to “read” these characters, and may cause them to read nonsense when interpreting a document. Instead of using icon fonts, CSS sprites or other techniques are a better way of including an icon, either with the use of hidden text for screen readers to give textual meaning to the icon or, better yet, using a text label alongside the image.

### Color Contrast

The final common mistake to look out for is color contrast. Although this is often considered to live in the realm of visual design, the way colors are implemented can make a big difference to the accessibility of a page. It can be easy to assume that when text is on top of an appropriate background image, that the color contrast is fine, but in the case of an image failing, or being slow to load, a background-color attribute should also be set, alongside the background-image, that should give sufficient contrast to the text on top of it.

Often, when white text is used on a dark background image, and the default background color of the page is white, then at first white text will render on top of the white background until the image loads, which makes for a less-than-optimal loading experience. Similarly, the use of color alone as an indicator of something is not reliable, as color blindness is fairly common. When color is used, then another way of indicating what the color means is also needed (for example, text or an icon).

## Summary

As a professional, you have an obligation to build your products in a way that doesn’t discriminate against those who use assistive technology. Fortunately, the Web and browsers have been built in a way that reduces the level of friction needed to support accessibility tools beyond what native apps can offer. Bending web standards, or implementing completely custom components out of generic HTML elements, can lead to breaking compatibility with assistive technologies, potentially alienating a sizeable portion of your user base.

There are many different types of accessibility to consider, some of which use tools such as screen readers to deliver experiences, and others that are solely implemented in browsers and designs. It’s important to treat these AT tools similar to other web browsers, and correct bugs that are found in them, as well as testing your site using interaction mechanisms beyond the standard mouse or touchscreen.

Like many cross-functional requirements, accessibility should be built in from the start of a project, and there are several techniques to address this, both for structuring a document using semantic HTML and setting up JavaScript event handlers for particular event types and considering other flows of engaging with interactive elements. In particular, HTML can be augmented with ARIA attributes to provide hints to AT tools when non-standard HTML is used, and JavaScript should handle focus in the same way that it considers click- or touch-based interaction methods.

If you remember one thing from this chapter, it should be to test your site in the way that many of your users will be interacting with it, either with a screen reader or with keyboard shortcuts, as this will identify the biggest issues preventing your site or app from being accessible.