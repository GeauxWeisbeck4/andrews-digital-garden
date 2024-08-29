---
id: 01J6DG2PTYNXNHJ0XT2234BE8H
modified: 2024-08-28T18:04:04-04:00
title: Learn Go Lang Chapter 2 - Conditionals
description: Learning how to use conditionals in Go Lang
tags:
  - go-lang
  - programming
  - conditionals
  - boot-dev
---
# Conditionals

`if` statements in Go do not use parentheses around the condition:

```go
if height > 4 {
    fmt.Println("You are tall enough!")
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

`else if` and `else` are supported as you might expect:

```go
if height > 6 {
    fmt.Println("You are super tall!")
} else if height > 4 {
    fmt.Println("You are tall enough!")
} else {
    fmt.Println("You are not tall enough!")
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

Fix the bug on line `12`. The `if` statement should print "Message sent" if the `messageLen` is _less than or equal to_ the `maxMessageLen`, or "Message not sent" otherwise.

## Tips

Here are some of the comparison operators in Go:

- `==` equal to
- `!=` not equal to
- `<` less than
- `>` greater than
- `<=` less than or equal to
- `>=` greater than or equal to

# The initial statement of an if block

An `if` conditional can have an "initial" statement. The variable(s) created in the initial statement are _only_ defined within the scope of the `if` body.

```go
if INITIAL_STATEMENT; CONDITION {
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Why would I use this?

It has two valuable purposes:

1. It's a bit shorter
2. It limits the scope of the initialized variable(s) to the `if` block

For example, instead of writing:

```go
length := getLength(email)
if length < 1 {
    fmt.Println("Email is invalid")
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

We can do:

```go
if length := getLength(email); length < 1 {
    fmt.Println("Email is invalid")
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

In the example above, `length` isn't available in the parent scope, which is nice because we don't need it there - we won't accidentally use it elsewhere in the function.

# Switch

Switch statements are a way to compare a value against multiple options. They are similar to if-else statements but are more concise and readable when the number of options is more than 2.

```go
func getCreator(os string) string {
    var creator string
    switch os {
    case "linux":
        creator = "Linus Torvalds"
    case "windows":
        creator = "Bill Gates"
    case "mac":
        creator = "A Steve"
    default:
        creator = "Unknown"
    }
    return creator
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

Notice that in Go, the `break` statement is not required at the end of a `case` to stop it from falling through to the next `case`. The `break` statement is implicit in Go.

If you _do_ want a `case` to fall through to the next `case`, you can use the `fallthrough` keyword.

```go
func getCreator(os string) string {
    var creator string
    switch os {
    case "linux":
        creator = "Linus Torvalds"
    case "windows":
        creator = "Bill Gates"

    // all three of these cases will set creator to "A Steve"
    case "Mac OS":
        fallthrough
    case "Mac OS X":
        fallthrough
    case "mac":
        creator = "A Steve"

    default:
        creator = "Unknown"
    }
    return creator
}
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The `default` case does what you'd expect: it's the case that runs if none of the other cases match.

## Assignment

I know we haven't covered function syntax in depth yet, but _bear_ with me.

Fix the bug in the `billingCost` function. If the `plan` is:

- "pro", the cost should be `20.0`
- "enterprise", the cost should be `50.0`