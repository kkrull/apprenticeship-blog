---
layout: post
title:  "Week 4 in Review"
date:   2018-04-07 13:44:00 -0500
categories: apprenticeship
---

Week 4 of my apprenticeship has gone by in the blink of an eye.  So quickly, in fact, that I think it would help me to spend some time making sense of it all.


## Background Tasks

As with any apprenticeship, I have a few recurring responsibilities try to work on a little bit every day.  I'll recap where I am on these topics, after the last week.


### Reading

At the end of the Week 3, I fell asleep somewhere in the About the Authors section of the [DevOps Handbook][devops-handbook].  I have since learned that it's best read while laying on my side, as it makes an unpleasant **_THUD_** if I happen to be holding it above my head when I fall asleep.  *That doesn't mean the book is boring, it just means I was really tired last week.*

As Week 4 draws to an end, I have enjoyed learning a bit about the 3 Ways and their foundations in recent improvements in manufacturing processes.  There were a lot more references to Toyota's manufacturing processes than I expected, and it's comforting to see a bit of formalism applied to the field (i.e., best practices are better in measurable ways rather relying on anecdotes and star power to convince people).  This also isn't the first place I've heard references to [The Goal: A Process of Ongoing Improvement][the-goal].  I'll have to add that to my reading list, for later.

I'd like to read the book in more detail at a later date, but so far a quick perusal of the introduction and Information Security chapters is all I have had time for right now.  I'm looking forward to my next book.

[devops-handbook]: https://www.amazon.com/DevOps-Handbook-World-Class-Reliability-Organizations/dp/B0767N1MM2/ref=sr_1_3?ie=UTF8&qid=1523128490&sr=8-3&keywords=devops+handbook
[the-goal]: https://www.amazon.com/Goal-Process-Ongoing-Improvement/dp/0884270610/ref=pd_lpo_sbs_14_img_0?_encoding=UTF8&psc=1&refRID=RX4Y07BDC9PS4WY6Y1S6


### Blogging

It took a couple weeks to get comfortable with Jekyll, but so far it has done a good job at everything I wanted.  With the technical aspects out of the way, the next part to refine is my writing process.

I'm working on writing smaller, more focused articles this week, and so far it is helping.  One of the Week 3 articles involved a lot of soul-searching and was a whopping 1,349 words.  I can get a decent idea of my writing rate by comparing the time stamps of the last commit and of the article itself, which puts me at about 180 words / hour.  

My experiences with the [Personal Software Process][personal-software-process] have influenced me to measure the size and rate of things like this, and experience tells me that the biggest gains are to be had by writing less, instead of trying to write faster.

[personal-software-process]: https://en.wikipedia.org/wiki/Personal_software_process


### Minimax

I haven't always done this every day, like I was hoping to.  Earlier attempts in prior weeks have taken a while, and I was often getting stuck at the point of introducing recurrence / evaluation of multiple plays for the same game state.

By the end of Week 4, I had made it all the way through the kata (albeit, as a two-part process).  A pairing session with my mentors later in the week was really helpful in identifying friction points that were slowing me down, and I'm looking forward to trying some of the suggested simplifications next week.

After a week, what seemed a nearly insurmountable challenge now seems a bit more possible.  I won't really believe it until I have done it, but the path to completing it in a short enough time to demonstrate is now a bit clearer.


## `cob_spec` – Test-Drive an HTTP server

Progress has been a lot slower than I would have liked for myself, and I'm trying to keep it all in perspective.  It has taken a while to get comfortable with the new programming paradigms, conventions, and quirks I'm experiencing by learning Go.  It also takes some investment on any new project to get a walking skeleton that has a structure I'm happy with and to be able to test-drive its behavior.

I don't know how long it takes for others to do this, but to me it sometimes feels like I'm taking *forever*.

However when I look back, I am reasonably satisfied with my progress.  By the end of Week 3, I only had a walking skeleton that passed 1 of the tests in `cob_spec`.  There was technical debt in missing features (lack of a graceful way to stop the server) and in missing test code coverage.  My initial design of using a blocking `http.Server#Listen` method with a bunch of client-supplied channels was starting to feel more and more brittle, as I became more and more aware of the risk of clients mis-constructing a channel or blocking on the wrong channel at the wrong time.

There have been some useful gains in Week 4 when I stop to think about it:

- I'm up to 2 specs passing on `cob_spec`.
- The outstanding technical debt has been addressed, without taking on a lot more.  The test coverage / techniques are there now, and the only new technical debt this week is in DoS-proofing the `http.RequestParser`.
- It's parsing the first line in the HTTP request.
- I'm really appreciating the `goroutine` and `chan` primitives in Go.  Despite some of the other quirks that can be annoying, it really seems like they got the fundamentals right on creating and synchronizing asynchronous processes.

There were also a lot of investments that I hope will pay off in Week 5 and beyond:

- I'm more comfortable with testing asynchronous processes in Go now.
- Using constructor functions is helpful in making sure buffered channels (for synchronization) get created correctly.
- I'm a bit wiser about looking out for rogue, test-muddling instances of my server now.  Old debugging and testing sessions apparently left in their wake a dozen or so `go build` and `gohttp` processes that were still accepting TCP connections.
- I can run FitNesse from the command line now, instead of firing up a server, browsing to a web page, and clicking a button.
- I'm slightly less lost with the 4+ packages that contain a `Reader`-like interface, now.  That's `io.Reader`, `bytes.Buffer`, `bufio.Reader` and one more I'm not remembering right now, for folks playing at home.


## What's the moral of the story?

Looking back, Week 4 has been fairly productive.  Sometimes that's hard to see in the day to day flurry of activity.

I have realized a couple of things this week:

1. **A little bit of internal state goes a long way:**  Years of functional programming taught me the perils of having mutable state, which led me to my first design of using a client-supplied channel to trigger shutting down the server (`http.Server#Listen(chan bool)`).  This got fairly complicated, however, and I found that changing it to have separate `http.Server#Start` and `http.Server#Shutdown` methods allowed me to manage the `Listener` inside the server and make the client a lot simpler.
1. "I'll write one test by 9PM and go to bed...or I won't and I'll go to bed anyway."  I was wasting a lot of time and energy in Week 3 by trying to push through exhaustion and get something done, only to find the extra time and effort were not paying off.  When I truly accepted the possibility of failure, I was able to relax, write one test, and call it good for the night.


## Next Week

I'm sure next week will be at least as busy as those before it, but I'm hopeful that I'm getting far enough along in the learning and design curves that I can get a lot further on my HTTP server and on my Minimax kata.
