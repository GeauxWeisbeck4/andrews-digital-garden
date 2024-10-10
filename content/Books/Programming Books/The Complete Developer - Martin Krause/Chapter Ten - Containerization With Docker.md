---
id: 01J9QDEJB6Z6RGYX3VB79T12RV
modified: 2024-10-08T21:15:13-04:00
title: Chapter Ten - Containerization With Docker
description: How to use Docker to containerize apps
tags:
  - docker
  - full-stack
  - dev-ops
  - web-development
  - programming
  - books
---
Professional full-stack developers frequently work with Docker and, more broadly, containers. _Docker,_ an open source containerization platform, solves three common problems.

First, it lets us run a particular version of some software, such as Node.js, for each of our projects. Second, it decouples the development environment from our local machine and creates a reproducible way to run the application. Third, unlike traditional virtual machines, Docker containers run on a shared host. Therefore, they are smaller in size and consume less memory than classic virtual machines, which emulate a complete system and are often hardware specific. As a result, container-based applications are lightweight and easy to scale. These advantages have made Docker the most appreciated development platform in recent years.

This chapter covers the fundamentals of Docker. We first walk through the steps required to containerize our Next.js application by creating a Docker container running the latest Node.js version and serving the application from inside the container. Then we explore the concept of a microservice architecture and create two microservices using Docker.

### The Containerization Architecture

In their daily lives, developers must regularly switch between applications that require different versions of the same library. For example, a JavaScript-focused developer might need a different Node.js or TypeScript version for each of their projects. Of course, they could switch the installed Node.js version on their local machine with tools such as nvm whenever they need to work on a different project. But instead of resorting to crude hacks, they could choose a more elegant solution.

Using Docker, we can separate our application or its services into independent containers, each of which provides a service-specific environment. These containers run on an operating system of our choosing (often Debian, Ubuntu, or Alpine), with only the dependencies necessary to this particular application. Containers are isolated from one another and communicate through defined APIs.

When we use a Docker container during the development process, we facilitate the application’s later deployment. After all, the container provides a location-independent version of our application that is platform agnostic. Therefore, we already know that our application works with the installed dependencies and that no conflicts or additional installation steps are necessary. Instead of setting up a remote server with the required software and then deploying and testing our application afterward, we can simply move our Docker container to the server and spin it up there.

In situations when we need to move to a different server, scale our application, add additional database servers, or distribute instances across several locations, Docker lets us deploy our application by using the same straightforward process. Instead of managing different hosts and configurations, we can effectively build a platform-agnostic application and run the same containers everywhere.

### Installing Docker

To check whether you already have Docker installed, open the command line and run docker -v. If you see a version number higher than 20, you should be able to follow along with the examples in this chapter. Otherwise, you’ll need to install the most recent version of Docker from Docker Inc. Go to [_https://www.docker.com/products/docker-desktop/_](https://www.docker.com/products/docker-desktop/). Then choose the Docker desktop installer for your operating system and download it. Execute the application and check the Docker version number on the command line. It should match the one you downloaded.

### Creating a Docker Container

Docker has several components. The physical or virtual machine on which the Docker daemon runs is the _host system_. While you’re developing your application locally, the host is your physical machine, and when you deploy your container, the host is the server that runs the application.

We use the _Docker daemon service_ on the host system to interact with all components of the Docker platform. The daemon provides Docker’s functionality through APIs and is the actual Docker application installed on our machine. Access the daemon using the docker command from the command line. Run docker --help to display all possible interactions.

We use Docker _containers_ to run our containerized applications. These containers are running instances of a particular Docker image, which is the artifact that contains the application. Each Docker image relies on a Dockerfile, which defines the configuration and the content of the Docker image.

#### Writing the Dockerfile

A _Dockerfile_ is a text file containing the information we need to set up a Docker image. It commonly builds upon some existing base image, such as a bare-bones Linux machine on which we’ve installed additional software or a pre-provisioned environment. For example, we might use a Linux image with Node.js, MongoDB, and all relevant dependencies installed.

Often, we can build upon an official image. For example, [Listing 10-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter10.xhtml#Lis10-1) shows the basic Dockerfile we use to containerize our refactored Next.js application. Dockerfiles contain keywords followed by commands, and we use the FROM keyword here to select the official Node.js Docker image. Create a file called _Dockerfile_ in your project’s root directory, next to the _package.json_ file, and add the code in [Listing 10-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter10.xhtml#Lis10-1) to it.

```
FROM node:current

WORKDIR /home/node
COPY package.json package-lock.json /home/node/
EXPOSE 3000
```

Listing 10-1: A simple Dockerfile for a typical Node.js-based application

The image we’ve selected contains a preconfigured Node.js system running on Debian. The version tag current gives you the most recent Node.js version; alternatively, we could provide a particular version number here. Hence, if you need to lock any application to a specific Node.js version, this is the line to do so. You could also use the slimmer node:current-slim image, a lightweight Debian distribution that contains only the software packages necessary to run Node.js. However, we need MongoDB’s in-memory server, so we’ll choose the regular image. You can see a list of the available images at [_https://hub.docker.com_](https://hub.docker.com/). Other images you’ll probably use in your career include those for WordPress, MySQL, Redis, Apache, and NGINX.

Finally, we use the WORKDIR keyword to set the working directory inside the Docker image to the user’s home directory. All future commands will now execute in this directory. We use the COPY keyword to add the _package .json_ and _package-lock.json_ files to the working directory. A Node.js application runs on port 3000 by default, so we use the EXPORT keyword to choose port 3000 for TCP connections. This connection will provide access to the application from outside the container.

#### Building the Docker Image

To create a Docker image from the Dockerfile, we use the docker image build command. During the build process, the Docker daemon reads the Dockerfile and executes the commands defined there to download and install software, copy local files into the image, and configure the environment. Run the following next to your Dockerfile to build the image from it:

```
$ docker image build --tag nextjs:latest .
[+] Building 11.9s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                   0.1s
 => => transferring dockerfile: 136B                                   0.0s
 => [1/2] FROM docker.io/library/node:current-alpine@sha256:HASH 0.0s
 => [2/2] WORKDIR /home/node                                           0.0s
 => => naming to docker.io/library/ nextjs:latest
```

The --tag flag gives the image the name nextjs and sets its version to latest. Now we can easily refer to this specific image at a later time. We use a period (.) at the end of the command to set the build context, limiting the docker build command’s file access to the current directory. In the output, the Docker daemon indicates that it successfully built the tagged image.

Now, to verify that we have access to the image, run the following. This command lists all locally available Docker images:

```
$ docker image ls
REPOSITORY    TAG        IMAGE
nextjs        latest     98b28358e19a
```

As expected, our newly created image has a random ID (98b28358e19a), is tagged as nextjs, and is available in the latest version. The Docker daemon may also display additional information, such as the size and age of the image, which aren’t relevant to us for now.

Docker provides additional commands for managing local and remote images. You can view a list of all available commands by running docker image --help. For example, to remove an existing image from your local machine, use docker image rm:

```
$ docker image rm <name:version or ID>
```

After a while, you’ll find that you’ve collected unused or outdated versions of your images, so deleting them to free up space on your machine with docker image prune is a good practice.

#### Serving the Application from the Docker Container

Docker containers are running instances of Docker images. You could use the same Docker image to spin up multiple containers, each with a unique name or ID. Once the container is running, you can synchronize local files to it. It listens on an exposed TCP or UDP port, so you can connect to it and execute commands inside it using SSH.

Let’s containerize our application. We’ll spin up the Docker container from our image, map the local Next.js files to the working directory, publish the exposed port, and finally start the Next.js development server. We can do all of this using docker container run:

```
$ docker container run \
--name nextjs_container \
--volume ~/nextjs_refactored/:/home/node/ \
--publish-all \
nextjs:latest npm run dev
> refactored-app@0.1.0 dev
> next dev

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
event - compiled client and server successfully in 10.9s (208 modules)
```

At first glance, this command might look complicated, but once we take a closer look at it, you’ll easily understand what it is doing. We pass it several flags, starting with the --name flag, which assigns a unique name to the running container. We’ll use this name to identify the container later.

Then we use the --volume flag to create a Docker volume. _Volumes_ are a simple way to share data between containers. Docker itself manages them, and they let us synchronize our application files to the _home/node/_ directory inside the container. We use the format source:destination to define a volume, and depending on your file structure, you might need to adjust the absolute path to this folder. In this example, we map _/nextjs_refactored/_ from the user’s home folder into the container.

The --publish-all flag publishes all exported ports and assigns them to random ports on the host system. We use docker container ls later to view the ports for our application. The last two arguments are intuitive: nextjs:latest points to the Docker image we want to use for the container, and npm run dev starts the Next.js development server as usual. The console output shows that the Node.js app inside the container is running and listening on port 3000.

#### Locating the Exposed Docker Port

Unfortunately, as soon as we try to access our Next.js application on port 3000, the browser notifies us that it isn’t accessible; no application is listening there. The problem is that we didn’t map the exposed Docker port 3000 to the host’s port 3000. Instead, we used the --publish-all flag and assigned a random port to the exposed Docker port.

Let’s run docker container ls to see details about all running Docker containers:

```
$ docker container ls
CONTAINER ID   IMAGE             PORTS                     NAMES
dff681898013   nextjs:latest     0.0.0.0:55000->3000/tcp   nextjs_container
```

Search for the name we assigned to our container, _nextjs_container_, and notice that port 55000 on the host maps to the Docker port 3000. Hence, we can access our application at _http://localhost:55000_. Open this URL in your browser. You should see the Next.js application.

If you glance at the URL bar, you’ll notice that the port we use to access the application is different from the one used in previous chapters because it is now running inside the Docker container. Try to access all of the pages and APIs we created previously before moving to the next section.

#### Interacting with the Container

You can view a list of all Docker commands for interacting with containers by running docker container --help. In most contexts, though, you’ll find it sufficient to know just a few of these. For example, use exec to execute commands inside an already running Docker container. We could use exec to connect to a shell inside the container by passing it the -it flag and the path to the shell, such as _/bin/sh_. The -i flag is short for --interactive, whereas -t runs a pseudoterminal. The interactive option lets us interact with the container, and the tty pseudoterminal keeps the Docker container running so that we can actually interact with it:

```
$ docker container exec -it <container ID or name> /bin/sh
```

The kill command stops a running Docker container:

```
$ docker container kill <containerid or name>
```

We can select the container by name or by using the container ID shown in the list of local running containers.

### Creating Microservices with Docker Compose

Docker provides us with a way to break up an application into small, autonomous units, called _microservices_. A microservice-driven architecture splits an application into a collection of self-contained services that communicate through well-defined APIs. It’s a relatively new architectural concept that gained traction around the late 2000s to early 2010s, when Docker and other tools that allowed for easier partitioning and orchestration of server resources became available. These tools form the technical foundation of a microservice architecture.

Microservices have several advantages. First, each independent service has a single purpose, which reduces its complexity. Therefore, it is more testable and maintainable. We can also deploy the microservices separately, spin up multiple instances of a single microservice to improve its performance, or swap it out altogether without affecting the whole application. Contrast these features with a traditional monolithic application whose user interface, middleware, and data storage exist in one single program built from a single code base. Even if a monolith uses a more modular approach, the code base couples them tightly, and you can’t swap out the elements easily.

Another characteristic feature of microservices is that dedicated teams can own just a single service and its code base. This means that they can select the appropriate tools, frameworks, and programming languages on a per-service basis. On the other hand, you’d typically use a single core language to write a monolithic application.

Now that you know how to create a single container from scratch, we’ll practice creating multiple containers; each will serve one part of an application. One way to use microservices is to create one service for the frontend and a second for the backend. The Food Finder application we’ll create in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml) will use this structure. The main benefit of this approach is that it lets us use a preconfigured MongoDB image for the database. For the example in this chapter, we’ll create a second service that watches our weather service and reruns its test suite as soon as the file changes. To do so, we’ll use the Docker Compose interface and define our microservice architecture in a _docker-compose.yml_ file.

#### Writing the docker-compose.yml File

We define all services in _docker-compose.yml_, a text file in the YAML format. This file also sets the properties, dependencies, and volumes for each service. Most properties are similar to the command line flags you specify when creating Docker images and containers. Create the file in the root folder of your application and add the code from [Listing 10-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter10.xhtml#Lis10-2) to it.

```
version: "3.0"
services:
    application:
        image:
            nextjs:latest
        ports:
            - "3000:3000"
        volumes:
            - ./:/home/node/
        command:
            "npm run dev"
    jest:
        image:
            nextjs:latest
        volumes:
            - ./:/home/node/
        command:
            "npx jest ./__tests__/mongoose/weather/services.test.ts --watchAll"
```

Listing 10-2: A basic docker-compose.yml file that defines the application and Jest services

Every _docker-compose.yml_ file starts by setting the version of the Docker Compose specification used. Depending on the version, we can use different properties and values. We then define each service as a single property under services. As discussed, we want to have two services: our Next.js application running on port 3000 and the Jest service, which watches the _services .test.ts_ file we created in [Chapter 8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml) and reruns the tests as soon as we change a file. We limit the watch command to retest only the services. This limits the scope of the exercises, but of course, you can rerun all tests if you’d like.

Each service follows roughly the same structure. First we define the image from which Docker Compose should create each container. This can be an official distribution or a locally built one. We use the nextjs image in the latest version for both services. Then, instead of using the --publishAll flag, we map the ports directly from 3000 to 3000. By doing so, we can connect to the application’s port 3000 from the host’s port 3000.

With the volumes property, we synchronize the files and paths from the host system into the container. This is similar to the mapping we used in the docker run command, but instead of supplying an absolute path, we can use relative paths for the source. Here we map the whole local directory _./_ into the container’s working directory _/home/node_. As before, we can edit the TypeScript files locally, and the application inside the container always uses the latest version of the files.

Until now, these properties have matched the command line arguments we used in the docker run command. Now we add the command property, which specifies the command that each container executes on startup. For the application service, we’ll start Next.js with the usual npm run dev command, whereas the Jest service should call Jest directly through npx. Providing the path to the test file and the --watchAll flag causes Jest to rerun the tests when the source code changes.

#### Running the Containers

Start the multi-container app with the docker compose up command. The output should look similar to what is shown here:

```
$ docker compose up
 [+] Running 2/2
 ⠿ Container application-1     Created                       0.0s
 ⠿ Container jest-1  Recreated                               0.4s
Attaching to application-1, jest-1
application-1     |
application-1     | > refactored-app@0.1.0 dev
application-1     | > next dev
application-1     |
application-1     | ready - started server on 0.0.0.0:3000, URL:
application-1     | http://localhost:3000
jest-1            | PASS __tests__/mongoose/weather/services.test.ts
jest-1            |  the weather services
jest-1            |     API storeDocument
jest-1            |       ✓ returns true  (9 ms)
jest-1            |       ✓ passes the document to Model.create()  (6 ms)
jest-1            |     API findByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code to Model.findOne()  (1 ms)
jest-1            |     API updateByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code and the new data to
jest-1            |         Model.updateOne()  (1 ms)
jest-1            |     API deleteByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code Model.deleteOne()  (1 ms)
jest-1            |
jest-1            | Test Suites: 1 passed, 1 total
jest-1            | Tests:       8 passed, 8 total
jest-1            |    0 total
jest-1            | Time:        4.059 s
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i.
```

The Docker daemon spins up all services. As soon as the application is ready, we see the status message from the Express.js server and can connect to it on the exposed port 3000. At the same time, the Jest container runs the tests for the weather services and reports that all are successful.

#### Rerunning the Tests

Now that we’ve started the Docker environment, let’s verify that the command to look for changes in the code and rerun tests is working as intended. To do so, we need to modify the source code to trigger Jest. Therefore, we open the _mongoose/weather/service.ts_ file and modify the contents by adding a blank line and then saving the file. Jest should rerun the test inside the container, as you can see from the output in [Listing 10-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter10.xhtml#Lis10-3).

```
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i.
jest-1            |
jest-1            | PASS __tests__/mongoose/weather/services.test.ts
jest-1            |   the weather services
jest-1            |     API storeDocument
jest-1            |       ✓ returns true  (9 ms)
jest-1            |       ✓ passes the document to Model.create()  (6 ms)
jest-1            |     API findByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code to Model.findOne()  (1 ms)
jest-1            |     API updateByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code and the new data to
jest-1            |         Model.updateOne()  (1 ms)
jest-1            |     API deleteByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code Model.deleteOne()  (1 ms)
jest-1            |
jest-1            | Test Suites: 1 passed, 1 total
jest-1            | Tests:       8 passed, 8 total
jest-1            |    0 total
jest-1            | Time:        7.089 s
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i
```

Listing 10-3: Rerunning the tests on files changed with jest --watchAll

All tests continue to pass. Connect to _http://localhost:3000_ and verify that your browser can still render the application.

#### Interacting with Docker Compose

Docker Compose provides a complete interface for managing microservice applications. You can see a list of available commands by running docker compose --help. The following are the most essential.

We use docker compose ls to get a list of all locally running Docker applications defined in _docker-compose.yml_ files. The command returns the name and status of the application:

```
$ docker compose ls
```

To shut down all running services defined in the _docker-compose.yml_ file in the current directory, run docker compose kill, which sends a SIGKILL command to the primary process inside each container:

```
$ docker compose kill
```

To kill the services with a more graceful SIGTERM command, use the following:

```
$ docker compose down
```

Instead of forcing a shutdown, this command gracefully removes all processes, containers, networks, and volumes created by docker compose up.

### Summary

Using the Docker containerization platform makes it easy to deploy applications and use a microservice architecture. This chapter covered the building blocks of the Docker ecosystem: the host, the Docker daemon, Dockerfiles, images, and containers. Using Docker Compose and Docker volumes, you split your application into single, self-contained services.

To unleash the full potential of Docker, read the official tutorials at [_https://docs.docker.com/get-started/_](https://docs.docker.com/get-started/) or those at [_https://docker-curriculum.com_](https://docker-curriculum.com/). In the next chapter, you’ll start to build the Food Finder application. This full-stack web application will build upon the knowledge you’ve gained in all previous chapters.