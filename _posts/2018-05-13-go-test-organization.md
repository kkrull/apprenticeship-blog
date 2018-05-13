---
layout: post
title:  "Go Organize Your Test Code"
date:   2018-05-13 14:30:00 -0500
categories: go testing
---

Last time I wrote about [Testing in Go]({{ site.baseurl }}{% post_url 2018-04-06-one-double-per-type %}) I talked about writing helper code (in the form of mocks and stubs), but I didn't talk about _where_ to put that helper code.

I've tried out a couple of ideas by now and have some thoughts to share.


## Go's simple rules that make you look in the mirror

In deciding how to organize (test) code in Go, you're faced with a few constraints:

* Only 1 package per directory.  Sort of.  More on that in a minute.
* Naming the package with a name other than the name of the subdirectory containing the files attracts small mammals that will gnaw away at your code reviews with their disproportionately large teeth until you give in.
* Source files containing test code shall be co-mingled with the production code they target
    * **EXCEPT** there is a strange exception to the rule of "one package per directory".  In a directory `path/to/unicorn` you shall create 1 or more source files in the `unicorn` package, but you may also create any number of source files in the `unicorn_test` package and still put them in the same directory.
    * **AND** Those files in the `..._test` package shall end in `_test.go`.

When you step back and consider just _how much test code you will be writing when doing TDD_, that's actually quite a large constraint on a codebase that is generally *larger* than the codebase being tested.  

Think about that and life's other fundamental philosophical questions, next time you're doing a diaper change.

1. If the tests and the production code are in separate packages, _how do you test members (the lowercase types and functions) that are only visible inside the package?_  Should a type be internal if and only if it is an implementation detail that even a white box test can not exercise?  Or should you think like a Pythonista and make just about everything accessible (starting with an upper-case letter) to the outer world, assuming they all know what they are doing?
1. If two or more source file in the same `_test` package need the same helper code, _which source file(s) do you put it in?_  You _can_ just have one test file use the helpers defined in another test file, but how do you find anything when the package and its tests start getting large?  Should that code live in its own file?  If so, isn't is weird to have a `helper_test.go` file that doesn't actually contain any tests?
1. If two different `test` packages need the same test helper code, should `package_that_came_second_test` depend upon `package_that_came_first_test`, or should they both depend upon a separate helper package in some neutral location?

If the Golang creators are all knowing and all powerful, _did they ever foresee such a problem?_  I can imagine a counter-argument along the lines of "you should only be testing at the unit test level, and each of those tests should be testing distinct behaviors, and those distinct behaviors should not need to share any helper code".

Yeah, I get it.  But consider this:

1. A testing pyramid will - by definition - have mid- and large-sized tests that overlap in scope with small-sized unit tests.  For example, a type that generates HTTP responses (that need to be parsed) is also part of a larger system that needs to verify that the correct responses are given to a set of requests.  While the larger-scoped tests won't directly observe or interact with the parts in the middle, the parts at the beginning and the end of the workflow get shared by both types of tests.
1. A well-proportioned and completely disjoint testing pyramid doesn't get built overnight when you're doing top-down or outside-in design.  What started life as a test on one component morphs into a mid-sized integration test on multiple components each time you extract a type.  The tests don't always experience the same transformation, or the transformation of the test code from one integration test to two unit tests happens at a later time.


## Dedicated test helper package



In the early days of my Go project, the codebase was quite small and volatile.  This volatility resulted in large part 

- Separate test package, so production code doesn't accidentally use test types
- Test package mirrors production packages
- Prefix for 'mock' and 'stub'
- Separate source files for each production package
- Suitability for small- to medium-sized project. May not stand up for a big project. Ironically this was intended for small projects, but was kind of geared towards large projects by making one package for everything


## Second pass

- put packge-specific mocks and helpers in the `_suite_test.go` file.
- it's hard to make a secondary source file with another name, because it has to end in _test.go to be picked up by the test runner. But if it doesn't have any tests, well that's just weird.
- then it's visible within the package and not visible to production code
avoids warnings of untested packages
- maintains proximity / locality to where things are used
- pretty much everything so far has been a mock. haven't had many stubs or spies. Is this a sign of the early stages of the project and the use of mocks for interface discovery, or has my testing style changed?


## Takeaways

* You need a convention to help you organize test code
* The perils of using separate packages
* The advantages of putting it in the suite file 
