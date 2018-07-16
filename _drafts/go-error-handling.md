---
layout: post
title:  "A Survey of Error Handling Patterns in Go"
date:   2018-07-15 15:49:00 -0500
categories: go errors
---

It's not hard to find articles that are critical of Go's chosen mechanism for representing and handling errors.


## Intro

How it works

* Error handling techniques are a bit different
* important when learning a new language to consider why they made those choices you don't understand, and what you're missing from your own experiences.
* Errors are a value -- cite the Rob Pike article
* that means you can handle those values any old way
* On the plus side, it's harder to ignore an error value than it is to ignore an unchecked exception.

https://blog.golang.org/errors-are-values

> The key lesson, however, is that errors are values and the full power of the Go programming language is available for processing them.


Observations

* this wasn't an arbitrary decision
* can I find sources for how exceptions break referential integrity?
* other languages tend towards generic, reusable solutions, but that goes a bit against the Go ethos of doing something simple that works right here and now.  why not just write a for loop?


General advice

* You should either handle an error, or not handle it but delegate it to a higher level (to the caller). Handling the error and returning it is bad practice as if the caller also does the same, the error might get handled several times.
* Handle it and don't return it to the caller, or return it to the caller.  Knew that already.  Works fine for multiple layers of abstraction.
* Guiding observation: There's no getting around the guard clauses, is there?  The only thing I can do is organize them differently.


Goals

* understand what techniques are available
* get past some of the polarization
* remember - not having your favorite mechanism for error handling does make you think


My example code:

* unclear control flow: 2 failures, one success, one fallback
* function is kind of long
* variable scope/lifecycle is a bit inconsistent


## Plain old if statements (make no change)

* Errors are a value.  You handle these values like you would handle any other value.
* Start with my original request parsing example


If you stop reading here and stick to if statements

* you're likely to be writing idiomatic Go code
* if you have tested and they are passing, you may be able to move on
* although a bit polarizing, the ability to have working code and move on without getting distracted is valuable


## Try/catch (not going to happen)

Defer, Panic, and Recover is an option if used locally.  Its classification of idiomatic is debatable.

Try/catch is commonly desired:

* A lot of people want to see try/catch
* Worth noting that other techniques are available
* Breaks referential integrity (research Scala/Odersky) -- https://stackoverflow.com/a/28993780
  https://livebook.manning.com/#!/book/functional-programming-in-scala/chapter-4/1
  https://github.com/fpinscala/fpinscala/blob/master/exercises/src/main/scala/fpinscala/errorhandling/Option.scala
* or, cite the article pointing out the hard-to-distinguish correct vs. incorrect code with try/catch


## First approach - refactor your control flow

Function call wrapping: I could have refactored it to

* one parse function that reads the line and parses it, handling the two error cases
* one handler function that uses the implemented handler or falls back to not implemented
* at that point, I'm back to a simple if/else and that's idomatic and non-crazy

What would wrapper functions look like?  doThirdThing(neededFor3, doSecondThing(neededFor2, doFirstThing(neededFor1)))
  Each function would have to have a guard clause that returns the error right away, if there is already an error

https://stackoverflow.com/questions/37346694/go-best-practice-to-handle-error-from-multiple-abstract-level


## Closure over errors

Cite Rob Pike article.

https://blog.golang.org/error-handling-and-go


> In Go, error handling is important. The language's design and conventions encourage you to explicitly check for errors where they occur (as distinct from the convention in other languages of throwing exceptions and sometimes catching them). In some cases this makes Go code verbose, but fortunately there are some techniques you can use to minimize repetitive error handling.


## Wrapper with pattern matching-ish?  (sort of like Either)

Cite the swish / pattern matching example.


## Either types (domain specific)

What the example would look like

* No generics support as of now.
* Would a hand-rolled, domain specific one be very hard to write? Maybe it's not as bad as I think.
* Is that really much different than the `catch` macro from my scheme days?  All that's doing is wrapping.


Note: This probably goes against the grain in terms of what is considered idiomatic.


## Promise-lite?

https://github.com/chebyrash/promise

* type-safety has been abandoned
* IMO this is not idiomatic.


## State machine-ish

* show the example code
* it's long; I wouldn't recommend it in all cases
* you're really not saving on if guards; you're just moving them around and naming intermediate states


## Continuation Passing Style

* What the example would look like, in CPS
* Is it practical?  

True CPS - can the next success/old error/new error function be passed in as a second parameter?

https://codeburst.io/behind-continuations-passing-style-practical-examples-in-go-b59f2b6fdb2c


## Summary

* I can use a state machine when I really want to, but it's usually more verbose than it's worth.
* I can just live with functions containing multiple points of failure being longer than I prefer.
* Making my own custom error types really isn't that bad
