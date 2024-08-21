---
id: 01J5TW839J0PEG1TWY3RPGZGNS
modified: 2024-08-21T12:29:11-04:00
title: Prisma Architecture, Engine, and Design
description: A look at what Prisma looks like under the hood and from the view point of a software architect
tags:
  - architecture
  - software-engineering
  - prisma
  - orm
  - system-design
---
# Engines

From a technical perspective, Prisma Client consists of three major components:

- JavaScript client library
- TypeScript type definitions
- A query engine

All of these components are located in the [generated `.prisma/client` folder](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/generating-prisma-client#the-prismaclient-npm-package) after you ran `prisma generate`.

This page covers relevant technical details about the query engine.

## Prisma engines[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#prisma-engines "Direct link to Prisma engines")

At the core of each module, there typically is a [Prisma engine](https://github.com/prisma/prisma-engines) that implements the core set of functionality. Engines are implemented in [Rust](https://www.rust-lang.org/) and expose a low-level API that is used by the higher-level interfaces.

A Prisma engine is the **direct interface to the database**, any higher-level interfaces always communicate with the database _through_ the engine-layer.

As an example, Prisma Client connects to the [query engine](https://www.prisma.io/docs/orm/more/under-the-hood/engines) in order to read and write data in a database:

![Prisma engine](https://www.prisma.io/docs/assets/images/typical-flow-query-engine-at-runtime-73ffdee4acc20a853bbd431dc12fb64f.png)

### Using custom engine libraries or binaries[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#using-custom-engine-libraries-or-binaries "Direct link to Using custom engine libraries or binaries")

By default, all engine files are automatically downloaded into the `node_modules/@prisma/engines` folder when you install or update `prisma`, the Prisma CLI package. The [query engine](https://www.prisma.io/docs/orm/more/under-the-hood/engines) is also copied to the generated Prisma Client when you call `prisma generate`. You might want to use a [custom library or binary](https://github.com/prisma/prisma-engines) file if:

- Automated download of engine files is not possible.
- You have created your own engine library or binary for testing purposes, or for an OS that is not officially supported.

Use the following environment variables to specify custom locations for your binaries:

- [`PRISMA_QUERY_ENGINE_LIBRARY`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_query_engine_library) (Query engine, library)
- [`PRISMA_QUERY_ENGINE_BINARY`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_query_engine_binary) (Query engine, binary)
- [`PRISMA_SCHEMA_ENGINE_BINARY`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_schema_engine_binary) (Schema engine)
- [`PRISMA_MIGRATION_ENGINE_BINARY`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_migration_engine_binary) (Migration engine)
- [`PRISMA_INTROSPECTION_ENGINE_BINARY`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_introspection_engine_binary) (Introspection engine)

warning

- `PRISMA_MIGRATION_ENGINE_BINARY` variable is deprecated in [5.0.0](https://github.com/prisma/prisma/releases/tag/5.0.0).
- The Introspection Engine is served by the Migration Engine from [4.9.0](https://github.com/prisma/prisma/releases/tag/4.9.0). Therefore, the `PRISMA_INTROSPECTION_ENGINE` environment variable will not be used.
- The `PRISMA_FMT_BINARY` variable is used in versions [4.2.0](https://github.com/prisma/prisma/releases/tag/4.2.0) or lower.

#### Setting the environment variable[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#setting-the-environment-variable "Direct link to Setting the environment variable")

You can define environment variables globally on your machine or in the `.env` file.

##### a) The `.env` file[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#a-the-env-file "Direct link to a-the-env-file")

Add the environment variable to the [`.env` file](https://www.prisma.io/docs/orm/more/development-environment/environment-variables/env-files).

- Linux, Unix, MacOS
- Windows

```
PRISMA_QUERY_ENGINE_BINARY=custom/my-query-engine-unix
```

> **Note**: It is possible to [use an `.env` file in a location outside the `prisma` folder](https://www.prisma.io/docs/orm/more/development-environment/environment-variables/managing-env-files-and-setting-variables).

##### b) Global environment variable[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#b-global-environment-variable "Direct link to b) Global environment variable")

Run the following command to set the environment variable globally (in this example, `PRISMA_QUERY_ENGINE_BINARY`):

- Linux, Unix, MacOS
- Windows

```
export PRISMA_QUERY_ENGINE_BINARY=/custom/my-query-engine-unix
```

#### Test your environment variable[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#test-your-environment-variable "Direct link to Test your environment variable")

Run the following command to output the paths to all binaries:

```
npx prisma -v
```

The output shows that the query engine path comes from the `PRISMA_QUERY_ENGINE_BINARY` environment variable:

- Linux, Unix, MacOS
- Windows

```
Current platform     : darwinQuery Engine         : query-engine d6ff7119649922b84e413b3b69660e2f49e2ddf3 (at /custom/my-query-engine-unix)Migration Engine     : migration-engine-cli d6ff7119649922b84e413b3b69660e2f49e2ddf3 (at /myproject/node_modules/@prisma/engines/migration-engine-unix)Introspection Engine : introspection-core d6ff7119649922b84e413b3b69660e2f49e2ddf3 (at /myproject/node_modules/@prisma/engines/introspection-engine-unix)
```

### Hosting engines[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#hosting-engines "Direct link to Hosting engines")

The [`PRISMA_ENGINES_MIRROR`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_engines_mirror) environment variable allows you to host engine files via a private server, AWS bucket or other cloud storage. This can be useful if you have a custom OS that requires custom-built engines.

```
PRISMA_ENGINES_MIRROR=https://my-aws-bucket
```

## The query engine file[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#the-query-engine-file "Direct link to The query engine file")

The **query engine file** is different for each operating system. It is named `query-engine-PLATFORM` or `libquery_engine-PLATFORM` where `PLATFORM` corresponds to the name of a compile target. Query engine file extensions depend on the platform as well. As an example, if the query engine must run on a [Darwin](https://en.wikipedia.org/wiki/Darwin_(operating_system)) operating system such as macOS Intel, it is called `libquery_engine-darwin.dylib.node` or `query-engine-darwin`. You can find an overview of all supported platforms [here](https://www.prisma.io/docs/orm/reference/prisma-schema-reference#binarytargets-options).

The query engine file is downloaded into the `runtime` directory of the generated Prisma Client when `prisma generate` is called.

Note that the query engine is implemented in Rust. The source code is located in the [`prisma-engines`](https://github.com/prisma/prisma-engines/) repository.

## The query engine at runtime[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#the-query-engine-at-runtime "Direct link to The query engine at runtime")

By default, Prisma Client loads the query engine as a [Node-API library](https://nodejs.org/api/n-api.html). You can alternatively [configure Prisma to use the query engine compiled as an executable binary](https://www.prisma.io/docs/orm/more/under-the-hood/engines#configuring-the-query-engine), which is run as a sidecar process alongside your application. The Node-API library approach is recommended since it reduces the communication overhead between Prisma Client and the query engine.

![Diagram showing the query engine and Node.js at runtime](https://www.prisma.io/docs/assets/images/query-engine-node-js-at-runtime-462396e7623f826c6b070819fd061c92.png)

The query engine is started when the first Prisma Client query is invoked or when the [`$connect()`](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/databases-connections/connection-management) method is called on your `PrismaClient` instance. Once the query engine is started, it creates a [connection pool](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/databases-connections/connection-pool) and manages the physical connections to the database. From that point onwards, Prisma Client is ready to send [queries](https://www.prisma.io/docs/orm/prisma-client/queries/crud) to the database (e.g. `findUnique()`, `findMany`, `create`, ...).

The query engine is stopped and the database connections are closed when [`$disconnect()`](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/databases-connections/connection-management) is invoked.

The following diagram depicts a "typical flow":

1. `$connect()` is invoked on Prisma Client
2. The query engine is started
3. The query engine establishes connections to the database and creates connection pool
4. Prisma Client is now ready to send queries to the database
5. Prisma Client sends a `findMany()` query to the query engine
6. The query engine translates the query into SQL and sends it to the database
7. The query engine receives the SQL response from the database
8. The query engine returns the result as plain old JavaScript objects to Prisma Client
9. `$disconnect()` is invoked on Prisma Client
10. The query engine closes the database connections
11. The query engine is stopped

![Typical flow of the query engine at run time](https://www.prisma.io/docs/assets/images/typical-flow-query-engine-at-runtime-73ffdee4acc20a853bbd431dc12fb64f.png)

## Responsibilities of the query engine[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#responsibilities-of-the-query-engine "Direct link to Responsibilities of the query engine")

The query engine has the following responsibilities in an application that uses Prisma Client:

- manage physical database connections in connection pool
- receive incoming queries from the Prisma Client Node.js process
- generate SQL queries
- send SQL queries to the database
- process responses from the database and send them back to Prisma Client

## Debugging the query engine[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#debugging-the-query-engine "Direct link to Debugging the query engine")

You can access the logs of the query engine by setting the [`DEBUG`](https://www.prisma.io/docs/orm/prisma-client/debugging-and-troubleshooting/debugging) environment variable to `engine`:

```
export DEBUG="engine"
```

You can also get more visibility into the SQL queries that are generated by the query engine by setting the [`query` log level](https://www.prisma.io/docs/orm/reference/prisma-client-reference#log-levels) in Prisma Client:

```
const prisma = new PrismaClient({  log: ['query'],})
```

Learn more about [Debugging](https://www.prisma.io/docs/orm/prisma-client/debugging-and-troubleshooting/debugging) and [Logging](https://www.prisma.io/docs/orm/prisma-client/observability-and-logging/logging).

## Configuring the query engine[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#configuring-the-query-engine "Direct link to Configuring the query engine")

### Defining the query engine type for Prisma Client[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#defining-the-query-engine-type-for-prisma-client "Direct link to Defining the query engine type for Prisma Client")

[As described above](https://www.prisma.io/docs/orm/more/under-the-hood/engines#the-query-engine-at-runtime) the default query engine is a Node-API library that is loaded into Prisma Client, but there is also an alternative implementation as an executable binary that runs in its own process. You can configure the query engine type by providing the `engineType` property to the Prisma Client `generator`:

```
generator client {  provider   = "prisma-client-js"  engineType = "binary"}
```

Valid values for `engineType` are `binary` and `library`. You can also use the environment variable [`PRISMA_CLIENT_ENGINE_TYPE`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_client_engine_type) instead.

info

- Until Prisma 3.x the default and only engine type available was `binary`, so there was no way to configure the engine type to be used by Prisma Client and Prisma CLI.
- From versions [2.20.0](https://github.com/prisma/prisma/releases/2.20.0) to 3.x the `library` engine type was available and used by default by [activating the preview feature flag](https://www.prisma.io/docs/orm/reference/preview-features/client-preview-features#enabling-a-prisma-client-preview-feature) "`nApi`" or using the `PRISMA_FORCE_NAPI=true` environment variable.

### Defining the query engine type for Prisma CLI[​](https://www.prisma.io/docs/orm/more/under-the-hood/engines#defining-the-query-engine-type-for-prisma-cli "Direct link to Defining the query engine type for Prisma CLI")

Prisma CLI also uses its own query engine for its own needs. You can configure it to use the binary version of the query engine by defining the environment variable [`PRISMA_CLI_QUERY_ENGINE_TYPE=binary`](https://www.prisma.io/docs/orm/reference/environment-variables-reference#prisma_cli_query_engine_type).