---
id: 01J9QD9ZEDAHRY2QR95S75WCZ3
title: Chapter Eight - Testing With the Jest Framework
modified: 2024-10-08T21:16:32-04:00
description: Using Jest to test our software
tags:
  - programming
  - web-development
  - full-stack
  - tests
  - jest
  - books
---
Whenever you modify your code, you risk causing unforeseen side effects in another part of your application. As a result, guaranteeing the integrity and stability of a code base can be challenging. To do so, developers follow two main strategies.

In the first, an architectural pattern, we split our code into small, self-contained React components. By nature, these components don’t interfere with one another. Hence, changing one shouldn’t lead to any side effects. In the second, we perform automated unit testing, which this chapter covers using the Jest framework.

In the following sections, we discuss the essentials of automated unit testing and the benefits of using it. You’ll learn how to write a test suite in Jest and use its reports to improve your code. You’ll also handle dependencies by using code doubles. Lastly, you’ll explore other kinds of tests you might want to run against your application.

### Test-Driven Development and Unit Testing

Developers sometimes use the technique of _test-driven development (TDD)_, in which they write their automated tests before implementing the actual code to be tested. They first create a test to evaluate that the smallest possible unit of code would work as expected. Such a test is called a _unit test_. Next, they write the minimum amount of code necessary to pass the test.

This approach has distinct benefits. First, it lets you focus on your app’s requirements by explicitly defining the code’s functionality and edge cases. Therefore, you have a clear picture of its desired behavior, and you can identify unclear or missing specifications sooner rather than later. When you write tests after completing the functionality, they might reflect the behavior you implemented rather than the behavior you require.

Second, limiting yourself to writing only necessary code prevents your functions from becoming too complex and splits your application into small, understandable sections. Testable code is maintainable code. In addition, the technique ensures that your tests cover a large portion of the app’s code, a metric called _code coverage_, and by running the tests frequently during development, you’ll instantly recognize bugs introduced by new lines of code.

Depending on the situation, the _unit_ targeted by a unit test can be a module, a function, or a line of code. The tests aim to verify that each unit works in isolation. The single lines inside each test function are the test _steps_, and the whole test function is called a test _case_. Test _suites_ aggregate test cases into logical blocks. To be considered reproducible, the test must return the same results every time we run it. As we will explore in this chapter, this means that we must run the tests in a controlled environment with a defined dataset.

Facebook developed the Jest testing framework in conjunction with React, but we can use it with any Node.js project. It has a defined syntax for setting up and writing tests. Its _test runner_ executes these tests, automatically replaces any dependencies in our code, and generates a test-coverage report. Additional npm modules provide custom code for testing DOM or React components and, of course, adding TypeScript types.

### Using Jest

To use Jest in a project, we must install its required packages, create a directory for all test files, and add an npm script that will run the tests. Execute the following in your Next.js application’s root directory to install the framework, as well as type definitions from DefinitelyTyped as development dependencies:

```
$ npm install --save-dev jest @types/jest
```

Then create the directory in which to save your tests. Jest uses the ___tests___ folder by default, so make one in your root directory. Next, to add the npm script _test_ to your project, open the _package.json_ file and modify the scripts object to match the one in [Listing 8-1](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-1).

```
  "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "next lint",
      "test": "jest"
  },
```

Listing 8-1: The package.json file with the new text command

Now we can run tests with the npm test command. Usually, build servers execute this command by default during the build process. Lastly, to enable TypeScript support in Jest, add the ts-jest transpiler:

```
$ npm install --save-dev ts-jest
```

Also create a _jest.config_ file to add TypeScript by running npx ts-jest config:init.

### Creating an Example Module to Test

Let’s write some example code to help us understand unit testing and TDD. Say we want to create a new module in our app, _./helpers/sum.ts_. It should export a function, sum, that returns the sum of its parameters. To follow a TDD pattern, we’ll begin by creating test cases for this module.

First we need to create the function that will run our tests. Create a file called _sum.test.ts_ in the default test directory and add the code from [Listing 8-2](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-2).

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

});
```

Listing 8-2: The empty test suite

We import the sum function we’ll write later and use Jest’s describe function to create an empty test suite. As soon as we run the (nonexistent) tests with npm test, Jest should complain that there is no file called _sum.ts_ in the _helpers_ directory. Create this file and folder now, at the root directory of your project. Within the file, write the sum function shown in [Listing 8-3](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-3).

```
const sum = () => {};
export {sum};
```

Listing 8-3: The bare bones of the sum function

Now run the tests again with npm test. Because the code just exports a placeholder sum function that returns nothing, the Jest test runner again complains. This time, it informs us that the test suite needs to contain at least one test.

Let’s look at the anatomy of a test case and add a few test cases to the _sum.test.ts_ file during the process.

### Anatomy of a Test Case

There are two types of unit tests: state and interaction based. An _interaction-based_ test case verifies that the code under evaluation invokes a specific function, whereas a _state-based_ test case checks the code’s return value or resulting state. Both types follow the same three steps: arrange, act, and assert.

#### Arrange

To write independent and reproducible tests, we need to first _arrange_ our environment by defining prerequisites, such as test data. If we need these prerequisites for only one particular test case, we define them at the beginning of the case. Otherwise, we set them globally for all tests in the test suite by using the beforeEach hook, which gets executed before each test case, or the beforeAll hook, which gets executed before all tests run.

If, for example, we had some reason to use the same global dataset for each test case and knew that our test steps would modify the dataset, we would need to re-create the dataset before each test. The beforeEach hook would be the perfect place to do this. On the other hand, if the test cases merely consume the data, we’d need to define the datasets only once and so would use the beforeAll hook.

Let’s define two test cases and create the input values for each. Our input parameters will be specific to each test case, so we’ll declare them inside the test cases instead of using a beforeEach or beforeAll hook. Update the _sum.test.ts_ file with the code from [Listing 8-4](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-4).

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {
    test("two plus two is four", () => {
        let first = 2;
        let second = 2;
        let expectation = 4;
    });

    test("minus eight plus four is minus four", () => {
        let first = -8;
        let second = 4;
        let expectation = -4;
    });
});
```

Listing 8-4: The test suite containing the arrange steps

The describe function creates our test suite, which comprises two calls to the test function, each of which is a test case. For both, the first argument is the description we see on the test runner’s report.

Each of our tests evaluates the result of the sum function. The first checks the addition feature, verifying that 2 plus 2 returns 4. The second test confirms that the function correctly returns negative values as well. It adds 4 to −8 and expects a result of −4.

You might want to check the return type of the sum function, too. Usually, we would have done so, but because we’re using TypeScript, there is no need for this additional test case. Instead, we can define the return type in the function signature, and TSC will verify it for us.

#### Act

As soon as the test runner executes a case, the test steps _act_ on our behalf by invoking the code under test with the data for the particular test case. Each test case should test exactly one feature or variant of the system. This step is the line of code that invokes the function to execute. [Listing 8-5](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-5) adds it to the test cases in _sum.test.ts_.

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
        let first = 2;
        let second = 2;
        let expectation = 4;
        let result = sum(first, second);
    });

    test("minus eight plus four is minus four", () => {
        let first = -8;
        let second = 4;
        let expectation = -4;
        let result = sum(first, second);
    });

});
```

Listing 8-5: The test suite containing the act steps

Our new lines call the sum function and pass it the values we defined as parameters. We store the returned values in the result variable. In your editor, TSC should throw an error along the lines of Expected 0 arguments, but got 2. This is fine, as the sum function is just an empty placeholder and doesn’t yet expect any parameters.

#### Assert

The final step of our test case is the _assertion_ that the code fulfills the expectations we defined. We create this assertion with two parts: the Jest expect function, used in conjunction with a _matcher_ function from Jest’s _assert_ library that defines the condition for which we are testing. Depending on the unit test’s category, this condition could be a specific return value, a state change, or the invocation of another function. Common matchers check whether a value is a number, a string, and so on. We can also use them to assert that a function returns true or false.

Jest’s _assert_ library provides us with a built-in set of basic matchers, and we can add additional ones from the npm repository. One of the most common assert packages is _testing-library/dom_, used to query the DOM for a particular node and assert its characteristics. For example, we can check for a class name or attribute or work with native DOM events. Another common assert package, _testing-library/react_, adds utilities for React and gives us access to the render function and React hooks in our asserts.

Because each test case evaluates one condition in one unit of code, we limit each test to one assert. In this way, as soon as the test run succeeds or fails and the test reporter generates the report, we can easily spot which test assumption failed. [Listing 8-6](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-6) adds one assert per test case. Paste it into the _sum.test.ts_ file.

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
        let first = 2;
        let second = 2;
        let expectation = 4;
        let result = sum(first, second);
        expect(result).toBe(expectation);
    });

    test("minus eight plus four is minus four", () => {
        let first = -8;
        let second = 4;
        let expectation = -4;
        let result = sum(first, second);
        expect(result).toBe(expectation);
    });

});
```

Listing 8-6: The test suite containing the assert steps

These lines use the expect assert function with the toBe matcher to compare the expected result to be the same as our expectation. Our test cases are now complete. Each follows the _arrange, act, assert_ pattern and verifies one condition. [Appendix C](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/appendix-C.xhtml) lists additional matchers.

### Using TDD

Our test cases still haven’t executed, and if you run npm test, the test runner should fail immediately. TSC checks the code and throws an error for the missing parameter declarations on the sum function:

```
FAIL  __tests__/sum.test.ts
  • Test suite failed to run
--snip--
Test Suites: 2 failed, 2 total
Tests:       0 total
Snapshots:   0 total
```

It’s time to implement this sum function. Following the principles of TDD, we’ll incrementally add features to the code and run the test suites after each addition, continuing this process until all tests pass. First we’ll add those missing parameters. Replace the code in _sum.ts_ with the contents of [Listing 8-7](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-7).

```
const sum = (a: number, b: number) => {};

export {sum};
```

Listing 8-7: The sum function with added parameters

We’ve added the parameters and typed them as numbers. Now we rerun the test cases and, as expected, they fail. The console output tells us that the sum function doesn’t return the expected results. This shouldn’t surprise us, because our sum function doesn’t return anything at all:

```
FAIL  __tests__/sum.test.ts (5.151 s)
  the sum function
    × two plus two is four (6 ms)
    × minus eight plus four is minus four (1 ms)

  • the sum function › two plus two is four
    Expected: 4
    Received: undefined

  • the sum function › minus eight plus four is minus four
    Expected: -4
    Received: undefined

Test Suites: 1 failed, 1 total
Tests:       2 failed, 2 total
Snapshots:   0 total
Time:        5.328 s, estimated 11 s
```

The code in [Listing 8-8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-8) adds this functionality to the _sum.ts_ file. We type the function’s return type as a number and add the two parameters.

```
const sum = (a: number, b: number): number => a + b;

export {sum};
```

Listing 8-8: The complete sum function

If we rerun npm test, Jest should report that all test cases succeed:

```
PASS  __tests__/sum.test.ts (8.045 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four (2 ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        8.291 s
```

As you can see, everything worked.

#### Refactoring Code

Unit tests are particularly useful when we need to refactor our code. As an example, let’s rewrite the sum function so that, instead of two parameters, it accepts an array of numbers. The function should return the sum of all array items.

We begin by rewriting our existing test cases into a more compact form and then expanding the test suite to verify the new behavior. Replace the code in the _sum.test.file_ with [Listing 8-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-9).

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
        expect(sum([2, 2])).toBe(4);
    });

    test("minus eight plus four is minus four", () => {
        expect(sum([-8, 4])).toBe(-4);
    });

    test("two plus two plus minus four is zero", () => {
        expect(sum([2, 2, -4])).toBe(0);
    });

});
```

Listing 8-9: The test suite for the refactored sum function

Notice that we rewrote the test cases in a more compact form. Explicitly splitting the arrange, act, and assert statements across multiple lines may be easier to read, but for simple test cases, such as those in [Listing 8-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-9), we often write them in one line. We’ve changed their functionality to accommodate the new requirements. Instead of accepting two values, our sum function receives an array with numbers. Again, the TSC instantly notifies us of the mismatching parameters between the sum function in the test suite and the actual implementation.

Once we’ve written our tests, we can rewrite our code. [Listing 8-10](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-10) shows the code for the _helpers/sum.ts_ file. Here the sum function now accepts an array of numbers as a parameter and returns a number.

```
const sum = (data: number[]): number => {
    return data[0] + data[1];
};

export {sum};
```

Listing 8-10: The rewritten sum function in the helpers/sum.ts file

We changed the parameter to an array of numbers. This fixes the TypeScript error caused by the test suite in [Listing 8-9](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-9). But because we’re following TDD and making only one functional change at a time, we keep the function’s original behavior of adding two values. As expected, one of the test cases fails when we run the automated tests with npm test:

```
FAIL  __tests__/sum.test.ts (7.804 s)
  the sum function
    ✓ two plus two is four (7 ms)
    ✓ minus eight plus four is minus four (1 ms)
    ✕ two plus two plus minus four is zero (9 ms)

  • the sum function › two plus two plus minus four is zero
    Expected: 0
    Received: 4

Test Suites: 1 failed, 1 total
Tests:       1 failed, 2 passed, 3 total
Snapshots:   0 total
Time:        8.057 s, estimated 9 s
```

The third test case, which tests the new requirement, is the one that failed. Not only did we expect this result, but we also wanted the test to fail; this way, we know that the tests themselves are working. If they succeeded before we implemented the corresponding functionality, the test cases would be faulty.

With the failing test as the baseline, it is now time to refactor the code to accommodate the new requirement. Paste the code in [Listing 8-11](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-11) into the _sum.ts_ file. Here we refactor the sum function to return the sum of all array values.

```
const sum = (data: number[]): number => {
    return data.reduce((a, b) => a + b);
};

export {sum};
```

Listing 8-11: The corrected sum function with array.reduce

Although we could loop through the array with a for loop, we use modern JavaScript’s array.reduce function. This native array function runs a callback function on each array element. The callback receives the return value of the previous iteration and the current array item as parameters: exactly what we need to calculate the sum.

Run all the test cases in our test suite to verify that they are working as expected:

```
PASS  __tests__/sum.test.ts (7.422 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four
    ✓ two plus two plus minus four is zero

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        7.613 s
```

The test runner should show that the code passed every test.

#### Evaluating Test Coverage

To measure exactly which lines of code our test suites cover, Jest generates a test-coverage report. The higher the percentage of code our tests assess, the more thorough they are, and the more confident we can be about the application’s quality and maintainability. As a general rule of thumb, you should aim for code coverage of 90 percent or above, with a high coverage for the most critical part of your code. Of course, the test cases should add value by testing the code’s functions; adding tests just to increase the test coverage is not the goal we are aiming for. But as soon as you’ve tested your code base thoroughly, you can refactor existing features and implement new ones without worrying about introducing regression bugs. A high code coverage verifies that changes have no hidden side effects.

Modify the npm test script in the _package.json_ file by adding the --coverage flag to it, as shown in [Listing 8-12](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-12).

```
  "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "next lint",
      "test": "jest --coverage"
  },
```

Listing 8-12: Enabling Jest’s test-coverage feature in the package.json file

If we rerun the test suite, Jest should show what percentage of the code our unit tests cover. It generates a code-coverage report and stores it in the _coverage_ folder. Compare your output with the following:

```
PASS  __tests__/sum.test.ts (7.324 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four
    ✓ two plus two plus minus four is zero (1 ms)
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |     100 |      100 |     100 |     100 |
  sum.ts  |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        7.687 s, estimated 8 s
```

The report shows the coverage broken down by statements, branches, functions, and lines. We see that our simple sum function has a code coverage of 100 percent across all categories. Hence, we know that we’ve left no code untested and can trust that the test cases reflect the function’s quality.

### Replacing Dependencies with Fakes, Stubs, and Mocks

We mentioned that our tests should run in isolation, without depending on external code. You might have wondered how to handle imported modules; after all, as soon as you import code, you add a dependency to the unit under evaluation. Those third-party modules might not work as expected, and we don’t want our code to depend on the assumption that they all operate correctly. Consequently, you should provide a set of test cases for each imported module to verify its functionality. They, too, are units to test.

Separately, instead of importing modules into our other code units, we need to replace them with _test doubles_ that return a defined set of static data tailored to the test. Test doubles replace an object or a function, effectively removing a dependency. Because they return a defined dataset, their response is known and predictable. You can compare them to stunt doubles in movies.

Besides replacing an object or function, test doubles have a second important purpose: they record their calls and let us spy on them. We can thus use them to test whether the test double has been called at all, how often, and which arguments it received. There are three main types of test doubles: fakes, stubs, and mocks. However, you’ll sometimes hear the term _mock_ to refer to all three.

#### Creating a Module with Dependencies

To practice using test doubles in our sum function, we’ll create a new function that calculates a specified number of values in the Fibonacci sequence. The _Fibonacci sequence_ is a pattern in which each subsequent number is the sum of the previous two, a simple use case for a sum module.

All developers must figure out how fine-grained their test cases need to be. The Fibonacci sequence is a good example, because trying to test every possible number submitted to the function would be useless, as the sequence has no end. Instead, we want to verify that the function properly handles edge cases and that its underlying functionality works. For instance, we’ll check how it handles an input with a length of 0; in that case, the function should return an empty string. Then we’ll test how it calculates a Fibonacci sequence of any length longer than 3. Create the _fibonacci.test.ts_ test suite inside the ___tests___ folder and then add the code from [Listing 8-13](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-13) to it.

```
import {fibonacci} from "../helpers/fibonacci";

describe("the fibonacci sequence", () => {

    test("with a length of 0 is ", () => {
        expect(fibonacci(0)).toBe(" ");
    });

    test("with a length of 5 is '0, 1, 1, 2, 3' ", () => {
        expect(fibonacci(5)).toBe("0, 1, 1, 2, 3");
    });

});
```

Listing 8-13: The test suite for the fibonacci function

We define two test cases: one that checks for a length input of 0 and another that calculates a Fibonacci sequence of five numbers. Both tests follow the _arrange, act, assert_ pattern in the compact variant we used earlier.

After we’ve created the test cases, we can move on to writing the Fibonacci function code. Create the _fibonacci.ts_ file in the _helpers_ folder, next to the _sum.ts_ file, and add the code from [Listing 8-14](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-14) to it.

```
import {sum} from "./sum";

const fibonacci = (length: number): string => {
    const sequence: number[] = [];
    for (let i = 0; i < length; i++) {
        if (i < 2) {
            sequence.push(sum([0, i]));
        } else {
            sequence.push(sum([sequence[i - 1], sequence[i - 2]]));
        }
    }
    return sequence.join(", ");
};

export {fibonacci};
```

Listing 8-14: The fibonacci function

We import the sum function from the module we created earlier in this chapter. It is now a dependency that we’ll need to replace with a test double later. Then we implement the fibonacci function, which accepts the length of the sequence to calculate and returns a string. We store the current sequence in an array so that we have a simple way to access the two previous values needed to calculate the next one. Notice that the first number in the sequence is always 0 and the second is always 1. Finally, we return a string with the requested number of values. If you save this code and rerun the test suites, both _sum.test.js_ and _fibonacci.test.ts_ should pass successfully.

#### Creating a Doubles Folder

Because we import the sum function in the Fibonacci module, our code has an external dependency. This is problematic for testing purposes: if the sum function breaks, the test for the Fibonacci sequence will fail as well, even if the logic of the Fibonacci implementation is correct.

To decouple the test from the dependency, we’ll replace the sum function in the _fibonacci.ts_ file with a test double. Jest can replace any module that has an identically named file saved in a ___mocks___ subdirectory adjacent to the test file during the test run. Create such a folder in the _helpers_ folder next to the test file and place a _sum.ts_ file inside it. Leave the file empty for now.

To enable the test double, we call the jest.mock function, passing it the path to the original module saved in the test file. In [Listing 8-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-15), we add this call to _fibonacci.test.ts_.

```
import {fibonacci} from "../helpers/fibonacci";

jest.mock("../helpers/sum");

describe("the fibonacci sequence", () => {
    test("with a length of 0 is ", () => {
        expect(fibonacci(0)).toBe(" ");
    });
    test("with a length of 5 is '0, 1, 1, 2, 3' ", () => {
        expect(fibonacci(5)).toBe("0, 1, 1, 2, 3");
    });

});
```

Listing 8-15: The test suite for the fibonacci function with the test double

This new line replaces the sum module with the test double. Now let’s create all three basic types of test doubles, adding their code to the file in the ___mocks___ folder.

#### Using a Stub

Stubs are merely objects that return some predefined data. This makes them very simple to implement but limited in use; often, returning the same data isn’t enough to mimic a dependency’s original actions. [Listing 8-16](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-16) shows a stub implementation for the sum function’s test double. Paste the code into the _sum.ts_ file inside the ___mocks___ folder.

```
const sum = (data: number[]): number => 999;

export {sum};
```

Listing 8-16: A stub for the sum function

The stubbed function has the same signature as the original function. It accepts the same arguments, an array of numbers, and returns a string. Unlike the original, however, this test double always returns the same number, 999, regardless of the data it received.

To successfully run the test suites with this stub function, we’d need to adjust our expectations about what our code will do. Instead of returning five numbers in the Fibonacci sequence, it would produce the string 999, 999, 999, 999, 999. If we see such a string, we know that the sum function was called five times. Experiment with the stub, modifying the test suite’s expectations to match it. Then restore the matchers to those shown in [Listing 8-15](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-15) so that you can use them for the upcoming tests.

#### Using a Fake

Fakes are the most complex kind of test double. They are working implementations of the original functionality, but unlike the real implementation, they provide only the functionality necessary for the unit test. Their implementation is simplified and often doesn’t cater to edge cases.

The fake for the sum adds the first and second items of the array manually, instead of using array.reduce. This simplified implementation strips the sum function of the ability to sum more than two data points, but for the Fibonacci sequence, it is sufficient. The reduced complexity makes it easy to understand and less prone to error. Replace the content of the _sum.ts_ file inside the ___mocks___ folder with the code in [Listing 8-17](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-17).

```
const sum = (data: number[]): number => {
    return data[0] + data[1];
}
export {sum};
```

Listing 8-17: A fake for the sum function

Our fake uses a simple mathematical plus operator (+) to add the first and second items of the data parameter. Its main benefit is that it returns a result similar to that of the actual implementation. We can now run the test suites, and they should pass successfully without our having to adjust our expectations, returning the Fibonacci sequence.

#### Using a Mock

Mocks lie somewhere between stubs and fakes. Although less sophisticated than fakes, they return more realistic data than stubs. While they don’t simulate a dependency’s true behavior, they can react to the data they receive.

For example, our naive mock implementation of the sum function will return a result from a hardcoded hash map. Replace the code in the ___mocks__/sum.ts_ file with the code from [Listing 8-18](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-18), which inspects the request and enables the Fibonacci calculator to use the original test suites.

```
type resultMap = {
    [key: string]: number;
}

const results : resultMap= {
    "0 + 0": 0,
    "0 + 1": 1,
    "1 + 0": 1,
    "1 + 1": 2,
    "2 + 1": 3
};

const sum = (data: number[]): number => {
    return results[data.join("+")];
}

export {sum};
```

Listing 8-18: A mock for the sum function

We create a type, called resultMap, that uses a string as a key and a number as a value. Then we use the newly created type for a hash map that stores our desired responses. Next, we define the mock function with the same interface as the original implementation. Inside the mock, we calculate the key to use in the hash map based on the parameters we receive. This lets us return the correct dataset and produce an actual Fibonacci sequence. The main benefit of using the mock over sum is that we can control its outcome, as it returns values from a known dataset.

Conveniently, Jest provides us with helpers to work with test doubles. The jest.mock function replaces imported modules with mocks. The jest.fn API creates a basic mock that can return any kind of predefined data, and jest.spyOn lets us record calls to any function without modifying it. We will use all of those in [Exercise 8](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Exe8) on page 146.

In typical developer contexts, you won’t bother with the subtle differences between stubs, fakes, and mocks and will use the term _mock_ as a generic term for test doubles. Don’t spend too much time overengineering your mocks; they’re just tools to help you test your code.

### Additional Types of Tests

The tests covered in this chapter so far are the most common ones you’ll encounter as a full-stack developer. This section briefly explains additional types of tests and when to use them. These aren’t meant to replace unit tests; rather, they supplement unit tests by covering specific aspects of your implementation that aren’t otherwise testable. For example, because unit tests run in isolation, they can’t evaluate the interaction between modules. Theoretically, if every function and module passes its test, the whole program should work as expected. Practically, you’ll often face issues caused by faulty module documentation. Commonly, the documentation will claim that an API returns a specific type, but the actual implementation will return a different one.

#### Functional Tests

While unit tests examine the implementation of a feature from a developer’s perspective, _functional tests_ cover the user’s perspective by verifying that code works as the user expects it to work. Put otherwise, these tests check that a given input results in an expected output. Most functional tests are a type of _black-box_ test, which ignores the module’s internal code, side effects, and intermediate results and tests only the interfaces. Functional tests do not generate a code-coverage report. Generally, a quality assurance manager will write and use functional tests during a system testing stage. By contrast, developers write and use unit tests during development.

#### Integration Tests

You learned that the goal of a unit test is to check the smallest possible section of code in isolation. An _integration test_ is the complete opposite. It verifies the behavior of entire subsystems, whether they be layers of code, such as an app’s data storage mechanism, or specific functions consisting of multiple modules. Integration tests check the integration of the subsystem in the context of the current environment. Hence, they never run in isolation and typically don’t use test doubles.

Integration tests are helpful for finding three types of problems. The first type is problems related to _inter-module communication_, which is the communication between modules. Common problems are faulty internal API integrations and undetected side effects, such as a function that doesn’t delete old files before writing new data to the filesystem. The second type is problems related to the _environment_, which describes the hardware and software setup the code runs on. Different software versions or hardware configurations can introduce significant issues for your code. The most common problem for full-stack developers involves differences in Node.js versions and outdated dependencies in the modules.

The third type is problems related to _gateway communications_, which relates to testing any API communication with a third-party API gateway. Any communication with external APIs should be tested with integration tests. This is the only instance in which integration tests might use test doubles, such as stubbed versions of the external API, in order to simulate a specific API behavior, like a timeout or successful request. As with functional tests, a quality assurance manager generally writes and uses integration tests. Developers do so less often.

#### End-to-End Tests

You can think of the _end-to-end test_ as a combination of functional tests and integration tests. As another kind of black-box test, they examine the application’s functionality across the full stack, from the frontend to the backend, in a specific environment. These business-facing tests should provide confidence that the overall application is still working as expected.

End-to-end tests run the application in a specific environment. Often, the complexity of the many dependencies increases the risk of flaky tests in which the application works correctly but the environment causes the tests to fail. End-to-end tests are thus the most time-consuming to create and maintain. Due to their complexity, we must craft them carefully. During execution, they are known to be slow, prone to encountering timeouts, and, like nearly all black-box tests, unable to provide detailed error reports. Therefore, they test only the most critical business-facing scenarios. Generally, a quality assurance manager writes them.

#### Snapshot Tests

The tests described earlier in this chapter check the code against some assertion. By contrast, a _snapshot test_ compares the application’s current visual (or user interface) state to a previous version of it. Hence, these tests are also called visual regression tests. In each test, we create new snapshots and then compare them with ones stored earlier, providing a cheap way to test user interface components and complete pages. Instead of manually creating and maintaining tests that describe every property of an interface, such as a component’s height, width, position, and colors, we can establish a snapshot containing all of these properties.

One way to perform this type of test is to create and compare screenshots. Generally, a headless browser renders the component; the test runner waits for the page to render and then captures an image of it. Unfortunately, this process is relatively slow, and headless browsers are flaky. Jest takes a different approach to snapshot testing. Instead of working with headless browsers and image files, it renders the React user interface components to the virtual DOM, serializes them, and saves them as plaintext in _snap_ files stored in the ___snapshots___ directory. Hence, Jest snapshot tests are much more performant and less flawed. The Food Finder application you’ll build in [Part II](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/part2.xhtml) will use snapshot tests to verify the integrity of the build and test your React components.

Exercise 8: Add Test Cases to the Weather App

As long as you follow the basic principles we’ve discussed, there is no right or wrong way to test your code. Unit, snapshot, and end-to-end tests are all different tools in your tool belt, and you must strike a balance between the time you spend writing the tests and the usefulness of each. There is also no consensus on what to test. While you should strive for 90 percent code coverage or higher, the general rule of thumb is to cover at least the most critical parts of your application with unit tests and then write some integration tests to verify that your application works on each deployment.

When it comes to our weather application, we’ll want our test cases to cover four core aspects. First we’ll add unit tests to evaluate the middleware and services. Even though the REST API endpoints and React user interface component are easy to test directly in the browser, we’ll add test cases for both of them: a basic snapshot test for the user interface component and an end-to-end test for the REST API endpoint _/v1/weather/[zipcode].ts_.

We’ve opted to test the REST endpoint rather than the GraphQL API for simplicity’s sake, as each REST endpoint has its own file, while all GraphQL APIs share an entry point, making their testing more complex. However, testing this GraphQL API would make an excellent exercise for exploring end-to-end-tests after you’ve finished the chapter.

#### Testing the Middleware with Spies

The middleware to connect to the database is a core part of the application, but we can’t access it directly, as it doesn’t expose any API. We can only implicitly test it by examining the database or by running a query through Mongoose, some service, or an API endpoint. Each of these methods would work, but if we want to test the connection to the database as a unit test, we need to do so in a way that isolates that component as much as possible.

To do so, we’ll use Jest’s built-in spies to verify that our middleware successfully calls all the functions necessary for establishing the connection to the MongoDB memory server. Navigate to your ___tests___ folder and create a new folder, _middleware_, and a file, _db-connect.test.ts_, inside it. Then copy the code from [Listing 8-19](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-19) into the file.

```
/**
 * @jest-environment node
 */

import dbConnect from "../../middleware/db-connect";
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";

describe("dbConnect ", () => {
    
    let connection: any;

    afterEach(async () => {
        jest.clearAllMocks();
        await connection.stop();
        await mongoose.disconnect();
    });

    afterAll(async () => {
        jest.restoreAllMocks();
    });

    test("calls MongoMemoryServer.create()", async () => {
        const spy = jest.spyOn(MongoMemoryServer, "create");
        connection = await dbConnect();
        expect(spy).toHaveBeenCalled();
    });

    test("calls mongoose.disconnect()", async () => {
        const spy = jest.spyOn(mongoose, "disconnect");
        connection = await dbConnect();
        expect(spy).toHaveBeenCalled();
    });

    test("calls mongoose.connect()", async () => {
        const spy = jest.spyOn(mongoose, "connect");
        connection = await dbConnect();
        const MONGO_URI = connection.getUri();
        expect(spy).toHaveBeenCalledWith(MONGO_URI, {dbName: "Weather"});
    });

});
```

Listing 8-19: The __tests__/middleware/db-connect.test.ts suite for the database connection

Most of this code resembles the test suites you wrote earlier in this chapter. But instead of testing simplified example code, we’re now testing real code, which requires us to make some adjustments.

First we set the testing environment for Jest to node, which simulates a Node.js runtime. Later, when writing snapshot tests, we’ll use Jest’s default environment, called jsdom, which simulates a browser by providing a window object, as well as all the usual DOM properties and functions. By always setting these environments in the file, we avoid issues caused by using the wrong environment. Then, as usual, we import the packages we need.

Now we can start writing the test suite for the dbConnect function. We define a connection variable in the test suite’s scope to store the database connection, and then we can access the MongoDB’s server instance, including its methods and properties. For example, we’ll use these to stop the connection and disconnect from the server after each test to guarantee that each test case is independent.

To be able to store the connection, we first need to return the mongoServer constant from the dbConnect function in the file _db-connect.ts_. Open the file and add the line return mongoServer to the dbConnect function right before the function’s closing curly bracket (}). From time to time, you’ll need to modify the code you wrote earlier to accommodate the requirements of your tests. In other words, you need to adapt the code to make it testable.

Now we use the connection we just exposed and set up the afterEach hook, which runs after each test case, to reset the mocked functions to their initial mocked state, thus clearing all previously gathered data. This is necessary, because otherwise, the spies would report information gained during the previous calls, as they retain their state across all test suites. Also, we re-create the database connection for each test case. Therefore, we need to stop the current connection and explicitly disconnect from the database after each test. Then we set up the afterAll hook to remove all mocks and restore the original functions through the restoreAllMocks function.

Our test cases should all follow the _arrange, act, assert_ pattern. As we review them, you might find it useful to open the _db-connect.ts_ file in the _middleware_ folder and follow along. The initial test case verifies the call to the create function on the MongoMemoryServer, as this is the first function that we call in the _db-connect.ts_ file. To do so, we create a spy with the jest .spyOn method. As arguments, this method takes the name of an object and the object’s method on which to spy. Then we act on the code under test and call the dbConnect function. Finally, we assert that the spy has been called.

The second test case works similarly except that it spies on a different method. We use it to check that mongoose.disconnect was called successfully during the execution of dbConnect. The third test case introduces a new matcher. Instead of verifying only the call itself with toHaveBeenCalled, we now also verify the call’s arguments using toHaveBeenCalledWith. Here we grab the connection string directly from the connection and store it in the variable MONGO_URI. We also hardcode the database we want to connect to. Then we call the matcher, passing it the expected arguments and verifying that they meet our expectations.

Now run the test suites with npm test. All tests should pass with 100 percent test coverage.

#### Creating Mocks to Test the Services

While the tests we wrote for the middleware were quite simple, the service tests are a bit more complicated. If you open the _mongoose/weather/services.ts_ file, you’ll see that the services depend on WeatherModel, which is Mongoose’s gateway to the MongoDB collection. Each service calls a method on the model that, in turn, requires a database connection. We won’t reevaluate those database connections here; instead, the goal of this test suite will be to verify that the service functions call the correct WeatherModel functions. To do so, we’ll create a mock WeatherModel that exposes the same set of APIs as mocked functions.

We first write the mocked model. Following convention, we create the file _mongoose/weather/__mocks__/model.ts_ and add the code in [Listing 8-20](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-20).

```
import {WeatherInterface} from "../interface";

type param = {
    [key: string]: string;
};

const WeatherModel = {
    create: jest.fn((newData: WeatherInterface) => Promise.resolve(true)),
    findOne: jest.fn(({zip: paramZip}: param) => Promise.resolve(true)),
    updateOne: jest.fn(({zip: paramZip}: param, newData: WeatherInterface) =>
        Promise.resolve(true)
    ),
    deleteOne: jest.fn(({zip: paramZip}: param) => Promise.resolve(true))
};
export default WeatherModel;
```

Listing 8-20: The mock for the WeatherModel

We implement WeatherInterface and define the new param type, which is an object with key-value pairs that we use to type the first parameter. We make the mocked WeatherModel the default export and use an object that implements the four methods of the actual WeatherModel, each of which takes the same parameters as the original. They also take the original Mongoose model’s method. Because they are asynchronous functions, we return a promise resolved to true.

Now we can write the test suite for the services. These check that each service returns true upon success and calls the correct method of the mocked WeatherModel. Create the file _/__tests__/mongoose/weather/services.test.ts_ and add the code from [Listing 8-21](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-21) to it.

```
/**
 * @jest-environment node
 */
import {WeatherInterface} from "../../../mongoose/weather/interface";
import {
    findByZip,
    storeDocument,
    updateByZip,
    deleteByZip,
} from "../../../mongoose/weather/services";

import WeatherModel from "../../../mongoose/weather/model";
jest.mock("../../../mongoose/weather/model");

describe("the weather services", () => {

    let doc: WeatherInterface = {
        zip: "test",
        weather: "weather",
        tempC: "00",
        tempF: "01",
        friends: []
    };

    afterEach(async () => {
        jest.clearAllMocks();
    });

    afterAll(async () => {
        jest.restoreAllMocks();
    });

    describe("API storeDocument", () => {
        test("returns true", async () => {
            const result = await storeDocument(doc);
            expect(result).toBeTruthy();
        });
        test("passes the document to Model.create()", async () => {
            const spy = jest.spyOn(WeatherModel, "create");
            await storeDocument(doc);
            expect(spy).toHaveBeenCalledWith(doc);
        });
    });

    describe("API findByZip", () => {
        test("returns true", async () => {
            const result = await findByZip(doc.zip);
            expect(result).toBeTruthy();
        });
        test("passes the zip code to Model.findOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "findOne");
            await findByZip(doc.zip);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip});
        });
    });

    describe("API updateByZip", () => {
        test("returns true", async () => {
            const result = await updateByZip(doc.zip, doc);
            expect(result).toBeTruthy();
        });
        test("passes the zip code and the new data to Model.updateOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "updateOne");
            const result = await updateByZip(doc.zip, doc);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip}, doc);
        });
    });

    describe("API deleteByZip", () => {
        test("returns true", async () => {
            const result = await deleteByZip(doc.zip);
            expect(result).toBeTruthy();
        });
        test("passes the zip code Model.deleteOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "deleteOne");
            const result = await deleteByZip(doc.zip);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip});
        });
    });
  
});
```

Listing 8-21: The updated test suite in __tests__/mongoose/weather/services.test.ts

As in the previous test suite, we begin by setting up the environment and importing modules. We also import WeatherModel and call jest.mock with the path to the mocked model we created, effectively replacing the original model in the code under test. Then we create a document containing some test data. We store it in the constant doc and will pass it to the mocked model’s methods. As done previously, we use the afterEach hook to reset all mocks after each test and the afterAll hook to remove the mocks and restore the original functions after all test cases have been finished.

We create a nested test suite for each of the four services. Each has the same two unit tests: one to verify the return value upon success with the toBeTruthy matcher and one to spy on a particular WeatherModel mock function. The code follows the same pattern as the previous test suite and uses the same matchers as well.

The code-coverage report we receive after running npm test shows that we tested around 70 percent of the service code. If you take a look at the uncovered lines listed in the last column, you’ll see that they contain the console.log(err); output. This output is used whenever an asynchronous call to the model’s methods fails:

```
PASS  __tests__/mongoose/weather/services.test.ts
PASS  __tests__/middleware/dbconnect.test.ts (7.193 s)

--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
--------------------|---------|----------|---------|---------|-------------------
All files           |   83.63 |      100 |   88.23 |   82.35 |
 middleware         |     100 |      100 |     100 |     100 |
  db-connect.test.ts|     100 |      100 |     100 |     100 |
 mongoose/weather.  |   77.41 |      100 |     100 |   75.86 |
  services.test.ts  |   70.83 |      100 |     100 |   70.83 |8,20-22,33-35,43-45
--------------------|---------|----------|---------|---------|-------------------
```

For the purposes of this chapter, we’ll leave these lines uncovered. Otherwise, we could modify the mocked model to throw an error—for example, by supplying an invalid document—and then add a third test case per service verifying the error.

#### Performing an End-to-End Test of the REST API

Sophisticated API tests might use a dedicated API testing library such as _SuperTest_, which provides matchers for HTTP status codes and simplifies the handling of requests and responses. Alternatively, they might use a GUI tool like Postman. In this example, we’ll merely test that the returned data matches our expectations by using the native fetch method.

Unlike the previous tests, this one doesn’t isolate any single component, as our goal is to verify that all components of the system work together as expected. To check whether the API returns a proper response from the database when supplied some input, our end-to-end test will make certain assumptions: that all layers have already been tested independently, that the database contains its initial seed data, and that our application runs at _http://localhost:3000/_.

To verify our first assumption, open the API endpoint file _pages/api/v1/weather/[zipcode].ts_. You’ll notice that the API code imports two functions, findByZip from the service module and the middleware’s dbConnect, both of which we’ve already tested. The second assumption is also satisfied; the database loads the initial seed on each startup. Create the file _zipcode.e2e .test.ts_ in ___tests__/pages/api/v1/weather/_ and add the code from [Listing 8-22](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-22).

```
/**
 * @jest-environment node
 */

describe("The API /v1/weather/[zipcode]", () => {
    test("returns the correct data for the zipcode 96815", async () => {
        const zip = "96815";
        let response = await fetch(`http://localhost:3000/api/v1/weather/${zip}`);
        let body = await response.json();
        expect(body.zip).toEqual(zip);
    });
});

export {};
```

Listing 8-22: The test suite for the REST API

We set the environment to node and then define the test suite with one test case. In it, we supply a ZIP code that matches one of the initial seed datasets. Then we use the native fetch method, which has been available since Node.js version 17.5, to call our weather API on our localhost and check whether the returned ZIP code is the same as the one passed as a parameter. We add an empty export statement to define this file as an ES6 module.

The test should pass and have 100 percent code coverage. Now that we’re confident that the core of our application is working as expected, we can test the user interface components.

When using fetch, there are two common error messages you might encounter. The first, ECONNREFUSED, tells you that fetch could not connect to your application because it is not running. Use npm run dev to start the application or adjust the port in the fetch call if you’re not using port 3000. The second error mentions that the test exceeded the timeout of 5,000 ms for a test. If you started your application for the purposes of testing and did not use a previously running application, Next.js compiles the API route as soon as the test consumes it. Depending on your environment, this might take longer than the default timeout. Add the line jest.setTimeout(20000); before the describe method at the top of your file to increase the timeout and make the test wait 20,000 ms instead of 5,000 ms.

#### Evaluating the User Interface with a Snapshot Test

Snapshot tests verify that a page’s rendered HTML didn’t change between two test runs. To achieve this with Jest, we must first prepare our environment. Add the jsdom environment, react-testing-library, and the react-test-renderer to the project:

```
$ npm install --save-dev jest-environment-jsdom
$ npm install --save-dev @testing-library/react @testing-library/jest-dom
$ npm install --save-dev @types/react-test-renderer react-test-renderer
```

We’ll need these to simulate a browser environment and render React components during our test cases. Now we’ll modify the _jest.config.js_ file in our root directory accordingly. Replace its content with the code in [Listing 8-23](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-23).

```
const nextJest = require("next/jest");
const createJestConfig = nextJest({});

module.exports = createJestConfig(nextJest({}));
```

Listing 8-23: The updated jest.config.js file

This code imports the _next/jest_ package and exports a Jest configuration with the default properties of a Next.js project. It is the simplest form of Next.js-compatible Jest configuration. If you take a look at the official Next.js setup guide at [_https://nextjs.org/docs/testing_](https://nextjs.org/docs/testing), you’ll see that it outlines some basic configuration options, none of which we need.

##### The First Version

A snapshot test renders a component or a page, takes a snapshot of it as serialized JSON, and stores it in a ___snapshots___ folder next to the test suite. On each consecutive run, Jest compares the current snapshot with the stored reference. As long as they are the same, the snapshot test passes. To generate the initial snapshot, create a new folder, ___tests__/pages/components_, and the file _weather.snapshot.test.tsx_, and then add the code in [Listing 8-24](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-24) to it.

```
/**
 * @jest-environment node
 */

import {act, create} from "react-test-renderer";
import PageComponentWeather from "../../../pages/components/weather";

describe("PageComponentWeather", () => {
    test("renders correctly", async () => {
        let component: any;
        await act(async () => {
            component =
                await create(<PageComponentWeather></PageComponentWeather>);
        });
        expect(component.toJSON()).toMatchSnapshot();
    });
});
```

Listing 8-24: The snapshot test for PageComponentWeather

The first lines of our snapshot test set the environment to jsdom and import the test renderer’s act and create methods to test the React component, which we import in the next line.

Next, we write the simulated user behavior and wrap the component’s creation in the asynchronous act function. As you might have guessed, this function draws its name from the _arrange, act, assert_ pattern and ensures that all relevant updates to the DOM have been applied before proceeding with the test case. It is required for all statements that cause updates to the React state, and here, it delays the test execution until after the useEffect hook runs.

Then we write a test case that awaits the create function, which renders the JSX component. This lets us generate HTML in a simulated browser environment and store the result in a variable. We await the component’s rendering so that the HTML is available for our follow-up interactions before we continue with the test case. Then we serialize the rendered component to a JSON string and use a new matcher, toMatchSnapshot, which compares the current JSON string with the stored reference.

A trial run shows that all tests succeed. We see two interesting things— that the test created one snapshot and that we achieved a test coverage of 81 percent:

```
PASS  __tests__/mongoose/weather/services.test.ts
PASS  __tests__/pages/api/v1/weather/zipcode.e2e.test.ts
PASS  __tests__/middleware/dbconnect.test.ts (7.193 s)
PASS  __tests__/pages/components/weather.snapshot.test.tsx

---------------------|---------|----------|---------|---------|-------------------
File                 | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
---------------------|---------|----------|---------|---------|-------------------
All files            |   83.63 |      100 |   88.23 |   82.35 |
 middleware          |     100 |      100 |     100 |     100 |
  db-connect.test.ts |     100 |      100 |     100 |     100 |
 mongoose/weather    |   77.41 |      100 |     100 |   75.86 |
  services.test.ts   |   70.83 |      100 |     100 |   70.83 |8,20-22,33-35,43-45
 pages/api/v1/       |         |          |         |         |
  weather            |         |          |         |         |
    [zipcode].ts     |     100 |      100 |     100 |     100 |
 pages/components    |   81.81 |      100 |      60 |      80 |
  weather.tsx        |   81.81 |      100 |      60 |      80 |8,12
---------------------|---------|----------|---------|---------|-------------------
Snapshot Summary
 › 1 snapshot written from 1 test suite.
```

You can look at the created snapshot by opening the _weather.snapshot.test.tsx.snap_ file in the ___snapshots___ folder. It should look fairly similar to the code in [Listing 8-25](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-25), and you’ll see that it is nothing more than the rendered HTML saved as a multiline template literal. Your HTML might not be identical to that shown here; the important aspect is that it looks the same after each test run when react-test-renderer rendered the component.

```
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`PageComponentWeather renders correctly 1`] = `
<h1
    data-testid="h1"
    onClick={[Function]}
>
    The weather is
    sunny
    , and the counter shows
    0
</h1>
`;
```

Listing 8-25: The weather.snapshot.test.tsx.snap file with the serialized HTML

We also see that the counter is set to 0, which indicates that the useEffect did not run before we created the snapshot. If you open the component’s file and check the uncovered lines, you’ll learn that they relate to the click handler that increases the state variable and, as suspected, the useEffect hook. We want to test these core functionalities as well.

##### The Second Version

We’ll modify the test code to cover the previously untested functionalities. Paste the code from [Listing 8-26](https://learning.oreilly.com/library/view/the-complete-developer/9781098168810/xhtml/chapter8.xhtml#Lis8-26) into the snapshot test file.

```
/**
 * @jest-environment node
 */

import {act, create} from "react-test-renderer";
import PageComponentWeather from "../../../pages/components/weather";

describe("PageComponentWeather", () => {
    test("renders correctly", async () => {
        let component: any;
        await act(async () => {
            component = await create(<PageComponentWeather></PageComponentWeather>);
        });
        expect(component.toJSON()).toMatchSnapshot();
    });
 
    test("clicks the h1 element and updates the state", async () => {
        let component: any;
        await act(async () => {
            component = await create(<PageComponentWeather></PageComponentWeather>);
            component.root.findByType("h1").props.onClick();
        });
        expect(component.toJSON()).toMatchSnapshot();
    });
 
});
```

Listing 8-26: The updated snapshot test

In the updated code, we’ve added another test case that finds the headline on the page and simulates a user clicking it. Remember from previous chapters that this increases the state variable counter. Again, we await the creation of the component and use the act function.

If you rerun the tests, you should see a failure. The test runner tells us that the snapshots do not match:

```
FAIL  __tests__/pages/components/weather.snapshot.test.tsx
  • PageComponentWeather › renders correctly
--snip--
 › 1 snapshot failed.
--snip--
Snapshot Summary
 › 1 snapshot failed from 1 test suite.
› Inspect your code changes or run `npm test -- -u` to update them.
```

Because we modified the test case to wait for the useEffect hook and set the state variable counter to 1 instead of 0, the DOM changed as well. Follow the test runner’s advice and rerun the tests with npm test -- -u to create a new, updated snapshot. The tests should now succeed, reporting a test coverage of 100 percent for our component.

Try experimenting with your newfound knowledge. For example, can you write a snapshot test for the page routes in the _pages_ directory or a set of end-to-end tests for the GraphQL API?

### Summary

You should now be able to create automated tests with Jest and, more broadly, design a testing plan on your own to strike a balance between effort and reward. We discussed the benefits of TDD and unit testing and then used the _arrange, act, assert_ pattern to develop a simple sum function following test-driven principles. Next, we used three types of test doubles to replace the sum function while calculating the Fibonacci sequence. Finally, we added unit and snapshot tests to our existing Next.js application, created a mock of a Mongoose model, and used spies to verify our assumptions.

To learn more about Jest and automated testing, consult the official Jest documentation at [_https://jestjs.io/docs/getting-started_](https://jestjs.io/docs/getting-started). In the next chapter, you’ll explore the differences between authorization and authentication and how you can leverage OAuth in your applications.