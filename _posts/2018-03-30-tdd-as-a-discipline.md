---
layout: post
title:  "Test-Driven Development: More Discipline than Dogma (Part 3)"
date:   2018-03-30 11:45:00 -0500
categories: tdd philosophy
---

This is the conclusion to an [on-going series]({{ site.baseurl }}{% post_url 2018-03-23-tdd-is-not-dogmatic %}) discussing my philosophy of Test-Driven Development.


## Intro

So far I've shared my thoughts on why Test-Driven Development can appear to be a dogmatic process and how it really only fits the definition insomuch as TDD is a set of principles.

If TDD isn't dogmatic, then how should we explain why people do something that's hard and whose benefits are difficult to quantify?  How does this change in viewpoint help anybody?

My hope is to acknowledge the imperfections of the process while also reducing the stigma that it carries in some circles.  If at the end of the day you're still not persuaded to try it yourself — but you think twice about judging somebody else for trying it - I'll consider that a win.  If the conversation can shift from assuming that TDD folks are making unjustifiable decisions that waste time, to assuming that we're all just regular people making rational decisions based upon our experiences, I think that would be a good thing for our field.


## More of a Discipline

I like to think of TDD as more of a discipline — something that we know is hard and choose to practice anyway, because we think it will be helpful in the long run.

I found a [few definitions of discipline][mw-discipline] that resonated with me:

- a rule or system of rules governing conduct or activity
- orderly or prescribed conduct or pattern of behavior
- training that corrects, molds, or perfects the mental faculties or moral character

Is it a system of rules governing activity, and is there a pattern to how that activity is conducted?  Check and check.

Does following TDD provide feedback or training that affects your thinking about the code you're writing?  Does it cause you to make generalizations that you carry into the next project's design?  You bet it does.  Having to jump through hoops just to inject a collaborator or having dozens of lines of setup code in a single test will make you think twice about creating collaborators out of thin air and having long functions with multiple responsibilities.

Finally, let's avoid the debate over "moral character".  I'm not making any such claims here, and I doubt that the folks who developed the software for the Saturn V rocket followed TDD or were lacking in any sense of professionalism or character.  Just to be sure, I double checked the [source code][agc-codebase], and I didn't see any mention of RSpec.


## What else have I tried?

Why do I bother to do something that's hard?  *Because it works for me better than everything else I have tried*.

### Cowboy coding

At the start of my career, I wrote code the same way everybody else who was fresh out of college did at the time: *Cowboy-style*.  And sure enough, I experienced all the problems that everybody else had at the time: 

- It was really hard to add features to the existing code.  The only thing saving me from the fear of touching the Big Ball of Mud was my ignorance of the risks.  I soon learned, the hard way.
- No release could be completed without nights and weekends spent on integration, deployment, and validation activities.  Every release needed somebody to be the hero.
- Since everything was always late, there was no time to learn a better way of doing things.  I had to write code with the feeling that there must surely be a better way, if only I had a few hours to go and find out.


### Test-After

So I learned about JUnit and unit testing.  It didn't take long to single-handedly bring the test coverage on one project from 1-2% up to a **mighty 5%**.  *RAWR.*

Looking back, my code had a lot of tight coupling (statics everywhere, and collaborators made in constructors), and it wasn't very SOLID.  I started testing way too late in the game to make much of an influence on my design choices.

The fact that I was in trouble began to dawn on me when — on one 60,000 line project - I was having trouble getting above about 90 tests because all the tests depended upon the same data set (because they all used the same, global collaborators).  I literally had to stop writing tests, take my hands off the keyboard, and go to the whiteboard to map out a set of test data that would work for every test I wanted to write.

That project was late, too.

The other issue I found with this approach is no manager wanted to hear that the code was "done", but I couldn't move on to another feature yet because I wasn't done testing.  And - due to the unnecessary complexities of the production code - I was never, really sure when I was done testing.


### Waterfall

I later joined an organization that followed the [Team Software Process][tsp-wikipedia].  The forthrightness and humility about everyone's inevitable creation of defects that is baked into this process is laudable, and we had an ambitious goal: the product should have fewer than 0.5 defects / KLOC when all was said and done.

The solution was to use a waterfall style process of doing requirements, design, coding, and testing with quality controls (personal reviews and peer inspections) at each stage.

Despite a genuine interest in code readability and code quality from some of the brightest people I have ever known, the team was surprisingly dogmatic about not making even the *slightest* change to the production code for the sake of making it easier to test.  No reasonable argument could be made on any case by case basis for changing any part of the production code, so that we could test it.  The result was an [Ice Cream Cone][test-strategy] test strategy that kept being pushed further and further down the schedule.

Many years and millions of dollars later, that program failed, too.


### Test-Driven Development

I later joined a team that was pretty good at following Agile and in using TDD.  I went from 0 to shipping in a few, short weeks.  And some teams would consider that a long time, for ramping up.

There wasn't any great mystery whether my new code was working, what the status of each feature was, or how much longer would be needed to test something.  The domain and architecture were no less complicated than any of the other projects I had been on.  The team — as remarkable as each person was — wasn't doing anything that my former colleagues were incapable of doing.

The key difference — in my view — was in everybody's *choice* to learn and follow the principles of Test-Driven Development.

That's not to say that there weren't defects, that everything shipped on-time all the time, or that every single line was test-driven.  But more often than not: the process was followed, and we got results.


## Conclusion

The critical reader may be able to smell other issues with my former projects, or at least point out that correlation (of practice to results) is not causation.  I may well agree with you, on those points.

My main goal with this series is for readers to have more forgiving  impressions of teams doing TDD and more realistic expectations of its results.  It's not going to solve all your problems, and your individual results may vary.  You may have some other way of developing a high quality codebase, or you may not have the time to try in your current project.  You may even try — and fail — to do TDD and wonder if the principles are too laborious or impractical for today's high-paced demands.

For all I know, we may look back years later and see more flaws in TDD and more merit in some alternative.  Some other way of developing software may come along later that works better, or that works more often.

However - for the time being - Test-Driven Development is the *best way I know so far*.


[agc-codebase]: https://github.com/chrislgarry/Apollo-11/
[mw-discipline]: https://www.merriam-webster.com/dictionary/discipline
[test-strategy]: https://watirmelon.blog/testing-pyramids/
[tsp-wikipedia]: https://en.wikipedia.org/wiki/Team_software_process
