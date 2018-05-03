---
layout: post
title:  "Organize Your Test Code in Go FTW"
date:   2018-05-03 13:30:00 -0500
categories: go testing
---

Last time I wrote about [Testing in Go]({{ site.baseurl }}{% post_url 2018-04-06-one-double-per-type %}) I talked about writing helper code (in the form of mocks and stubs), but I didn't talk about _where_ to put that helper code.

I've tried out a couple of ideas by now and have some thoughts to share.


## Small Projects: Separate test package

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
