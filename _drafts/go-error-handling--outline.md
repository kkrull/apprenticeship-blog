---
layout: post
title:  "Error Handling Techniques in Go"
date:   2018-07-15 15:49:00 -0500
categories: go errors
---

## Takeaways

A review of error handling techniques in Go that is more comprehensive and less polarizing than most articles out there.

* Understand a variety of techniques that are available, for error handling in Go.
* Understand the pros and cons of each technique.
* More acceptance and moving on with how Go handles errors.


## Intro

* Error handling techniques are a bit different
* Errors are a value -- cite the Rob Pike article
* that means you can handle those values any old way


My example code:

* 2 failures, one success, one fallback


## first approach - refactor your code

* I could have refactored it to
** one parse function that reads the line and parses it, handling the two error cases
** one handler function that uses the implemented handler or falls back to not implemented
* at that point, I'm back to a simple if/else and that's idomatic and non-crazy

## Try/catch (not going to happen)

Try/catch is commonly desired:

* A lot of people want to see try/catch
* Worth noting that other techniques are available
* Breaks referential integrity (research Scala/Odersky)
* or, cite the article pointing out the hard-to-distinguish correct vs. incorrect code with try/catch


## Plain old if statements

* Errors are a value.  You handle these values like you would handle any other value.
* Start with my original request parsing example


## Either types (domain specific)

* What the example would look like


## Continuation Passing Style

* What the example would look like, in CPS
* Is it practical?  


## Summary

