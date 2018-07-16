---
layout: post
title:  "A Survey of Error Handling Patterns in Go"
date:   2018-07-15 15:49:00 -0500
categories: go errors
---

When you're learning another language, there can be periods of frustration where you're having trouble expressing an idea that would have been easier in a more familiar language.  It's natural to wonder why the new language was designed that way, and it's easy to get fooled into thinking that - if you're having trouble expressing an idea in a language - it must be the fault of the language's creators.  This line of reasoning can lead you to using the new language in ways that are not idiomatic for that language.

One such topic that has challenged my own conceptions is how errors are represented, triggered, and handled in Go.  To recap:

* An error in Go is any type that implements the [`error` interface][golang-error-type], by providing an `Error() string` method.
* A function returns an `error` just like it returns any other value.  [Multiple return values][golang-multiple-return-values] are used to distinguish errors from the function's normal return value.
* Errors are handled by checking the value(s) returned from a function and propagated to higher layers of abstraction through simple returns (perhaps with an embellishment that adds more details to the error message).

For example, consider a function that resolves a host address and starts listening for TCP connections.  There are two things that can go wrong, so there are two error checks:

{% highlight go %}
func StartListening(host string, port uint16) (net.Listener, error) {
	address, addressErr := net.ResolveTCPAddr("tcp", fmt.Sprintf("%s:%d", host, port))
	if addressErr != nil {
		return nil, fmt.Errorf("StartListening: %s", addressErr)
	}

	listener, listenError := net.ListenTCP("tcp", address)
	if listenError != nil {
		return nil, fmt.Errorf("StartListening: %s", listenError)
	}

	return listener, nil
}
{% endhighlight %}


This topic has its own entry on the [Go FAQ][golang-faq-errors], and the community has offered a wide range of opinions on the matter.  Depending upon your past experiences, you may be inclined to think:

* Go should [implement some form of exception handling][morgan-go-error-handling] that allows you to write `try/catch` blocks to group error-generating code and distinguish it from error-handling code.
* Go should [implement some form of pattern matching][yager-go-is-not-good] that offers a concise way to wrap errors and apply distinct transformations to values and errors, alike.

While those are useful features in other languages, they are not likely to be implemented anytime soon in Go.  Instead, let's take a look at a few ways that we can write idiomatic Go code today, using the features that are already available.


[golang-faq-errors]: https://golang.org/doc/faq#exceptions
[golang-error-type]: https://golang.org/pkg/builtin/#error
[golang-multiple-return-values]: https://golang.org/doc/effective_go.html#multiple-returns
[morgan-go-error-handling]: https://opencredo.com/why-i-dont-like-error-handling-in-go/
[yager-go-is-not-good]: http://yager.io/programming/go.html


## Plain old if statements (make no change)

* Errors are a value.  You handle these values like you would handle any other value.
* Start with my original request parsing example


If you stop reading here and stick to if statements

* you're likely to be writing idiomatic Go code
* if you have tested and they are passing, you may be able to move on
* although a bit polarizing, the ability to have working code and move on without getting distracted is valuable




## Intro

How it works

* Errors are a value -- cite the Rob Pike article
* that means you can handle those values any old way
* On the plus side, it's harder to ignore an error value than it is to ignore an unchecked exception.

https://blog.golang.org/errors-are-values

> The key lesson, however, is that errors are values and the full power of the Go programming language is available for processing them.


Observations

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


## Notes from FP in Scala

s safer and retains referential transparency, and through the use of higher-order functions, we can preserve the primary benefit of exceptions

We can prove that y is not referentially transparent. Recall that any RT expression may be substituted with the value it refers to, and this substitution should preserve program meaning. If we substitute throw new Exception("fail!") for y in x + y, it produces a different result, because the exception will now be raised inside a try block that will catch the exception and return 43:

Another way of understanding RT is that the meaning of RT expressions does not depend on context and may be reasoned about locally, whereas the meaning of non-RT expressions is context-dependent and requires more global reasoning. For instance, the meaning of the RT expression 42 + 5 doesn’t depend on the larger expression it’s embedded in—it’s always and forever equal to 47. But the meaning of the expression throw new Exception("fail") is very context-dependent—as we just demonstrated, it takes on different meanings depending on which try block (if any) it’s nested within.

There are two main problems with exceptions:

As we just discussed, exceptions break RT and introduce context dependence, moving us away from the simple reasoning of the substitution model and making it possible to write confusing exception-based code. This is the source of the folklore advice that exceptions should be used only for error handling, not for control flow.
Exceptions are not type-safe. The type of failingFn, Int => Int tells us nothing about the fact that exceptions may occur, and the compiler will certainly not force callers of failingFn to make a decision about how to handle those exceptions. If we forget to check for an exception in failingFn, this won’t be detected until runtime.

Java’s checked exceptions at least force a decision about whether to handle or reraise an error, but they result in significant boilerplate for callers. More importantly, they don’t work for higher-order functions, which can’t possibly be aware of the specific exceptions that could be raised by their arguments. For example, consider the map function we defined for List:


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


However foreign the Go authors' choices in error handling may seem, it's clear from the various blogs and talks they have given that their design choices were not arbitrary.  I try to remember in cases like this that the burden of inexperience lies with me - not with the language creators - and that I can learn to think of an old problem in new ways.
