---
layout: post
title:  "Consequences of Programming by Wishful Thinking"
date:   2018-04-12 16:56:00 -0500
categories: design process
---

[sicp]: http://groups.csail.mit.edu/mac/classes/6.001/abelson-sussman-lectures/
[walking-skeleton]: http://alistair.cockburn.us/Walking+skeleton
[wishful-thinking]: http://wiki.c2.com/?WishfulThinking

I've been doing a lot of top-down development lately, using a couple of guiding principles:

- Programming by [wishful thinking][wishful-thinking]: Where you write abstract code that uses imaginary, lower-level components before actually implementing those components.  It's described pretty well in the [Structure and Interpretation of Computer Programs][sicp] lectures.
- [Walking Skeleton][walking-skeleton]: Where you create a small end-to-end implementation first, relying more on high-level Acceptance Tests and less on low-level Unit Tests to drive the early stages of a project.


## Where Does This Lead?

Starting a project from the outermost layers and working your way inwards inevitably means that you as a client can only observe (and your tests can only demand) **behavior**.  By that, I'm talking more about what you can observe from the outside (black box style testing) than the implementation details that are a part of the overall solution (white box style testing).


### Tests Have a Wide Scope

In the example of my HTTP server, that means my earliest tests are making real TCP connections to my server, sending it real messages, and parsing real responses:

{% highlight go %}
Context("when the server is running", func() {
	BeforeEach(func() {
		server = http.MakeTCPServerOnAvailablePort(contentRoot, "localhost")
		Expect(server.Start()).To(Succeed())
	})

	It("responds to HTTP requests", func(done Done) {
		for i := 0; i < 2; i++ {
			conn, connectError = net.Dial("tcp", server.Address().String())
			Expect(connectError).NotTo(HaveOccurred())
			writeString(conn, "GET / HTTP/1.1\r\n\r\n")
			expectHttpResponse(conn)
		}

		close(done)
	})
})
{% endhighlight %}

This sort of test is great for being pretty far removed from the implementation details of the underlying production code.  If you just have 1 or 2 of these tests, chances are they will last a long time with minimal modification.  In these early stages, the coupling of test code to production code is low, so the tests are pretty resilient to change.

As the project grows, adding similarly-scoped tests for each new feature quickly becomes difficult.  What started life as 1 or 2 resilient tests grows into more tests that collectively get more and more brittle.  There are a couple of good indicators when your testing strategy is too heavily skewed towards integration testing:

1. It gets increasingly difficult to make modified versions of these tests that get the system into the state you need, or that can verify the expected outcome.
1. Making one change breaks more than 1 or 2 tests.

Put in the context of this example, switching to another transport or application layer, or even upgrading to HTTP 2 could mean the entire test suite suddenly breaks and has to be updated in a lot of places.


### Effects on Production Code

In an effort to avoid making premature or incorrect abstractions, I try to stick with this process for a little while until I get a couple more features working.  This inevitably leads to types, functions, and source files of production code that are on the large side.  By that, I mean types longer than 100-200 lines and methods longer than 5 lines (see [Extract Till You Drop][extract-till-you-drop]).

Sooner or later, the tests grow in size and more distinct responsibilities reveal themselves within the production code.  If I have implemented a feature and _then_ felt comfortable with identifying a suitable abstraction for a distinct responsibility, I use refactorings like [Extract Method][extract-method] and [Replace Method with Method Object][replace-method-with-method-object] to start introducing new types that do something distinct with the same kind of data.  My approach to this is still rather informal, where I extract a bunch of methods and then group those that operate on the same parameters into the same object.

After doing this, I'm left with more units of production code than scopes of tests.  In other words, tests that use the interfaces of _N_ types are now reaching through the outer types to test _N + M_ types, where _M_ is the number of types extracted during refactoring.

[extract-method]: https://refactoring.com/catalog/extractMethod.html
[extract-till-you-drop]: https://sites.google.com/site/unclebobconsultingllc/one-thing-extract-till-you-drop
[replace-method-with-method-object]: https://refactoring.com/catalog/replaceMethodWithMethodObject.html


### What About the Old Tests?

Admittedly, the early days of a new project wind up following more of an [Ice Cream Cone][ice-cream-cone] test strategy.  This begs the question:

**When should integration tests be converted into unit tests?**

I try to manage all this with a couple rules of thumb:

1. Extract new types that will be related to a change I need to make.  In other words, I try to avoid doing it just because something felt big.
1. Leave existing integration tests in-place where possible, but **add unit tests for new behavior**.

That means I still have some integration tests that cover multiple behaviors from multiple types, and I've got a handful of unit tests on the newly extracted types.  It's kind of gross — because there's not one place to look for a description of a type's behavior anymore — but I'm trying to make good decisions about my time.

Besides, there's a chance that the newly extracted type may be sufficiently isolated in responsibilities and stable enough that I won't have to change it.  If that code never changes, then it may not be worthwhile to suddenly move the tests down to that lower level as soon as the type is extracted.

On top of that, there are some other risks with moving the tests down too early:

1. I may still have made the wrong abstraction.  Leaving the tests at a higher scope leaves me free to change my mind a little while longer.
1. For any test of _N_ expected behaviors, extracting a component means you will need _N + 1_ tests — the original _N_ tests you had on the actual behaviors, and 1 more to cover the interface points with the extracted type.

For this reason, I try to do more of a pay-as-you-go philosophy on shifting integration-style tests down to unit-style tests.  I haven't developed any formal approach to this yet; it's usually driven by pain in maintaining the older, integration-style tests.

I'm still struggling with deciding when and what to extract or — put another way — when to stop, once I have started.

[ice-cream-cone]: https://medium.com/@fistsOfReason/testing-is-good-pyramids-are-bad-ice-cream-cones-are-the-worst-ad94b9b2f05f


## A Final Thought: Why Extract at All?

Well, if the system is indeed stable — and I have just written the last feature it will ever need — and if it works well enough that no bugs will ever need to be fixed — and I will never need to answer any detailed questions about how it works — then no, there's not much point in extracting types other than making myself feel better.

But chances are I *will* need to do at least one of those things.  That's why I try to do extraction judiciously, in more of an on-demand manner.
