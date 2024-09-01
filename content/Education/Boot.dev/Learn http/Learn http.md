---
id: 01J6K76N8811ZXHJFE63K92A8M
title: Learn http
description: Using http protocol in Go lang
modified: 2024-08-30T23:22:58-04:00
tags:
  - go-lang
  - http
  - boot-dev
  - programming
---
# Communicating on the web

Instagram would be pretty terrible if you had to manually copy your photos to your friend's phone when you wanted to share them. Modern applications need to be able to communicate information _between devices_ over the internet.

- Gmail doesn't just store your emails in variables on your computer, it stores them on computers in their data centers
- You don't lose your Slack messages if you drop your computer in a lake, those messages exist on Slack's [servers](https://en.wikipedia.org/wiki/Web_server)

## How does web communication work?

When two computers communicate with each other, they need to use the same rules. An English speaker can't communicate verbally with a Japanese speaker, similarly, two computers need to speak the same language to communicate.

This "language" that computers use is called a [protocol](https://en.wikipedia.org/wiki/Communication_protocol). The most popular protocol for web communication is [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview), which stands for Hypertext Transfer Protocol.

## Fantasy Quest Online

Throughout this course, we'll be building parts of an online game called "Fantasy Quest". It's a typical fantasy RPG with one key difference: all our player's data is stored online, on our servers.

## Assignment

Take a look at the `getItemData` function that I've provided in `http.go`. It retrieves information about items from Fantasy Quest's servers via HTTP as a slice of bytes `[]byte`.

In `main.go` do the following:

1. Convert the slice of bytes to a string with the built-in [`string()` type conversion](https://go.dev/tour/basics/13).
2. Print the string representation of the bytes to the console.

_It should look like a big ugly string of text._

## Tip

Notice how none of the data that is logged to the console was generated within our code! That's because the data we retrieved is being sent over the internet from our servers via HTTP.
`main.go`
```go
package main

import (
	"fmt"
	"log"
)

func main() {
	items, err := getItemData()
	if err != nil {
		log.Fatalf("error getting item data: %v", err)
	}

	// Don't edit above this line
	var data string = string(items)
	fmt.Println(data)
	// ?
}

```
`http.go`
```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func getItemData() ([]byte, error) {
	res, err := http.Get("https://api.boot.dev/v1/courses_rest_api/learn-http/items")
	if err != nil {
		return []byte{}, fmt.Errorf("error creating request: %w", err)
	}
	defer res.Body.Close()

	data, err := io.ReadAll(res.Body)
	return data, err
}
```
