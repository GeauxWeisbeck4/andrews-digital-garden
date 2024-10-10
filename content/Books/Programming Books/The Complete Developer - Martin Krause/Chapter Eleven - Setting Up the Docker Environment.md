---
id: 01J9QDKJ2D4HTPDPWFCAA40X9Z
modified: 2024-10-08T21:17:45-04:00
title: Chapter Eleven - Setting Up the Docker Environment
description: Setting up Docker for creating FoodFinder
tags:
  - docker
  - full-stack
  - books
  - web-development
---
In this part of the book, you’ll build a full-stack application from scratch by using the knowledge you’ve acquired so far. While previous chapters explained parts of the technology stack, the remaining chapters focus on the code in more detail.

This chapter describes the application you’ll build and walks you through configuring the environment using Docker. While I recommend reading previous chapters before you start writing code, the only real requirement is that you have Docker installed and running before moving on. Consult [Chapter 10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter10.xhtml) for instructions on doing so.

> NOTE

_You can download the complete source code for the Food Finder application at_ [http://www.usemodernfullstack.dev/downloads/food-finder](http://www.usemodernfullstack.dev/downloads/food-finder) _and a ZIP file with only the required assets from_ [http://www.usemodernfullstack.dev/downloads/assets](http://www.usemodernfullstack.dev/downloads/assets)_._

### The Food Finder Application

The Food Finder application shows a list of restaurants and their locations. The user can click these to see additional details about each location. In addition, they can log in to the app with their GitHub accounts by using OAuth so that they can maintain a wish list of locations.

Behind the scenes, we’ll write this simple single-page application in TypeScript. After setting up the local environment, we’ll build the backend and middleware with Next.js, Mongoose, and MongoDB, which we’ll seed with initial data. Then we’ll add GraphQL to expose an API layer through which we can access a user’s wish list. To build the frontend, we’ll use our knowledge of React components, Next.js pages, and routing. We’ll also add an OAuth authorization flow with _next-auth_ to let users log in with GitHub. Finally, we’ll write automated tests with Jest to verify the integrity and stability of the application.

### Building the Local Environment with Docker

Docker decouples the development environment from our local machine. We’ll use it to create self-contained services for each part of our application. In the _docker-compose_ file, we’ll add one service for the backend, which provides the MongoDB database, and a second to run the Next.js application hosting the frontend and the middleware.

To start the development, create a new empty folder, _code_. This folder will serve as the application’s root and contain all the code for the Food Finder application. Later in this chapter, we’ll use the create-next-app helper command to add files to it.

Next, create an empty _docker-compose.yml_ file and a _.docker_ folder in this root folder. In the file, we will define the two services for our environment and store the seed data we need to create the container.

#### The Backend Container

The backend container provides nothing but the app’s MongoDB instance. For this reason, we can use the official MongoDB image, which Docker can download automatically, from the Docker registry without creating a custom Dockerfile.

##### Seeding the Database

We want MongoDB to begin with a prefilled database that contains a valid set of initial datasets. This process is called seeding the database, and we can automate it by copying the seeding script _seed-mongodb.js_ into the container’s _/docker-entrypoint-initdb.d/_ directory on startup. The MongoDB image executes the scripts in this folder against the database defined in the MONGO_INITDB_DATABASE environment variable if there is no data in the container’s _/data/db_ directory on startup.

Create a new folder, _foodfinder-backend_, in the _.docker_ folder, and then copy into the newly created folder the _seed-mongodb.js_ file from the _assets.zip_ file you downloaded earlier. The seed file’s content should look similar to [Listing 11-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml#Lis11-1).

```
db.locations.insert([
    {
        address: "6220 Avenue U",
        zipcode: "NY 11234",
        borough: "Brooklyn",
        cuisine: "Cafe",
        grade: "A",
        name: "The Roasted Bean",
        on_wishlist: [],
        location_id: "56018",
    },
--snip--
    {
        address: "405 Lexington Avenue",
        zipcode: "NY 10174",
        borough: "Manhattan",
        cuisine: "American",
        grade: "A",
        name: "The Diner At The Corner",
        on_wishlist: [],
        location_id: "63426",
    }
]);
```

Listing 11-1: The seed-mongodb.js file

You can see that the script interacts directly with a collection in the MongoDB instance that we’ll set up in the next section. We use MongoDB’s insert method to fill the database’s location collection with the documents. Note that we are working with the _native_ MongoDB driver to insert the documents instead of using Mongoose. We do so because Mongoose is not installed on the default MongoDB Docker image, and inserting the documents is a relatively simple task. Although we do not use Mongoose for seeding the database, the documents we insert need to match the schema we define with Mongoose later.

##### Creating the Backend Service

We can now define the backend service in the Docker setup. Add the code from [Listing 11-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml#Lis11-2) into the empty _docker-compose.yml_ file we created earlier.

```
version: "3.0"
services:
    backend:
        container_name: foodfinder-backend
        image: mongo:latest
        restart: always
        environment:
            DB_NAME: foodfinder
            MONGO_INITDB_DATABASE: foodfinder
        ports:
            - 27017:27017
        volumes:
            - "./.docker/foodfinder-backend/seed-mongodb.js:
/docker-entrypoint-initdb.d/seed-mongodb.js"
            - mongodb_data_container:/data/db

volumes:
    mongodb_data_container:
```

Listing 11-2: The docker-compose.yml file with the backend service

We first define the container’s name so that we can easily reference it later. As discussed earlier, we use the latest version of the official MongoDB image and specify that this container should always be restarted if it stops. Next, we use the environment variables to define the collections we’ll use with MongoDB. We define two of those: DB_NAME points to the collection we’ll use with Mongoose, and MONGO_INITDB_DATABASE points to the seed script. The scripts in _/docker-entrypoint-initdb.d/_ use this latter collection by default.

We want the script to populate our application’s database, so we set both variables to the same name, foodfinder, and thus we have a prefilled database for our Mongoose model.

Then we map and expose the container’s internal port 27017 to the host’s port 27017 so that the MongoDB instance is accessible to the application at _mongodb://backend:27017/foodfinder_. Notice that the connection string contains the service name, the port, and the database. Later, we store this connection string in the environment variables and use it to connect to the database from the middleware. Finally, we map and copy the seed script to the setup location and save the database data from _/data/db_ into the Docker volume _mongodb_data_container_. Because we want to split the string across two lines, we need to wrap it in double quotes (") according to the YAML conventions.

Now complete the Docker setup with docker compose up:

```
$ docker compose up
[+] Running 2/2
 ⠿ Network foodfinder_default                      Created                 0.1s
 ⠿ Container foodfinder-backend                    Created                 0.3s
Attaching to foodfinder-backend

foodfinder-backend  | /usr/local/bin/docker-entrypoint.sh: running /docker
                    /entrypoint-initdb.d/seed-mongodb.js
```

The output shows us that the Docker daemon successfully created the foodfinder-backend container and that the seeding script was executed during startup. Instead of going through the hassle of installing and maintaining MongoDB locally or finding a free or low-cost cloud instance, we’ve added MongoDB to our project with just a few lines of code in the _docker -compose_ file.

Stop the container with CRTL-C and remove it with docker compose down:

```
$ docker compose down
[+] Running 2/2
 ⠿ Container foodfinder-backend                     Removed                 0.0s
 ⠿ Network foodfinder_default                       Removed
```

Now we can add the frontend container.

#### The Frontend Container

Now we’ll create the containerized infrastructure for the frontend and middleware. Our approach will involve using create-next-app to scaffold the Next.js application, as we did in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml), relying on the official Node.js Docker image to decouple the application from any local Node.js installation.

As we’ll execute all Node.js-related commands inside this container, we technically don’t even need Node.js installed on our local machine; nor must we make sure the Node.js versions we use comply with Next.js’s requirements. Also, npm might install packages that are optimized for the operating system on which it is running, so by using npm inside the container, we ensure that npm installs the correct versions for Linux.

Nonetheless, we’ll want Docker to synchronize the Node.js _modules_ folder to our local system. This will allow our IDE to automatically use the installed dependencies, such as the TypeScript compiler and ESLint. Let’s start by creating a minimal Dockerfile.

##### Creating the Application Service

We add the combined frontend and middleware service to our Docker setup by placing the code from [Listing 11-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml#Lis11-3) into the services property of the project’s _docker-compose.yml_ file.

```
--snip--
services:

    application:
        container_name: foodfinder-application
        image: node:lts-alpine
        ports:
            - "3000:3000"
        volumes:
            - ./code:/home/node/code
        working_dir: /home/node/code/
        depends_on:
            - backend
        environment:
            - HOST=0.0.0.0
            - CHOKIDAR_USEPOLLING=true
            - CHOKIDAR_INTERVAL=100
        tty: true
    backend:
--snip--
```

Listing 11-3: The docker-compose.yml file with the backend and application service

The service for the Food Finder application follows the same structure as the service for the backend. First we set the container’s name. Then we define the image to be used for this particular service. While the backend service used the official MongoDB image, we now use the official Node.js image with the current LTS version running on Alpine Linux, a lightweight Linux distribution that requires significantly less memory than a Debian-based image.

We then expose and map port 3000, making the application available on _http://localhost:3000_, and map the local application’s code directory into the container. Next, we set the working directory to the _code_ directory. We specify that our container requires a running backend service, because the Next.js application will need a working connection to the MongoDB instance. In addition, we add environment variables. In particular, chokidar supports hot-reloading for the Next.js code. Finally, setting the tty property to true makes the container provide an interactive shell instead of shutting down. We’ll need the shell to execute commands inside the container.

##### Installing Next.js

With both services in place, we can now install Next.js inside the container. To do so, we need to start the container with docker compose up:

```
$ docker compose up

[+] Running 3/3
 ⠿ Network foodfinder_default                      Created                 0.1s
 ⠿ Container foodfinder-backend                    Created                 0.3s
 ⠿ Container foodfinder-application                Created                 0.3s
Attaching to foodfinder-application, foodfinder-backend
--snip--
foodfinder-application  | Welcome to Node.js ...
--snip--
```

Compare this command line output with the previous docker compose up output. You should see that the application container started successfully and that it runs a Node.js interactive shell.

Now we can use docker exec to execute commands inside the running container. Doing so has two main advantages. First, we don’t need any particular version of Node.js (or any version at all) on our local machine. Second, we run the Node.js application and npm commands on the Node.js Linux Alpine image so that the dependencies will be optimized for Alpine instead of for our host system.

To run npm commands inside the container, use docker exec -it foodfinder-application followed by the command to run. The Docker daemon connects to the terminal inside the container and executes the provided command in the application container’s working directory, _/home/node/code_, which we set previously. Let’s install the Next.js application there using the npx command discussed in [Chapter 5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter5.xhtml):

```
/home/node/code# docker exec -it foodfinder-application \
npx create-next-app@latest foodfinder-application \
--typescript --use-npm
Need to install the following packages:
  create-next-app
Ok to proceed? (y)
✔ Would you like to use ESLint with this project? ... No / Yes
Creating a new Next.js app in /home/node/code/foodfinder-application.

Success! Created foodfinder-application at /home/node/code/foodfinder-application
```

We set the project name to _foodfinder-application_ and accept the defaults. The rest of the output should look familiar to you.

As soon as the scaffolding is done, we can start the Next.js application with npm run dev. If you visit _http://localhost:3000_ in your browser, you should see the familiar Next.js splash screen. The _foodfinder-application_ folder should be mapped into the local _code_ folder, so we can edit the Next.js-related files locally.

##### Adjusting the Application Service for Restarts

Currently, connecting to the application container requires running docker exec after each restart through docker compose up and then calling npm run dev manually. Let’s make two minor adjustments in our application service to allow for a more convenient setup. Modify the file to match [Listing 11-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml#Lis11-4).

```
--snip--
services:
--snip--
    application:
--snip--
        volumes:
            - ./code:/home/node/code
        working_dir: /home/node/code/foodfinder-application
--snip--
        command: "npm run dev"
--snip--
```

Listing 11-4: The docker-compose.yml file to start Next.js automatically

First, change the working_dir property. Because we’re working with Next.js, we set it to the Next.js application’s root folder, _/home/node/code/foodfinder-application_, which contains the _package.json_ file. Then we add the command property with a value of npm run dev. With these two modifications, each docker compose up call should instantly start the Next.js application. Try starting the containers with docker compose up; the console output should show that Next.js runs and that it’s available at _http://localhost:3000_:

```
$ docker compose up
[+] Running 3/3
 ⠿ Network foodfinder_default                      Created    0.1s
 ⠿ Container foodfinder-backend                    Created    0.3s
 ⠿ Container foodfinder-application                Created    0.3s
Attaching to foodfinder-application, foodfinder-backend
foodfinder-application  |
foodfinder-application  | > foodfinder-application@0.1.0 dev
foodfinder-application  | > next dev
foodfinder-application  |
foodfinder-application  | ready - started server on 0.0.0.0:3000,
foodfinder-application  | url: foodfinder-application  | http://localhost:3000
foodfinder-application  | info  - Loaded env from /home/node/code/foodfinder-
foodfinder-application  | application/.env.local
```

If you visit _http://localhost:3000_ in your browser, you should see the Next.js splash screen without having to start the Next.js application manually.

Note that, if you’re using Linux or macOS without being the administrator or root user, you’ll need to adjust the application service and the startup command. Because the Docker daemon runs as a root user by default, all files it creates require root privileges. Your regular user doesn’t have those and cannot access those files. To avoid these possible issues, modify your setup so that the Docker daemon transfers the ownership to your user. Start by adding the code in [Listing 11-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter11.xhtml#Lis11-5) to the application service in the _docker-compose_ file.

```
services:
--snip--
    application:
--snip--
        user: ${MY_USER}
--snip--
```

Listing 11-5: The docker-compose.yml file with the user property

We add the user property to the application service and use the environment variable MY_USER as the property’s value. Then we modify the docker compose commands so that, on startup, we add the current user’s user ID and group ID to this environment variable. Instead of a plain docker compose up call, we use the following code:

```
MY_USER=$(id -u):$(id -g) docker compose up
```

We use the id helper program to save the user ID and group ID in the format userid:groupid to our environment variable, which the _docker-compose_ file then picks up. The -u flag returns the user ID, and the -g flag returns the group ID.

### Summary

We’ve set up our local development environment with Docker containers. With the _docker-compose.yml_ file we created in this chapter, we decoupled the application development from our local host system. Now we can switch our host systems and, at the same time, ensure that the Food Finder application always runs with the same Node.js version. In addition, we added a container running our MongoDB server, to which we’ll connect in the next chapter when we implement our application’s middleware.