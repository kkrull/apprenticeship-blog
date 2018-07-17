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
func (router RequestLineRouter) parseRequestLine(reader *bufio.Reader) (Request, Response) {
	requestLineText, err := readCRLFLine(reader) //(line string, err Response)
	if err != nil {
		//No input, or it doesn't end in CRLF
		return nil, err
	}

	requested, err := parseRequestLine(requestLineText) //(RequestLine, err Response)
	if err != nil {
		//Not a well-formed HTTP request line with {method, target, version}
		return nil, err
	}

	if request := router.routeRequest(requested); request != nil {
		//Well-formed, executable Request to a known route
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

Take Go's [official code format][gofmt] and its stance on [forbidding unused variables and imports][golang-faq-unused], for example.  I may disagree with the way some code is formatted and think that there are reasonable times to allow unused variables, but tools like [`goimports`][goimports] make it easy to follow standards, and the compiler doesn't leave you with much of a choice.  The time that would otherwise be spent deliberating alternatives can now be re-focused on other code that might be more important.

Back to the code in question - we can explore different structures that may clarify its control flow, but the lack of general-purpose, higher-order functions in Go limits our options.  We can experiment with different abstractions, but at the end of the day we're just moving around the `if` statements.


[gofmt]: https://golang.org/cmd/gofmt
[goimports]: https://godoc.org/golang.org/x/tools/cmd/goimports
[golang-faq-unused]: https://golang.org/doc/faq#unused_variables_and_imports


## Non-idiomatic options

You may be familiar with other approaches to error handling and control flow that are used in other languages, and it can be tempting to try to apply your favorite technique to Go.  Let us briefly consider a few tempting options that are likely to raise eyebrows in the Go community, in all but the rarest of circumstances.


### Defer, Panic, Recover

The first is called [Defer, Panic, and Recover][golang-blog-defer-panic-recover], where `panic` and `recover` are used to simulate the effects of `throw` and `catch` in other languages like Java and C#.  There are a few points worth noting here:

* The Go authors do [cite a case where it's used in Go standard libraries][json-decode], but they have also been careful to keep panics from escaping to the outside world.  In most cases, `panic` is reserved for truly catastrophic errors (much like the [`Error` class in Java][java-error] is used for non-recoverable errors).
* The theoretical argument that exceptions break Referential Transparency: Martin Odersky describes this well in his book [Functional Programming in Scala][fp-in-scala-book].  To sum it up: code that can throw an exception can evaluate to different values depending on whether or not it is surrounded in a `try/catch` block, so the programmer has to be aware of more, global context.  There's a nice [example of this on GitHub][fp-in-scala-example] and a [concise explanation in this answer][stackoverflow-referential-transparency].
* The practical argument: It's [difficult to distinguish correct and incorrect exception-based code][chen-harder-to-recognize].

The Go authors tend to make distinctions between expected branches in control flow (valid and invalid input, for example) and large-scale events that threaten the process as a whole.  If you are to remain in the good graces of the Go community, you should make an effort to reserve `panic` for the latter.


[chen-harder-to-recognize]: https://blogs.msdn.microsoft.com/oldnewthing/20050114-00/?p=36693/
[fp-in-scala-book]: https://www.manning.com/books/functional-programming-in-scala
[fp-in-scala-example]: https://github.com/fpinscala/fpinscala/blob/master/exercises/src/main/scala/fpinscala/errorhandling/Option.scala
[golang-blog-defer-panic-recover]: https://blog.golang.org/defer-panic-and-recover
[java-error]: https://docs.oracle.com/javase/8/docs/api/?java/lang/Error.html
[json-decode]: https://golang.org/src/encoding/json/decode.go
[stackoverflow-referential-transparency]: https://stackoverflow.com/a/28993780


### Higher-order functions and wrapper types

Go authors like Rob Pike say over and over again to [just write a "for" loop][robpike-filter], but it's hard to resist thinking about the example problem as a series of transformations:

    bufio.Reader -> string -> RequestLine -> Request  
    
_Shouldn't we be able to write a `map` function for this?_

Go is a statically typed language without generics, so your options are to declare types and functions with the types specific to your domain, or abandon type safety altogether.

Imagine what it would look like if you tried to write `map`:

{% highlight go %}
// Sure, we can declare a few commonly-used variations...
func mapIntToInt(value int, func(int) int) int { ... }
func mapStringToInt(value string, func(string) int) int { ... }

// ...but how does this help?
func map(value interface{}, mapper func(interface{}) interface{}) interface{} { ... }
{% endhighlight %}

There are options that abandon type safety, like [this Go Promise library][github-go-promise].  However, take a close look at how the example code carefully sidestep the issue of type safety by using the output value in a function - `fmt.Println` - that doesn't care about its type.

{% highlight go %}
var p = promise.New(...)
p.Then(func(data interface{}) {
    fmt.Println("The result is:", data)
})
{% endhighlight %}

Making a a wrapper like Scala's [`Either` type][scala-either] doesn't really solve the problem either, because it would need to have type-safe functions to transform happy- and sad-path values.  It would be nice to be able to write the example function a bit more like this:

{% highlight go %}
func (router RequestLineRouter) parseRequestLine(reader *bufio.Reader) (Request, Response) {
	request, response := newStringOrResponse(readCRLFLine(reader)).
		Map(parseRequestLine).
		Map(router.routeRequest)

	if response != nil {
		return nil, response
	} else if request == nil {
		//Technically, this doesn't work because we now lack the intermediate value
		return nil, requested.NotImplemented()
	} else {
		return request, nil
	}
}
{% endhighlight %}

But look how much single-use code you would have to write to support that:

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

So the various forms of writing higher-order functions wind up being non-idiomatic, impractical, or both.  Go simply is not a functional language.


[github-go-promise]: https://github.com/chebyrash/promise
[robpike-filter]: https://github.com/robpike/filter
[scala-either]: https://www.scala-lang.org/api/2.9.3/scala/Either.html


## Back to basics

Now that we have seen how functional styles are not likely to be helpful, let's [remind ourselves][robpike-errors-are-values]:

> The key lesson, however, is that errors are values and the full power of the Go programming language is available for processing them.

The other bit of good news is that you can't unknowingly ignore a returned error, like you can with an unchecked exception.  The compiler will force you at a minimum to declare the error as `_`, and tools like [`errcheck`][errcheck] do a good job of keeping you honest.

[errcheck]: https://github.com/kisielk/errcheck
[robpike-errors-are-values]: https://blog.golang.org/errors-are-values


### Refactor your workflow

Looking back at the example code, there are 2 definite errors (input not ending in CRLF and not-well-formed CRLF lines), 1 clearly successful response, and 1 default response.

Why don't we just group those together with some more functions?

{% highlight go %}
func (router RequestLineRouter) parseRequestLine(reader *bufio.Reader) (Request, Response) {
	requested, err := readRequestLine(reader)
	if err != nil {
		//No input, not ending in CRLF, or not a well-formed request
		return nil, err
	}

	return router.routedRequestOrNotImplemented(requested)
}

func readRequestLine(reader *bufio.Reader) (*RequestLine, Response) {
	requestLineText, err := readCRLFLine(reader)
	if err == nil {
		return parseRequestLine(requestLineText)
	} else {
		return nil, err
	}
}

func (router RequestLineRouter) routedRequestOrNotImplemented(requested *RequestLine) (Request, Response) {
	if request := router.routeRequest(requested); request != nil {
		//Well-formed, executable Request to a known route
		return request, nil
	}

	//Valid request, but no route to handle it
	return nil, requested.NotImplemented()
}
{% endhighlight %}

Here, we're back to using plain old Go code.  We get the benefit of a smaller `parseRequestLine` at the cost of a couple of helpers functions.  You get to decide which end of the trade-off works for you: more functions that are smaller, or fewer functions that are a bit larger?


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
