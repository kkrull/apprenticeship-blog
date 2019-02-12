---
layout: post
title:  "DRAFT The Role of Prototyping in Software Craftsmanship"
date:   2019-02-12 08:50:00 -0500
categories: prototyping craftsmanship
---

## Takeaways

Goals:

1. Make room for reasonable people who are in the discovery phase of their work to prototype
  - without feeling the need and hindered productivity in doing TDD right from the start.
  - You don't have to spend every minute of development, doing TDD.
  - There are valuable skills in prototyping that can be useful in an Agile / Craftsmanship-oriented team.
  - Highlight specific techniques that can be helpful for prototyping.
1. Introduce specific techniques that can be used to create a production version of a prototyped idea.
1. Share my opinion of when - and how - to do each phase of delivery.
  - Distinguish the separate phases / roles of prototyping and making production code.
  - Don't pretend, advertise, or re-label prototype code as production code.  The prototype is focused and therefore
    intentionally limited.

Specifically:

- Criteria for entering a prototype/production state
  - Enter prototype if you: don't know where to start, don't know if an algorithm, library, framework, or design will
    work at all, don't know if you'll like a design, don't know if it will work for your customers, or don't know if
    your team will wish to pursue another alternative.
- Criteria for leaving a prototype/production state
  - Enter production when you have built up enough confidence on the areas of greatest uncertainty that you feel
    comfortable with some commitment -- in the prodcution code -- to that aapproach.
  - Go back to prototyping if you find out it doesn't work (despite your earlier prototyping efforts).
  - Go to done when the production code meets the acceptance criteria, is well-tested, and is well-factored (readable).


## Outline

The big idea

- It's ok - and even encouraged - to prototype when you're still trying to figure out if/how it's all going to work.
  - Will a library or framework work (as expected)?
  - Will an algorithm work?
  - How do I want to structure my code?  Sometimes you just need to get an idea out of your system and a rough sketch of
    an idea to evaluate its suitability for your purposes.
  - Will an idea be intuitive and effective for users?
- Just know when to prototype and when to write production code.
- Be transparent about what phase you're in.


Philosophy on prototyping

- What's the goal?  _Figure out if an idea works at all_.  **In isolation**.
- Goal: Avoid re-work of writing a bunch of production code that winds up not working, not being accepted by customers,
  or not being accepted by the team.
- No hack is too big.  A larger concern is that you might not be making a _big enough hack_ to be a good use of your
  time.
- Be as direct as possible.  Go from A directly to B.  This means there will be fewer components that are larger in
  size, when compared to the production version of the solution.
- When you have a working idea, remember what it doesn't do
  - handle errors that can be reasonably expected, even with a limited feature scope
  - enable future bug fixes, because of its factoring
  - enable future extension, again because of its factoring
- What should you not be doing?
  - Implementing lots of features at once
  - Using complicated frameworks like Spring (unless your prototype is intended to figure out how Spring itself works)
  - **Claiming that you are done**
- Why is this useful?  You're getting early feedback, which is something you're generally trying to do in Agile.


Philosphy on production code

- What's the goal?  _Make code that has a limited scope, and does that in an intuitive and bug-free manner_.  (Yes there
  will be bugs, despite the best practices).
- Also the goal is to write code that can be understood - over time - by any developers (not just you).  It should be
  easy to tell when it has been broken, and where, and what it was supposed
  to be doing.  So you need tests.
- Goal: It should be easy to fix, when you do find bugs.
- Goal: It should be easy to extend, when you need new features.
- Goal: You should have repeatable, automated, objective evidence that it works.  So you need tests.  So you need it be
  easy to test.  So you have to follow SOLID principles -- especially dependency injection.  As a result, there will be
  more components to test.  Where the prototype was intentionally a direct line from A to B, writing SOLID code will
  necessarily introduce indirect relationships through interface points.
- Goal: What it does do, it should do **well**
- Goal: It should handle reasonable edge- and error-conditions that are in scope for the current set of features.  This
  is scope that is explicitly excluded from the prototype.  This means there will be more branches in the code.
- What's at stake is not merely your own time.  It's the time of your customers, your teammates, and of your future
  self.  What makes sense in the moment may not be so clear down the line, unless you are intentional about being
  expressive in your code.
- It's an act of _hubris_ to think that it's ok to leave code for the customers and for the team that you alone can
  understand.  If your teammates have trouble understanding your code, that's _your_ fault.
- Do one thing -- introduce the Taguchi Design of Experiments.
- Have multiple ideas?  Build multiple, independent prototypes.


TODO: Share some links to other materials on Software Craftsmanship, SOLID principles, and the need for TDD.  Don't
review the whole pantheon; just ackowledge that there are important reasons for this philosphy and provide a few links
for new readers to start reading more.


Techniques for prototyping code

- Even though you're not doing TDD for every line of code you write, it can be helpful to write a high-level acceptance
  test that gives you quick feedback on whether or not your prototype works (on the happy path).
- The walking skeleton -- supported by an acceptance test -- can be attractive if:
  - You don't have to spend a lot of time getting a new test framework up and running
  - there isn't a bunch of other technical debt that would make writing that test take too long
- Even if there is tech debt like services / databases that are hard to manage from within the test, you can hack the
  experience and start those up manually before starting your test.  That's not great for production, but it can give
  you some necessary support during the prototyping process.
- Even just writing the Gherkin without writing the glue code can be a helpful exercise.
- In general, always write new code in new places before deleting old code.  This includes prototype code.
- Make a whole separate repository if there's anything in the old repository that's going to slow down your ability to
    express teh idea
- Make a `prototype/` folder in the same repository you use for production code.  Git can handle plenty of directories
  with other stuff in them. You don't have to have to be locked into a "one artifact, one build process, and one Git
  repo" model.  You can have reference / prototype code that is built out of band (or not at all).  Keep it as long as
  you need it; delete it when you don't.
- **When do you stop?** When yoiu know if the idea works and is acceptable.
- Refer to the Pretotype It book.
- Add in some of the techniques and case studies from my Zagaku talk.
- What skills do you need?
  - Suppress impulses to implement any features that aren't of the greatest concern / uncertainty.
  - Do something full-stack -- see an idea through to the end.
- Communicate:
  - Tell your team.  Be upfront about it.  Openly accept and embrace the fact that you don't know what you are doing.
  - Make a separate spike story card, or a separate sub-task in the development story (if and only if the prototype is
    expected to not take long).


Techniques in using a prototype to guide development for production code

- TDD as always, when writing production code.
- Use a separate Git repo, branch, or different directories in the same branch to distinguish prototype code from
  production code.
- Recommend starting fresh on the production code, instead of trying to refactor the prototype into production code.
- Your design may deviate quite a lot from the prototype as the tests apply design pressure on your components.  That's
  ok -- embrace it.
- Open up two IDEs, if you want.


## Does this put me at odds with the Craftsmanship community?


## Does this put me at odds with the development community, at large?


