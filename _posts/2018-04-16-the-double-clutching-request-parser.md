---
layout: post
title:  "The Case of The Double-Clutching RequestParser"
date:   2018-04-16 19:53:00 -0500
categories: go errors
---


Way back in my first posts on Go, I explored [some of my gut reactions]({{ site.baseurl }}{% post_url 2018-03-19-learning-go-3--error-handling %}) to error handling in Go.  Although I have plenty of experience in returning both normal output and errors from functions, I am still struggling to get comfortable with how to handle that concisely *at the call site*.

Is there something I'm missing?  Do Go developers have some secret to dealing with code where lots of errors can happen, and I just haven't learned it yet?  Could it be that the Go community just accepts this pattern and sees no reason to try anything else?


## Exhibit A

I find myself writing a lot of Go code like this:

{% highlight go linenos %}
func (parser RFC7230RequestParser) parseRequestLine(reader *bufio.Reader) (Request, Response) {
    requestLineText, err := readCRLFLine(reader)
    if err != nil {
        return nil, err
    }

    requested, err := parseRequestLine(requestLineText)
    if err != nil {
        return nil, err
    }

    if request := parser.routeRequest(requested); request != nil {
        return request, nil
    }

    return nil, &servererror.NotImplemented{Method: requested.Method}
}
{% endhighlight %}


### It's too long

For a workflow consisting of 3 steps, it feels like I'm constantly stopping what I'm doing to check for an error before carrying on again.

Consider the happy path through this code: there's a simple pipeline that converts the data from line 2–through an intermediate type on line 7–to the final return type on line 12.  So...shouldn't this function be about 4 lines of code?

There's also duplication of the error check (lines 3 and 8) and of the error-return (lines 4 and 9).


### Scope

Another smell with this function has to do with the scope of the variables holding the errors.  This leads me either to:

* use distinct names for each step's error, which means old errors are kept in scope way longer than I need them.
* reuse the same variable name and ponder why a line containing `:=` doesn't complain that the variable exists already.

Ponder that for a moment in light of the fact single-variable form of `:=`.

{% highlight go %}
answer := 42
answer := 43
fmt.Printf("Answer to the universe: %d\n", answer)
{% endhighlight %}

This code doesn't compile.  So why should the case with 1 new variable and 1 old one be ok?


### Confusing Control Flow

Finally, the procedure written in the function is just plain confusing.

If it were a fail-fast procedure, I would expect to see 1 or more guard clauses punctuated by a successful return at the end.  A successful return outside of the curly braces would tell me at a glance that there's weird stuff going on inside the curly braces, and life as normal outside of them.

If it were–for lack of a better word–an optimistic procedure, I would expect the opposite: a clause to return something successful followed by 1 or more edge cases that return errors.

However in this case, it's the 3rd clause that matters.  The happy path returns from some place deep within a sandwich of edge cases. 


## Is there a way out of this?

When I get a chance, there are a few ideas I could explore:

* The first two error cases might be rolled into the same function.  This increases the call depth but reduces the number of branches from 4 to 3.  I'm not sure if that's a trade-off I'm willing to make.
* Maybe I could turn each step into its own function, and call them in a heavily nested fashion.  Something like: 

> request, err := parser.RouteRequest(parseRequestLine(readCRLFLine(reader))

I'm not even sure if that would work, and it's hard to imagine it being very easy to read.

* Write a stripped-down `Promise` or `Either` type, that's specific to the domain.  It's hard to imagine this being small, and it would be next to impossible to reuse due to the lack of templating / generics.
* Maybe some sort of state-machine / Continuation Passing Style?  There would be a function for each state, that calls the next function, and so on...

Or maybe like other annoyances with Go, I'm just supposed to accept it and move on?  Could the lack of options be useful constraint that propels me to write more features?
