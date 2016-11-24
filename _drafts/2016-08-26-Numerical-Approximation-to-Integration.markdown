---
layout: post
title: 'Numerical Approximations to Integration'
---

Calculating \\(P(a < X < b)\\) may seem straightforward in most context,
yet there are times where a numerical approximation to this value is required.
In this short article we look at several straightforward ways we can numerically
approximate this value.

The running example will consider approximating an integral over some interval
of the standard normal.

$$P(a < Z < b) = \int_a^b \phi(z) dz$$

where \\(\phi\\) is the density of the standard normal given by,

$$\frac{1}{\sqrt{2\pi}} e^{-z^2/2}$$

Every Statistics book include tables in the Appendix that give the CDF of
\\(\phi\\), such that the probability above reduces to \\(\Phi(b) - \Phi(a)\\).

We will make use of the R programming language to code these approximations and
make use of the vector operations built in. This code will easily translate to
any programming language.

## A Riemann Approximation

Riemann integration is what we all learn in Calc I, we can take the same idea
of binning and apply it here to "integrate" over some interval (a, b]. Recall
the idea of Riemann Integration is to divide the **domain** over which
we are trying to integrate into some small intervals. We then calculate the
the height of each of the midpoints of these intervals, and sum. As we increase
the number of bins we split the domain in (take the limit as \\(n \rightarrow
\infty\\)) the better we get the actual value.

The R implementation is straightforward and presented below.

~~~r
# constants
m <- 5000 # no of bins
w <- (b - a) / m # rectangle width
a <- 0; b <- 1 # bounds

# binning
grid <- seq(a + w/2, b - w/2, length = m) # binning start and end points
h <- (1/sqrt(2 * pi)) * exp(-grid^2 / 2) # kernel of normal

# calculate integral
sum(w * h) # width * height
~~~

~~~
[1] 0.3413447
~~~

Note that the `sum` function operates elements-wise over the vector, so as one would expect in a mathematical sense. List comprehension may be used in languages that
do not implement vector operations.

We can compare this to the "table" approach (\\(\Phi(b) - \Phi(a)\\)) using R's built in Normal function,

~~~r
pnorm(1) - pnorm(0)
~~~

~~~
[1] 0.3413447
~~~

As we can see for most applications this is good enough. The next method takes
the same idea of Riemann Integration, but instead of an evenly spaced grid over
the domain we will be using randomly choosen points.

## Monte Carlo Integration

Monte Carlo Integration is named so because instead of choosing points along the
domain systematically, we do so randomly. Specifically we use a random uniform,
\\(Unif(0, 1)\\) to select points. Apart from the way we choose these points,
the rest of the implementation remains the same, and presented here.

~~~r
m <- 5000
a <- 0
b <- 1
w <- (b - a) / m
u <- runif(m, a, b)
h <- dnorm(u)

sum(w * h)
~~~

~~~
[1] 0.3409085
~~~

A couple things to note. First we have that points are chosen at random along
the domain, using `runif()`, these points are the "midpoints" used the Riemann
version. Also note that we use the `dnorm()` built-in function. We could have used
the kernel as we did in the first implementation.

### Error Rate

The fact that we are randomly sampling from the domain of the integration means
we carry some type of error between each run. Here we take a closer look. First
we encampsulate the above integration in a function,

~~~r
mcIntegrate <- function(m, a, b, d = "dnorm") {
  w <- (b - a) / m
  u <- runif(m, a, b)
  h <- match.fun(d)(u)

  return (sum(h * w))
}
~~~

and run the integration 1000 times,

~~~r
values <- numeric(1000)
for (i in 1:1000) {
  values[i] <- mcIntegrate(5000, 0, 1)  
}

actual <- pnorm(1) - pnorm(0)
errors <- abs(values - actual)

# mean of the errors
mean(errors) # [1] 0.0005323245
quantile(errors, c(0.025, 0.975))
~~~

~~~
2.5%      97.5%
0.00002309 0.00151030
~~~

As we can see on average the errors are within .0005 of the actual value,
moreover 95% of the runs are within 0.00002309 and 0.00151030 of the actual
value.

## Acceptance-Rejection Region

The Acceptance-Rejection method simply samples at random from an area that we know
holds the density. Once this is done, we simply "accept" those points under the
curve, calculate this proportion as the approximation.

![Accept and Reject](/assets/post_assets/numerical-integration/accept-reject.png)

Once again the implementation itself is rather straightforward.

~~~r
m <- 5000
u <- runif(m, 0, 1)
h <- runif(m, 0, 0.4)
acc <- mean(h < dnorm(u))
0.4 * acc
~~~

~~~
[1] 0.3424
~~~
