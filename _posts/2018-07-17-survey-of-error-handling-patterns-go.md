---
layout: post
title:  "Exploring Error Handling Patterns in Go"
date:   2018-07-17 09:49:00 -0500
categories: go errors
---

When you're learning another language, there can be periods of frustration where you're having trouble expressing an idea that would have been easier in a more familiar language.  It's natural to wonder why the new language was designed that way, and it's easy to get fooled into thinking that — if you're having trouble expressing an idea in a language — it must be the fault of the language's authors.  This line of reasoning can lead you to using a language in ways that are not idiomatic for that language.

One such topic that has challenged my own conceptions is how errors are handled in Go.  To recap:

* An error in Go is any type implementing the [`error` interface][golang-error-type] with an `Error() string` method.
* Functions return `error`s just like any other value.  [Multiple return values][golang-multiple-return-values] distinguish errors from normal return values.
* Errors are handled by checking the value(s) returned from a function and propagated to higher layers of abstraction through simple returns (perhaps adding details to the error message).

For example, consider a function that resolves a host address and starts listening for TCP connections.  There are two things that can go wrong, so there are two error checks:

{% highlight go %}
func Listen(host string, port uint16) (net.Listener, error) {
	addr, addrErr := net.ResolveTCPAddr("tcp", fmt.Sprintf("%s:%d", host, port))
	if addrErr != nil {
		return nil, fmt.Errorf("Listen: %s", addrErr)
	}

	listener, listenError := net.ListenTCP("tcp", addr)
	if listenError != nil {
		return nil, fmt.Errorf("Listen: %s", listenError)
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

There may not be much to change in the prior example to make it shorter or simpler, but writing an `if` statement after each function call can start to feel like it's getting out of hand:

{% highlight go %}
func (router HttpRouter) parse(reader *bufio.Reader) (Request, Response) {
	requestText, err := readCRLFLine(reader) //string, err Response
	if err != nil {
		//No input, or it doesn't end in CRLF
		return nil, err
	}

	requestLine, err := parseRequestLine(requestText) //RequestLine, err Response
	if err != nil {
		//Not a well-formed HTTP request line with {method, target, version}
		return nil, err
	}

	if request := router.routeRequest(requestLine); request != nil {
		//Well-formed, executable Request to a known route
		return request, nil
	}

	//Valid request, but no route to handle it
	return nil, requestLine.NotImplemented()
}
{% endhighlight %}

There are a few things to be desired in this method:

1. The total number of lines in this method suggests that some helper functions [could be extracted][extract-till-you-drop].
1. Returning a successful value from the middle of two error cases makes it hard to read.
1. It's not clear if the second `err` is a new variable at a new address, or re-assignment of the first one.  It's not entirely consistent the single-variable form of `:=`, which forbids re-assignment.

[extract-till-you-drop]: https://sites.google.com/site/unclebobconsultingllc/one-thing-extract-till-you-drop


## First option: Keep it and move on

While this version of `parse` does bug me, using an `if` guard after each possible error is the idiomatic way of handling errors in Go.  We will explore some other ways to refactor this code, but keep in mind that _this code does what it needs to do without abusing features of the language_.  There is an advantage to Go's sometimes Spartan philosophy: When there is one, clear way to do something, you can accept it and move on (even if you're not thrilled with the standard).

Take Go's [official code format][gofmt] and its stance on [forbidding unused variables and imports][golang-faq-unused], for example.  I may disagree with the way some code is formatted and think that there are reasonable times to allow unused variables, but tools like [`goimports`][goimports] make it easy to follow standards, and the compiler doesn't leave you with much of a choice.  The time that would otherwise be spent deliberating alternatives can now be re-focused on other code that might be more important.

Back to the code in question — we can explore different structures that may clarify its control flow, but the lack of general-purpose, higher-order functions in Go limits our options.


[gofmt]: https://golang.org/cmd/gofmt
[goimports]: https://godoc.org/golang.org/x/tools/cmd/goimports
[golang-faq-unused]: https://golang.org/doc/faq#unused_variables_and_imports


## Non-idiomatic options

You may be familiar with other approaches to control flow that are used in other languages, and it can be tempting to try to apply your favorite technique to Go.  Let us briefly consider a few tempting options that are likely to raise eyebrows in the Go community, in all but the rarest of circumstances.


### Defer, Panic, Recover

The first is called [Defer, Panic, and Recover][golang-blog-defer-panic-recover], where `panic` and `recover` are used sort of like `throw` and `catch` in other languages.  There are a few points worth noting here:

* The Go authors do [cite a case where it's used in Go standard libraries][json-decode], but they have also been careful to keep panics from escaping to the outside world.  In most cases, `panic` is reserved for truly catastrophic errors (much like the [`Error` class in Java][java-error] is used for non-recoverable errors).
* The theoretical argument that exceptions break Referential Transparency: Martin Odersky describes this well in his book [Functional Programming in Scala][fp-in-scala-book].  To sum it up: code that can throw an exception can evaluate to different values depending on whether or not it is surrounded in a `try/catch` block, so the programmer has to be aware of global context in order to avoid bugs.  There's a nice [example of this on GitHub][fp-in-scala-example] and a [concise explanation in this answer][stackoverflow-referential-transparency].
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
type any interface{}
func mapFn(value any, mapper func(any) any) any {
	return mapper(value)
}
{% endhighlight %}

There are options that abandon type safety, like [this Go Promise library][github-go-promise].  However, take a close look at how the example code carefully sidesteps the issue of type safety by using the output value in a function that accepts any type (`fmt.Println` does ultimately use reflection to type its argument).

{% highlight go %}
var p = promise.New(...)
p.Then(func(data interface{}) {
    fmt.Println("The result is:", data)
})
{% endhighlight %}

Making a wrapper like Scala's [`Either` type][scala-either] doesn't really solve the problem either, because it would need to have type-safe functions to transform happy- and sad-path values.  It would be nice to be able to write the example function a bit more like this:

{% highlight go %}
func (router HttpRouter) parse(reader *bufio.Reader) (Request, Response) {
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

type ParseRequestLine func(text string) (*RequestLine, Response)
func (either *StringOrResponse) Map(parse ParseRequestLine) *RequestLineOrResponse {
	if either.err != nil {
		return &RequestLineOrResponse{data: nil, err: either.err}
	}

	requestLine, err := parse(either.data)
	if err != nil {
		return &RequestLineOrResponse{data: nil, err: either.err}
	}

	return &RequestLineOrResponse{data: requestLine, err: nil}
}

type RequestLineOrResponse struct {
	data *RequestLine
	err Response
}

type RouteRequest func(requested *RequestLine) Request
func (either *RequestLineOrResponse) Map(route RouteRequest) (Request, Response) {
	if either.err != nil {
		return nil, either.err
	}

	return route(either.data), nil
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


### Function grouping

Looking back at the example code, there are 2 definite errors (input not ending in CRLF and not-well-formed HTTP request lines), 1 clearly successful response, and 1 default response.

Why don't we just group those cases together?

{% highlight go %}
func (router HttpRouter) parse(reader *bufio.Reader) (Request, Response) {
	requested, err := readRequestLine(reader)
	if err != nil {
		//No input, not ending in CRLF, or not a well-formed request
		return nil, err
	}

	return router.requestOr501(requested)
}

func readRequestLine(reader *bufio.Reader) (*RequestLine, Response) {
	requestLineText, err := readCRLFLine(reader)
	if err == nil {
		return parseRequestLine(requestLineText)
	} else {
		return nil, err
	}
}

func (router HttpRouter) requestOr501(line *RequestLine) (Request, Response) {
	if request := router.routeRequest(line); request != nil {
		//Well-formed, executable Request to a known route
		return request, nil
	}

	//Valid request, but no route to handle it
	return nil, line.NotImplemented()
}
{% endhighlight %}

Here we get the benefit of a smaller `parse` at the cost of a couple of extra functions.  You get to decide whether you prefer _more_ functions, or _larger_ functions.


### Happy- and sad-paths in parallel

One could also refactor the existing functions to handle happy and sad paths at the same time.

{% highlight go %}
func (router HttpRouter) parse(reader *bufio.Reader) (Request, Response) {
	return router.route(parseRequestLine(readCRLFLine(reader)))
}

//Same as before
func readCRLFLine(reader *bufio.Reader) (string, Response) { ... }

func parseRequestLine(text string, prevErr Response) (*RequestLine, Response) {
	if prevErr != nil {
		//New
		return nil, prevErr
	}

	fields := strings.Split(text, " ")
	if len(fields) != 3 {
		return nil, &clienterror.BadRequest{
			DisplayText: "incorrectly formatted or missing request-line",
		}
	}

	return &RequestLine{
		Method: fields[0],
		Target: fields[1],
	}, nil
}

func (router HttpRouter) route(line *RequestLine, prevErr Response) (Request, Response) {
	if prevErr != nil {
		//New
		return nil, prevErr
	}

	for _, route := range router.routes {
		request := route.Route(line)
		if request != nil {
			//Valid request to a known route
			return request, nil
		}
	}

	//Valid request, but unknown route
	return nil, &servererror.NotImplemented{Method: line.Method}
}
{% endhighlight %}

Instead of 4 `if` guards in the original example, we're down to 3 without adding any more functions.  The trade-off is having to read the top-level `parse` inside-out.


### Closure for errors

You can also create a closure over the first error.  The example code [from the article][pike-error-closure] starts like so:

{% highlight go %}
_, err = fd.Write(p0[a:b])
if err != nil {
    return err
}
_, err = fd.Write(p1[c:d])
if err != nil {
    return err
}
_, err = fd.Write(p2[e:f])
if err != nil {
    return err
}
{% endhighlight %}

The author creates a `func` that applies the next step as long as no error has been encountered yet.

{% highlight go %}
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
if err != nil {
    return err
}
{% endhighlight %}

This approach works well when you can pass each step in your workflow to the closure, which suggests that each step should operate on the same types.  In some cases, this can work great.

In the example in this blog, you would have to create a `struct` to close over the error and write separate `applyParseText` / `applyParseRequest` / `applyRoute` functions for each step in the workflow, which is probably more trouble than it's worth.


[pike-error-closure]: https://blog.golang.org/errors-are-values


## Conclusion

Although Go's design choices in error handling may seem foreign at first, it's clear from the various blogs and talks its authors have given that these choices were not arbitrary.  Personally, I try to remember that the burden of inexperience lies with me — not with the Go authors — and that I can learn to think of an old problem in new ways.

When I started writing this article, I thought it might be possible to apply experiences from other, more functional languages to make my Go code simpler.  This experience has been a good reminder to me of what Go authors have been emphasizing all along: sometimes it's helpful to write a few functions specific to your own purposes and move on.
