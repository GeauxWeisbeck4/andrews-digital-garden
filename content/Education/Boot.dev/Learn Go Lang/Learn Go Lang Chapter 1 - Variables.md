---
id: 01J6DE909BRJVKRXRWZ1D218RX
modified: 2024-08-28T17:55:22-04:00
title: Learn Go Lang Chapter 1 - Variables
tags:
  - go-lang
  - variables
  - boot-dev
  - programming
---
# Which Type Should I Use?

With so many types for what is essentially just a number, developers coming from languages that only have one kind of `Number` type (like JavaScript) may find the choices daunting.

## Prefer "default" types

A problem arises when we have a `uint16`, and the function we are trying to pass it into takes an `int`. We're forced to write code riddled with type conversions like:

```go
var myAge uint16 = 25
myAgeInt := int(myAge)
```

This style of code can be slow and annoying to read. When Go developers stray from the “default” type for any given type family, the code can get messy quickly. Unless you have a good _performance related_ reason, you'll typically just want to use the "default" types:

- `bool`
- `string`
- `int`
- `uint`
- `byte`
- `rune`
- `float64`
- `complex128`

## When should I use a more specific type?

When you're super concerned about performance and memory usage.

That’s about it. The only reason to deviate from the defaults is to squeeze out every last bit of performance when you are writing an application that is resource-constrained. (Or, in the special case of `uint64`, you need an absurd range of unsigned integers).

You can [read more on this subject here](https://blog.boot.dev/golang/default-native-types-golang/) if you'd like, but it's not required.

# Go is Statically Typed

Go enforces [static typing](https://developer.mozilla.org/en-US/docs/Glossary/Static_typing) meaning variable types are known _before_ the code runs. That means your editor and the compiler can display type errors before the code is ever run, making development easier and faster.

Contrast this with most dynamically typed languages like JavaScript and Python... Dynamic typing often leads to subtle bugs that are hard to detect. The code _must_ be run to catch syntax and type errors. (sometimes in production if you're unlucky 😨)

## Concatenating strings

Two strings can be [concatenated](https://en.wikipedia.org/wiki/Concatenation) with the `+` operator. But the compiler will not allow you to concatenate a `string` variable with an `int` or a `float64`.

## Assignment

Texttio uses [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) to log users in.

The code on the right has a type error. Change the type of `password` to a string (but use the same numeric value) so that it can be concatenated with the `username` variable.

## Answer
```go
package main

import "fmt"

func main() {
	var username string = "presidentSkroob"
	var password string = "12345"

	// don't edit below this line
	fmt.Println("Authorization: Basic", username+":"+password)
}
```
# Compiled vs Interpreted

You can run a compiled program _without_ the original source code. You don't need the compiler anymore after it's done its job. That's how most videogames are distributed! Players don't need to install the correct version of `python` to run a PC game: they just download the executable game and run it.

![compiler vs interpreter](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/7RBQRNA.png)

With interpreted languages like Python and Ruby, the code is interpreted at [runtime](https://en.wikipedia.org/wiki/Runtime_(program_lifecycle_phase)) by a separate program known as the "interpreter". Distributing code for users to run can be a pain because they need to have an interpreter installed, and they need access to the source code.

## Examples of compiled languages

- Go
- C
- C++
- Rust

## Examples of interpreted languages

- JavaScript (sometimes JIT-compiled, but a similar concept)
- Python
- Ruby

## Why build Textio in a compiled language?

One of the most convenient things about using a compiled language like Go for Textio is that when we deploy our server we don't need to include any runtime language dependencies like Node or a Python interpreter. We just add the pre-compiled binary to the server and start it up!

# Same Line Declarations

You can declare multiple variables on the same line:

```go
mileage, company := 80276, "Tesla"
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

The above is the same as:

```go
mileage := 80276
company := "Tesla"
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

At the top of the `main` function, declare a float called `averageOpenRate` and string called `displayMessage` _on the same line._

Initialize them to values:

- `.23`
- `is the average open rate of your messages`

before they're printed.

# Small memory footprint

Go programs are fairly lightweight. Each program includes a small amount of extra code that's included in the executable binary called the [Go Runtime](https://go.dev/doc/faq#runtime). One of the purposes of the Go runtime is to clean up unused memory at runtime. It includes a [garbage collector](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)) that automatically frees up memory that's no longer in use.

## Comparison

As a general rule, Java programs use _more_ memory than comparable Go programs. There are several reasons for this, but one of them is that Java uses a virtual machine to interpret bytecode at runtime and typically allocates more on the [heap](https://courses.grainger.illinois.edu/cs225/fa2022/resources/stack-heap/).

On the other hand, Rust and C programs use slightly _less_ memory than Go programs because more control is given to the developer to optimize the memory usage of the program. The Go runtime just handles it for us automatically.

## Idle memory usage

![idle memory](https://miro.medium.com/max/1400/1*Ggs-bJxobwZmrbfuoWGpFw.png)

In the chart above, [Dexter Darwich compares the memory usage](https://medium.com/@dexterdarwich/comparison-between-java-go-and-rust-fdb21bd5fb7c) of three _very_ simple programs written in Java, Go, and Rust. As you can see, Go and Rust use _very_ little memory when compared to Java.

# Computed Constants

Constants must be known at compile time. They are _usually_ declared with a static value:

```go
const myInt = 15
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

However, constants _can be computed_ as long as the computation can happen at _compile time_.

For example, this is valid:

```go
const firstName = "Lane"
const lastName = "Wagner"
const fullName = firstName + " " + lastName
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

That said, you _cannot_ declare a constant that can only be computed at run-time like you can in JavaScript. This breaks:

```go
// the current time can only be known when the program is running
const currentTime = time.Now()
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Assignment

Keeping track of time in a message-sending application is _critical_. Imagine getting an appointment reminder an hour **after** your doctor's visit.

Complete the code using a computed constant to print the number of seconds in an hour.

# Comparing Go's Speed

Go is _generally_ faster and more lightweight than interpreted or VM-powered languages like:

- Python
- JavaScript
- PHP
- Ruby
- Java

However, in terms of execution speed, Go does lag behind some other compiled languages like:

- C
- C++
- Rust

Go is a bit slower mostly due to its automated memory management, also known as the "Go runtime". Slightly slower speed is the price we pay for memory safety and simple syntax!

![speed comparison](https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/tUIWLob.png)

Textio is an amazing candidate for a Go project. We'll be able to quickly process large amounts of text all while using a language that is safe and simple to write.

# Formatting Strings in Go

Go follows the [printf tradition](https://cplusplus.com/reference/cstdio/printf/) from the C language. In my opinion, string formatting/interpolation in Go is _less_ elegant than Python's f-strings, unfortunately.

- [fmt.Printf](https://pkg.go.dev/fmt#Printf) - Prints a formatted string to [standard out](https://stackoverflow.com/questions/3385201/confused-about-stdin-stdout-and-stderr).
- [fmt.Sprintf()](https://pkg.go.dev/fmt#Sprintf) - Returns the formatted string

These following "formatting verbs" work with the formatting functions above:

## Default representation

The `%v` variant prints the Go syntax representation of a value, it's a nice default.

```go
s := fmt.Sprintf("I am %v years old", 10)
// I am 10 years old

s := fmt.Sprintf("I am %v years old", "way too many")
// I am way too many years old
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

If you want to print in a more specific way, you can use the following formatting verbs:

## String

```go
s := fmt.Sprintf("I am %s years old", "way too many")
// I am way too many years old
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Integer

```go
s := fmt.Sprintf("I am %d years old", 10)
// I am 10 years old
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

## Float

```go
s := fmt.Sprintf("I am %f years old", 10.523)
// I am 10.523000 years old

// The ".2" rounds the number to 2 decimal places
s := fmt.Sprintf("I am %.2f years old", 10.523)
// I am 10.52 years old
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

If you're interested in all the formatting options, you can look at the `fmt` package's [docs](https://pkg.go.dev/fmt#hdr-Printing).

## Assignment

Create a new variable called `msg` on line 9. It's a string that contains the following:

```
Hi NAME, your open rate is OPENRATE percentNEWLINE
```

![Copy icon](https://www.boot.dev/img/copy_icon.svg)

- Replace `NAME` with the variable `name`,
- Replace `OPENRATE` with the variable `openRate` rounded to the nearest "tenths" place, e.g `10.523` should be rounded down to `10.5`
- The word percent should appear as part of the string following the open rate value
- Replace `NEWLINE` with the newline [`\n`](https://en.wikipedia.org/wiki/Newline) escape sequence.

_Note: The Boot.dev code editor won't output any print statments that do not end with a newline. `fmt.Println` automatically ends with a newline, but `fmt.Print` does not._

## Answer

```go
package main

import "fmt"

func main() {
	const name = "Saul Goodman"
	const openRate = 30.5

	msg := fmt.Sprintf("Hi %v, your open rate is %v percent", name, openRate)

	// don't edit below this line

	fmt.Println(msg)
}

```

# Runes and String Encoding

In many programming languages (cough, C, cough), a "character" is a single byte. Using [ASCII](https://www.asciitable.com/) encoding, the standard for the C programming language, we can represent 128 characters with 7 bits. This is enough for the English alphabet, numbers, and some special characters.

In Go, strings are just sequences of bytes: they can hold arbitrary data. However, Go also has a special type, [`rune`](https://go.dev/blog/strings), which is an alias for `int32`. This means that a `rune` is a 32-bit integer, which is large enough to hold any [Unicode](https://home.unicode.org/) code point.

When you're working with strings, you need to be aware of the encoding (bytes -> representation). Go uses [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding, which is a variable-length encoding.

## What does this mean?

There are 2 main takeaways:

1. When you need to work with individual characters in a string, you should use the `rune` type. It breaks strings up into their individual characters, which can be more than one byte long.
2. We can use [zany](https://www.merriam-webster.com/dictionary/zany) characters like emojis and Chinese characters in our strings, and Go will handle them just fine.

## Assignment

Boots is a _bear_. (Not a dog, haters).

1. Run the code as-is. Notice that the simple string "boots" has 5 bytes, and 5 runes (characters).
2. Update the `name` variable to be the [bear emoji](https://emojipedia.org/bear) instead of the word "boots".

If you've got it right, you should notice that the emoji is only one rune, but it takes up 4 bytes.