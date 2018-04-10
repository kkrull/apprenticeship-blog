---
layout: post
title:  "Interface Segregation in Go"
date:   2018-04-07 13:44:00 -0500
categories: apprenticeship go
---

As my HTTP server gets a few more packages, I've been thinking about interfaces in Go a lot today.

Go has the neat feature of being able to define and implement an interface *without any formal declaration on the implementing side*.  This is unlike Java and C# (where some sort of `implements` clause is required on the implementor side) and is more like the duck-typing used in Ruby.  However–unlike Ruby–there's a compiler to check that a type expected to implement an interface has all the methods in that interface, rather than just a runtime check in Ruby.


## Example

Take this code for example, where a `TCPServer` needs a type with a `ParseRequest` method:

{% highlight go linenos %}
//server.go
type TCPServer struct {
	Parser RequestParser
}

func (server TCPServer) handleConnection(conn *net.TCPConn) {
	reader := bufio.NewReader(conn)
	request, parseError := server.Parser.ParseRequest(reader)
	...
}

type RequestParser interface {
	ParseRequest(reader *bufio.Reader) (Request, *ParseError)
}
{% endhighlight %}


Note how the implementing type — `RFC7230RequestParser` — need not declare any relationship to the interface:

{% highlight go linenos %}
//request.go
import (
	"bufio"
	"strings"
)

type RFC7230RequestParser struct{}

func (parser RFC7230RequestParser) ParseRequest(reader *bufio.Reader) (Request, *ParseError) {
	request, err := parseRequestLine(reader)
	...
}
{% endhighlight %}

See how there is no mention of `RequestParser`, or even an import of `server`?


## Consequences on Dependencies

What does this mean, and why might the language creators have made this choice when they designed the language?

First of all, there is an impact on modules and dependencies.  In a language like Java, the implementing class would need to have the interface in its classpath so that it can import it and declare the implementation.  If the implementation and the interface are in separate jars:

- the implementation jar depends upon the jar containing the interface, to compile and run.
- circular dependencies aren't allowed, so it's impossible for the jar containing the interface to have any dependencies back to the jar containing the implementation.

Eliminating circular dependencies and being careful about how you organize your code into modules (jars) is a good thing, but it can easily turn into a mess on a large, undisciplined project by:

1. **Leaving every single type in one module:** This can lead to code with tangled dependencies and type hierarchies.  Suspicious, two-way relationships among the packages in the same jar are hidden from view, until you try to untangle it.
1. **Splitting up the types in an unstable project among arbitrarily defined modules**: This leads to moving classes across packages, jars, and even code repositories causing much weeping and gnashing of teeth for existing clients.  It usually happens when you're in the middle of writing something that depends upon the classes somebody else just moved, or when you tried to be nice and upgrade your dependencies.

The traditional approach is to make a stable `widget-api.jar` that `widget-implementation.jar` (and possibly a separate `widget-client.jar`) depends upon.  As is often said, any problem in software can be solved by adding a layer of indirection.

However, the devil is often in the practical details of when to move what types into what jars.  It's rare that I have seen this process go smoothly, in all but the most disciplined organizations.


## Questions in Go

Go simplifies the problem by reducing the number of dependencies.  The implementation doesn't have to know a thing about the interface it's satisfying.

Along with the greater flexibility comes a few more questions; namely — **who should use which type?**


### Client code: Use the interface

Which type should clients use — the interface (`type I interface{}`) or the implementation (`type X struct {}`)?  This one seems pretty simple — the client should deal in terms of the interface.  Why else would the interface exist, if not for use here?

It also makes sense to put the `interface{}` right in the same source file (or at least the same package) as the client.


### Implementation code

Should code in the implementor's package refer to the implementing type, or the interface type?

Most of the time the implementation won't have a choice and will have to
 talk about itself, *but what about constructor functions*?  Should `func NewServer` return the implementation type `type http.TCPServer struct {}` or interface type `type Server interface{}`?  
 
This one is a little less clear to me, but for now I am having **constructors return the implementation type**.  On one hand, declaring that a constructor returns the interface type would retain some flexibility in being able to return a different type, later, without affecting existing clients.

On the other hand, doing that would lead to a circular dependency between two packages.  Besides, it seems reasonable that a constructor in the `http` package should return its own type (`http.TCPServer`) and not somebody else's.


### Tests on the implementation

What about tests on the implementation?  Should the test use the interface or concrete type itself to drive the desired behavior?

I almost always test through the interface, so I can be sure to avoid getting around a tedious interface by going straight to the implementing type.  I never gave this much thought before, but that does mean my tests have dependencies on two modules — the one with the interface, and the one with the implementation.

That doesn't seem so bad.  I just hadn't thought about it before.  And more important than the number of dependencies, they are all pointed squarely from the test to the interface/implementation packages.


## Conclusion

So far, Go seems to offer some pretty good separation of dependencies while still offering many of the advantages of a type checker.

The only major question I have left right now is how big the risk is of the compiler accepting a type that satisfies an interface in signature, but not in contract?  Could that become a frequent or head-scratching source of semantic errors in large codebases with many dependencies?

Time will tell.
