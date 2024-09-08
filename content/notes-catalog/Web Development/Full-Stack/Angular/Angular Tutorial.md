---
id: 01J75A39S5CVPTWAD7SAWCDAHS
title: Angular Tutorial
modified: 2024-09-06T23:59:25-04:00
description: The official Angular Tutorial
tags:
  - angular
  - full-stack
  - programming
  - tutorial
  - web-development
---
## [Conceptual preview of Angular components](https://angular.dev/tutorials/first-app/02-HomeComponent#conceptual-preview-of-angular-components)

Angular apps are built around components, which are Angular's building blocks. Components contain the code, HTML layout, and CSS style information that provide the function and appearance of an element in the app. In Angular, components can contain other components. An app's functions and appearance can be divided and partitioned into components.

In Angular, components have metadata that define its properties. When you create your `HomeComponent`, you use these properties:

- `selector`: to describe how Angular refers to the component in templates.
- `standalone`: to describe whether the component requires a `NgModule`.
- `imports`: to describe the component's dependencies.
- `template`: to describe the component's HTML markup and layout.
- `styleUrls`: to list the URLs of the CSS files that the component uses in an array.

[Learn more about Components](https://angular.dev/api/core/Component)

1. ### [Create the `HomeComponent`](https://angular.dev/tutorials/first-app/02-HomeComponent#create-the-homecomponent)
    
    In this step, you create a new component for your app.
    
    In the **Terminal** pane of your IDE:
    
    1. In your project directory, navigate to the `first-app` directory.
        
    2. Run this command to create a new `HomeComponent`
        
        ```
        ng generate component home
        ```
        
        check
        
    3. Run this command to build and serve your app.
        
        Note: This step is only for your local environment!
        
        ```
        ng serve
        ```
        
        check
        
    4. Open a browser and navigate to `http://localhost:4200` to find the application.
        
    5. Confirm that the app builds without error.
        
        HELPFUL: It should render the same as it did in the previous lesson because even though you added a new component, you haven't included it in any of the app's templates, yet.
        
    6. Leave `ng serve` running as you complete the next steps.
        
2. ### [Add the new component to your app's layout](https://angular.dev/tutorials/first-app/02-HomeComponent#add-the-new-component-to-your-apps-layout)
    
    In this step, you add the new component, `HomeComponent` to your app's root component, `AppComponent`, so that it displays in your app's layout.
    
    In the **Edit** pane of your IDE:
    
    1. Open `app.component.ts` in the editor.
        
    2. In `app.component.ts`, import `HomeComponent` by adding this line to the file level imports.
        
        Import HomeComponent in src/app/app.component.ts
        
        check
        
        ```
        import {HomeComponent} from './home/home.component';
        ```
        
    3. In `app.component.ts`, in `@Component`, update the `imports` array property and add `HomeComponent`.
        
        Replace in src/app/app.component.ts
        
        check
        
        ```
          imports: [HomeComponent],
        ```
        
    4. In `app.component.ts`, in `@Component`, update the `template` property to include the following HTML code.
        
        Replace in src/app/app.component.ts
        
        check
        
        ```
          template: `    <main>      <header class="brand-name">        <img class="brand-logo" src="/assets/logo.svg" alt="logo" aria-hidden="true" />      </header>      <section class="content">        <app-home></app-home>      </section>    </main>  `,
        ```
        
    5. Save your changes to `app.component.ts`.
        
    6. If `ng serve` is running, the app should update. If `ng serve` is not running, start it again. _Hello world_ in your app should change to _home works!_ from the `HomeComponent`.
        
    7. Check the running app in the browser and confirm that the app has been updated.
        
        ![browser frame of page displaying the text 'home works!'](https://angular.dev/assets/images/tutorials/first-app/homes-app-lesson-02-step-2.png)
3. ### [Add features to `HomeComponent`](https://angular.dev/tutorials/first-app/02-HomeComponent#add-features-to-homecomponent)
    
    In this step you add features to `HomeComponent`.
    
    In the previous step, you added the default `HomeComponent` to your app's template so its default HTML appeared in the app. In this step, you add a search filter and button that is used in a later lesson. For now, that's all that `HomeComponent` has. Note that, this step just adds the search elements to the layout without any functionality, yet.
    
    In the **Edit** pane of your IDE:
    
    1. In the `first-app` directory, open `home.component.ts` in the editor.
        
    2. In `home.component.ts`, in `@Component`, update the `template` property with this code.
        
        Replace in src/app/home/home.component.ts
        
        check
        
        ```
          template: `    <section>      <form>        <input type="text" placeholder="Filter by city" />        <button class="primary" type="button">Search</button>      </form>    </section>  `,
        ```
        
    3. Next, open `home.component.css` in the editor and update the content with these styles.
        
        Note: In the browser, these can go in `src/app/home/home.component.ts` in the `styles` array.
        
        ### Replace in src/app/home/home.component.css
        
        ```
        .results {  display: grid;  column-gap: 14px;  row-gap: 14px;  grid-template-columns: repeat(auto-fill, minmax(400px, 400px));  margin-top: 50px;  justify-content: space-around;}input[type="text"] {  border: solid 1px var(--primary-color);  padding: 10px;  border-radius: 8px;  margin-right: 4px;  display: inline-block;  width: 30%;}button {  padding: 10px;  border: solid 1px var(--primary-color);  background: var(--primary-color);  color: white;  border-radius: 8px;}@media (min-width: 500px) and (max-width: 768px) {  .results {      grid-template-columns: repeat(2, 1fr);  }  input[type="text"] {      width: 70%;  }   }@media (max-width: 499px) {  .results {      grid-template-columns: 1fr;  }    }
        ```
        
        check
        
    4. Confirm that the app builds without error. You should find the filter query box and button in your app and they should be styled. Correct any errors before you continue to the next step.