---
title: "When Go Slice Bounds Get Hazy"
date: 2017-09-01T23:00:32+10:00
tags: ["golang", "slices"]
---

Earlier this week, I thought I stumbled across a bug in the go standard library.  
Earlier this week, I was wrong.  
  
While browsing the net/http sources, I spotted what appears to be an [index out of range](https://github.com/golang/go/blob/f9cf8e5ab11c7ea3f1b9fde302c0a325df020b1a/src/net/http/request.go#L869) in calls to `http.Request.BasicAuth()`.
  
Decoding `func parseBasicAuth()`:

- If the value string has prefix "Basic "
- base64 decode the remainder of the string
- find the index of the first colon (:) in the decoded result
- return username and password as the substrings before and after the colon, respectively

But wait - what if there's nothing after the colon?  
  
I posited that crafting an HTTP request with an Authorization header with nothing after the colon should result in a runtime error, so quickly threw together a simple http server that calls to `BasicAuth()`, started it, then tested with the following client code:  
```
	enc := base64.StdEncoding.EncodeToString([]byte("username:"))
	auth := fmt.Sprintf("Basic %s", enc)
	r.Header.Set("Authorization", auth)

	http.DefaultClient.Do(r)
```
... and nothing happened.

Today I discovered that it's legal to slice an `array` or `string` *from* its length to its end.  
  
> For arrays or strings, the indices are in range if 0 <= low <= high <= len(a), otherwise they are out of range.  
>  -- *The Go Programming Language Specification*

Here is some code to demonstrate, runnable in [The Playground](https://play.golang.org/p/PtgLSjCl2g):
```
package main

import (
	"fmt"
)

func main() {

	// Start with a string
	s := "index"
	fmt.Println("The string is:", s)

	// What's at index 0?
	fmt.Printf("The letter at index %d is: %s\n", 0, string(s[0])) // output: "i"

	// Now as a slice 0:0
	fmt.Printf("The letter at slice [%d:%d] is: %s\n", 0, 0, string(s[0:0])) // output: ""

	// Now as a slice 1:1
	fmt.Printf("The letter at slice [%d:%d] is: %s\n... and so on\n\n", 1, 1, string(s[1:1])) // output: ""

	// Now as a slice 0:1
	fmt.Printf("The letter at slice [%d:%d] is: %s\n", 0, 1, string(s[0:1])) // output: "i"

	// Now as a slice 0:1
	fmt.Printf("The letter at slice [%d:%d] is: %s\n... and so on\n\n", 1, 2, string(s[1:2])) // output: "n"

	// last index == len(s) - 1
	fmt.Printf("The letter at final index %d [len(s) - 1] is: %s\n", len(s)-1, string(s[len(s)-1])) // output: "x"

	// When we go beyond that, we panic
	printPanic(s)

	// But for some reason, this is OK when slicing
	fmt.Printf("\nThe letter at slice [%d:] is: %s\n", len(s), string(s[len(s):])) // output: ""
}

func printPanic(s string) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("panic recovered: ", r)
		}
	}()

	fmt.Printf("The letter at index %d is: ", len(s))
	fmt.Printf(" %s\n", string(s[len(s)]))
}
```
