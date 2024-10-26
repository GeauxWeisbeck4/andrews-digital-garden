---
id: 01JB58FGP72PH2RK6TD4R84QTM
title: Chapter 6 - Front End
modified: 2024-10-26T16:35:43-04:00
tags:
  - full-stack
  - books
---
# 6. Front End

Chris Northwood[1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#Aff2) 

(1)

Manchester, UK

Different teams will often have very different ideas of what part of their app is the front end. For some teams, the front end is literally just the HTML, CSS, and JavaScript that makes up their app. For others, this can include the logic and server-side code that generates the HTML too, with the “back end” being simply the APIs. So, to avoid ambiguity, this chapter defines the front end as the bit of your server that generates the HTML, as well as any code that runs in the browser.

There are three key technologies to keep in mind when it comes to code that runs in the browser: HTML, CSS, and JavaScript. JavaScript is such a large topic that it is broken out into its own chapter, but below we cover HTML and CSS, which are specification languages that specify the structure and presentation of your web page. JavaScript is a fully specified programming language that allows you to add interactivity to the browser and manipulate the structure and presentation.

For a back-end developer coming to front end, everything can seem a bit too foreign. All of a sudden, you’re using new languages and new toolchains, and you have to target multiple runtimes. But it’s important to remember that any back-end techniques you know and love can apply to the front end too. Don’t panic; just start copying snippets of JQuery from Stack Overflow into a single .js file, as you’re writing legacy code from the start.

For the front-end developer, this might feel like home turf, but there is much to be learned from the techniques that may traditionally live in the domain of the back-end developer.

## HTML

Hypertext Markup Language is the language at the core of the web. It is not necessarily a programming language for building applications, but instead a language for describing documents. There are two fundamental approaches to expressing structured documents in software engineering: markup and standoff. In markup languages, like HTML, the structure and annotation are inline with the text and contents of the document.

HTML annotates sections of text with tags, denoted by angle brackets that give the corresponding section a meaning, or as a way of embedding non-text elements within a page. Standoff annotation mechanisms instead leave a document as is, and then have a second file that describes the structure. For example, you may have a plain text file containing the text to be described, and then a second annotation file describing things like “the characters between positions 112-118 are bold.” The benefit of markup over standoff is that keeping the document and the annotations in sync is much easier, as they can be manipulated as one, but the downside is that if you want to have text in your document that happens to look like a tag, you must encode (or escape) that text to indicate it should not be treated as a tag.

Hypertext is a specific form of a technology called hypermedia, which was of much interest to researchers in the 1990s when HTML was developed. Hypermedia refers to any non-linear media—for example, a video that you can navigate in any order you want, rather than following only one path. The navigation occurs by following hyperlinks, which are references to other documents or bits of media. Tim Berners-Lee thought of this as a web of documents, hence the term World Wide Web.

This book will not teach you HTML, but there are many other good resources out there, and it is important to be aware of the basic principles when building an HTML page. HTML tags are used to express the structure of a document, and although they do also imply a specific form of rendering in a browser, this structure and meaning are also used in other ways—search engines, accessibility tools, and more rely on this structure being in place and being meaningfully applied to a document in order to correctly interpret a web page.

## From Server to Browser

At the core of any web site or application is some way of creating HTML and delivering that to the user via a web browser. This used to be done by hand-writing some HTML and just serving those as files from disk, and this is still a valid technique today, but it limits the amount of “dynamism” available, as the same page must be served to everyone. Although JavaScript can be used to add some interactivity and dynamism to the client’s browser, it is very common for the actual HTML that is delivered to the browser to be generated on the server side in an application in response to a request, rather than simply serving a static file from disk. Even when a dynamic application is not required, it can be useful to generate static HTML on disk by assembling them from common templates on disk, using a tool known as a static site generator (one example of this is Jekyll). These tools work similarly to the dynamic server-generated code, but instead generate all possible outputs up front, rather than on request.

When constructing a web page, you will often encounter different concerns for various different parts of the page. For example, a common navigation bar or footer might be the same on every single page, but other components might need some information from a database, or another system, to be inserted. For static components, like the footer, it is very common to simply write fragments of HTML that are assembled together by the web browser, but for dynamic components that can change, there is a dazzling array of libraries and techniques for doing this. To further complicate things, the dynamic content in the database might also hold its own styling data, in addition to how the component which presents that data is styled—this is referred to as rich content.

The approaches for building an HTML document server-side generally break down into the following techniques: auto-generation, tree transformation, and string interpolation. There is some crossover between these three techniques, and they can be combined.

Auto-generation happens when an underlying library automatically creates the HTML for you, perhaps by directly serializing a class. Auto-generation can often lead to problems and should only be used in very small doses—for example, if you’re using a tool like Django’s forms, which renders out a model to a simple HTML form. With auto-generation, you often have limited control over what the resulting HTML looks like, and as that HTML is your integration point between not only your content, your styling, and your front-end logic, but also your users, it’s important for you to control it. The method is quick, and can seem useful if you’re working in a “full stack” framework, but as discussed in the Designing Systems chapter, it’s often better to pick multiple smaller tools and wire them together to make something more than the sum of its parts than it is to use a one-size-fits-all solution. Auto-generation can be considered an anti-pattern, something that seems good at first and will help you move quickly, but as time goes on will cause you an increasing amount of pain.

One exception that is worth calling out is Facebook’s React. Although React takes an auto-generation approach, using the JSX extension to JavaScript, there is a strong link between what you write and what is output that mitigates most of the downsides and offers considerable upside. Indeed, when using JSX, React appears to have more in common with a string interpolation technique than a traditional auto-generation approach.

Tree transformation is a technique that takes content or a template written in some other non-HTML language and translates it directly into an HTML tree, without going through an intermediate stage such as being parsed with classes into models. Sometimes the source document is itself represented as a tree (like XML or HAML), and other times it’s just flat content (like Markdown or BB Code).

### What is the HTML Tree?

The term “tree” comes from computer science, referring to a particular type of data structure. A tree consists of a set of nodes and a set of pointers associated with those nodes. The root of the tree is a single node that has pointers to other nodes, and these pointers are known as branches. Eventually, a node does not have any pointer and this is known as a leaf. Additionally, no node can point to a node that another node points at, so there are no loops, or nodes on multiple branches. If you sketch out a diagram of this, you end up with something that looks like a tree (although often computer scientists start with the “root” at the top, so you get an upside-down tree). Relations between the nodes in a tree are expressed in the same terminology as in human ancestry, as is shown in Figure [6-1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#Fig1). The node that points to another particular node in the tree is called the parent, and the nodes it points at are the children. Nodes that share the same parent are referred to as siblings.

![../images/471976_1_En_6_Chapter/471976_1_En_6_Fig1_HTML.jpg](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_6_Chapter/471976_1_En_6_Fig1_HTML.jpg)

Figure 6-1

A tree structure. From the point of view of the node shaded black, the parent and children are labeled.

HTML conforms to this tree-like structure. An HTML element can have other elements nested inside of it, which you could call the children of that node, but this reflects the branches in a tree. Take for example the following HTML:

<html>

    <body>

        <div>

            <h1>An HTML tree</h1>

            <p>Hello world</p>

        </div>

    </body>

</html>

If you represented that as a tree, as in Figure [6-2](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#Fig2), you would end up with a structure like this:

![../images/471976_1_En_6_Chapter/471976_1_En_6_Fig2_HTML.png](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781484241523/files/images/471976_1_En_6_Chapter/471976_1_En_6_Fig2_HTML.png)

Figure 6-2

An HTML document structured as a tree

Trees are common data structures, and other syntaxes for writing HTML can also be represented with this kind of nesting structure, which can be transformed into a tree and then output as HTML.

For dynamic rich content, tree transformation is a powerful technique. It often does not make sense to store content in a database as HTML, as this can leave it brittle or linked to specific implementation details, such as CDN URLs or CSS class names, that are hard to migrate. Instead, you should consider using another markup language (such as Markdown, BBcode, or wiki syntax), which is then translated to HTML at render time. For simple cases, where the source syntax is well-known, there are many libraries that can help you do this. The downside here is that you often have limited ability to hook in any extra HTML attributes (such as class names), which can make writing CSS selectors or JavaScript against those elements tricky.

Although there are many libraries that will do the translation for you, there are others that require you to implement the translation rules yourself. The most well-known example of this is XSLT, which has native support for translating XML into HTML, although the exact matching rules have to be written in “XSL,” or Extensible Stylesheet Language (which itself is XML). Although XML has developed a poor reputation, generally due to inappropriate usage or over-complicated schemas, it is very effective at certain tasks. XML itself grew from the HTML standard as a more generic language, and there was even an aborted attempt to deprecate HTML and replace it with an XML-based variation known as XHTML.

XML is very adept at annotating text inline, and other formats, such as JSON, can make the process clunky. Creating your own XML schema—with tags that have semantic meaning in your domain—and storing it in your database can sometimes work better than storing HTML. In this scenario, when this content directly maps to the HTML output, XSLT should be seriously considered. Listing [6-1](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#PC2) shows how you may store content as XML in a database, but then map it to some output HTML as shown in Listing [6-2](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#PC3).

<article>

        <paragraph>This is some example content.</paragraph>

        <image id="a9be99f7" alt="Example" />

</article>

Listing 6-1

Sample XML that refers to an image by ID

<article>

        <p>This is some example content.</p>

        <img src="https://cdn.example.com/images/a9be99f7.jpg"

           alt="Example">

</article>

Listing 6-2

An example of what that XML may be translated to when rendered to HTML

You may consider storing the HTML directly, but this can be problematic. For example, if your CDN URL changes, you may then have dead image links in your database, or if you refer to CSS classes, it will be harder to refactor your CSS without breaking content.

There are scenarios where content is not stored as HTML, but where direct tree translation isn’t the best approach. For example, instead of writing something that translates JSON directly into HTML, it can be beneficial to load those into models that are then passed to the view. Then, string interpolation is used.

Although tree transformation is an effective technique for handling dynamic and rich content, it is less effective when it comes to managing static components. Template languages such as HAML or Jade have become common as a mechanism to write an abstracted version of HTML, which is then translated into HTML at render time. Although this has the upside of enforcing the validity of the generated HTML, there are often downsides. The most important one is that it adds a layer of indirection between the code you see in the browser and the code you see on disk, which can make debugging harder. Often, these languages make inline tagging (such as annotating a single word in a sentence) clumsy, as shown in the following example, as well as hiding important HTML details, such as whitespace collapsing. In complex systems, it is often necessary to fall back to inline HTML in addition to HAML, which can clutter your code, and styles of different engineers between two different languages mixed together can conflict.

%p

  This is how you insert

  %a{:href => "foo.html"} a link

  with HAML.

Given these downsides, and the limited upsides of using such a tool, it is best to avoid this kind of HTML generation style for your actual templates. If you’re especially worried about the risk of introducing invalid HTML into your codebase, you can incorporate HTML validation into your automated tests (and it is probably a good idea to do so anyway) to still reap the benefits of these transformation layers.

String interpolation is by far the most common way to create templates. This technique works by taking a typical HTML file and adding a special type of tag that is interpreted by a library. This template is rendered by passing a number of objects to a renderer, as well as the template name, and these can be referred to within these tags—for example, calling a method on an object in order to return a string that is echoed into the view.

<article>

        <h2 class="title">{{ title }}</h2>

        <p class="abstract">{{ abstract }}</p>

        <a href="{{ link.href }}">{{ link.text }}</a>

</article>

PHP deserves special mention here, as the entire language is based on the idea of string interpolation. However, in an MVC world, mixing the domain logic with the templating language can make the code harder to maintain due to working in different domains in the same file. As such, many modern PHP frameworks actually come with their own templating language, rather than using PHP’s support for this interpolation directly. This introduces an artificial constraint on how much logic can exist in the template to enforce these good practices. Most other web frameworks in other languages will come with their own templating languages too, such as Ruby’s ERB in Rails, Python’s Jinja in Django and Flask, or Handlebars in JavaScript. Although the exact syntax may vary between these templating languages, the overarching principles remain the same.

Although it is common to hear that templates should not contain any logic, this is impractical in most real-world applications, and most templating languages include simple logic, such as if statements and loops. Even with these limited constraints, it can be easy to introduce significant logic to a template, violating the single responsibility principle and making that logic hard to test. These should be refactored into a more appropriate place such as the controller (for example, computing the value of an if statement and passing the result through as a boolean to the view), or on the model or view model being queried.

Another common feature of templates is the ability to include other templates. This is a powerful technique that can help with code organization and minimize re-use. Although it may not seem like it, many refactoring techniques for logic code also apply here. A particularly long page can be broken down into a series of smaller templates, even if they are not reused, which can make the intention and structure of the page easier to see.

One important feature of some templating languages is support for template inheritance. Template inheritance is somewhat similar to inheritance in object-oriented classes. A base template will typically define a full HTML document, with some common header/footers, and then will include gaps, which the individual templates can fill with their content (typically the page body). This can be a good technique for reducing duplication, although in template languages that do not support this, a common solution is to have “header” and “footer” template files included at the top and bottom of the file to provide that level of functionality.

In addition to generating HTML in a server-side application, or all at once with a static site generator, there is a technique known as server-side includes, or edge-side includes, that will assemble pages from individual components for presentation to a user’s device. In server-side includes, the server that reads a file from disk will parse it and dynamically include another file to create a new composition to the user in response to a directive in the file. In edge-side includes, this is done using an intermediate proxy server. This approach can be powerful if you want to decouple the applications that produce the different components of a single web page, and also include different caching rules for each component.

## Styling

HTML specifies the structure of a web page, and CSS (Cascading Style Sheets) specifies the styling and presentation of it. At the core of CSS are selectors and rules. The selectors specify which HTML elements these rules apply to, and the rules describe the value of the presentation properties that get applied to any element that the selector matches.

        .header {

            background-color: #eee;

        }

The preceding code is an example of CSS. The selector is .header, which indicates that the rules in the block below should apply to any object with a class of header, and that it should set the background-color property of those objects to #eee, which is a way of expressing a light gray color. The syntax of pure CSS is very regular and follows this format, and this book doesn’t aim to cover the whole range of CSS properties and values, but the Mozilla Developer Network (MDN) is a good resource that does cover the whole range.

In addition to classes, CSS selectors allow you to target elements by their tag name (e.g., with a selector of p); an element with an ID (#body for an element <div id="body">...</div>); or by attributes ([href] for all elements with an attribute of href). A special selector of * exists that applies to all elements, and attribute selectors can also use operators to test the contents of a value, not just the presence:

- [attr="foo"] where an attribute equals the value foo (e.g., [data-languages="en"] would match <span data-languages="en">)
    
- [attr*="foo"] where an attribute includes the substring foo (e.g., [data-languages*="en"] would match <span data-languages="german english">)
    
- [attr^="foo"] where an attribute starts with the substring foo (e.g., [href^="https"] would match any secure link, but not one that might have https elsewhere in the path)
    
- [attr$="foo"] where an attribute ends with the substring foo (e.g., [href$=".pdf"] would match any link to a PDF file)
    
- [attr~="foo"] where an attribute includes the whole word foo (e.g., [data-languages~="en"] would match <span data-languages="fr en de"> but not <span data-languages="english">, as in the latter en is not the whole word)
    
- [attr|="foo"] where an attribute is exactly foo, or a foo followed by a hyphen and some other text (e.g., [data-language|="en"] would match <span data-language="en-GB">)
    

Any attribute operator can also be used case-insensitively by adding an i before the final bracket (e.g., [data-languages*="en" i] would match <span data-languages="English">).

Selectors can also be nested—for example .body p says to apply to all <p> tags that occur under an element that has a class of body, and combined, for example, p.author applies to <p> tags that specify a class of author. An HTML tag can carry multiple classes—for example, p.author would still apply to <p class="author large">, as would p.large and p.author.large. Nesting and combination can be used together—e.g., .body p.author for all <p> tags with a class of author that have an ancestor in the DOM somewhere with a class of body.

Methods of nesting can also be made more specific. In the previous selector of .body p there can be many elements nested between the element that has the body class and the p, but using the operator > we could say .body > p which means only the p tags that are directly children of a tag with a class of body are targeted.

Other operators target siblings in a DOM tree. Take, for example, the following HTML:

<div class="article">

  <p>some introductory text</p>

  <img class="figure" ...>

  <p>Some more text</p>

  <h3>A sub section</h3>

  <p>Line 1</p>

  <p>Line 2</p>

</div>

<p>Continuing text</p>

If we had a selector in our CSS of .article img + p, then the p that immediately follows the img would get styled according to the properties specified in that definition. The other <p>s would not, in contrast with .article h3 ~ p. The ~ here means to style all subsequent siblings, so the two lines that come after the h3 heading would pick up that style, but not the line that comes after the div, as moving up a level of nesting means they are no longer siblings (the parent DOM node is different).

CSS also allows us to target “pseudo-classes,” which corresponds to how a state may change. For example, a:hover would apply when an element is being hovered over (for instance, changing the style of a button). There are many pseudo-classes, and online sources such as the MDN cover the full list. Similarly, there are pseudo-elements, the most famous of which are ::after and ::before (for compatibility reasons, these can be specified with one colon, like pseudo-classes). These pseudo-elements refer to elements that do not exist in the DOM, but can be made to behave like them—for example, :before can be combined with a background-image to make an icon appear inline that is purely defined in CSS. A detailed description of these is beyond the scope of this book, but there are many good reference materials on how to use them.

It is the “C” of CSS that causes issues. Multiple style sheets can apply to a document, and if there are multiple selectors that target a specific instance of a HTML tag on a page, then the cascading aspect of CSS comes in and a set of weightings is used to determine which property should “win” and be the one that is actually applied.

All browsers have a default style sheet, which causes a page to at least have some styling (default fonts, different sizes for headings, etc.), rather than just render a blank screen when a document is loaded, but this will always lose out to any style sheets specified by the site itself. The original intent was for users to be able to specify their own stylesheet that would override anything else (for example, to increase font size for those with vision impairment), but this was never well used in practice and ended up being over complicated. One of the main issues with CSS is that it can make scoping hard, so it’s easy to add a rule that has an unintended side effect on another component, or an odd interaction. These types of bugs are generally referred to as specificity bugs, and there are a number of techniques to deal with these and to minimize the risk of introducing them in the first place.

Another confusing part of CSS is that some properties not only apply to the element that the selector specifies, but also any elements nested inside it. This can be useful in avoiding large amounts of duplication—for example, if you have a heading and a paragraph inside a div, then you can set a font color on the div and it will apply to the heading and the paragraph, rather than causing too much duplication, but this can sometimes cause unexpected side effects.

CSS also lacks the ability to directly express certain concepts in a straightforward way—the most famous example of this is vertical alignment of a series of blocks (although this has improved somewhat). These concepts have to be expressed by building the desired effect out of a more fundamental set of properties, which can lead to common code patterns (sometimes called “CSS hacks”) that can make it hard to see exactly what the direct effect of each property is, as it is the combined effect that is wanted. Another fundamental limitation is the inability to write a selector in a form that says, “apply to this element, but only if this element has a child node that this selector matches.” That is to say, you cannot write a selector like .product:has-child(.price), which would style an element with class product if it had a child class price. This is due to CSS being applied in a top-down way. If you could style a parent based on its children, then the renderer would have to look ahead to see which elements are coming up in the render, which has a significant performance overhead, or go back and re-render the parent when the child is being rendered. The most common workaround for this is to have classes like .product--has-price added in the server-side code and styling that, rather than attempt to do it in pure CSS.

These restrictions are a source of frustration for many developers, but the simplicity of CSS means that there is not much to learn, even if some of it is not obvious. This simplicity has become de-facto enforced, as it is hard to make significant changes to the language in a backwards compatible way. New properties and values can be added, as browsers will simply ignore those it does not recognize, but any more significant change to the language’s syntax is harder to introduce. As backwards compatibility (supporting older browsers) is often an important concern in web development, writing CSS often requires writing “lowest common denominator” CSS, which lacks developer niceties, like variables and mix-ins.

To work around this, the concept of CSS pre-processors was introduced. These are transpilers that accept some other language and output valid CSS. Common transpilers are Less and Sass, which all support specifying selectors and rules like normal CSS, but have a different syntax for declaration and add additional features to the language over base CSS. SCSS is a variation of Sass that allows you to write Sass, but in a syntax that is closer to normal CSS than normal Sass or Less. For this reason, SCSS has become very popular, and when most people refer to Sass, they specifically mean the SCSS variation. One reason SCSS is popular is that plain CSS is also valid SCSS, so it offers an easy upgrade route for people moving from traditional CSS-only front ends, and is also familiar to developers.

An early powerful feature of Sass and other pre-processors was the ability to support variables for reuse, although CSS has now itself adopted variables. For example, you could have one file that defines your color palette, and then reuse that variable to create multiple themes. Listings [6-3](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#PC8), [6-4](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#PC9) and [6-5](https://learning.oreilly.com/library/view/the-full-stack/9781484241523/html/471976_1_En_6_Chapter.xhtml#PC10) define 3 SCSS files, and when rendered with Sass this will output two CSS files, one of which declares white-on-black, and the other black-on-white.

$background-color: #000;

$text-color: #fff;

@import "main";

Listing 6-3

dark.scss

$background-color: #fff;

$text-color: #000;

@import "main";

Listing 6-4

light.scss

body {

  background-color: $background-color;

  color: $text-color;

}

Listing 6-5

_main.scss

@import is another useful feature of SCSS files, allowing multiple files to be consolidated into one. The underscore in _main.scss indicates that it is a partial file, so only exists to be included by the top-level SCSS files, and will not be rendered into a CSS file.

Other useful features include functions, mixins, and extensions. For example, you could have a base class that is defined in SCSS as:

%button {

  ...

}

.add-button {

  @extends %button;

  background-image: url('add.png');

}

This creates a class called %button, but unlike normal CSS classes, it cannot be used directly in HTML. Instead, it must be extended by another selector, and anything defined in %button is also available in .add-button and anything else that extends it. On the flip side, you can also use includes. The two might seem functionally similar, but can suffer from performance issues. A class that is extended many times can have very long selectors when rendered into CSS, which can impact performance, whereas an include is simply repeated every time it appears in the final CSS, which can increase size. Which one to use in any given circumstance is hard to assess, and can require benchmarking to figure out the actual performance impact. An include in SCSS looks like this:

@mixin button() {

   ...

}

.add-button {

  @include button;

  background-image: url('add.png');

}

Mixins and includes also allow function like behavior, as they take parameters. For example, you might re-write the previous example as follows:

@mixin button($icon-url) {

  background-image: url($icon-url);

   ...

}

.add-button {

  @include button('add.png');

}

Parameters can also take default parameters, which makes them very effective. To override attributes that are inherited through @extends, you can specify the attribute in the selector that extends it, but this also complicates the final rendered CSS by having extensions and overrides rendered side-by-side.

PostCSS has a similar approach, in that it starts with plain CSS, but it supports plugins that add extra features to CSS. Some of these plugins enable SCSS-like features, whereas others simply enable new, non-backwards-compatible, CSS language features, so you can write pure CSS, but then output it in a backwards-compatible way.

There is an alternative to using features like SCSS’s extends and includes: using pure CSS and then mixing together CSS classes in the DOM. To do so requires some level of discipline, and a technique known as BEM is one method of achieving it. BEM stands for Block Element Modifier, and may just be seen as a naming scheme, but it goes beyond that. In BEM, a high-level component is called a block, and corresponds to a reusable or distinct component within your page structure. A block is usually made up of other HTML elements, which may look like this:

<div class="product-card">

  <h3 class="product-card__name">...</h3>

  <img class="product-card__photo" ...>

  <p class="product-card__description">...</p>

  <p class="product-card__price product-card__price--discounted">...</p>

  <a href="product-card__link button button--primary">...</a>

</div>

In the preceding example, we have a block, product-card, and then a number of elements within it, name, photo, description and link. The link is itself a block of type button, and that button is modified by being a primary button.

These class names demonstrate BEM’s naming scheme, which is “**block**__**element--****modifier,**” where element and modifier are optional. The CSS definition for this example would then define product-card as the high-level component and each element within it appropriately too, although there is an implicit contract dictating that the element may only be used inside of that block, which can simplify your definitions. Sometimes you may need to vary the behavior of a block, and this is where the modifier comes in. Instead of having multiple classes, such as primary-button, secondary-button, etc., for variations on a theme, which can result in duplication (or use SCSS extends/includes, which results in duplication in the rendered CSS), you can abstract common functionality to the block (or element) level and then only specify the differences of that specific variation from the base. This is demonstrated above by modifying button at the block level (in which case the elements may inherit that modification by specifying nested selectors in the CSS), or at the element level itself, as in the price element.

Even though BEM can be used alone, it’s also common to use it with SCSS and other similar tools, as it is a way of dealing with CSS’s specificity issues by avoiding excessive nesting, and dealing with namespace issues by ensuring elements are namespaces within a block. There are other approaches for dealing with CSS’s flat namespace, the most common of which is to simply prepend the name with a namespace. For example, if you are creating a shared module that can be embedded on other pages, like a “Share” button, you might start all of your classes with the name of your tool and a hyphen to avoid clashing with any names the site which contains your embed may use. Other technologies have emerged that dynamically tie CSS to DOM nodes, either in JavaScript or HTML, by being aware of which CSS classes are used by a DOM and then rewriting the names to include a unique identifier.

A useful tool to have when writing CSS is an “autoprefixer.” When a new CSS property or value is introduced, there is often a period when some browsers support an experimental version of it, which may have subtle differences from the final standardized version. In order to avoid introducing broken syntax for these experimental features, a browser manufacturer will introduce a “prefixed” version of the rule of value. For example, when a method for introducing rounded corners on boxes was created, you could support this in WebKit browsers by specifying -webkit-border-radius: 5px, until the final standardized specification was approved. With an autoprefixer, you can simply write the standardized version, and it will generate the prefixed versions too to support older browsers that allow the experimental versions but not the standardized version.

When doing front-end work in CSS, it is also likely you’ll come across CSS frameworks. When using build tools like NPM with SCSS, these act as libraries and utilities that you can import into your code, but are also standalone CSS files that you can embed, providing many classes for you to use in your HTML.

CSS frameworks can range in their scope from relatively simple ones—like normalize.css, which tries to “reset” the CSS to a stripped-down set of defaults that are the same across all browsers—to larger ones, which define helpers to build grid layouts and typography rules, to fully-featured ones like Foundation and Bootstrap, which also include styling for many common elements such as forms, menus, etc. Many organizations will develop an in-house CSS framework that complies with their house style and includes components and styles that are reused across many different pages.

CSS is called Cascading Style Sheets because multiple rules can apply to a particular DOM element, and the process of figuring out the exact value of a property to apply to a DOM element is known as the cascade. Sometimes, there will be a conflict when the same property is defined multiple times, so the cascade must figure out the most specific rule to apply, and there are a number of limitations in doing this.

The first has to do with ancestry in the DOM. If you defined a font-family in an HTML selector, and then override that in a p selector, then any text inside a <p> tag will take the font defined in the p selector, as that is closer to it in the HTML tree structure. Note that not all CSS properties are inherited by child nodes; many only apply to the exact node being targeted.

If there are multiple selectors that apply to the same node, and that node is either the exact node (or at the same level of ancestry, for properties inherited from a parent), then we must look at the actual selector that defined that property and compute a specificity value.

To compute the actual specificity of a selector, you first count its number of IDs, then the number of classes, attributes, and pseudo-classes, and finally the number of elements and pseudo-elements. You then compare, in that order, the counts of each type, and the first count that is higher is the most specific. For example, a selector with two IDs is always more specific than a selector with one, even if the latter has more classes specified, as IDs are always more specific than classes. In another example, if neither selector specifies an ID, or if the number of IDs in the selector is the same, then classes do matter, and the selector with the most classes is the most specific. Continuing, if the number of IDs and classes in a selector is the same, then it is decided by elements.

The final rule for determining which to apply, when all else is equal, is which one comes last in the CSS definition file. So, if you have two p selectors, the one that comes second will override any values set in the first.

There are specificity calculators that can be used to determine the specificity value for a selector, which can help with debugging. However, avoiding using complex specificity rules is preferable, as it can get very complicated quite quickly, so it’s important to choose a selector carefully, making sure it is the lowest specificity it needs to be to accomplish a task. BEM encourages using very flat CSS, where most selectors are a single class (perhaps with pseudo-elements) to manage specificity in this way. This can lead to verbose HTML, as an element might need many CSS selectors if it combines different classes, but BEM proponents believe this is a good trade-off. The use of IDs is also commonly discouraged in CSS.

A special case rule exists in CSS known as !important, e.g., specifying a rule such as font-weight: bold !important. The ! may look like a negation, but it’s a form of declaring that this rule is the most important and should be more specific than any other declaration. Use of !important is considered bad form, as it can lead to inflexibility when trying to work on the same code base at a later date, and normally indicates that there have been some other badly managed specificity rules that need to be addressed, since you might have quite a tangled code base. One of the few scenarios in which to consider using !important is when you need to override a foreign stylesheet, but in all other cases, you should continue using normal specificity rules instead. !important is itself bound by specificity rules; if you have two rules that are !important that apply to a DOM element, the same process as above is applied, and the most specific !important gets applied. !important does not allow you to bypass specificity—it just creates a new type of specificity to consider.

The final special case to consider is that of the inline style, e.g., <span style="..."></span>. In this case, the properties defined inline are always the most specific.

## Components

The UX chapter covers approaches to designing front-end experiences, and a very common one is to break down a single web page into a series of reusable components, which can then re-appear on other pages. Sometimes this reuse can span multiple web sites across the same organization too. This makes sense for someone coming from a development background, who may try to structure code as reusable modules, classes, and functions, and it’s something we can do when building UI components too.

The challenge with UI components on the Web is that it’s hard to define it as a single self-contained package, unlike in other areas such as mobile development, where you can link a library and import a class. Take, for example, an e-commerce web site, which might want to show thumbnails of an item on both a search results page and in a “similar items” box on a product result page. At the very least, you need to insert some common HTML onto both pages, and this can be achieved by using an “include” directive or similar in a templating language (where the template could be parameterized), but you’ll almost always then want to include some styles and potentially supporting JavaScript. You could include the script and CSS inline with the HTML template, but this leads to performance losses, as it can result in lots of duplication if a template is used several times, slowing down performance, and CSS styles should be declared in the <head> of the page to be compliant with the specification. Some frameworks, such as WordPress, allow these templates to declare any additional styles and JavaScript to be loaded, which are then correctly included in the rendered page and deduplicated, but this can also lead to performance issues, especially on less reliable connections such as mobile, as the browser now has many small CSS or JS files to fetch.

The most common way to work around these performance issues is to bundle the styles and any scripting for all the components used on the site into a single file for CSS and JavaScript for the entire site, and then serve that to the user. This works well for small to medium style sites, because although the first visit to the site might require downloading a script file that contains unused JavaScript or unused CSS, that file can then be cached and reused as a user navigates a site. There is a slight overhead on each page, but this approach normally offers the best balance between performance and overheads, as long as the single files do not get too bloated.

This approach may seem to violate the “don’t repeat yourself” principle, since a component may need to be specified in several places: in the HTML where it’s used, in your CSS file to be imported, and in your JavaScript file. However, this is normally the least complex thing to do.

When building a shared component, there are two predominant ways to actually structure the code, your choice of which can be influenced by the frameworks you’re using or the preferred style of the community. The first is to have a folder per type of resource, and then a file for each part of your component. For example:

- templates/thumbnail.html.j2
    
- assets/styles/_thumbnail.scss
    
- assets/scripts/thumbnail.js
    

The other is to group them together into a directory, e.g.:

- components/thumbnail/template.html.j2
    
- components/thumbnail/_component.scss
    
- components/thumbnail/component.js
    

Sometimes there may be other assets to consider. For example, you may want to split a particularly large component up into a number of JavaScript files or CSS components, or sub-templates, but you may also need to include any image assets, such as SVGs or custom fonts. The basic method of doing this remains the same, but the exact details will depend on the build tool you are using. Some will automatically copy them into the right place or transform them to inline when they are included, but others might need additional steps.

Both of the approaches above assume that your entire site is in a single code base, but for larger sites, or when there are shared components, you will often want to make them available as packages that can be installed using your package manager. NPM (and compatible alternatives, such as Yarn) have become the largest ecosystem for front-end component sharing (as well as for back-end JavaScript code when using Node). NPM is discussed further in the JavaScript chapter, but it is a repository of published packages and a command line tool for installing these from a package definition. NPM supports private repositories, which can be an effective way to share your code, either through directly linking to Git repos, using a hosted service they provide, or self-hosting your own repository. Many JavaScript build tools integrate directly with NPM-installed JavaScript modules, but other tools like CSS or a template loader might require you to either specify a full path to the file to be included or adjust a setting to look in the right directories.

Using NPM, you can split out a particular component to its own folder and repository, publishing it to a place where other projects depend on it. One downside here is if you need to always have the exact same version of a component deployed across your entire site, it can be difficult because you will need to update the version in every place it is used when a dependency is changed, and then deploy that change. This is a fundamental flaw in every component that ends up being bundled in a specific web site, where you cannot update that component in isolation. If this use case is important for you, it is best to avoid bundling it at all, and bring it in directly into a page through <script> and <link> tags, or using the <iframe> approach, discussed below.

As sites get larger, the overhead of having a single bundle for the entire site can be overwhelming, and the most common technique for dealing with this is called “code splitting.” Code splitting is a feature of your build tool, and when used it will create multiple bundles of code (CSS or JavaScript) instead of one. You can then choose to include only the relevant bundles on a particular set of pages, often selecting a basic-level bundle for common components across all pages (like navigation), and then a bundle for any page-specific functionality that gets loaded in. A naive approach to this method might create two distinct bundles, but often several bundles will have some shared libraries or styles, which will result in the library or components being bundled several times—one in each bundle. Some build tools are smart enough to identify this scenario and will automatically create other bundles that contain the common functionality. Implementing code splitting does require you to define the bundles or how they are split up, and this can become complex to maintain, so it’s advisable to only introduce this if performance benchmarks actually identify significant issues with the single-bundle approach.

For single-page web apps, there are other approaches that can simplify the developer experience. In a single-page web app, HTML templating is often done in the client with JavaScript, bringing the templating and scripting into one place and becoming closer to the model used in most non-web development. Further approaches, classified as “CSS-in-JS,” also bring the styling into the JavaScript code base. At the time of writing, these approaches are fairly immature, and can cause performance issues or require heavily tying your code to a particular build tool, thus adding a new substantial dependency.

One approach to CSS-in-JS is for the styling to be managed at run-time by the JavaScript, which can have extremely negative performance implications, especially for responsive designs. An alternative approach is to have the build tool look at where the styles have been defined in JavaScript and extract them into a CSS file, which then behaves like a traditional bundle in the browser. The latter approach is extremely promising, although the tools to do so are currently immature and adoption (at the time of writing) is still limited. With the CSS extraction approach, you often get additional benefits, such as rewriting class names to avoid conflicts if two different classes have the same names (working around CSS’s global namespace).

A radically different approach to sharing components between different web pages is to use IFrames. Although frames have been long thought of as a problematic technology, these weaknesses do not necessarily apply to IFrames. IFrames can be accessible and interact well with web pages, with the only real complications coming into play if you have to dynamically size an IFrame to fit its contents. An IFrame is essentially embedding an external webpage into a page; the parent page needs only know the URL of the component to be embedded, which takes it completely out of the bundling process above, and allows for the component to be deployed and managed independently of the page that uses it. It does introduce another set of risks if the server hosting the IFrame component goes down, in which case it will show as an error on the parent page, but does leave it otherwise unaffected.

Creating an IFrame component is similar to creating any other web page on a site, in that it needs a URL that renders HTML, but this URL is normally not directly exposed to users because the component by itself is not meaningful. IFrame components are also useful if you want to share content to be embedded directly by third-party web sites that may have many different build systems or styles (in which case it would be hard to otherwise provide a component for them), or if your component needs access to cookies or similar on your domain that third parties will not be able to access due to cross-origin rules.

In the event a component needs to be parameterized, this can be done by passing query parameters in the URL and then letting the component that renders the contents of the IFrame react appropriately. A JavaScript API (window.postMessage) also allows the parent page to communicate with the embedded IFrame if further interactivity is required, but otherwise access to the DOM of the parent or the child can be limited unless they are hosted on the same domain. This can cause some issues, so IFrames are often only used for relatively self-contained components, and not those that need to interact significantly with the state of the parent page.

There is no perfect answer to the problem of building a componentized web UI. The web community has recognized this and has started working on a set of specifications known as web components, which has a “native” way to solve this. With web components, you can define a custom element using JavaScript in the head of your document (or in a bundle), and when you want to include that component, you can use it as if it were a regular HTML element, passing any parameters as attributes to that tag.

Web components are a complex and evolving beast. Styling is handled using something known as the shadow DOM, which allows you to specify styles that only apply within the scope of that HTML element, but also stops document-level styles from affecting the contents of the custom element. As web components become more mature and browser support improves, it seems sensible to expect these to become much more common than they are at the time of writing.

## Responsive Design

Responsive design, or responsive web design (RWD) , is a technique used to design a web page that will adapt itself to the specific capabilities of a particular device. This technique was mostly enabled by a feature of CSS known as media queries, and before responsive design was widespread, it was common to build multiple versions of a site—one for desktop and another for mobile—and then serve different HTML based on the user agent (the string the browser sends to indicate the version) of the browser. This often required maintaining two similar but separate code bases, and as a result many mobile sites were simply limited versions of a desktop site, and as users moved more and more to mobile, this became unsustainable. The expansion of the kinds of screens that web browsers could appear on (mobiles, tablets, laptops, watches, TVs, fridges) resulted in the practice of creating one version that can adapt to many devices.

It is not without its downsides, though. You often end up having to code for the lowest common denominator (mobile phones do not have the processing power of laptops, for example), which can lead to some compromise for higher-end devices. A technique known as progressive enhancement, which is discussed later on in the book, can be used to overcome this, but this is often not as simple as having one plain version per site. Sadly, this trade-off between multiple simple sites, which require a lot of work to maintain, versus one complex one persists to this day, but the consensus appears to be that the additional complexity is still less work than even creating just two versions of one site.

There may be circumstances where responsive design is not needed—for example, an internal app that is only being deployed by a call center can probably be desktop only, so it is always worth considering that option.

When designing a responsive web site, it is useful to categorize the different screen sizes needed and then create designs that work on those screen sizes, although some sites simply have one basic layout and ensure that the content scales to all screen sizes equally, which is a quick and easy solution if possible for your content and design. For others, you may want to separate the layout of a site into different categories. This can be as simple as “mobile” and “non-mobile,” but often includes intermediate categories such as “tablet” and might consider device orientation as well. Each category is not a fixed layout, but possibly covers a wide range of devices and screen sizes in each category, so variable width is often considered here. Often, the largest category has a maximum width set when it becomes a fixed-width design.

The boundaries of each category are known as breakpoints. A breakpoint is where the layout of a page changes from one category to another. For example, you might want to set a breakpoint of 400px, where below 400px is the mobile category, and equal to or above 400px is the tablet category. Note here that a pixel in CSS does not necessarily correspond to a physical pixel on the display. This is discussed further in the UX chapter, but as a reminder, in CSS, 1px corresponds to a pixel if the device’s display is 96dpi, and the physical pixel usage is adjusted if it is higher (or lower).

Radically changing the layout at every breakpoint can be confusing if a user switches between them (for example, if they rotate a device, or resize a browser window on a laptop), and would be much more difficult to implement. Small changes to the layout (perhaps moving from a single column to several) rather than reordering an entire page is normally preferred.

Other assumptions often get baked into these different screen sizes. For example, one might assume that mobile and tablet screen sizes will be used for touch, and any buttons or other clickable areas should be scaled up for that, but smaller laptops (or side-by-side web browsers on a desktop) might not be touch screen, and conversely, touch-screen laptops and desktops with bigger screen sizes are increasingly common. Similarly, mobile devices are often assumed to be held closer to the face, so text can be made relatively smaller. These assumptions are baked into designs between each breakpoint, but it’s important to be aware of them, and that it’s actually device width/height/orientation being used as a proxy for these wider assumptions, and relying on those factors alone will sometimes steer you wrong.

CSS media queries are a way of defining a block of CSS that only applies when the screen characteristics match a certain set of criteria. For example, if you wanted to say that text should turn blue on mobile, you might write the following:

@media (max-width: 400px) {

        p {

          color: blue;

  }

}

The preceding code dictates that the CSS applies when the browser width is less than 400px.

You can combine multiple statements into a single query, so that the inner rules only apply when all are set. Combining min-width with max-width therefore allows you to specify ranges:

@media (min-width: 401px) and (max-width: 1024px) {

  ...

}

The above would then only apply between 401-1024px. It’s important to highlight that we start at 401px for a reason! If we reused the 400px value, as we did in the mobile-only query , then if the device happened to be at exactly 400px, then both sets of media queries would apply. This would be an odd edge case to catch, as min-width and max-width are inclusive (equivalent to “more than or equal to” and “less than or equal to,” respectively), so they do overlap.

In addition to width, you can also specify height in the same way, as well as device-width and device-height, which allows you to specify the absolute size of the device you are on, rather than the current size of the browser (so you can, for example, always render a desktop site on a desktop, even if it’s made smaller), but this is not necessarily a good idea. orientation is another property of use, and can be specified with values of portrait or landscape to further target a particular configuration. There are many other properties, including aspect ratios, and newer features such as whether or not a user can hover with their current input device, that can be used to be more explicit about assumptions made about touch screens, etc. Media queries can also be constructed using operators other than and, including or which behaves as you expect, and not which can only be used to negate an entire query, rather than a particular parameter within it. Commas can be used to specify multiple different queries that apply (similar to the way a normal CSS declaration can specify multiple classes that match). Finally, it is interesting to note that the original purpose of media queries was to specify a “print” stylesheet—one that only applies when a page is printed out.

The actual breakpoint values are worth breaking out to variables when using a CSS variant that supports it, as they can be duplicated and might need to be tweaked during development. This also makes it easy to keep them consistent, having a single set of break points across an entire site, rather than different components having their own set of breakpoints applying to them.

Responsive design with media queries is often combined with a technique known as “mobile first,” which is discussed later in this chapter.

## Progressive Enhancement

At the core of the very identity of the Web is the metaphor of documents. Each web page was one document, and web sites consisted of a collection of those documents. Each document had a globally unique reference—a URL—that could be used to look it up, and documents were hyperlinked together using these references. It was truly ground-breaking, and for years this documents metaphor prevailed. Many standards were written based on the idea that web pages were simply documents, and then tools arose that exploited this, such as accessibility toolkits and search engines.

Then, people started putting things on the web that weren’t documents. Web sites such as Hotmail launched, and didn’t quite conform to some of those core principles of the Web. Nowadays, we call these types of sites web apps. These happily coexisted with the original web and worked well enough with the document metaphor that it didn’t break too many of those tools. This harmony was achieved using a technique called progressive enhancement.

Progressive enhancement is the idea of layering functionality on top of a simple document to give a richer experience. And it wasn’t only these new web apps that used progressive enhancement; traditional document-driven web sites started using it too. In the early days of the web, well-formed documents with very little styling were the norm. Then CSS came along, allowing people to style their documents, and it wasn’t unheard of for people to apply custom stylesheets in their own browser. For example, users who had trouble seeing might increase the default font size or add high color contrast to their default experience. When JavaScript came along, the same scenario applied. Web sites worked without JavaScript, and for browsers that supported it, JavaScript improved the user experience.

However, in the late 1990s and early 2000s, Windows started dominating the scene, along with Internet Explorer. Although sites that were content based continued to conform to standards and to the technique of progressive enhancement, web apps, especially intranet web apps, stopped doing so, as it was easier to assume the user was browsing with Internet Explorer. This was exacerbated by Internet Explorer’s adoption of proprietary extensions, which enabled new types of web apps. Progressive enhancement usually requires more engineering discipline than not using the approach, and in those environments, there was no need for it. With the rise of mobile browsers and the breakup of Microsoft’s monopoly on the web browser, these web apps suddenly became a large chunk of legacy code.

Similarly, with the rise of the mobile web, many sites that had been built around that particular environment struggled to adapt to mobile phones. Many organizations had to suddenly build a separate mobile site and then, years later, merge them back together using the rising technique of responsive web design. Sites that had used the technique of progressive enhancement had an easier migration to deal with. Many simply served a mobile version of their CSS template, but could leave their HTML documents as they were.

The early days of mobile web were similar to the early days of the Web on the desktop. Dozens of different devices, with different browsers and capabilities, made progressive enhancement the only game in town. However, like the consolidation of the desktop market to a smaller number of players, the mobile browser market has become dominated by Android and Apple. At the same time, JavaScript’s capabilities increased to a point where the browser could do more and more, and a web app could rely on the server less. A slew of frameworks arose, embracing the idea of building these kind of web apps (Angular and Meteor, among many others). These were generally incompatible with the idea of progressive enhancement, and in some use cases that was okay. However, these tools also got used in many situations where it _wasn’t_ okay. The abandonment of progressive enhancement in these cases suddenly caused problems when things that you could previously take for granted on the web if you were standards compliant—search engine discoverability, compatibility with accessibility tools—were no longer there. Additionally, these type of web apps can have long startup times, causing a slow page load, and this was especially true on smartphones.

### To Progressively Enhance, or Not?

Progressive enhancement is all about identifying the core functionality of your web site and making it available to all, and then applying any additional functionality or nice-to-haves as layers beyond that. As web technology and browsers have improved, the core experience has gotten richer. For example, ensuring that your page works without CSS is no longer strictly necessary. On the flip side, devices on flaky network connections, or issues with your server, can cause web pages to load without any style sheets.

You may think the same argument applies to JavaScript—few browsers nowadays do not support JavaScript, and unless there is a shaky connection, JavaScript should be loaded. However, not all search engines will execute JavaScript, or will only execute a limited subset of it. Furthermore, although it is completely possible to build fully accessible web applications with JavaScript, having to implement those semantics yourself is harder than using the built-in functionality of the browser. Also, JavaScript has more failure modes. A typo in a CSS file will not break all of your styling (although it may break part of it), but an error in your JavaScript could hinder critical functionality, or completely alter how the page is displayed.

Regardless of what type of web site you’re building, at its core is the HTML document you deliver to the user. For search engines and accessibility tools, this is still the most important element. Most will understand some of the layers on top, but only to a certain extent. For content-heavy web sites, this document should contain the very essence of your page, and it should be renderable before any JavaScript gets applied to it. For web apps, this is much harder to do, as for many, the essence of what the site does cannot be expressed using only the simple building blocks of HTML, or often to do so would require significant server-side effort. For some web apps, a decision is made on a component-by-component level as to whether progressive enhancement is used or not, but for others, the decision is made to not attempt at all. The single-page application pattern is one example where progressive enhancement is abandoned.

The decision of whether or not to use progressive enhancement is fundamental to the entire front-end experience, so it should be considered with care. For most content web sites, any decision to not apply progressive enhancement techniques can cause long-term issues with the web site. In some circumstances, short-term engineering gains can be made, but this is offset by the higher cost of implementing search engine optimization and re-adding the appropriate hooks for accessibility purposes. Progressive enhancement has stood the test of time, and has shown value again and again, but like many things in software engineering, the answer to the question “should I progressively enhance?” is, “it depends.” The decision of whether or not to not apply the technique should be made carefully.

Not using progressive enhancement, combined with techniques for performance, such as only loading the JavaScript when a page has first rendered (to speed up time before the page becomes usable), can leave the page in awkward half states, where a button may be visible, but clicking on it does nothing, as the JavaScript has not added any events to it yet. In a progressive enhancement context, this is less annoying, as it can fall back to the non-progressively enhanced version (such as submitting a form the traditional way, rather than with AJAX). However, when there are entire features being delivered using JavaScript, if these are only added after page load, this can cause annoying flashes as the page re-renders, or cause the content to jump around. To deal with these, you can use a small inline script tag to add a class to the <body> tag, and then use CSS to hide them by default until that script runs. If the inline script is the first thing in the body, elements will be immediately visible as they are rendered on browsers that support it, perhaps in a “disabled” or “loading” state, until they are completely enabled once the JavaScript has been loaded.

### Mobile First

Complementary to progressive enhancement is a technique known as “mobile first.” This technique starts with the assumption that browsers on smartphones are the least-capable browsers: the hardware is under-powered, network connections the least reliable, and interaction mechanisms the most limited. This technique is often used in the interaction design stages too, where the designs are developed with the most constrained environments in mind first, and as those constraints are lifted, more can be added. Many teams find it easier to add functionality to supported devices than to have to shoehorn an experience for bigger or more capable devices into a smaller one.

With mobile first, it’s useful to establish a baseline. A decade ago, a BlackBerry or Nokia smartphone might have been the starting point, but nowadays it’s more likely to be a cheap smartphone, which can be running a very capable browser but be constrained by either how out of date it is, or the performance of the device. Once you have established this baseline, you can design and build a version that targets that device, and then figure out what other, more capable, classes of device you want to target. A more capable smartphone might be very similar, perhaps with a bigger or higher definition screen, but then you might move to a tablet, where the screen is much bigger, and the way people hold and interact with the device completely changes. The last class of devices to consider are desktop or laptop devices, where users navigate using a mouse (or trackpad), which gives you much finer control over interactions with the UI, as well as features such as hovering.

If we try to apply the technique in reverse, then the value of mobile-first becomes clear. If we assumed desktop first, and then used hovering as part of the UI, then when a mobile variant is developed, either a workaround to enable the same interaction would be needed, which may not be obvious for mobile users, or significant rework would need to be done to the interface to take into account the fact that the user will not always be able to hover. By addressing designs in a mobile-first context, we either eliminate the use of desktop-only interactions like hover, or only use them for additional “nice-to-haves” rather than critical parts of the functionality.

Mobile first goes beyond the way you approach building the UI, though. By dealing up front with the fact that a network may be unreliable, you can build with those constraints in mind from the start, rather than having to go back and retrofit or rewrite code.

Most web sites nowadays will also reference external media, and mobile first encourages you to provide assets that are an appropriate size or quality for mobile devices. Take the example of images. There’s no point in downloading a 4K image for a phone with a low-resolution screen on a slow connection. As mobile phones generally have smaller screens than laptops, it may make sense to generate <img> tags with an src of a small resolution of the image appropriate for a mobile phone as the lowest common denominator. JavaScript can then be used to check the size of the actual screen and replace that src with a more appropriately sized image, but starting with a small image gives the fastest experience on mobile, while enhancing it for less constrained devices later. This kind of behavior is becoming increasingly abstracted away in the HTML specification. For images, srcset now does this without requiring any JavaScript, and for media like audio or video, formats such as MPEG-DASH also support adapting the media size and bitrate appropriately.

Mobile first, when used with responsive web design, normally involves defining all your mobile styling without using any media queries, and then overriding any mobile-only styles with media queries for larger devices and screen sizes.

### Feature Detection

For many, the simplest level of progressive enhancement is “does the core of the experience work without JavaScript?” and this is not a bad starting point. However, as discussed above, there are some experiences, especially in web apps, where the core essence is not expressible using HTML and CSS alone. But progressive enhancement is not about the binary notion “is JavaScript there or not,” and can cover much more.

Take, for example, geolocation. Although the JavaScript API exists in modern browsers, there is no guarantee it will actually work (for example, if the device has no GPS chip, or it cannot get a signal and GeoIP is inconclusive), so you should not rely on that API for the core experience of your site. For example, if you are building a “find a store” feature, then you may want to start with a simple search form that has no dependency on JavaScript, and only expose a “Use current location” button if the Geolocation API is available. This is a technique called feature detection, where a certain area of functionality on your web site is only enabled when a corresponding feature is detected in the user’s browser. Implementing detection for a particular feature is dependent on the particulars of that feature , but there are libraries that can abstract this away.

Another technique that has seen significant use is the “cuts the mustard” technique. This goes beyond the simple “does JavaScript run” approach; instead, a test is run to determine if the browser has a baseline support for modern JavaScript. Using this kind of technique can help with testing, because instead of a large combination of browsers and devices with different features to test, testing can be divided into browsers and devices that “cut the mustard” and those that don’t. Browsers that don’t cut the mustard get a non-JavaScript experience, which is better than having a subtly broken web site, and gives you an easier way of confirming your site works for search engines and in other situations where JavaScript cannot be relied upon.

### Progressive Enhancement of Styles

Progressively enhancing CSS is more of a binary affair. It’s usual for a site to minify all CSS down into one file, and for that to be loaded—therefore, either the page is completely styled, or not at all. When a CSS file has not loaded, it is usually very obvious to the viewer that something has gone wrong (and if they’re using it on a mobile device with a bad signal, they possibly think the problem is on their end, not the site’s), but even here you can make the best of a bad situation with progressive enhancement. When building a site, by focusing on the structure and content of the page in HTML, if CSS fails to load, then at least users can still read whatever you’ve published (even if it’s not pretty). This is good for search engine optimization too. A good rule of thumb is if the page still makes some kind of sense without CSS being applied, then a search engine can probably get some meaningful data from it.

Unfortunately, CSS is not quite as simple as an all-or-nothing process of loading a file. Like JavaScript, there are differing levels of support for features and CSS properties in different browsers, but unlike JavaScript, CSS has no feature detection. Fortunately, browsers will ignore CSS statements they do not recognize, which can eliminate most of the need for progressive enhancement. For example, if you need to support legacy Internet Explorer, you would be wise to accept that elements will not be pixel perfect in those older browsers. Similarly, some browsers only accept “prefixed” versions of properties (sometimes with different syntax for values than what was eventually standardized). Tools like an autoprefixer can behave like JavaScript polyfills to allow you to access these.

It is slightly harder to manage significant changes in CSS that browsers can just not parse. The most famous example of this was for legacy versions of Internet Explorer (prior to 9) that did not support media queries. As developers embraced mobile-first progressive enhancement and responsive web design, this gave IE users a significantly poorer experience. The common workaround was to produce an IE-only stylesheet, corresponding to the “desktop” break point, and then use IE conditional comments to include that only on IE. This was definitely not progressive enhancement in its purest form, but a hack where the trade-off made sense.

### When Not Using Progressive Enhancement

For web sites that have not been built using progressive enhancement, there is often a need to retrofit the advantages of progressive enhancement. The most common example is the need for discoverability by search engines, and for fast load times.

Additionally, one of the more common arguments against progressive enhancement is that it can increase engineering costs; functionality has to be implemented on both the server and client sides. The solution to both of these is server-side rendering, but the mechanism for doing so varies radically.

There are many proxies and servers out there that will render a page using a headless browser and then serve the resulting content directly to the user, but this can interfere with single-page application frameworks, which do not assume any content already exists. Some frameworks can bind themselves to server-side rendered content and use that as their initial state, speeding up build time. Sometimes these proxies will only do so if they detect a user agent belonging to a search engine, a technique that is often penalized by search engines if they discover it.

One technique that has emerged as a result of the popularity of NodeJS to reduce engineering costs (building the same logic and views server- and client-side) is “Isomorphic JavaScript” (or “Universal JavaScript”). Using JavaScript on the server, this renders the views using the same JavaScript code that is used in the client, with some initial state. Modules can then be shared server- and client-side, without having to re-implement the core of the view.

## Search Engine Optimization

Search engine optimization (SEO) can often be seen as a dark art, but is a necessity for any content-based web site in this day and age, as search engines are a primary way that people browse the Web. In reality, most people will be coming to your site either via a link on social media or a web app, or by searching for it. Entering URLs by hand has become the preserve of power users.

SEO is essentially the mechanism of making the content of your site easily understandable by search engines so that they rank your site highly for relevant terms, as well as adopting any other positive technical signals the search engines use to rank. The water has been muddied by ethically questionable SEO agencies that apply short-term hacks like adding loads of unnecessary keywords onto a page, or even more dubious techniques like hidden text, or serving search engines different content. Another important way search engines rank content is the number of incoming links from reputable sites. Again, some questionable agencies might spam your URL onto sites via comments in an attempt to boost your ratings, but these kinds of hacks are short-term at best, and often damaging in the long run.

Search engine optimization is definitely needed for content-type web pages, but even for application-type pages, some can be useful. Although many of these pages may be hidden behind a login and aren’t directly searchable, it is possible that things such as landing pages will be, so applying these kinds of techniques will help there.

The search engine market is unquestionably dominated by Google, which does not reveal exactly how results are ranked, but has indicated that some specific factors are important. The ethos of Google and other search engines is that the things that matter to humans are the things that matter to the engine. Ultimately, technical methods cannot correct for bad content, so getting the right content, including effective headlines and a good standard of English, is the single most important thing you can do. The second is to make sure that Google can understand your page. Although search engine crawlers can execute some JavaScript, it is not clear how much of the language they support. Fortunately, making a site easily parseable by browser—using techniques such as progressive enhancement—and ensuring that it is accessible means that search engines can also parse it, so a web site that is well-designed for human users should also work well for SEO purposes.

However, it isn’t just the actual content of the page that can be beneficial to search engines. There are specific bits of metadata and structured tags that should be added to your HTML to help boost your results. The <title> tag in HTML is one such example, as this is the name of the document as it shows up in search results. The simplest level beyond this is what is called a meta tag, which can be added inside the <head> of your HTML document to indicate additional bits of data, such as a summary of the document, known as a meta description. Specific search engines and social media also support more specific tags, such as “rich snippets,” to indicate media or other elements that stand out on a results page, or when sharing on social media to generate a small preview window. Adding these kinds of tags will make your site stand out over ones that do not, but almost everyone uses them, so not having them is more likely to mean that your content will have a low rank, rather than their inclusion guaranteeing a high rank.

Beyond the content and metadata, there are other signals that search engines take into account when ranking results. For example, if your site is not responsive to the size of a mobile screen, it will not appear as high for searches conducted from a mobile phone. Speed and security are important, too. Google in particular will punish sites that are slow to render and load, because users often will not wait more than a few seconds for a site to load. This makes it doubly important for you to manage the performance of your site. Web sites that are not HTTPS will also not perform as well on Google as those that are served over a secure connection. Google often uses these signals to push the web in the direction it would like it to go, towards a faster, more secure future, and it should not be surprising that Google pushes more technical improvements as search signals to improve the quality of the Web in general.

The final set of important signals for search engines, as discussed above, is the number and quality of incoming links to your site. If you have the same content split over many pages, then there are often multiple URLs that people can link to, simplifying your site design so there’s only piece of content with one address that is linked to means that that one link will get a higher rank than having a larger number of links each with a lower rank. In terms of actually getting those external sites to link to yours, this is a completely non-technical problem! Linking to the site from social media and using press releases to link to your content, or even other sites linking back, will help.

The final word on managing URLs for SEO is that if your URLs change, you will often find your search rank dropping as you lose incoming links. It is possible to mitigate this with correct use of HTTP redirects, but there will always be some intermediate phase between adding the redirect and the new page or URL getting the appropriate rank. In this case, it is important to have a good and sensible URL structure up front. There will be times when you find you have to delete content and change redirects, and when that happens, it might seem at first that you can just restructure your internal navigation and the site will be fine, but don’t forget about any incoming links, and always set up redirects appropriately.

This is only a brief overview of SEO , as it is a fast-moving field that changes from day to day. Any book that tries to cover anything other than high-level concepts will become outdated very quickly. However, it is not a dark art that can only be implemented by specialists from SEO agencies; it’s something that a full stack developer should be able to do with minimum fuss by understanding these principles.

## Build Tools

When using a tool like Sass, and even when not, there are often steps we should take in order to transform our source code into the version that gets delivered to the user’s browser. This is similar to the way compiled languages might need some sort of build tool in order to compile and assemble Java classes into a .jar for distribution, and indeed some of those traditional language tools have developed some capability to handle front-end code. However, the most mature and common tools for building front-end code are those that were designed specifically for that purpose. There are many tools, each with their own philosophies and approaches, and many new ones appearing all the time.

Some of these tools are targeted towards one task—for example, Compass is responsible for building a CSS file from Sass input—but others are more general and orchestrate a number of tasks—for example, Webpack tries to satisfy all front-end building concerns. There are also task-running tools that can orchestrate the different parts of your build tooling into one command. Some popular tools include Gulp or Grunt, which might combine a number of the more-focused tools, or tools like NPM scripts or makefiles, which are often used with tools like Webpack.

Given that the front-end world evolves very quickly, I will not give any concrete recommendations here, as they will undoubtedly become outdated very quickly, but I will discuss a general approach to the types of activities you may want to live in your build tool. Often, you will configure your build tool to use a pipeline, starting with source files and outputting a final build, with each stage taking the output of the previous. These stages will vary, but will often develop as follows:

- Starting with an entry point, processing it to find all the imports, and concatenating them together into one built output
    
- Transpiling each import (for example, converting JavaScript into a backwards-compatible variant, or Sass into CSS)
    
- Generating source maps that can be used to help debug the final build by mapping it back to the original code (a source map describes how the compiled code relates to the original, and is supported by major browsers to allow you to debug the original code, rather than the compiled version)
    
- Minimizing the code or images, through compression or removing white space to minimize the file size, which can improve performance for the end user
    

Your build tools will also want to run any tests against your code. These could be unit and integration tests, style checkers (sometimes known as linters that ensure your code matches best practices and your preferred style), or other analysis tools that identify common bugs.

Most build tools will have two modes—one to output a “production” build, and another to better support development. Keeping the difference between your production builds and development builds as small as possible can minimize bugs that only crop up in production, but some are necessary. For large projects, minification can take a long time, so that is sometimes skipped in development mode in order to minimize waiting time after making changes. On the flip side, source maps that are invaluable for debugging can be large and slow down production, so are skipped. Some libraries go further, sometimes by including extended debug logging or checking values at runtime, which are then skipped for performance reasons in the production build.

Another common difference between development and production builds is that often production builds will run once with the output being uploaded to a CDN, whereas development builds will happen in a “watch” mode, where a build tool will watch the source files for changes and then automatically rerun the build. This process can make use of caching in order to be much quicker than a full build or test run. Some build tools support the ability to go further, using techniques such as live reloading or hot module reloading. In these cases, the build tool will start up a web server to serve the built assets, but insert additional code that will cause your web browser to connect to that web server using a web socket or similar. The web server will then tell the browser to automatically refresh the page if the code has changed, or, in more advanced cases, reload a single component of the page for even faster development.

Do not underestimate the complexity or power of a well-tuned build tool. Correctly configuring the build can have real performance impacts for your end users, as well as greatly enhance your workflow. Do your research to find what works best for your stack, and spend some time setting up a skeleton that works for you and that you can reuse.

## Summary

The front end of your application is the bit that your users actually interact with. On the web, HTML determines the layout of your document, CSS dictates the visual styling, and JavaScript can be used to give a degree of interactiveness by manipulating the HTML structure and CSS rules.

All apps start by delivering HTML to a web browser, even if it’s only a small amount to bootstrap a rich web app. For many sites, the bulk of the HTML is generated server-side, and there are different ways of generating this: automatically using libraries and frameworks, which has limited flexibility; tree transformation, where some other data structure such as XML or HAML is translated directly into HTML; or string interpolation, where the HTML is built using file- or string-based templates.

CSS is structured into selectors and rules. The selectors describe which parts of the HTML the rules should apply to, and the rules then specify the exact effect on the rendered content. Some rules, such as fonts, cascade such that they apply to children of the targeted element, and when several rules can apply to a particular HTML element, specificity is used to determine which one takes precedence. CSS can also target pseudo-elements and pseudo-classes, which do not exist in the HTML but represent virtual content or distinct states.

CSS has for a long time been a fairly limited language. Other languages have been introduced, such as Sass or Less, that compile into CSS but provide additional syntactic sugar, such as variables or inheritance, over the plain CSS language.

It’s common to take a component-based approach to building your front end, and this plays well with the approach of many UX designers. There are several approaches to this-such as handling the HTML template, CSS and JavaScript separately-although in some rich applications, you can bundle these together with some potential performance cost. These components can be bundled as shared code or hosted and embedded as IFrames.

Two important techniques for the front end are responsive design and progressive enhancement. Responsive design is where the styling of the site uses media queries to change based on the size of the user’s screen, and is often combined with a mobile-first methodology, where the basic design is mobile-friendly and the site responds as the screen gets larger. With progressive enhancement, all users get the same basic experience, and then JavaScript is used to improve the user experience when the device supports it. This increases the reach of your application by not depending on particular features, but making use of them when they do exist.

The final part of front-end design to consider is non-human users of your web sites. This can include search engines, which index your site and allow it to be searched, and many of the techniques that make sites usable and accessible can make your site attractive to search engines.