---
layout: post
title:  "DRAFT TDD Tips: What do to when you're stuck"
date:   2018-04-06 08:00:00 -0500
categories: go tdd
---


## When you're stuck on one test

I got stuck a few times this week part way through test-driving.  TDD practitioners often recommend not taking too big a leap in the behavior of the prodcution code when writing the next test – that you should write _just enough_ test code to force your production code into doing something interesting.  

It's a simple idea that is sometimes difficult for me to put into practice.  There are times when the next logical step requires a significant change to the interface.  Take this early iteration on the Minimax algorithm, for example.

{% highlight go %}
func Minimax(game Game) Result {
	if game.FindWinner() == game.MaximizingPlayer() {
		return Result{Score: 1}
	} else if game.FindWinner() == game.MinimizingPlayer() {
		return Result{Score: -1}
	} else if game.IsOver() {
		return Result{Score: 0}
	}

	return Result{Move: game.AvailableMoves()[0]}
}

type Game interface {
	AvailableMoves() []Move
	FindWinner() Player
	IsOver() bool
	MaximizingPlayer() Player
	MinimizingPlayer() Player
}
{% endhighlight %}

That's enough to score endgame scenarios and to pick _something_.  Any player can pick _something_, so `Player` isn't even part of the function call yet.  It wasn't necessary.

Now what about the next logical step – picking the penultimate move with the best score?  Now you have to consider whether it's the maximizing player (who will pick the move resulting in the highest score) or the minimizing player (who will pick the opposite).

## Notes

Just write 1 test, if you're stuck

Don't be afraid to write a test just on the return type, instead of making a large next test for a relatively complex jump in behavior (best avoided, but sometimes can't help it)

write top-down always and then extract as a refactoring? but it's too big and you've already done the hard work to test it from the outside. maybe it's better to extract and test-drive the smaller part when I get a few test cases in to the top-down writing. Example - parsing HTTP requests.

