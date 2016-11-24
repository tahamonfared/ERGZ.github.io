---
layout: post
title: 'The Law of Large Numbers with Coins'
---

Limiting behavior in probabilistic simulation are what allow us
to have any sort of confidence in the results they yield. In this post we look at the
important **Law of Large Numbers** through a coin tossing example and observe
how its implementation guarantees a simulation will yield a stable result.

## Why Coin Tosses?

Coin tosses exhibit two very important (makes our work easier) properties. **First**,
each coin toss is independent of any other. This will allow our simulation to be
free of any kind of trend. This of course is not true in general, and we will look
at an example where this does not hold. **Second**, finite variance. Coin tosses
can be modeled by a binomial distribution, whose finite variance will allow the
simulation to converge.

## The Experiment

The experiment is simple. Suppose we have a fair coin, we will toss the coin
a large number of times each time noting the side that lands up. Common knowledge
of course leads us to believe that \\(P(heads) = P(tails) = 1/2\\), but at what
point in the process of tossing are we sure to be at least very close to this number?

## The Law of Large Numbers (LLN)

We can formally state the experiment above as follows:

Let \\(H_n = 0\\) denote that the nth toss was tails, and \\(H_n = 1\\)
denote that the nth toss was heads. Any single toss is a Bernouilli trial, and
as noted above n such trials yields a \\(Binom(n, .5)\\). Then the LLN says that
for some \\(\varepsilon > 0\\) as n (number of tosses) tends to \\(+\infty \\)
the proportion of heads is within \\(\varepsilon\\) of 1/2.

$$\lim_{n \rightarrow \infty} P\left( |Y_n - 1/2| < \varepsilon \right) = 1$$

which we can write as,

$$Y_n \overset{p}{\to} 1/2$$

which we read as \\(Y_n\\) converges in probability to 1/2.
