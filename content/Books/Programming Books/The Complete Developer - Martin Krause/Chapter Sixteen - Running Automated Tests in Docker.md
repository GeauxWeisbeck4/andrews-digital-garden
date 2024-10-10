---
id: 01J9QDVNESQM035K2SQ5E7Y59F
modified: 2024-10-08T21:21:35-04:00
---
In this short final chapter, you’ll write a couple of automated tests that verify the state of the Food Finder application. Then you’ll configure a Docker service to continuously run them.

We’ll focus on evaluating the application’s header by using a snapshot test and mocking the user session. We won’t create tests for the other components or our middleware, services, or APIs. However, I encourage you to build these on your own. Try using browser-based end-to-end tests, with a specialized framework such as Cypress or Playwright, to test entire pages. You can find installation instructions and examples for both frameworks at [_https://nextjs.org/docs/testing_](https://nextjs.org/docs/testing).

### Adding Jest to the Project

Install the Jest libraries with npm:

```
$ docker exec -it foodfinder-application npm install --save-dev jest \
jest-environment-jsdom @testing-library/react @testing-library/jest-dom
```

Next, configure Jest to work with our Next.js setup by creating a new file called _jest.config.js_ containing the code in [Listing 16-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter16.xhtml#Lis16-1). Save the file in the application’s root folder.

```
const nextJest = require("next/jest");

const createJestConfig = nextJest({
    dir: "./",
});

const customJestConfig = {
    moduleDirectories: ["node_modules", "<rootDir>/"],
    testEnvironment: "jest-environment-jsdom",
};

module.exports = createJestConfig(customJestConfig);
```

Listing 16-1: The jest.config.js file

We leverage the built-in Next.js Jest configuration, so we need to configure the project’s base directory to load the _config_ and _.env_ files into the test environment. Then we set the location of the module directories and the global test environment. We use a global setting here because our snapshot tests will require a DOM environment.

Now we want to be able to run the tests with npm commands. Therefore, add the two commands in [Listing 16-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter16.xhtml#Lis16-2) to the scripts property of the project’s _package.json_ file.

```
    "test": "jest ",
    "testWatch": "jest --watchAll"
```

Listing 16-2: Two commands added to the package.json file’s scripts property

The first command executes all available tests once, and the second continuously watches for file changes and then reruns the tests if it detects one.

### Setting Up Docker

To run the tests using Docker, add another service to _docker-compose.yml_ that uses the Node.js image. On startup, this service will run npm run testWatch, the command we just defined. In doing so, we’ll continuously run the tests and get instant feedback about the application’s state. Modify the file to match the code in [Listing 16-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter16.xhtml#Lis16-3).

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

    application:
        container_name: foodfinder-application
        image: node:lts-alpine
        working_dir: /home/node/code/foodfinder-application
        ports:
            - "3000:3000"
        volumes:
            - ./code:/home/node/code
        depends_on:
            - backend
        environment:
            - HOST=0.0.0.0
            - CHOKIDAR_USEPOLLING=true
            - CHOKIDAR_INTERVAL=100
        tty: true
        command: "npm run dev"

    jest:
        container_name: foodfinder-jest
        image: node:lts-alpine
        working_dir: /home/node/code/foodfinder-application
        volumes:
            - ./code:/home/node/code
        depends_on:
            - backend
            - application
        environment:
            - NODE_ENV=test
        tty: true
        command: "npm run testWatch"

volumes:
    mongodb_data_container:
```

Listing 16-3: The modified docker-compose.yml file with the jest service

Our small service, named _jest_, uses the official Node.js Alpine image we’ve used previously. We set the working directory and use the volumes property to make our code available in this container as well. Unlike our application’s service, however, the _jest_ service sets the Node.js environment to test and runs the testWatch command.

Restart the Docker containers; the console should indicate that Jest is watching our files.

### Writing Snapshot Tests for the Header Element

As in [Chapter 8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml), create the ___tests___ folder to hold our test files in the application’s root directory. Then add the _header.snapshot.test.tsx_ file containing the code in [Listing 16-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter16.xhtml#Lis16-4).

```
import {act, render} from "@testing-library/react";
import {useSession} from "next-auth/react";
import Header from "components/header";

jest.mock("next-auth/react");
describe("The Header component", () => {
    it("renders unchanged when logged out", async () => {
        (useSession as jest.Mock).mockReturnValueOnce({
            data: {user: {}},
            status: "unauthenticated",
        });
        let container: HTMLElement | undefined = undefined;
        await act(async () => {
            container = render(<Header />).container;
        });
        expect(container).toMatchSnapshot();
    });

    it("renders unchanged when logged in", async () => {
        (useSession as jest.Mock).mockReturnValueOnce({
            data: {
                user: {
                    name: "test user",
                    fdlst_private_userId: "rndmusr",
                },
            },
            status: "authenticated",
        });
        let container: HTMLElement | undefined = undefined;
        await act(async () => {
            container = render(<Header />).container;
        });
        expect(container).toMatchSnapshot();
    });
});
```

Listing 16-4: The __tests__/header.snapshot.test.tsx file

This test should resemble those you wrote in [Chapter 8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml). Note that we import the useSession hook from _next-auth/react_ and then use jest.mock to replace it in the _arrange_ step of each test. By replacing the session with a mocked one that returns the state, we can verify that the header component behaves as expected for both logged-in and logged-out users. We describe the test suite for the Header component by using the arrange, act, and assert pattern and verify that the rendered component matches the stored snapshot.

The first test case uses an empty session and the _unauthenticated_ status to render the header in a logged-out state. The second test case uses a session with minimal data and sets the user’s status to _authenticated_. This lets us verify that an existing session shows a different user interface than an empty session does.

If you write additional tests, make sure to add them to the ___tests___ folder.

### Summary

You’ve successfully added a few simple snapshot tests to verify that the Food Finder application works as intended. Using an additional Docker service, you can continuously verify that additional developments won’t break the application.

Congratulations! You’ve successfully created your first full-stack application with TypeScript, React, Next.js, Mongoose, and MongoDB. You’ve used Docker to containerize your application and Jest to test it. With the knowledge gained in the book and its exercises, you’ve laid the foundation for your career as a full-stack developer.