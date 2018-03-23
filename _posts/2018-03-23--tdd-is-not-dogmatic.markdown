---
layout: post
title:  "Test-Driven Development: More Discipline than Dogma (Part 1)"
date:   2018-03-22 08:04:00 -0500
categories: tdd philosophy
---

Have you ever met somebody who is really serious about the practice of Test-Driven Development (TDD), and wondered what made that person dive in the deep end?  Does this person seem rational in every other way, except for his or her bewildering insistence that -- regardless of the complexity of a project -- production code may only be written after a test compels it into existence?  Does this seem idealistic, like something that's fun as a hobby but not cut out for the real world?

Before labeling the person and condemning the practice, it's worth considering the differences between being *dogmatic*, and being *disciplined*.


## What does it mean to be dogmatic?

The term ["dogma" is defined][dogma-definition] as

> a principle or set of principles
> laid down by an authority
> as incontrovertibly true

Let's break that down and see how it applies to Test-Driven Development.


### A set of principles?

Is TDD a set of principles?  You bet it is!  The [Red Green Refactor cycle][red-green-refactor] consists of this set of rules:

1. You must write a failing test before you write any production code.
1. You must not write more of a test than is sufficient to fail, or fail to compile.
1. You must not write more production code than is sufficient to make the currently failing test pass.

While you may have your own reasons for choosing to follow — or not follow — these rules in any given circumstance, the rules are pretty objective.  You're either following them, or you're not.

So far, TDD seems to fit the definition of a dogma.


### ...laid down by an authority?

I suppose your opinion on this matter depends upon your experiences.

In my case, I have certainly been encouraged to (or discouraged from) use TDD on whatever I was doing for my day job.  I've worked in companies with cultures at both ends of the spectrum — where my practice of TDD either made me fit in well, or be the odd one out.  

There are certainly thought leaders in our community who promote the principles of TDD and teach techniques for doing it well, but I've never test-driven code simply because someone told me I had to.  So I don't think practicing TDD makes you dogmatic, by this criteria.


### ...as incontrovertibly true?

Again, it probably depends upon who you're talking to and what you assume about that person's level of objectivity on the matter.  There has been [quite a lot of debate][is-tdd-dead] on whether or not you must practice TDD in order to call yourself a professional.  Some well-known authors have held the position that [you must use TDD to be a professional][no-tdd-equals-unprofessional] and have [later evolved][professionalism-and-tdd] to acknowledging well-crafted work that has been achieved without it.

I'm no doctor, but I would argue that — with the example of doctors washing their hands to prevent infections in their patients — we have a pretty good understanding of what causes infections and how hand washing addresses those causes.  The outcomes and practices are also more objective in these cases — doctors either washed their hands (or didn't) and patients either got infected (or didn't).

With software development and TDD, it's not so clear-cut.  People argue over the scope, intention, and direction of software projects and therefore disagree over the criteria for defects.  Is a product buggy because it interfered with the user's ability to use it intuitively, or is it just a requirement that hasn't been fulfilled yet?  Is "software quality" defined by users from the outside, or does the ability to maintain and extend its internal structure count too?  It can also be hard to find a consensus among developers that TDD would be the only or best way to improve the quality, of a given project.  For any given advocate of using TDD for a given project, it's often not hard to find a dissenter who believes that a different choice of platform / language / programming styles — or mastery of those — would be the better way to proceed.

It seems to me that the lack of consensus and of a clear, causal relationship between TDD and quality leaves plenty of room to struggle with the questions of why, how, and when to best use TDD.  This in turn leads me to argue that:

*A developer is not being dogmatic, merely for his or her choice to use Test-Driven Development.*


## What is it, then?

If the practice of TDD is not dogmatic, then what is it?  What are you supposed to call those weird people who like to use it, anyway?

Stay tuned for the next part in the series!


[dogma-definition]: https://www.google.com/search?q=Dictionary#dobs=dogma
[is-tdd-dead]: https://martinfowler.com/articles/is-tdd-dead/
[no-tdd-equals-unprofessional]: http://programmer.97things.oreilly.com/wiki/index.php/The_Three_Laws_of_Test-Driven_Development
[professionalism-and-tdd]: https://8thlight.com/blog/uncle-bob/2014/05/02/ProfessionalismAndTDD.html
[red-green-refactor]: http://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html
