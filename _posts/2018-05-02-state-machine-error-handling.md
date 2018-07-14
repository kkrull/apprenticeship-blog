---
layout: post
title:  "State Machines to the Rescue?"
date:   2018-05-02 17:50:00 -0500
categories: go errors
---

Last time I wrote about [error handling in Go]({{ site.baseurl }}{% post_url 2018-04-16-the-double-clutching-request-parser %}), I mentioned an idea of using something akin to Continuation Passing Style to make error handling of a multi-step workflow more readable.

Today I thought I'd give that a try.


## The Problem

To re-state the problem I'm trying to address, I didn't much care for the stop-and-go style of checking for errors after every step.

{% highlight go linenos %}
func (parser RFC7230RequestParser) parseRequestLine(reader *bufio.Reader) (Request, Response) {
	  //One source of error
    requestLineText, err := readCRLFLine(reader)
    if err != nil {
        return nil, err
    }

		//Another source of error
    requested, err := parseRequestLine(requestLineText) 
    if err != nil {
        return nil, err
    }
    
		//It worked
    if request := parser.routeRequest(requested); request != nil {
        return request, nil 
    }

		//Another error
    return nil, &servererror.NotImplemented{Method: requested.Method} 
}
{% endhighlight %}


## The Inpiration: Functional Programming

It's a common practice in functional programming to make a state machine for complex workflows.  The big idea is to **make a function for each state**.  Some of those functions represent logical steps in a workflow, while others represent error states.  You make a transition from one state to the other **simply by calling the function** that represents the desired state.

It might look something like this, in Scheme:

{% highlight scheme %}

(define (master-plan coyote roadrunner)
  (define (set-trap acme-object) ; step 1
    (if (and (reliable? acme-object)
             (not (eq? (name coyote) "Wile E. Coyote")))
      (capture roadrunner) ; go to step 2
      (fall-off-cliff coyote))) ; transition to an error state
    
  (define (capture roadrunner) ; step 2
    (cond
      [(escapes? roadrunner) (go-hungry)]
      [else (dine-on roadrunner)]))
      
  (define (dine-on roadrunner) `#(ok pat-tummy)) ; victory!
  (define (fall-off-cliff who) `#(POOF ,who)) ; one kind of error
  (define (go-hungry) `#(rumbling-tummy ,coyote)) ; another error
  
  (set-trap 'anvil)) ; start the state machine
{% endhighlight %}

If all goes according to plan, `(master-plan coyote road-runner)` will return `#(ok pat-tummy)`.  Anything else is some sort of error, depending upon what ill fate befell the coyote.


## Application to Go

Although that example is a bit contrived, I have seen real examples of this pattern being used in mission-critical software.  It takes a bit of time to get accustomed to reading Scheme, but the examples I've seen have been sufficiently expressive to help the developer identify flaws in the design and drive out some bugs.  That's one sign of an appropriate application of a useful design pattern.

Will the same work in Go?  Let's see what it looks like:

{% highlight go %}
func Parse(reader *bufio.Reader) (ok *requestMessage, err Response) {
    stateMachine := &parser{reader: reader}
    return stateMachine.ReadingRequestLine() //Start the machine
}

//A state machine that parses an HTTP request
type parser struct {
    reader *bufio.Reader
}

//Initial state
func (parser *parser) ReadingRequestLine() (ok *requestMessage, badRequest Response) {
    requestLine, err := parser.readCRLFLine()
    if err != nil { //First kind of error
        return nil, err //Transition to error state
    }

    return parser.parsingRequestLine(requestLine) //Go to state 2
}

//OK State 2
func (parser *parser) parsingRequestLine(requestLine string) (ok *requestMessage, badRequest Response) {
    fields := strings.Split(requestLine, " ")
    if len(fields) != 3 { //Second kind of error
        return nil, &clienterror.BadRequest{DisplayText: "incorrectly formatted or missing request-line"}
    }

    return parser.parsingTarget(fields[0], fields[1]) //Go to state 3
}

//OK State 3
func (parser *parser) parsingTarget(method, target string) (ok *requestMessage, badRequest Response) {
    path, query, _ := splitTarget(target)
    requested := &requestMessage{
        method: method,
        target: target,
        path:   path,
    }

    return parser.parsingQueryString(requested, query)
}

//OK State 4
func (parser *parser) parsingQueryString(requested *requestMessage, rawQuery string) (ok *requestMessage, badRequest Response) {
    if len(rawQuery) == 0 {
        return parser.readingHeaders(requested)
    }

    stringParameters := strings.Split(rawQuery, "&")
    for _, stringParameter := range stringParameters {
        nameValueFields := strings.Split(stringParameter, "=")
        if len(nameValueFields) == 1 {
            requested.AddQueryFlag(nameValueFields[0])
        } else {
            decodedValue, _ := PercentDecode(nameValueFields[1])
            requested.AddQueryParameter(nameValueFields[0], decodedValue)
        }
    }

    return parser.readingHeaders(requested)
}

//OK State 5
func (parser *parser) readingHeaders(requested *requestMessage) (ok *requestMessage, badRequest Response) {
    isBlankLineBetweenHeadersAndBody := func(line string) bool { return line == "" }

    for {
        line, err := parser.readCRLFLine()
        if err != nil {
            return nil, err
        } else if isBlankLineBetweenHeadersAndBody(line) {
            return requested, nil
        }
    }
}

func (parser *parser) readCRLFLine() (line string, badRequest Response) {
    maybeEndsInCR, _ := parser.reader.ReadString('\r')
    if len(maybeEndsInCR) == 0 {
        return "", &clienterror.BadRequest{DisplayText: "end of input before terminating CRLF"}
    } else if !strings.HasSuffix(maybeEndsInCR, "\r") {
        return "", &clienterror.BadRequest{DisplayText: "line in request header not ending in CRLF"}
    }

    nextCharacter, _ := parser.reader.ReadByte()
    if nextCharacter != '\n' {
        return "", &clienterror.BadRequest{DisplayText: "message header line does not end in LF"}
    }

    trimmed := strings.TrimSuffix(maybeEndsInCR, "\r")
    return trimmed, nil
}
{% endhighlight %}

The approach is analogous to the sequence of function calls in the Scheme example.  Anything that is complex or that can produce an error gets its own state, and hence its own function.


## Thoughts

Is this a good, generalized approach for Go?  I'm not so sure, but here are some points that made it appealing to me in this case:

* The workflow has *several* sources of error.  If there are a lot of steps that are more likely to work, I don't know that this would be worth the effort.
* The workflow is fairly complicated.  I already needed to extract methods to make it more readable anyway, and having a name for each state is helpful in reasoning about the problem.

The astute reader will point out that I have just as many guard clauses (the `if err != nil` checks) as I would have had without the state machine.  That's true, but at least this way they're not all in a long sequence in a large function.

Your mileage may vary, but this technique may be helpful from time to time.