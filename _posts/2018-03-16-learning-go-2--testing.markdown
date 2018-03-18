---
layout: post
title:  "Getting into Go Part 2: Testing"
date:   2018-03-16 12:00:09 -0500
categories: go testing
---

In the last post, I was just learning how to write and build a simple program in Go.  Now it's time to see how to test some code.


## Why All the Fuss?

Testing is a big deal to me — big enough for me to write [my own open source library][javaspec] for it.  There are plenty of reasons why it's important to invest in testing early in the life of a project, and much has been written about the topic.  I'm just going to list a few of the more important reasons to me:

- It helps me minimize the number of bugs I have to debug at any given time.  If a test fails when I expected it to pass, the problem is likely to have occurred in one of the tiny changes made within the past few minutes.
- I can make incremental, objectively verifiable progress.  A bunch of untested code awaiting big-bang integration may **completely fail** to meet any requirements, whereas incomplete, test-driven code tends to at least do *something* useful.
- If I'm interrupted, it's easier to remember what I needed to do next when I get back to it.  A pending test with real, unmet expectations jogs my memory faster than a `TODO` comment or Post-it notes.
- The act of explaining what the code should do makes me think about the problem more clearly, just as writing about any idea helps me think it through.  I take care to name my tests well, which means I'm using *two* languages to describe the production code: the English phrase describing the intended behavior and the programming language used to verify it.

It also happens to be useful when learning a new language.  Think about how many untested assumptions you have when you're using a language for the first time and how vital it is to have prompt feedback on your code.


## Using the built-in "testing" package

There is a built-in package — [`testing`][testing-package] — that ships with Go, so that seems like a good place to start.  It's pretty similar to what you get with [JUnit 3][junit3] and Maven when you run [`mvn test`][maven-test].

After a few red-green-refactor cycles on a "hello world" test, I wound up with this xUnit-style test:

{% highlight go %}
package greet

import "testing"

func TestGreet(t *testing.T) {
  expected := "Hello World!"
  actual := Greet()
  if actual != expected {
    t.Errorf("Expected <%v> but was <%v>", expected, actual)
  }
}
{% endhighlight %}

Running it is a simple matter of `go test` from the package source directory:

{% highlight bash %}
$ go test
PASS
ok      github.com/kkrull/gosandbox     0.005s
{% endhighlight %}


## It works, but...

While I'm impressed that there is an out-of-the-box solution for testing, this still leaves something to be desired.


### It's hard to describe behavior

A `testing` test is a function that starts with `Test`, so you're stuck with as much as you can manage in CamelCase.  Although `TestGreetGivenNoNameSaysHelloWorld` would be a better description of the intended behavior, I usually see people using this style of testing with names like `TestGreet1` and `TestGreet2` instead.

**This makes it harder for anyone reading the tests to tell how one test differs from the others.**


### Invoking the tests is distracting

Switching windows from the editor to a command prompt to run `go test` is annoying to me.  When I'm doing TDD, I try to wait for the test results and read them before making the next change.  I can easily make hundreds of these edits in a single sitting.  **I want that feedback to come quickly and effortlessly.**

What slows down the process of responding to what the tests are telling me?  Switching focus to another window and moving my fingers off home row so I can use the arrow keys to run the test command again.  It may not seem like a big deal, but the reduced cognitive load is quite liberating.

[Guard][guard] is a neat tool that can help enormously with this problem.  

These gems...

{% highlight ruby %}
group :development do
  gem 'guard'
  gem 'guard-shell'
end
{% endhighlight %}

...and this `Guardfile`...

{% highlight ruby %}
guard :shell do
  watch /(.*).go/ do |m|
    timestamp = Time.now.strftime('%H:%M:%S')
    "#{timestamp} #{m[0]} changed"
    `go test`
  end
end
{% endhighlight %}

...automate running `go test` whenever you save a Go source file.  Note that this uses [guard-shell] instead of [guard-go], as the latter appears to be so badly outdated that it no longer runs.

It's a significant improvement over running tests manually, but there is a noticeable lag from saving the source file to `guard` starting up a new test process and showing you the results.  It also seems odd to require a Go developer to install Ruby.

There are still some places where this could be better, but it's an improvement.


### Manual assertions are extra work and lack expressive names

See the part in the test where I had to write the comparison, describe the discrepancy between the actual and expected values, and explicitly fail the test if the values don't match?  That's a lot of work that will be done over and over again.  It could do with some de-duplication and some naming.  In other words, an assertion library would really come in handy.

I found [Gomega][gomega] after a brief search, which offers something much easier to read:

{% highlight go %}
import . "github.com/onsi/gomega"
...
Expect(Greet()).To(Equal("Hello World!"))
{% endhighlight %}

The `import .` line above allows you to use `Expect` and the other functions from Gomega without the `gomega.` prefix.  Some other reading suggests that there is some disagreement in the community over whether this is a good idea, but I think having expressive test code is more important than the occasional risk of two packages exporting the same functions.  Besides, it's not hard to [use another identifier][import-syntax] to refer to two packages that export the same-named function.


## Crossroads

Is it common for other Go developers to wrestle with the lack of naming and assertions?  It's worth considering this question for a few minutes, since I am new to the language.  Maybe they have a different, more idiomatic way of writing their tests that works better for this language.

It turns out that the Go authors do have an [opinion on the matter][table-driven-test], recommending using data tables as inputs to a relatively small number of test functions.  That way, you get lots of test cases without declaring so many functions.  They cite [the tests in the `fmt` package][fmt-test-example] as an example of a good, table-driven test.

While it does economize on the amount of syntax per test case, I can't help but notice that:

- The test source file is 1,843 lines long, which is well over 3 times as large as the code it's testing.  It looks like they have put quite a lot of effort into making this individual test file as small as possible, but in my opinion it's quite difficult to tell where the test functions are or what they're testing.
- There are hundreds of lines of table data, which are allocated to about 20 tests.
- There are comments next to the data for tests cases that are rare, surprising, or non-intuitive.

I don't doubt that there are a lot of edge- and corner-cases to consider in this package, but I can't help thinking that this test would be extraordinarily difficult to maintain.  Are there separate documents describing the intended behavior and linking requirements to test cases?  Are test sources, production sources, requirements and specifications all kept perfectly in sync when there are changes?

I have to conclude for now that I'm not satisfied with the idiomatic approach.  I don't necessarily expect that a direct port of rspec or mocha to Go would be the best way to test Go, but I do think it's worth investigating what other, more experienced Go developers have to offer.


## Other testing libraries

Thankfully, there are a [few other testing libraries][testing-package-question] to try:

- [gospec][gospec] is not maintained anymore and does not appear to be compatible with modern Go, so that's out.
- [GoConvey][goconvey] has an impressive set of features, and a snazzy, web-based test runner.  However, I'd like to retain my ability to work on the command-line, at least for now.
- That leaves [Ginkgo][ginkgo] which — aside from my tendency to misspell it — looks like it has the features I want.  I'll gladly use strings and functions/lambdas any day of the week to structure my tests.


### Ginkgo

There are a couple of steps to get up and running, once you have fetched the sources with `go get`:

1. `ginkgo bootstrap` generates a `testing`-compatible runner for your tests, so you can still use `go test` to run your specs.  For me, this winds up in `greet_suite_test.go`.
1. `ginkgo generate <test name>` generates a skeleton file in which to write your specs, which becomes `greet_test.go` in my example.
1. Fire up `ginkgo watch` in your source directory to run specs whenever you save a source file.

At last!  I finally have a test cycle I'm satisfied with.  I was able to get to a fully working implementation of my toy `Greet` function.

{% highlight go %}
//greet_suite_test.go
package greet_test

import (
  "testing"

  . "github.com/onsi/ginkgo"
  . "github.com/onsi/gomega"
)   
  
func TestGreet(t *testing.T) { 
  RegisterFailHandler(Fail)
  RunSpecs(t, "Greet Suite")
}
{% endhighlight %}

{% highlight go %}
//greet_test.go
package greet_test

import (
  . "github.com/onsi/ginkgo"
  . "github.com/onsi/gomega"
  
  . "github.com/kkrull/gosandbox/greet"
)   
  
var _ = Describe("Greet", func() {
  It("Greets the world when no names are given", func() {
    Expect(Greet()).To(Equal("Hello World!"))
  })

  It("Greets a single person by their name", func() {
    Expect(Greet("George")).To(Equal("Hello George!"))
  })

  It("Greets two people by name, joining words with 'and'", func() {
    Expect(Greet("George", "Judy")).To(Equal("Hello George and Judy!"))
  })

  It("Greets 3 or more people by a comma-separated list of their names", func() {
    Expect(Greet("George", "Judy", "Astro")).To(Equal("Hello George, Judy, and Astro!"))
  })
})
{% endhighlight %}

{% highlight go %}
//greet.go
package greet

import "strings"
  
func Greet(who ...string) string {
  switch {
  case who == nil:
    return "Hello World!"
  case len(who) == 1:
    return "Hello " + who[0] + "!"
  case len(who) == 2:
    return "Hello " + strings.Join(who, " and ") + "!"
  default:
    allButLast := strings.Join(who[0:len(who)-1], ", ")
    return "Hello " + allButLast + ", and " + who[len(who)-1] + "!"
  } 
}
{% endhighlight %}

The test runner's output is fairly concise when tests are passing, and it's reasonably helpful in the event of a failure.  Here's an example:

{% highlight bash %}
$ ginkgo watch
Identified 1 test suite.  Locating dependencies to a depth of 1 (this may take a while)...
Watching 1 suite:
  . [1 dependency]
Running Suite: Greet Suite
==========================
Random Seed: 1521411392
Will run 4 of 4 specs

••••
Ran 4 of 4 Specs in 0.000 seconds
SUCCESS! -- 4 Passed | 0 Failed | 0 Pending | 0 Skipped PASS
...make a breaking change...
•! Panic [0.000 seconds]                                                                                                                                         
Greet                                                                                                                                                            
/Users/kkrull/go/src/github.com/kkrull/gosandbox/greet/greet_test.go:10                                                                                          
  Greets the world when no names are given [It]                                                                                                                  
  /Users/kkrull/go/src/github.com/kkrull/gosandbox/greet/greet_test.go:11                                                                                        
                                                                                                                                                                 
  Test Panicked                                                                                                                                                  
  runtime error: slice bounds out of range
{% endhighlight %}


## Conclusion

Now that I have invested in my ability to test and get immediate feedback on my code, I think I'm ready to write some more serious code in Go.

Was it worth it to spend this time investigating testing libraries?  I think so.  I'd much rather spend a day figuring this out up-front instead of during crunch time when things aren't working.  Few experiences are more frustrating than realizing that I'm doing something in a clumsy way, knowing that there *has* to be something better, and also knowing that there is no time left to try anything else right now.


[fmt-test-example]: https://github.com/golang/go/blob/master/src/fmt/fmt_test.go
[ginkgo]: http://onsi.github.io/ginkgo
[goconvey]: http://goconvey.co
[gomega]: http://onsi.github.io/gomega
[gospec]: https://github.com/orfjackal/gospec
[guard]: http://guardgem.org
[guard-go]: https://github.com/victorcoder/guard-go
[guard-shell]: https://github.com/guard/guard-shell
[import-syntax]: https://golang.org/ref/spec#Import_declarations
[javaspec]: https://javaspec.info
[junit3]: http://junit.sourceforge.net/junit3.8.1/javadoc/junit/framework/TestCase.html
[maven-test]: https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Setting_Up_Your_Project_to_Use_the_Build_Lifecycle
[table-driven-test]: https://github.com/golang/go/wiki/TableDrivenTests
[testing-package]: https://golang.org/pkg/testing
[testing-package-question]: https://stackoverflow.com/questions/16869957/rails-rspec-like-testing-in-google-go

