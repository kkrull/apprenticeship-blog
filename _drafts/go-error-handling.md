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


## A longer example

There's not much I would change in the prior example to make it shorter or simpler, but the notion of putting an `if` statement after each function call starts to feel like it's getting out of hand in larger examples like this:

{% highlight go %}
func (router RequestRouter) parseRequestLine(reader *bufio.Reader) (ok Request, err Response) {
	requestLineText, err := readCRLFLine(reader)
	if err != nil {
		//No input, or it doesn't end in CRLF
		return nil, err
	}

	requested, err := parseRequestLine(requestLineText)
	if err != nil {
		//Not a well-formed HTTP request line with {method, target, version}
		return nil, err
	}

	if request := router.routeRequest(requested); request != nil {
		//Well-formed request to a known route
		return request, nil
	}
	
	//Valid request, but no route to handle it
	return nil, requested.NotImplemented()
}
{% endhighlight %}

There are a few things I don't like about this method:

1. The total number of lines in this method suggests that I should extract (and name) some helpers, but that seems like it would be overkill when there are only 3 or 4 steps in the happy path.
1. The happy path returns from the middle of two error cases, instead of being at the very beginning or the very end.  This makes it hard to read.
1. What exactly is the scope of `err` anyway?  Normally `:=` prevents you from re-assigning an old variable in single-assignment statements, but the multi-assignment version seems a bit more relaxed.  Is the second `err` re-assigned over the first one, at the same address?  Or is a distinct `err` at a distinct address created, after the first drops out of scope?


## First option: Keep it and move on

While this version of `parseRequestLine` does bug me, using an `if` guard after each possible error is the idiomatic way of handling errors in Go.  We will explore some other ways to refactor this code, but keep in mind that _this code does what it needs to do without abusing features of the language_.  There is an advantage to Go's sometimes Spartan philosophy: When there is one, clear way to do something, you can accept it and move on (even if you're not thrilled with the standard).

Take Go's [official code format][gofmt] and its stance on [forbidding unused variables and imports][golang-faq-unused], for example.  I may disagree with the way some code is formatted and think that there are reasonable times to allow unused variables, but tools like [`goimports`][goimports] make it easy to follow standards, and the compiler doesn't leave you with much of a choice.  The time that would otherwise be spent deliberating alternatives can now be re-focused on other code that might be more important, when you stop to think about it.

Back to the code in question - we can explore different structures that may improve readability, but the lack of general-purpose, higher-order functions in Go limits our options.  We can experiment with different abstractions, but at the end of the day we're just moving around the `if` statements.


[gofmt]: https://golang.org/cmd/gofmt
[goimports]: https://godoc.org/golang.org/x/tools/cmd/goimports
[golang-faq-unused]: https://golang.org/doc/faq#unused_variables_and_imports


## Non-idiomatic options

Let us briefly consider a few tempting options that are likely to raise eyebrows in the Go community, in all but the rarest of circumstances.


### Defer, Panic, Recover

The first is called [Defer, Panic, and Recover][golang-blog-defer-panic-recover], where `panic` and `recover` are used to simulate the effects of `throw` and `catch` in other languages like Java and C#.  There are a few points worth noting here:

* Although the authors do [cite a case where it's used in Go standard libraries][json-decode], note that these details are never exposed to the client.  Outside of this function, `panic` is reserved for truly catastrophic, non-recoverable errors (much like the [`Error` class in Java][java-error]).
* The theoretical argument that exceptions break Referential Transparency.  Martin Odersky describes this well in his book [Functional Programming in Scala][fp-in-scala-book].  To sum it up: code that can throw an exception can evaluate to different values depending on whether or not it is surrounded in a `try/catch` block, so the programmer has to be aware of more, global context.  There's a nice [example of this on GitHub][fp-in-scala-example] and a [concise explanation in this answer][stackoverflow-referential-transparency].
* The practical argument that it can be [difficult to distinguish correct and incorrect exception-based code][chen-harder-to-recognize].


[chen-harder-to-recognize]: https://blogs.msdn.microsoft.com/oldnewthing/20050114-00/?p=36693/
[fp-in-scala-book]: https://www.manning.com/books/functional-programming-in-scala
[fp-in-scala-example]: https://github.com/fpinscala/fpinscala/blob/master/exercises/src/main/scala/fpinscala/errorhandling/Option.scala
[golang-blog-defer-panic-recover]: https://blog.golang.org/defer-panic-and-recover
[java-error]: https://docs.oracle.com/javase/8/docs/api/?java/lang/Error.html
[json-decode]: https://golang.org/src/encoding/json/decode.go
[stackoverflow-referential-transparency]: https://stackoverflow.com/a/28993780


### Higher-order functions and wrapper types

Go authors like Rob Pike say over and over again to [just write a "for" loop][robpike-filter], but it's hard to resist thinking about the example problem in terms of a series of transformations.  After all, isn't the example code just going through a sequence of steps?

* read the first line of text up to CRLF from the `bufio.Reader`
* parse that line into a `RequestLine{ Method, Target, Version string }`
* look through known routes for the given `RequestLine` to get an executable `Request{ Handle(client io.Writer) }`
* -or- return the first error `Reponse` encountered, during the process.

Go is a statically typed language without generics.  That means the following options are out:

* A type-safe [`Either` type][scala-either] like the one in Scala
* Wrapping normal and error values in a tuple, as in Erlang and Elixir
* Writing type-safe versions of higher-order functions like `map`

One could write a few trivial variations like `func mapIntToInt(value int, func(int) int) int` and `func mapStringToInt(value string, func(string) int) int`, but that doesn't help with our own, custom types.

If type safety is to be abandoned, there are options like [this Go Promise library][github-go-promise].  But why then are you using a language with static types if you're going to bypass type checking?

The final option is to write your own, domain-specific `Map` functions inside your own package.  That may be practical under some circumstances, but the example code grows unreasonably large.  For this code to work (which doesn't technically, because intermediate results are needed later):

{% highlight go %}
func (router RequestLineRouter) parseRequestLine(reader *bufio.Reader) (Request, Response) {
	request, response := newStringOrResponse(readCRLFLine(reader)).
		Map(parseRequestLine).
		Map(router.routeRequest)

	if response != nil {
		return nil, response
	} else if request == nil {
		return nil, requested.NotImplemented()
	} else {
		return request, nil
	}
}
{% endhighlight %}

It request a **significant** amount of support code:

{% highlight go %}
func newStringOrResponse(data string, err Response) *StringOrResponse {
	return &StringOrResponse{data: data, err: err}
}

type StringOrResponse struct {
	data string
	err Response
}

func (mapper *StringOrResponse) Map(makeRequestLine func(text string) (*RequestLine, Response)) *RequestLineOrResponse {
	if mapper.err != nil {
		return &RequestLineOrResponse{data: nil, err: mapper.err}
	}

	requestLine, err := makeRequestLine(mapper.data)
	if err != nil {
		return &RequestLineOrResponse{data: nil, err: mapper.err}
	}

	return &RequestLineOrResponse{data: requestLine, err: nil}
}

type RequestLineOrResponse struct {
	data *RequestLine
	err Response
}

func (mapper *RequestLineOrResponse) Map(routeRequest func(requested *RequestLine) Request) (Request, Response) {
	if mapper.err != nil {
		return nil, mapper.err
	}

	return routeRequest(mapper.data), nil
}
{% endhighlight %}

So this approach is either non-idiomatic, impractical, or both.


[github-go-promise]: https://github.com/chebyrash/promise
[robpike-filter]: https://github.com/robpike/filter
[scala-either]: https://www.scala-lang.org/api/2.9.3/scala/Either.html


## Back to Refactoring

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


## Plain old if statements (make no change)

* Errors are a value.  You handle these values like you would handle any other value.
* Start with my original request parsing example


If you stop reading here and stick to if statements

* you're likely to be writing idiomatic Go code
* if you have tested and they are passing, you may be able to move on
* although a bit polarizing, the ability to have working code and move on without getting distracted is valuable


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
