---
id: 01J62MCCEC4D0CXM5XSS3ZVX7R
title: Backstage Architecture
modified: 2024-08-24T12:45:03-04:00
description: A look at the architecture of Backstage
tags:
  - backstage
  - architecture
  - dev-ops
  - developer
  - programming
---
# Architecture overview

## Terminology[​](https://backstage.io/docs/overview/architecture-overview#terminology "Direct link to Terminology")

Backstage is constructed out of three parts. We separate Backstage in this way because we see three groups of contributors that work with Backstage in three different ways.

- Core - Base functionality built by core developers in the open source project.
- App - The app is an instance of a Backstage app that is deployed and tweaked. The app ties together core functionality with additional plugins. The app is built and maintained by app developers, usually a productivity team within a company.
- Plugins - Additional functionality to make your Backstage app useful for your company. Plugins can be specific to a company or open sourced and reusable. At Spotify we have over 100 plugins built by over 50 different teams. It has been very powerful to get contributions from various infrastructure teams added into a single unified developer experience.

## Overview[​](https://backstage.io/docs/overview/architecture-overview#overview "Direct link to Overview")

The following diagram shows how Backstage might look when deployed inside a company which uses the Tech Radar plugin, the Lighthouse plugin, the CircleCI plugin and the software catalog.

There are 3 main components in this architecture:

1. The core Backstage UI
2. The UI plugins and their backing services
3. Databases

Running this architecture in a real environment typically involves containerising the components. Various commands are provided for accomplishing this.

![The architecture of a basic Backstage application](https://backstage.io/assets/images/backstage-typical-architecture-c38b04130f70f294725a9f646df94d3a.png)

## User Interface[​](https://backstage.io/docs/overview/architecture-overview#user-interface "Direct link to User Interface")

The UI is a thin, client-side wrapper around a set of plugins. It provides some core UI components and libraries for shared activities such as config management. [[live demo](https://demo.backstage.io/catalog)]

![UI with different components highlighted](https://backstage.io/assets/images/core-vs-plugin-components-highlighted-de03cf2f3bb3f96bea4834f1db02172e.png)

Each plugin typically makes itself available in the UI on a dedicated URL. For example, the Lighthouse plugin is registered with the UI on `/lighthouse`. [[learn more](https://backstage.io/blog/2020/04/06/lighthouse-plugin)]

![The lighthouse plugin UI](https://backstage.io/assets/images/lighthouse-plugin-3543fecb164ca2e8dca3959c8b4909f4.png)

The CircleCI plugin is available on `/circleci`.

![CircleCI Plugin UI](https://backstage.io/assets/images/circle-ci-31440aae7a5b2c44a7ee75bfc4573b03.png)

## Plugins and plugin backends[​](https://backstage.io/docs/overview/architecture-overview#plugins-and-plugin-backends "Direct link to Plugins and plugin backends")

Each plugin is a client side application which mounts itself on the UI. Plugins are written in TypeScript or JavaScript. They each live in their own directory in the `plugins` folder. For example, the source code for the catalog plugin is available at [plugins/catalog](https://github.com/backstage/backstage/tree/master/plugins/catalog).

### Installing plugins[​](https://backstage.io/docs/overview/architecture-overview#installing-plugins "Direct link to Installing plugins")

Plugins are typically installed as React components in your Backstage application. For example, [here](https://github.com/backstage/backstage/blob/master/packages/app/src/App.tsx) is a file that imports many full-page plugins in the Backstage sample app.

An example of one of these plugin components is the `CatalogIndexPage`, which is a full-page view that allows you to browse entities in the Backstage catalog. It is installed in the app by importing it and adding it as an element like this:

```
import { CatalogIndexPage } from '@backstage/plugin-catalog';...const routes = (  <FlatRoutes>    ...    <Route path="/catalog" element={<CatalogIndexPage />} />    ...  </FlatRoutes>);
```

Note that we use `"/catalog"` as our path to this plugin page, but we can choose any route we want for the page, as long as it doesn't collide with the routes that we choose for the other plugins in the app.

These components that are exported from plugins are referred to as "Plugin Extension Components", or "Extension Components". They are regular React components, but in addition to being able to be rendered by React, they also contain various pieces of metadata that is used to wire together the entire app. Extension components are created using `create*Extension` methods, which you can read more about in the [composability documentation](https://backstage.io/docs/plugins/composability).

As of this moment, there is no config based install procedure for plugins. Some code changes are required.

### Plugin architecture[​](https://backstage.io/docs/overview/architecture-overview#plugin-architecture "Direct link to Plugin architecture")

Architecturally, plugins can take three forms:

1. Standalone
2. Service backed
3. Third-party backed

#### Standalone plugins[​](https://backstage.io/docs/overview/architecture-overview#standalone-plugins "Direct link to Standalone plugins")

Standalone plugins run entirely in the browser. [The Tech Radar plugin](https://demo.backstage.io/tech-radar), for example, simply renders hard-coded information. It doesn't make any API requests to other services.

![tech radar plugin ui](https://backstage.io/assets/images/tech-radar-plugin-4d2311a5ccbe580f9fc5f071c734f06c.png)

The architecture of the Tech Radar installed into a Backstage app is very simple.

![ui and tech radar plugin connected together](data:image/png;base64,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4XC4tLS0qqoqJycn+GmzsrK2bdsmlAFAKCNlzkcod3d3p6amHj9+PHiyoKCgsbGxo6OjsbGxuLi4o6MjNTV14Bmampqys7M7OzsHPtm/9SInJ6ewsLC3t7elpSU7O7v/dxhcFW5sbCwsLOzu7p43b14sFsvPzw8iODc3t6Ojo6KiIoj1WCzW1dWVnp7e0dHR0dGRk5PT2dkZiUQqKytNFwAIZaTMkAiHw01NTQNDORaLnThxorCwMCsrq7u7OxQKlZSURCKRSCTS0NDQ0dERCoXOOEljY2N2dvaqVasGhnL/BeZIJFJVVdXS0hJcLR74Ozx58mRwtvT09Pb29rKysoqKiuPHj6enp8disWg0WlVVlZGRUVlZ2drampaWFvkvPT09H3dHtekCAKGMlPkYqquri4uLgyrNyMhob2+PRqPBRdyKioqamprS0tKBPfr+UO7fQJyRkdG/TXlgKAfbJwaGcmpq6htvvBGLxSorK4NNFxUVFfn5+U1NTS0tLfn5+UFzd3d3Bz/Y5MmTe3t709LSgh/sjKw3XQAglJEy5140Gl26dOm8efPmzJkT3J7i5MmTWVlZhYWFoVCoq6urp6cnJycnNzc3Jyenvb29s7MzPz9/4Bnq6+uzsrLy8/MHPl9fXx8KhQoLC7Ozs0tLS6PRaFtbW2lpafBqW1tbenp6cM6gfdva2lJSUnp7e/v6+tLS0oKL3JFIJCcnJzs7O2juhoaGefPm5efnB2VfWVlZX19vugBAKCNlMF0AIJSRMpguABDKSBkwXQAIZaQMmC4AhDJSBkwXAEIZKQOmCwChjJQB0wWAUAYpg+kCAKGMlMF0AYBQRspgugBAKCNlMF0AIJQZIsnJyZNhaKSnp/tPDAChDAAAQhkAAIQyAAAIZQAAEMoAAIBQBgAAoQwAAEIZAACEMgAACGUAABDKAAAglAEAQCgDAIBQBgAAoQwAAEIZAAAuTv8PTp8myvYT4PUAAABQZVhJZk1NACoAAAAIAAIBEgADAAAAAQABAACHaQAEAAAAAQAAACYAAAAAAAOgAQADAAAAAQABAACgAgAEAAAAAQAAA7igAwAEAAAAAQAAAYEAAAAAmQvpBwAAACV0RVh0ZGF0ZTpjcmVhdGUAMjAyMC0xMC0wNVQwNzo1MzozNSswMDowMCgGuf8AAAAldEVYdGRhdGU6bW9kaWZ5ADIwMjAtMTAtMDVUMDc6NTM6MzUrMDA6MDBZWwFDAAAAEXRFWHRleGlmOkNvbG9yU3BhY2UAMQ+bAkkAAAASdEVYdGV4aWY6RXhpZk9mZnNldAAzOK24viMAAAAYdEVYdGV4aWY6UGl4ZWxYRGltZW5zaW9uADk1MttYhLwAAAAYdEVYdGV4aWY6UGl4ZWxZRGltZW5zaW9uADM4NWAKC/IAAAAASUVORK5CYII=)

#### Service backed plugins[​](https://backstage.io/docs/overview/architecture-overview#service-backed-plugins "Direct link to Service backed plugins")

Service backed plugins make API requests to a service which is within the purview of the organisation running Backstage.

The Lighthouse plugin, for example, makes requests to the [lighthouse-audit-service](https://github.com/spotify/lighthouse-audit-service). The `lighthouse-audit-service` is a microservice which runs a copy of Google's [Lighthouse library](https://github.com/GoogleChrome/lighthouse/) and stores the results in a PostgreSQL database.

Its architecture looks like this:

![lighthouse plugin backed to microservice and database](https://backstage.io/assets/images/lighthouse-plugin-architecture-bc78e37450e4ffd6a9149f752e142ada.png)

The software catalog in Backstage is another example of a service backed plugin. It retrieves a list of services, or "entities", from the Backstage Backend service and renders them in a table for the user.

### Third-party backed plugins[​](https://backstage.io/docs/overview/architecture-overview#third-party-backed-plugins "Direct link to Third-party backed plugins")

Third-party backed plugins are similar to service backed plugins. The main difference is that the service which backs the plugin is hosted outside of the ecosystem of the company hosting Backstage.

The CircleCI plugin is an example of a third-party backed plugin. CircleCI is a SaaS service which can be used without any knowledge of Backstage. It has an API which a Backstage plugin consumes to display content.

Requests going to CircleCI from the user's browser are passed through a proxy service that Backstage provides. Without this, the requests would be blocked by Cross Origin Resource Sharing policies which prevent a browser page served at [https://example.com](https://example.com/) from serving resources hosted at [https://circleci.com](https://circleci.com/).

![CircleCI plugin talking to proxy talking to SaaS Circle CI](https://backstage.io/assets/images/circle-ci-plugin-architecture-637c8554269d35240f4458083cca146f.png)

## Package Architecture[​](https://backstage.io/docs/overview/architecture-overview#package-architecture "Direct link to Package Architecture")

Backstage relies heavily on NPM packages, both for distribution of libraries, and structuring of code within projects. While the way you structure your Backstage project is up to you, there is a set of established patterns that we encourage you to follow. These patterns can help set up a sound project structure as well as provide familiarity between different Backstage projects.

The following diagram shows an overview of the package architecture of Backstage. It takes the point of view of an individual plugin and all of the packages that it may contain, indicated by the thicker border and italic text. Surrounding the plugin are different package groups which are the different possible interface points of the plugin. Note that not all library package lists are complete as packages have been omitted for brevity.

![Package architecture](https://backstage.io/assets/images/package-architecture.drawio-15aac8979d89a6c2f7eb24f04d8d3b32.svg)

### Overview[​](https://backstage.io/docs/overview/architecture-overview#overview-1 "Direct link to Overview")

The arrows in the diagram above indicate a runtime dependency on the code of the target package. This strict dependency graph only applies to runtime `dependencies`, and there may be `devDependencies` that break the rules of this table for the purpose of testing. While there are some arrows that show a dependency on a collection of frontend, backend and isomorphic packages, those still have to abide by important compatibility rules shown in the bottom left.

The `app` and `backend` packages are the entry points of a Backstage project. The `app` package is the frontend application that brings together a collection of frontend plugins and customizes them to fit an organization, while the `backend` package is the backend service that powers the Backstage application. Worth noting is that there can be more than one instance of each of these packages within a project. Particularly the `backend` packages can benefit from being split up into smaller deployment units that each serve their own purpose with a smaller collection of plugins.

### Plugin Packages[​](https://backstage.io/docs/overview/architecture-overview#plugin-packages "Direct link to Plugin Packages")

A typical plugin consists of up to five packages, two frontend ones, two backend, and one isomorphic package. All packages within the plugin must share a common prefix, typically of the form `@<scope>/plugin-<plugin-id>`, but alternatives like `backstage-plugin-<plugin-id>` or `@<scope>/backstage-plugin-<plugin-id>` are also valid. Along with this prefix, each of the packages have their own unique suffix that denotes their role. In addition to these five plugin packages it's also possible for a plugin to have additional frontend and backend modules that can be installed to enable optional features. For a full list of suffixes and their roles, see the [Plugin Package Structure ADR](https://backstage.io/docs/architecture-decisions/adrs-adr011).

The `-react`, `-common`, and `-node` plugin packages together form the external library of a plugin. The plugin library enables other plugins to build on top of and extend a plugin, and likewise allows the plugin to depend on and extend other plugins. Because of this, it is preferable that plugin library packages allow duplicate installations of themselves, as you may end up with a mix of versions being installed as dependencies of various plugins. It is also forbidden for plugins to directly import non-library packages from other plugins, all communication between plugins must be handled through libraries and the application itself.

### Frontend Packages[​](https://backstage.io/docs/overview/architecture-overview#frontend-packages "Direct link to Frontend Packages")

The frontend packages are grouped into two main groups. The first one is "Frontend App Core", which is the set of packages that are only used by the `app` package itself. These packages help build up the core structure of the app as well as provide a foundation for the plugin libraries to rely upon.

The second group is the rest of the shared packages, further divided into "Frontend Plugin Core" and "Frontend Libraries". The core packages are considered particularly stable and form the core of the frontend framework. Their most important role is to form the boundary around each plugin and provide a set of tools that helps you combine a collection of plugins into a running application. The rest of the frontend packages are more traditional libraries that serve as building blocks to create plugins.

### Backend Packages[​](https://backstage.io/docs/overview/architecture-overview#backend-packages "Direct link to Backend Packages")

The backend library packages do not currently share a similar plugin architecture as the frontend packages. They are instead simply a collection of building blocks and patterns that help you build backend services. This is however likely to change in the future.

### Common Packages[​](https://backstage.io/docs/overview/architecture-overview#common-packages "Direct link to Common Packages")

The common packages are the packages effectively depended on by all other pages. This is a much smaller set of packages but they are also very pervasive. Because the common packages are isomorphic and must execute both in the frontend and backend, they are never allowed to depend on any of the frontend or backend packages.

The Backstage CLI is in a category of its own and is depended on by virtually all other packages. It's not a library in itself though, and must always be a development dependency only.

### Deciding where you place your code[​](https://backstage.io/docs/overview/architecture-overview#deciding-where-you-place-your-code "Direct link to Deciding where you place your code")

It can sometimes be difficult to decide where to place your plugin code. For example should it go directly in the `-backend` plugin package or in the `-node` package? As a general guideline you should try to keep the exposure of your code as low as possible. If it doesn't need to be public API, it's best to avoid. If you don't need it to be used by other plugins, then keep it directly in the plugin packages.

Below is a chart to help you decide where to place your code.

![Package decision](https://backstage.io/assets/images/package-decision.drawio-c91bae580f0f2534f0582b38d288ed1e.svg)

## Databases[​](https://backstage.io/docs/overview/architecture-overview#databases "Direct link to Databases")

As we have seen, both the `lighthouse-audit-service` and `catalog-backend` require a database to work with.

The Backstage backend and its built-in plugins are based on the [Knex](http://knexjs.org/) library, and set up a separate logical database per plugin. This gives great isolation and lets them perform migrations and evolve separate from each other.

The Knex library supports a multitude of databases, but Backstage is at the time of writing tested primarily against two of them: SQLite, which is mainly used as an in-memory mock/test database, and PostgreSQL, which is the preferred production database. Other databases such as the MySQL variants are reported to work but [aren't tested as fully](https://github.com/backstage/backstage/issues/2460) yet.

## Cache[​](https://backstage.io/docs/overview/architecture-overview#cache "Direct link to Cache")

The Backstage backend and its built-in plugins are also able to leverage cache stores as a means of improving performance or reliability. Similar to how databases are supported, plugins receive logically separated cache connections, which are powered by [Keyv](https://github.com/lukechilds/keyv) under the hood.

At this time of writing, Backstage can be configured to use one of three cache stores: memory, which is mainly used for local testing, memcache or Redis, which are cache stores better suited for production deployment. The right cache store for your Backstage instance will depend on your own run-time constraints and those required of the plugins you're running.

### Use memory for cache[​](https://backstage.io/docs/overview/architecture-overview#use-memory-for-cache "Direct link to Use memory for cache")

```
backend:  cache:    store: memory
```

### Use memcache for cache[​](https://backstage.io/docs/overview/architecture-overview#use-memcache-for-cache "Direct link to Use memcache for cache")

```
backend:  cache:    store: memcache    connection: user:pass@cache.example.com:11211
```

### Use Redis for cache[​](https://backstage.io/docs/overview/architecture-overview#use-redis-for-cache "Direct link to Use Redis for cache")

```
backend:  cache:    store: redis    connection: redis://user:pass@cache.example.com:6379    useRedisSets: true
```

The useRedisSets flag is explained [here](https://github.com/jaredwray/keyv/tree/main/packages/redis#useredissets).

Contributions supporting other cache stores are welcome!

## Containerization[​](https://backstage.io/docs/overview/architecture-overview#containerization "Direct link to Containerization")

The example Backstage architecture shown above would Dockerize into three separate Docker images.

1. The frontend container
2. The backend container
3. The Lighthouse audit service container

![Boxes around the architecture to indicate how it is containerised](https://backstage.io/assets/images/containerised-18fbf8ed08f29df8fb922e4aa84c3864.png)

The backend container can be built by running the following command:

```
yarn run buildyarn run build-image
```

This will create a container called `example-backend`.

The lighthouse-audit-service container is already publicly available in Docker Hub and can be downloaded and run with

```
docker run spotify/lighthouse-audit-service:latest
```