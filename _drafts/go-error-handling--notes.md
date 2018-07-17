---
layout: post
title:  "DRAFT Golang: `<Insert Epiphany Here>`"
date:   2018-07-14 14:15:00 -0500
categories: go
---


## Takeaways

A review of error handling techniques in Go that is more comprehensive and less polarizing than most articles out there.

* Understand a variety of techniques that are available, for error handling in Go.
* Understand the pros and cons of each technique.
* More acceptance and moving on with how Go handles errors.


## Go error handling

* What would wrapper functions look like?  doThirdThing(neededFor3, doSecondThing(neededFor2, doFirstThing(neededFor1)))
  Each function would have to have a guard clause that returns the error right away, if there is already an error
* wrapper function to unwrap a successful value or go to an error handler?
* Try the Pike-monadic form of the error, where a helper function/type is deciding whether to NOP due to an error or do the thing
* True CPS - can the next success/old error/new error function be passed in as a second parameter?


### What have I learned since my recent writings?

* I can use a state machine when I really want to, but it's usually more verbose than it's worth.
* I can just live with functions containing multiple points of failure being longer than I prefer.
* Making my own custom error types really isn't that bad
* Domain-specific monads like Either/Optional/Promise are too much work to be worth it, in this example.


### Back to whatever I was doing before


#### Go blog

https://blog.golang.org/error-handling-and-go
And another one here by Rob Pike: https://blog.golang.org/errors-are-values

A function closure may help

```go
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
// and so on
if err != nil {
    return err
}
```


### Earlier Writing

* The Case of The Double-Clutching RequestParser
* State Machines to the Rescue?
* Getting into Go Part 3: Error Handling
