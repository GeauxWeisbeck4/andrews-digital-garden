---
id: 01J6G5Y3MWST229XTVP5C9PV8G
modified: 2024-08-29T19:26:02-04:00
description: Learning about TypeScript typing, generics, classes, and interfaces
title: Chapter 2 - TypeScript Deep Dive - Typing, Generics, Classes, and Interfaces
tags:
  - typescript
  - full-stack
  - programming
  - web-development
---
Topics:
- Advanced typing techniques
- Creating types from other types
- Interface and **object-oriented programming** (**OOP**) features
- Generics
- Promises

## Narrowing

When we talk about narrowing, we refer to the process of moving from a less precise type to a more precise type. For instance, a variable that initially has a type of **any** or **unknown** can be narrowed down to more specific types such as string, number, or custom types.

Let’s delve into a practical example to illustrate how narrowing works in a real-world scenario. In the following code, we’ll discuss a **narrowToNumber** function and a **getChatMessagesWithNarrowing** function, both of which utilize narrowing:

function narrowToNumber(value: unknown): number {
    if (typeof value !== 'number') {
        throw new Error('Value is not a number');
    }
    return value;
}

This function takes a parameter of type **unknown** and aims to ensure that this parameter is indeed a number. The usage of **typeof** in the **if** statement is a classic example of type guarding, a form of narrowing. If the value is not of type **number**, an error is thrown; otherwise, the function safely returns the value, now assured to be a number. This is a classic example of runtime type checking in TypeScript.

Let’s move to an example of how we can get messages on our backend. You don’t need to know a lot of the specifics of the backend-related code happening in this example, but I will explain the gist and the parts that are relevant:

async function getChatMessagesWithNarrowing(chatId: unknown, req: { authorization: string }) {
    const authToken = req.authorization;
    const numberChatId = narrowToNumber(chatId);
    const messages = await chatService.getChatMessages(numberChatId, authToken);
    if (messages !== null) {
        messages.map((message) => {
            console.log(`Message ID: ${message.id}, Feedback: ${message.feedback?.trim() ?? "no feedback"}`);
        });
        return {success: true, messages}
    } else {
        return {success: false, message: 'Chat not found or access denied'}
    }
}

Here, **chatId** is passed to the function with an **unknown** type. However, in our scenario, **chatId** is expected to be a number. This is where the **narrowToNumber** function becomes useful. By applying it to **chatId**, we are narrowing the type from a broad **unknown** type to a more specific type of **number** type. Further down in the code, we encounter the following instance of narrowing:

if (messages !== null) {
   ....
}

This line is a subtle but vital example of narrowing. Here, we check if **messages** is not **null** before proceeding. This act effectively narrows the type of **messages** from possibly **null** or an array of **messages** to definitively an array of **messages**. This ensures that the operations inside the **if** block are safe and won’t result in a **TypeError** error due to attempting to access properties on **null**.

Now, let’s look into another type feature – **null** types – in the next section.

## null types

In JavaScript and TypeScript, **null** is a primitive value that represents the intentional absence of any object value. When TypeScript’s strict **null** checking is enabled (which is highly recommended), variables must be explicitly typed to include **null** or **undefined** if they are intended to hold an empty value. This contrasts with JavaScript, where variables can implicitly be **null** or **undefined** without such strict type distinctions, often leading to unintended errors if not carefully managed. This forces developers to consciously handle **null** cases, leading to more robust and error-resistant code. We’ve seen it in action already in the previous section, but let’s look at it closer:

if (messages !== null) {
    // …
}

Here, the **messages !== null** **null** check is a direct application of handling **null** types. In this context, **messages** is expected to be an array or **null**. The check ensures that the following code only runs if **messages** is indeed an array and not **null**. This is a simple yet effective way to guard against **null**-related errors.

Next, let’s examine another snippet here:

console.log(`Message ID: ${message.id}, Feedback: ${message.feedback?.trim() ?? "no feedback"}`);

In this line, the use of the **?** optional chaining operator and the **??** nullish coalescing operator provides a powerful combination for dealing with **null** values, which helps to prevent runtime errors where we would expect a value but got **null**. The expression **message.feedback?.trim()** expression will only attempt to call **trim()** if feedback is not **null** or **undefined**, thus avoiding a potential runtime error. If feedback is nullish (that is, **null** or **undefined**), the nullish coalescing operator takes over, providing a **"no feedback"** fallback value.

**null** types in TypeScript are not just a feature of the language; they represent a mindset shift toward safer, more predictable coding practices, as **Uncaught TypeError: Cannot read properties of null** is an error that is very commonly met in JavaScript but is completely gone from TypeScript.

Now, in the next section, let’s look at function types.

## Function types

A function type allows you to specify the exact form a function should take: the types of its input arguments and its return type. This feature is helpful when you pass functions around the code as arguments or if you create a constant of a function type.

Let’s look at an example of how we can use a function type when we log the details of every message in our code:

type MapCallback = (message: IMessage) => void;
const logMessage: MapCallback = (message) => {
    console.log(`Message ID: ${message.id}`);
};
messages.map((message: IMessage) => {
    logMessage(message);
});

Here, **MapCallback** is a function type definition. It tells TypeScript that any function with this type should take one argument, **message**, which is of type **IMessage**, and it should return nothing (**void**). This function type becomes a template for creating functions with this specific structure.

**logMessage** is a function that’s explicitly declared to be of type **MapCallback**. This means **logMessage** must match the structure defined by **MapCallback** – it takes an **IMessage** object as an argument and does not return anything. TypeScript will enforce this structure, ensuring that **logMessage** is used correctly according to the defined function type.

Finally, **logMessage** is used within the **map** function. Each item in messages (assumed to be an array of **IMessage** objects) is passed to **logMessage**, which adheres to the structure defined by **MapCallback**. This ensures that the function is applied correctly to each **message** type and that we can only pass an argument of type **IMessage** to the **logMessage** function.

With this, we are ready to move to the next topic, creating types from other types, which will help us change existing types to get something new.

# Creating types from other types

In this section, we are going to cover a few techniques to adapt one type to another, such as **utility types**, **union types**, and **intersection types**. We will start by showing some useful utility functions for this job in TypeScript.

## Utility types

Utility types are a set of types provided by the language to transform existing types into new, modified versions. They offer a convenient way to alter properties of a type, making them optional, read-only, or excluding them, among other transformations. Let’s cover a few of them here:

- **Pick<Type, Keys>**: **Pick** constructs a new type by selecting a subset of properties from an existing type. **Pick** is used when you need a type with only specific properties from a parent type. This is useful for creating more focused and less bulky types. The following is an example of using **Pick**:
    
     interface IUser {
       id: number;
       name: string;
       email: string;
    }
    type UserPreview = Pick<IUser, 'id' | 'name'>;
    const userPreview: UserPreview = {id: '1', name: 'John'};
    
    Here, **UserPreview** contains only **id** and **name** from **IUser**.
    
- **Record<Keys, Type>**: **Record** generates a type with a set of keys and assigns a uniform type to these keys’ values. It is ideal for creating objects where keys have a common value type, often used for mapping or lookup purposes. Here is an example of using **Record**:
    
    type UserNamesById = Record<UserId, string>;
    const userNamesById: UserNamesById = {'1': 'John', '2': 'Alice'};
    
    **UserNamesById** is a dictionary object mapping **UserId** keys to string names.
    
- **Partial<Type>**: **Partial** turns all properties of a given type into optional properties. **Partial** is useful in situations such as updating parts of an object, where you don’t need to provide all properties. It provides flexibility in object creation. Here is how you would use **Partial** in code:
    
    type PartialIUser = Partial<IUser>;
    const partialUser: PartialIUser = {id: '1'};
    
    **PartialIUser** allows any combination of **IUser** properties, including incomplete objects, because all the properties are optional.
    
- **Required<Type>**: **Required** transforms all optional properties of a type into required ones. **Required** is the opposite of **Partial**. It enforces that all properties of the type must be provided, ensuring complete object definitions. The following is an example of using **Required**:
    
    type RequiredIUser = Required<PartialIUser>;
    const requiredUser: RequiredIUser = {id: '1', name: 'John', email: 'john@example.com'};
    
    **RequiredIUser** mandates that all properties, even those optional in **PartialIUser**, must be present.
    
- **Omit<Type, Keys>**: **Omit** creates a new type by omitting specified properties from an existing type. **Omit** is useful for creating a type that excludes certain properties from another type, which is particularly helpful in excluding sensitive or unnecessary properties. An example of **Omit** is as follows:
    
    type UserWithoutEmail = Omit<IUser, 'email'>;
    const userWithoutEmail: UserWithoutEmail = {id: '2', name: 'Alice'};
    
    **UserWithoutEmail** is a type similar to **IUser** but without the email property.
    
- **Readonly<Type>**: **Readonly** makes all properties of a type immutable post-creation. **Readonly** is used to create types where object properties shouldn’t be changed after the object is created, which is important for maintaining integrity in certain objects:
    
    type ReadonlyIUser = Readonly<IUser>;
    const user: ReadonlyIUser = {id: '1', name: 'John', email: 'john@example.com'};
    
    **ReadonlyIUser** guarantees that once an **IUser** object is created, its properties cannot be altered, as we cannot assign a new field to it afterward.
    

With this covered, let’s see how we can combine types together with union types, in the next section.

Union types

Union types, represented by a pipe sign (**|**), allow a variable to hold values that are a combination of two or more types, offering flexibility in defining types that can accept multiple, specific types of values. They’re essential in scenarios where a variable or function return type isn’t confined to a single type.

Let’s apply this concept to the provided examples and create a type for our **IMessage** interface:

type MessageType = "user" | "system";

Here, **MessageType** is a union type, meaning it can hold either a **"user"** or **"system"** value. Now, let’s add it to our interface, as shown here:

interface IMessage {
    type: MessageType;
    // other properties
}

In the **IMessage** interface, the type property must be either **"user"** or **"system"**, adhering to the **MessageType** union. However, union types don’t have to be primitive values. Let’s now look at how we can handle returning a union type from a function. The **getChatFromDb** function here illustrates a common use case for union types in function return values:

type DbChatSuccessResponse = {
    success: true;
    data: IChat;
};
type DbChatErrorResponse = {
    success: false;
    error: string;
};
function getChatFromDb(chatId: string): DbChatSuccessResponse | DbChatErrorResponse {
    const findChatById = (_: string) => ({} as IChat)
    const chat = findChatById(chatId);
    if (chat) {
        return {
            success: true,
            data: chat,
        };
    } else {
        return {
            success: false,
            error: "Chat not found in the database",
        };
    }
}

**getChatFromDb** can return either **DbChatSuccessResponse**, which represents a successful operation, or **DbChatErrorResponse**, which has properties for a failed response. This approach is useful for error handling and data fetching, where the outcome might differ significantly. Now, let’s handle the union type result of the function with narrowing in the following code:

const dbResponse = getChatFromDb("chat123");
if (dbResponse.success === true) {
    console.log("Chat data:", dbResponse.data);
} else {
    console.error("Error:", dbResponse.error);
}

In this snippet, the response from **getChatFromDb** is either a **success** or an **error** object. The **if dbResponse.success === true** check effectively distinguishes between these two possible return types. If **success** is **true**, TypeScript understands that **dbResponse** conforms to **DbChatSuccessResponse** and allows access to **dbResponse.data**. Otherwise, it treats **dbResponse** as **DbChatErrorResponse**, exposing the **error** property.

Let’s next look at a resembling technique called type intersections.

## Type intersections

Type intersection is a feature that creates a new type that includes all properties of combined types. It’s symbolized by an ampersand (**&**) and is particularly useful for composing complex types from simpler ones. In the following code, we are going to create a type for a database chat entity that must also have an **id** value:

type IDBEntityWithId = {
    id: number;
};
type IChatEntity = {
    name: string;
};
type IChatEntityWithId = IDBEntityWithId & IChatEntity;
const chatEntity: IChatEntityWithId = {
    id: 1,
    name: "Typescript tuitor",
};

Here, **chatEntity** is declared with the **IChatEntityWithId** type, so it must include both **id** from **IDBEntityWithId** and **name** from **IChatEntity**. This illustrates how intersection types enforce the presence of all properties from combined types. We can also achieve similar functionality by extending interfaces, which we are going to show in the following topic.

With this, we’ve covered the essential part of advanced typing techniques, and we are ready to talk more about how to reuse and write extensible code with **OOP** and **interfaces**.

# Interface and OOP features

Both OOP and interfaces serve a few very important purposes – they help to clearly define the structure of objects passed around, reuse code, and write extensible functionality. We’ve looked at interfaces before, but now, let’s talk about how we can extend their definitions.

## Interfaces

Extending interfaces allows for the creation of new interfaces that inherit properties from existing ones, thereby enhancing reusability and organization. Let’s draw on an example we used before, but now using extending interfaces there:

interface IMessageWithType extends IMessage {
    type: MessageType;
}
const userMessage: IMessageWithType = {
    id: 10,
    chatId: 2,
    userId: 1,
    content: "Hello, world!",
    createdAt: new Date(),
    type: "user",
};

The **IMessageWithType** interface extends the **IMessage** interface, which means it includes all properties from **IMessage** plus any additional properties defined in **IMessageWithType**. Here, **IMessageWithType** adds the **type** property, of type **MessageType**, to the existing structure. When creating a **userMessage** object of type **IMessageWithType**, it’s required to include all properties from both **IMessage** and **IMessageWithType**.

Now, let’s look at OOP functionalities that exist in TypeScript.

## OOP functionalities

**OOP** is a programming paradigm that uses objects and classes to create models based on the real world. TypeScript embraces core OOP principles, allowing developers to use polymorphism, abstraction, inheritance, and encapsulation using native syntax. Let’s look at these functionalities in detail here:

- **Polymorphism**: This allows objects of subclasses to be treated as objects of a common superclass. It’s about creating a structure where a function can utilize subclasses of the superclass interchangeably. Polymorphism is primarily achieved through interfaces and abstract classes. By defining a common interface or an abstract class, TypeScript allows different classes to implement the same structure or methods, thus enabling functions to work with objects of these different classes as if they were working with the base class.
- **Abstraction**: TypeScript uses abstract classes and interfaces to implement abstraction. These constructs allow you to define a standard template or contract that other classes can implement, encapsulating complex logic and exposing only the necessary parts.
- **Inheritance**: This is a mechanism where a new class extends (inherits from) an existing class, allowing for the reuse of code and creating a hierarchical relationship between classes. In TypeScript, inheritance is implemented using the **extends** keyword. A class can extend another class, inheriting its properties and methods.
- **Encapsulation**: This involves bundling data and methods that operate on the data within one unit, often a class, and restricting access to some of the object’s components, which ensures data integrity. Encapsulation in TypeScript is achieved through access modifiers such as **public**, **private**, and **protected**. These modifiers control the visibility and accessibility of class members, ensuring that internal details of a class are hidden and only exposed through a defined interface. Here is a description of access modifiers:
    - **public**: This is the default access level for class members. Members declared as **public** can be accessed from anywhere, meaning there’s no restriction on access. This includes access from within the class itself, from instances of the class, and from subclasses.
    - **private**: Members declared as **private** can only be accessed from within the class itself. They are not accessible from instances of the class or from subclasses. This access level is used to hide the internal state and functionality of the class from the outside, enforcing encapsulation.
    - **protected**: Members declared as **protected** can be accessed from within the class and also from subclasses. However, they are not accessible from instances of the class (unless through methods defined within the class or subclass). This allows for a more controlled form of accessibility, useful for cases where the subclass needs more intimate knowledge of the superclass without exposing members to the general public.

Let’s look at the following example that combines these techniques. We will define an **AbstractDatabaseResource** abstract class with common methods and an **abstract** method. An **InMemoryChatResource** concrete class will extend this abstract class and provide specific implementations for storing chat in memory:

abstract class AbstractDatabaseResource {
    constructor(protected resourceName: string) {
    }
    protected logResource(resource: { id: number }): void {
        console.log(`[${this.resourceName}] Resource logged:`, resource);
    }
    abstract get(id: number): { id: number } | null
    abstract getAll(): { id: number }[]
    abstract addResource(resource: { id: number }): void;
}
const inMemoryChatResource = new InMemoryChatResource();
const chat1: IChat = {
    id: 1,
    ownerId: 2,
    messages: []
};
inMemoryChatResource.addResource(chat1);
const retrievedChat1 = inMemoryChatResource.get(1);

Let’s discuss various techniques we have used here:

- **Abstraction**: The **AbstractDatabaseResource** class provides external methods that are used to manage database elements (methods such as **get**, **getAll**, and **addResource**), while it hides the complexity of the specific implementation. Users of the **AbstractDatabaseResource** class need only be concerned with the interface – what methods are available and what parameters they accept – not how these methods are implemented. This **separation of concerns** (**SoC**) makes the system easier to understand and use.
- **Inheritance**: **InMemoryChatResource** is a concrete class that extends **AbstractDatabaseResource**. This means it inherits its properties and methods, but also provides specific implementations for the abstract methods defined in the base class.
- **Encapsulation**: In **InMemoryChatResource**, the **resources** array is marked as **private**, meaning it can’t be accessed directly from outside the class. This encapsulation ensures that the internal representation of chat resources is hidden from external use. The **logResource** method in the abstract class is marked as **protected**, allowing it to be accessed within the class and its subclasses, but not outside.
- **Polymorphism**: While **InMemoryChatResource** has different implementations of the methods (such as **addResource** and **get**), they can be used interchangeably in contexts where an **AbstractDatabaseResource** class is expected. This is polymorphism, where objects of different classes can be treated as objects of a common superclass.
- **Instantiation and use**: We create an instance of **InMemoryChatResource** and use it to add and retrieve chat data. Despite the specific underlying implementation (in-memory array), the code only relies on the use of shared abstract class structure definition to know the methods and properties it can retrieve.

It is important to use interfaces and classes appropriately. As a rule of thumb, leverage interfaces for defining contracts and shapes of data, ensuring consistency across implementations and facilitating easy refactoring. Use classes to encapsulate data and behavior, taking advantage of inheritance and polymorphism to promote code reuse and maintainability, while keeping class definitions focused and avoiding overly complex inheritance hierarchies. Interfaces are good for defining the abstract structure that you are going to operate on in a function parameter, and classes are great for encapsulating complexity and provide only a simple and straightforward way to interact with the classes’ logic.

We can now work on the class code we introduced to cover two additional concepts: **generics** and **promises**.

# Generics

Generics in TypeScript are a tool for creating reusable components that can work with multiple types rather than a single one. This allows for better component abstraction while maintaining type safety as well. It also helps to write more flexible code that can adapt to different types. Let’s start with a basic example given here:

function printValue<T>(value: T): void {
    console.log(value);
}
printValue<number>(123);
printValue<string>("Hello");

**<T>** is a generic type parameter that allows this function to accept any type of value. When you call **printValue<number>(123)** and **printValue<string>("Hello")**, TypeScript treats **T** as **number** and **string**, respectively. This demonstrates how generics provide flexibility without losing the benefits of type checking.

We also can improve **InMemoryChatResource**, which we defined before. It works fine now, but what if we also need an in-memory implementation of **InMemoryUserResource** for **IUser** in addition to **IChat**? They basically have the same functionality; the only difference between them is the type of resource they handle. Generics can be helpful here. To illustrate that, let’s define a generic **GenericsInMemoryResource** class and create **chat** and **user** instances of it here:

class GenericsInMemoryResource<T extends { id: number }> extends AbstractDatabaseResource {
    private resources: T[] = [];
    constructor(resourceName: string) {
        super(resourceName);
    }
    get(id: number): T | null {
        const resource = this.resources.find((item) => item.id === id);
        return resource ? {...resource} : null;
    }
    getAll(): T[] {
        return [...this.resources];
    }
    addResource(resource: T): void {
        this.resources.push(resource);
        this.logResource(resource);
    }
}
const userInMemoryResource = new GenericsInMemoryResource<IUser>('user')
const chatInMemoryResource = new GenericsInMemoryResource<IChat>('chat')
userInMemoryResource.addResource({id: 1, name: 'Admin', email: 'admin@admin.com'});
chatInMemoryResource.addResource({id: 10, ownerId: userInMemoryResource.get(1)!.id, messages: []});

The **<T extends { id: number }>** syntax means **T** can be any type, but it must have an **id** property of type **number**. This ensures type safety while allowing for different resource types. Unlike **InMemoryChatResource**, which was limited to handling **IChat** objects, **GenericsInMemoryResource** can handle any type that meets the constraint. This eliminates the need for separate classes for each resource type, such as chats or users.

Instances such as **userInMemoryResource** and **chatInMemoryResource** demonstrate this using **GenericsInMemoryResource** with **IUser** and **IChat** types. Methods such as **addResource** and **get** work with the generic type **T**, making them adaptable to the specific type of resource being used.

The original **InMemoryChatResource** class is limited to handling chat data, while **GenericsInMemoryResource** is more flexible and scalable, capable of handling various types of resources. This leads to cleaner, more maintainable code, as you don’t need to create a new class for each resource type.

Generics significantly reduce code duplication by enabling a single class to manage multiple data types. This approach simplifies maintenance and enhances the adaptability of our code base.

Now, let’s briefly talk about how to handle promises with types.

# Promises

In TypeScript, when you declare a promise, you can use generics to indicate the type of data the promise will eventually return. This type indication ensures that the resolved value is consistent with expectations and allows TypeScript to provide relevant type checking and autocompletion. Let’s declare a **fetchData** function here that will imitate a network request with **setTimeout** and provide the type for its return value using the **Promise** type:

function fetchData(): Promise<string> {
    return new Promise((resolve) => {
        setTimeout(() => resolve("Data Fetched"), 1000);
    });
}

**fetchData** is a function returning a **Promise<string>** instance. This means the promise, when resolved, will return a string. Inside the promise, we simulate fetching data with **setTimeout** and resolve it with a **string** value, adhering to the declared return type **string**. This explicit typing of the promise’s resolved value ensures that any consumer of **fetchData** can expect a string.

With this, we have covered most aspects of TypeScript that we will need to build our chat application.

# Summary

In this chapter, we’ve delved into advanced TypeScript features, gaining a deeper understanding of concepts such as narrowing, **null** types, function types, and utility types, which are fundamental for developing robust web applications. This knowledge equips us with the tools to write safer, more predictable, and efficient code, reducing common JavaScript pitfalls. Moving forward, the next chapter will mark the beginning of our hands-on journey, where we’ll start building our application by configuring our backend runtime environment with **Bun**, laying the groundwork for our full stack TypeScript project.