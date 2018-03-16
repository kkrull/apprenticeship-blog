---
layout: post
title:  "Getting into Go Part 1: Learning the Basics"
date:   2018-03-15 17:00:09 -0500
categories: go setup
---

I'm learning Go this week, so it's time to go back to basics and see if I can
express a thought in a new language.  This of course means starting with the
ubiquitous "Hello World" program and trying to maintain a beginner's state of
mind.

I generally don't consider myself to really _know_ how to use a new library or
language until I can write tests for it, so I won't consider this exercise to
be done until I can use Test Driven Development for the code.  I'll also offer
some initial impressions along the way, which I'm sure to re-evaluate as I
spend more time with Go.


## Getting something to work

Getting Go installed on MacOS is a matter of running the installer and
optionally setting `GOPATH` if you want Go to work anywhere other than
`$HOME/go`.  The 
[installation instructions](https://golang.org/doc/install#testing) 
will get you up and running with this short program:

```go
package main

import "fmt"

func main() {
    fmt.Printf("hello, world\n")
}
```

A simple `go build` compiles your code to a self-contained, statically linked
binary for your OS and architecture.  You can 
[run more builds for different platforms](https://github.com/golang/go/wiki/WindowsCrossCompiling),
if needed.  If all goes well, you should see something like this:

```shell
$ ./hello 
hello, world
```

A further `go install` puts the binary in `$GOPATH/bin`, which they recommend
[adding to your path](https://golang.org/doc/code.html#GOPATH).


## Initial Thoughts

My first impression of Go is fairly positive: the example works as-advertised
in documentation that's appropriately detailed in 
[How to Write Go Code](https://golang.org/doc/code.html).
It's really convenient to have a single `go` program that handles all the
common things a developer needs to do without installing or configuring extra
tools like Maven or MSBuild.

At first, it seems a bit odd that code is typically organized first by a
[single, universal workspace](https://golang.org/doc/code.html#Organization),
then by (Git) repository, then by package.  I'm accustomed to build tools not
caring very much about where you store your source code or any packages created
from it, but my earlier experiences have either used interpreted languages or
have relied on a platform-independent form like Java bytecode or MSIL.  After a
bit of reflection, it makes sense that Go developers would — by necessity —
share source code directly and keep it all carefully organized so that clients
can build their own dependencies for their own platform. 

An interesting side-effect of this approach is that `go get` fetches directly
from Git repositories, meaning you don't just get the source code — you also
get the commit history. 

*Groovy.*


## What's Next?

At this point, I still haven't reached my goal of being able to *test* any of
this behavior.  Stay tuned for the next article on my experiences in testing
and test-driving Go code.

