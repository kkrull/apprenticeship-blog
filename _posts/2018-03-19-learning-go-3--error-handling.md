---
layout: post
title:  "Getting into Go Part 3: Error Handling"
date:   2018-03-19 18:02:00 -0500
categories: go errors
---

*Now the real fun begins.*

I'm writing my own HTTP server in Go as a learning exercise.  There are all kinds of things that can go wrong when communicating with a remote server, so this should serve as an excellent opportunity to learn about how error handling is done in Go.

Go provides a couple of easy functions to listen for and make TCP connections:

{% highlight go %}
import "net"

listener, serverError := net.Listen("tcp", "127.0.0.1:8080") //Server side
connection, clientError := net.Dial("tcp", "127.0.0.1:8080") //Client
{% endhighlight %}

Both of these functions return errors.  When I get in to actually communicating with my HTTP server, there are a wealth of additional errors that I will need to handle.  Among other things, connections can be dropped altogether, reset by gateways, or there can be protocol violations.

So I'd better learn how to do some error handling in Go, and its use of multiple return values to differentiate happy/error results suggests that this is going to be a bit different from what I'm used to.


## How to tell if an error happened

On the bright side, writing a basic [Gomega assertion for errors][gomega-error-matching] is pretty darn simple:

{% highlight go %}
listener, err := net.Listen("tcp", "127.0.0.1:8080")

//Choices when you are not expecting an error
Expect(err).To(BeNil()) //Error does not exist
Expect(net.Listen(...)).To(...) //Checks for an error, then matches the first returned value

//Choices when you expect an error
Expect(err).NotTo(BeNil()) //Doesn't exist
Expect(err.Error()).To(...) //Match the error message
Expect(err).To(MatchError(interface{})) //Structural match
{% endhighlight %}

Which way is better for checking the error?

- It's kind of hard to tell what the structure of the error should be.  It's [declared as simply `error`][listener-error], so it could change at any time.
- It's also kind of hard to tell what the error message should be, especially when you get it wrong.

{% highlight shell %}
Expected
  <*net.OpError | 0xc4200ae870>: {
      Op: "listen",
      Net: "tcp",
      Source: nil,
      Addr: nil,
      Err: {Err: "no such host", Name: "bogus", Server: "", IsTimeout: false, IsTemporary: false},
  }
to match error
  <string>: listen tcp: lookup bogus: no such host ---
{% endhighlight %}

I'm not sure what a good approach is right now to ensure you're getting the error you expect, without being tied to implementation details of the underlying library.  It will probably become more clear as I implement my own error messages or types, but I'm getting ahead of myself.

In either case, adding a [Gomega annotation][gomega-annotation] can add any missing details to the test failure.


## Error handling patterns

There's room for debate in how best to handle errors.  I used to think that throwing exceptions was an *inalienable right* of modern software development, that the days of returning `int` error codes in C were long behind us.

Then I learned a [variant of Scheme][swish-gen-server] that used pattern matching and dynamic typing to return either an error (a vector starting with a known symbol) or the successful result.  Here's one example:

{% highlight scheme %}
(match (catch (do-call server request timeout))
  [#(ok ,res) res] ;success
  [#(EXIT ,reason) ...]) ;failure
{% endhighlight %}

The pattern matcher was pretty good at complaining if you forgot to handle the error case, so in my experience it was a pretty reasonable alternative.

Another alternative is [Scala's Either type][scala-either], which gives you a type-safe way to return different types, depending upon whether there is an error.  I don't have much experience with it, but I seem to recall there being a sound argument in [Functional Programming in Scala][fp-in-scala]


## How does it work in Go?

Go uses an approach I haven't seen before, and it is documented in yet another FAQ of the form ["why doesn't Go have..."][faq-exception].  There's also a section in [Effective Go][effective-go-exception] and a separate [blog post by the authors][blog-exception].

The pattern I keep seeing is:

{% highlight go %}

{% endhighlight %}


{% highlight go %}
func DoSomethingThatMayFail() (result <type>, err error) { ... }

var goodResult, err := DoSomethingThatMayFail()
if err {
  //Handle it right away.  Don't try to do anything with goodResult.
}
{% endhighlight %}

On the bright side, I'm glad that the `error` interface is simple and that there's not a complex hierarchy of error types to navigate.  It's also a plus that the result isn't conveyed through modifying one of the parameters passed to the function, like the old days.

I don't have enough experience yet to know how well this is going to work out.  I've got more questions than answers at the moment:

1. Should I be making my own `error` types, like [this guy suggests][custom-error-types], that can be de-structured and understood by the client?
1. Is it simpler just to return `errors.New("bang!")` or `fmt.Errorf(...)` and test that a plausible looking message is returned from `Error()`?
1. Is chaining/wrapping errors a thing in Go, like it is in Java and somewhat in C#?  *Given the criticism leveled at Java's early insistence on checked exceptions, I may have just answered my own question.*

I don't have good answers for this right now.  Stay tuned to see if I figure it out!


[blog-exception]: https://blog.golang.org/error-handling-and-go
[custom-error-types]: https://medium.com/@sebdah/go-best-practices-error-handling-2d15e1f0c5ee
[effective-go-exception]: https://golang.org/doc/effective_go.html#errors
[faq-exception]: https://golang.org/doc/faq#exceptions
[fp-in-scala]: https://livebook.manning.com/#!/book/functional-programming-in-scala/chapter-4/1
[gomega-annotation]: http://onsi.github.io/gomega/#annotating-assertions
[gomega-error-matching]: http://onsi.github.io/gomega/#handling-errors
[listener-error]: https://golang.org/pkg/net/#TCPListener
[scala-either]: http://www.scala-lang.org/api/2.12.0/scala/util/Either.html
[swish-gen-server]: https://github.com/becls/swish/blob/master/src/swish/gen-server.ss