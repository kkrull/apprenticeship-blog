---
layout: post
title:  "DRAFT Golang: `<Insert Test code and philosophy here>`"
date:   2018-07-14 14:15:00 -0500
categories: go
---


## Testing techniques and organization in Go

* Where to put the test code
  In the suite file
  If it's used by multiple packages, it has to go in its own test package.
* Which libraries to use
  Ginkgo
  Gomega
* Roll your own mocks
  found myself using mocks over stubs/spies mostly, but I was also doing top-down design.
* One double per type


### What have I learned since my recent writings?

* I extracted a lot more interfaces that I might normally do, because I needed to mock collaborators
* Top-down then extract led to a bit more emphasis on the middle of the testing pyramid than I might normally intend,
  but it did fall into a repeatable pattern for the last several stories.
* If I can figure out a way to test package-local types or justify making all types exported from the package, then I
  will have learned something that I can share. (See http://localhost:4000/go/testing/2018/05/23/go-test-organization.html -- What's Left?)
* What is the golang philosophy on shared test helper code, anyway?
  What do they recommend for organizing test code?


### Am I saying anything original?

I think there's likely to be some original content, here.


### Does this put me at odds with the golang community?

Folks who prefer data-driven tests will probably find this unnecessarily complex.
Folks who prefer alternative methods of ensuring quality - who just aren't into TDD - will surely find this overly complex.
If you do Go and you're into TDD, I think this is fairly reasonable advice.


### What can I improve?

* Re-state "Go’s simple rules" for package organization in a less snarky fashion.
* Actually, that whole article could use less snark. (Go Think About Your Test Code)


### What can I draw upon?

* Test Code, Go Home
* Go Think About Your Test Code
* Thought of the Day: One Test Double per Type
