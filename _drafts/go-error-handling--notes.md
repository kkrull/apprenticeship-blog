---
layout: post
title:  "DRAFT Golang: `<Insert Epiphany Here>`"
date:   2018-07-14 14:15:00 -0500
categories: go
---


## Go error handling

* golang anti-pattern: use `panic`.
* golang norm: Just have slightly longer functions that have lots of if guards in them.
  Declare variables in the if-block, when you can.
* Functional state transition pattern.
  Revamp the Wile E. Coyote example.
  Maybe it would be better to write the example code in Scheme or Elixir.
* What would an "either" type container look like?
  No generics support as of now.
  Would a hand-rolled, domain specific one be very hard to write? Maybe it's not as bad as I think.
  Is that really much different than the `catch` macro from my scheme days?  All that's doing is wrapping.
* What would wrapper functions look like?  doThirdThing(neededFor3, doSecondThing(neededFor2, doFirstThing(neededFor1)))
  Each function would have to have a guard clause that returns the error right away, if there is already an error
* Would a domain-specific Promise type look any different?
* wrapper function to unwrap a successful value or go to an error handler?
* True CPS - can the next success/old error/new error function be passed in as a second parameter?


### What have I learned since my recent writings?

* I can use a state machine when I really want to, but it's usually more verbose than it's worth.
* I can just live with functions containing multiple points of failure being longer than I prefer.
* Making my own custom error types really isn't that bad
* _what error handling patterns exist, in general?_
* _what do gophers recommend?_


### Am I saying anything original?

Maybe.  Research it.

Not entirely.  This guy shows how to do CPS in Go: 
https://codeburst.io/behind-continuations-passing-style-practical-examples-in-go-b59f2b6fdb2c

### Me being original

What would be _original_ is if I'm not all for it or all against it.  
I should try a tone of making an honest attempt to use the language's features to the fullest, without abusing them.
I can also try to build a layman's case for why exceptions break referential integrity.
I can also give a more comprehensive set of options.

# TODO KDK

* Try the monadic form of the error, where a helper function/type is deciding whether to NOP due to an error or do the thing
* Try the CPS style: https://codeburst.io/behind-continuations-passing-style-practical-examples-in-go-b59f2b6fdb2c
* Come up with examples of other ways:
** Just put in the guard clauses and move on
** State machine pattern
** Domain-specific Either type
** Promises can be done, but there's no type safety: https://github.com/chebyrash/promise

### Back to whatever I was doing before

Apple's Swift programming language does not provide an exception handling mechanism.

Go. Best practice to handle error from multiple abstract level:
https://stackoverflow.com/questions/37346694/go-best-practice-to-handle-error-from-multiple-abstract-level

You should either handle an error, or not handle it but delegate it to a higher level (to the caller). Handling the error and returning it is bad practice as if the caller also does the same, the error might get handled several times.


This guy talks about error handling monads in Go and I have no idea what he's talking about:
https://awalterschulze.github.io/blog/post/monads-for-goprogrammers/


My thoughts:

* Handle it and don't return it to the caller, or return it to the caller.  Knew that already.  Works fine for multiple layers of abstraction.
* Most articles miss the problem with having multiple error-returning calls in the same function at a single level of abstraction.
  The problem there is in having to add a guard clause after every, single call.
* There's no getting around the guard clauses, is there?  The only thing I can do is organize them differently.
  Either one after another, after each error-prone function call.
  Or at the start of every CPS-style function, to propagate the old error to the next point.
* On the plus side, it's harder to ignore an error value than it is to ignore an unchecked exception.
* Nested ifs are a possibility, even though I find the nesting to be a smell of poorly-abstracted code.
  Usually when you see nesting, you can address it by extracting functions that handle the inner parts.
* Defer, Panic, and Recover is an option if used locally.  Its classification of idiomatic is debatable.


#### This guy sums it up well

https://opencredo.com/why-i-dont-like-error-handling-in-go/

He goes over the basic problem and shows the errWriter example from Rob Pike's blog.


#### Go blog

https://blog.golang.org/error-handling-and-go

In Go, error handling is important. The language's design and conventions encourage you to explicitly check for errors where they occur (as distinct from the convention in other languages of throwing exceptions and sometimes catching them). In some cases this makes Go code verbose, but fortunately there are some techniques you can use to minimize repetitive error handling.


And another one here by Rob Pike: https://blog.golang.org/errors-are-values


A function closure may help

```go
var err error
write := func(buf []byte) {
    if err != nil {
        return
    }
    _, err = w.Write(buf)
}
write(p0[a:b])
write(p1[c:d])
write(p2[e:f])
// and so on
if err != nil {
    return err
}
```

    The key lesson, however, is that errors are values and the full power of the Go programming language is available for processing them.


### Does this put me at odds with the golang community?

* Using a wrapper type instead of `(success, error)` multiple returns would most certainly be against golang idioms.


### What can I draw upon?

* The Case of The Double-Clutching RequestParser
* State Machines to the Rescue?
* Getting into Go Part 3: Error Handling
