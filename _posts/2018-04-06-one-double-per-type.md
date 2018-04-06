---
layout: post
title:  "Thought of the Day: One Test Double per Type"
date:   2018-04-06 08:00:00 -0500
categories: go tdd
---

It's been a week of mostly TDD in Go, and part of testing is to use [test doubles][the-little-mocker] (spies, stubs, mocks) for collaborators.  I haven't experimented with any of the libraries for creating spy/stub/mock objects in Go yet, so I've rolling my own.


## The Goal

The goal is to write test code that will cause a `http.TCPServer#handleConnection` method to parse an HTTP request and do one of two things:

1. Bail with a status code and reason, if the request can't be parsed.
1. Do something else useful, if the request can be parsed.

Here's what my version of the production code looked like, just to see what I'm going for:

{% highlight go %}
func (server TCPServer) handleConnection(conn *net.TCPConn) {
	reader := bufio.NewReader(conn)
	request, parseError := server.Parser.ParseRequest(reader) //Collaborator
	if parseError != nil {
		fmt.Fprintf(conn,
			"HTTP/1.1 %d %s\r\n",
			parseError.StatusCode, parseError.Reason)
		return
	}
	...
}
{% endhighlight %}


## How I got there

There were a couple of tests to drive out the behavior for how `http.TCPServer` interacts with `http.RequestParser`.

### One test for the happy path...

The test for the happy path:

- stubs a return value for `RequestParser` on line 4, and
- makes sure it gets called correctly on line 18.

{% highlight go linenos %}
Context("when it receives a request", func() {
	BeforeEach(func(done Done) {
		parser = &mock.RequestParser{
			ReturnsRequest: &http.Request{}} //Stub
		server = &http.TCPServer{
			Host:   "localhost",
			Parser: parser}

		Expect(server.Start()).To(Succeed())
		conn, connectError = net.Dial("tcp", server.Address().String())
		Expect(connectError).NotTo(HaveOccurred())
		close(done)
	})

	It("parses the request line as everything up to the first CRLF", func(done Done) {
		writeString(conn, "GET / HTTP/1.1\r\n\r\n")
		readString(conn)
		parser.VerifyReceived([]byte("GET / HTTP/1.1\r\n")) //Verify
		close(done)
	})
})
{% endhighlight %}

Note here how the test double itself (apparently) includes the assertion.  Hence, it's being used more like a **mock**.


### Another test for the sad path...

Here the test double is used more like a **stub**.  The focus of the test is to see what `http.TCPServer` does when `RequestParser` behaves a certain way.

{% highlight go linenos %}
Context("when the request parser returns an error", func() {
	BeforeEach(func(done Done) {
		parser = &mock.RequestParser{
			ReturnsError: &http.ParseError{ //Stub
				StatusCode: 400,
				Reason: "Bad Request"}}
		server = &http.TCPServer{
			Host:   "localhost",
			Parser: parser}

		Expect(server.Start()).To(Succeed())
		conn, connectError = net.Dial("tcp", server.Address().String())
		Expect(connectError).NotTo(HaveOccurred())
		close(done)
	})

	It("responds with an HTTP status line of the status code and reason returned by the parser", func(done Done) {
		writeString(conn, "GET / HTTP/1.1\r\n\r\n")
		Expect(readString(conn))
			.To(HavePrefix("HTTP/1.1 400 Bad Request\r\n")) //Expected outcome
		close(done)
	})
})
{% endhighlight %}


### The test double

The fake collaborator has to be able to define some behavior and verify some interactions.  I put it in the `mock` package since it has at least some mock behavior.

{% highlight go linenos %}
type RequestParser struct {
	ReturnsRequest *http.Request
	ReturnsError   *http.ParseError
	received       []byte
}

func (parser *RequestParser) ParseRequest(reader *bufio.Reader) (*http.Request, *http.ParseError) {
	allButLF, _ := reader.ReadBytes(byte('\r'))
	shouldBeLF, _ := reader.ReadByte()
	parser.received = appendByte(allButLF, shouldBeLF) //Record
	return parser.ReturnsRequest, parser.ReturnsError //Stubbed behavior
}

func appendByte(allButLast []byte, last byte) []byte {
	whole := make([]byte, len(allButLast)+1)
	copy(whole, allButLast)
	whole[len(whole)-1] = last
	return whole
}

func (parser RequestParser) VerifyReceived(expected []byte) {
	Expect(parser.received).To(Equal(expected)) //Verification
}
{% endhighlight %}

In order to get the tests to run, this test double has to do a few things:

1. It has to behave in the manner prescribed by the test — as seen on line 11
1. It has to record the interaction (line 10) and verify it happened in the intended manner (line 22)


## Design choices

This test double is driven by a couple of design choices:

1. Use reasonable defaults.
1. Make as few distinct test-double types as possible.

### Reasonable Defaults

I was inspired a few years ago in the importance of de-cluttering data in tests and in organizing test code by a really good talk at CodeMash ([Patterns of Effective Test Setup][poets-talk]).  While this isn't a particularly revolutionary idea — most testing libraries I've seen do this already – I find it helps to really pay attention to these sorts of details in my tests.  

So what's the big idea?  My test doubles always return some sort of zero value (`nil`, 0, false, an empty array, etc...) unless you tell it otherwise.  This avoids cluttering up the test by defining behavior that's not required to make the production code dance.

It also doesn't take any stand on enforcing plausible behavior: it's up to the tester to understand that this object _couldn't possibly return a parsed request and an error_.


### Note to self: Make as few test doubles as possible

If I can find a way to make one test double for a type instead of two, I'm all for that lately.  My recent experiences in test-driving each new behavior for Minimax with a new Game type taught me that.  It didn't take long to have 100+ lines of various `Game` types for just a few dozen lines of actual algorithm.

Sometimes I get pedantic and get all hung up on whether to call something a stub or a mock.  If I'm stubbing in one test and mocking in another, shouldn't I have two separate types?  [sinon.js][sinon-js] would make you call separate `stub` and `mock` methods for each test, for example.  

If you follow that line of reasoning, you might end up with:

- `stub.RequestParser` that returns a stubbed request.
- `stub.ExplodingRequestParser` that returns an error, instead of a request.
- `mock.MockRequestParser` that has the extra verification method with the assertions in it.

That might make me feel more "pure" somehow, but I found it's a lot more work to maintain 2 or more test doubles for the same type than it is to just have 1.  Each stub has to do all the same stuff (just return different values in different positions), and the mock has to do all the things the stub does and add a verification method.  If I were to split it all up, there would be a lot of duplication.  

What's even worse is when there's a change to the underlying type (the type being stubbed/mocked) — then you have to update 2 or more types instead of just 1.

And what would it save me in those cases?  This combined stub/spy/mock test double does it all, and it's pretty simple.  Who doesn't like simple?


[the-little-mocker]: https://8thlight.com/blog/uncle-bob/2014/05/14/TheLittleMocker.html
[poets-talk]: https://github.com/spetryjohnson/Talk-Patterns_of_Effective_Test_Setup/blob/master/POETS%20-%20CodeMash%202014.pptx
[sinon-js]: http://sinonjs.org/releases/v4.5.0/
