---
layout: post
title:  "Test-Driven Development: More Discipline than Dogma? (Part 2)"
date:   2018-03-28 07:31:00 -0500
categories: tdd philosophy
---

This is part of an [on-going series]({{ site.baseurl }}{% post_url 2018-03-23-tdd-is-not-dogmatic %}) discussing the Philosophy of Test-Driven Development.


## Why it seems dogmatic

It's understandable why Test-Driven Development may seem dogmatic to some.  It's so understandable that I'm even questioning the course I set out to take with this series (hence the question mark in the title).  I've been practicing TDD for several years, and there are times when I wonder if I'm taking it a bit too far.

For starters, it does satisfy *part* of the definition we began with earlier: TDD is indeed a set of principles.

There are some other areas where the water gets a little muddy, at least when viewed from an informal perspective of what one tends to associate with being dogmatic.


### ...wait for it...

In some respects, Test-Driven Development involves a fair amount of **delayed gratification**.

You have to tell yourself "no" a lot, every time you think about something you want to write, but don't have a test for it yet.  You have to be the [kid distracting yourself from the cookie][delayed-gratification-experiment], so you can be rewarded with it later.  

And in the case of TDD, it's not clear if the adults in the room will be sadistic enough to alter the deal and give you bowl of brussel sprouts instead, as your "reward" for all that waiting.

So if your vision of the finish line is *completing a draft of the production code* - and you assume that the production code resulting from a TDD session is exactly what you would have written anyway - the measured pace of writing no more code than you need to pass the tests may seem very slow, indeed.  It may defy your sense of reason for somebody to spend the extra time doing TDD to reach a certain point in the development process, when you assume that writing code is a linear sequence of knowing what you want to write, starting at the top, and working your way to the bottom.

*Sutained action in the face of contradictory facts?  Seems to me somebody's being a bit dogmatic!*


### There's no ground truth

*But wait!  You have to compare the time spent over the entire software development cycle*. You can't get away with only comparing the time taken to write the production code, **you have to compare the time spent on debugging and integration too**.

If there were a way to make an objective, universally agreed upon comparison, there probably wouldn't be much left to debate.  Unfortunately, nobody seems to agree on the definition of a bug, which part of the process is to blame, or what life would have been like if we did it the other way.

It's pretty straightforward to measure the *cost* (the effort) of doing TDD: it's the amount of time you spend test-driving the code, until the code is complete.  You just have to count how long you spent on each TDD session.

Measuring the effort of a test-last or test-never process is also fairly straightforward: it's the time spent on the coding plus any time spent on testing.  You should probably include the time spent on this feature by anyone in a QA or tester role, as well.

If we lived in a world without bugs, we would have enough information to make a reasonably objective comparison of the two approaches.  However, *what are the consequences of these two ways of writing code?* What costs and benefits are incurred later in the life of a project, due to the way things are done now?

It's really hard to say.  TDD can save you time in the long run, for reasons such as these:

- If TDD code has fewer bugs, how many is "fewer"?  How much time would you have needed to fix the bugs you avoided?
- If TDD code is smaller because you came up with a simpler algorithm, how much smaller is it?  How many lines of code did you avoid writing, during the TDD session?
- If TDD code is easier to extend and less brittle, how many lines of code will you avoid writing over the life of the project?

Sometimes test-driven code turns out exactly the way you thought it would, and the difference from TDD is simply having really good test coverage (and possibly, documentation) for your code.  Other times, the code takes a surprising turn towards something much simpler than you had in mind, as was the case with [Uncle Bob's Bowling Kata][bowling-tdd].  I've had similar experiences of my own.

But do cases like this *always* happen?  Is the amount of time available for feature work or the ease of adding new features clearly traced back to earlier decisions to use TDD?  While I do generally believe my use of TDD to be beneficial in many cases, I really don't have a good way to measure how well or how often it worked.

*Building up faith by cherry-picking facts from your own experiences?  You know what that sounds like...*


### TDD is hard

Test-Driven Development is a skill.  You have to learn the principles and practice until you get good at it, just like you would for writing concurrent programs on a multi-threaded platform, responsive UIs for a variety of screen sizes, or Continuation Passing Style in a functional language.  Colleges don't usually offer much training in the ways of TDD, and - when you start working - you tend to pick up the habits of those around you.

So you may not get much time to practice, without a fairly exceptional team or a lot of personal drive.

To make matters worse, it can be embarassingly difficult to test-drive even simple algorithms like [Minimax][minimax].  While it can be a fun challenge that makes you think in different ways about your design and your approach to TDD, it can also take a really long time to test-drive 15 lines of pseudocode.


## What? You can't be done yet!

You're right.  I'm not.

Hey for all I know, you may actually be a rock star.  You might be so knowledgeable of your domain and so experienced with your development stack that you really could write good code faster, if you just went ahead and wrote it.

Or you might be at such an early stage in your project that it makes more sense to implement a [Walking Skeleton][pryce-book] and worry about test-driving smaller components once the code stabilizes.

Or you might be over-estimating your capabilities or under-estimating the needs of your project.

Stay tuned for the exciting conclusion to (what is turning out to be) my Philosophy of TDD Trilogy.


[bowling-tdd]: https://cleancoders.com/episode/clean-code-episode-6-p2/show
[delayed-gratification-experiment]: https://en.wikipedia.org/wiki/Stanford_marshmallow_experiment
[minimax]: https://en.wikipedia.org/wiki/Minimax#Pseudocode
[pryce-book]: https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627
