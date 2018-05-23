---
layout: post
title:  "Test Code, Go Home"
date:   2018-05-23 17:55:00 -0500
categories: go testing
---

I kind of left you all hanging last time I wrote about [Testing in Go]({{ site.baseurl }}{% post_url 2018-05-13-go-test-organization %}), and it's finally time for me to organize my thoughts on this matter.

Just where do you put all this code that helps you write your tests, anyway?


## Put it in the `_suite_test.go` file

There it is.  That's the big reveal.

> Put your test helper functions, stubs, and mocks in the file ending in `_suite_test.go` - the one you use to run Ginkgo.

This of course assumes that you _are_ using Ginkgo, but – then again if you are using the built-in `testing` package – you're probably not reading this article anyway.  That's not meant as a dig on anybody; I just don't think you're likely to be interested in this topic if you're following the data-driven, test-after approach favored by the Go authors.  You probably just don't have a lot of mocks to organize.

For example:

{% highlight go %}
func TestFs(t *testing.T) {
	RegisterFailHandler(Fail)
	RunSpecs(t, "fs")
}

/* FileSystemResourceMock */

type FileSystemResourceMock struct {
	getPath    string
	headTarget string
}

func (mock *FileSystemResourceMock) Name() string {
	return "File system mock"
}

func (mock *FileSystemResourceMock) Get(client io.Writer, message http.RequestMessage) {
	mock.getPath = message.Path()
}

func (mock *FileSystemResourceMock) GetShouldHaveReceived(path string) {
	ExpectWithOffset(1, mock.getPath).To(Equal(path))
}

func (mock *FileSystemResourceMock) Head(client io.Writer, message http.RequestMessage) {
	mock.headTarget = message.Target()
}

func (mock *FileSystemResourceMock) HeadShouldHaveReceived(target string) {
	ExpectWithOffset(1, mock.headTarget).To(Equal(target))
}

/* Other types and helper functions... */
{% endhighlight %}

So why do I suggest this?  I like this approach so far because it solves the practical problem – most of the time – of putting helper code in the `_test` package and finding a corresponding `_test.go` file for it.  That makes it available to all the tests in the same package and to none of the production code.  Isn't that the problem we're trying to solve, here?

Besides, there's hardly anything in that suite file, anyway.
 
This works for me in most case, except when I need to...


## Use the Same Test Helper in Multiple Packages 
 
In practice, this doesn't happen terribly often for me, but sometimes I do need to use the same test helper in more than one place.  A good example is my `httptest.ResponseMessage` that parses HTTP response messages that are generated just about everywhere.

Yes I could push some logic down to a package that only writes HTTP response messages and test everything else at some higher level of abstraction, but I'm not ready to go there yet.  And we're still left with the problem in the general case.

So my approach so far is to make a separate package and directory that's readily identified as containing test-only code - hence the name `httptest`.  The downsides are:

* I _could_ mistakenly import this code into a production package and use it there.  Still, that's better than an earlier mistake of mine in which I monkey patched helper functions into production classes in Ruby, not realizing that the scope of that patch is entirely visible in the production code, as well.
* `go test ./...` generates an annoying warning saying that `httptest` doesn't have any tests.  Well _of course it doesn't have any tests!_.  You don't need to test the test code! (that's another mistake I have made)  


## What Else I Tried

Initially, I put all my test helpers into a single test package, with a separate source file corresponding to each production package I was testing.  So I had a `mock` package/directory with `http_mocks.go` for all the `http` helpers, `fs_mocks.go` for all the `fs` helpers, and so on.

It worked, and it read well.  It's pretty satisfying to write `mock.Request` in your test code to stand in for `http.Request` in the production code.

The only downside?  It got hard to maintain.  I felt it was too easy for me to move, delete, or rename a struct in the production code and not make a corresponding change in the test package.  Moving my test helpers into a more localized and specific location right next to the production and test code where it's used felt more cohesive to me.


## What's Left?

I still haven't answered the question of how (or if) to test package-local, production types that start in a lowercase letter.  I'm used to being able to create test code in a separate assembly but still in the same (Java) package, or with internals visible to test assemblies as in .NET.  The choice has always been there for me to let trusted test assemblies test some of the internals of a package, that I don't otherwise have any reason to expose to the outside world.

I'll have to give some more thought to why I want to make types package-visible and unit test them, anyway.
