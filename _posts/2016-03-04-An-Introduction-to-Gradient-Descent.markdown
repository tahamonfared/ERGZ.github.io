---
layout: post
title: "An Introduction to Gradient Descent"
date: 2016-03-04
categories: data statistics
description: "In this post"
---

Much of Machine and Statical Learning Algorithms can be formulated in a problem of mathematics called [Convex Optimization](#). Convex functions are essentially those functions for which if we draw a line segment from any point to another, the function will always lie beneath it.

<center><img src="/assets/post_assets/imgs/easy-intro-to-gd/convex-function-example.jpg"></center>



*The figure above shows a typical convex function. This function is in $\mathbb{R}^2$, but the same principle holds for functions in $\mathbb{R}^n$.*

In the context of Machine Learning this convex function represents a cost or objective function with the learning parameters we seek to minimize. A simple example of this is *Simple Linear Regression*,

$$f(x) = \beta_0 + \beta_1 x$$

The objective function we formulate from this is simply the summed square difference between the actual observed value and the value that the model predicts. We can write this mathematically as:

$$\sum_{i = 1}^n\left(Y_i - f(x_i) \right)^2 = \sum_{i = 1}^n\left(Y_i - \beta_0 + \beta_1 x_i \right)^2$$

we often call this the **Residual Sum of Squares** or *RSS* in short. And so the convex optmization problem can then be formulated to be:

$$\min_{\beta_0,\beta_1}\left[ \sum_{i = 1}^n\left(Y_i - \beta_0 + \beta_1 x_i \right)^2 \right]$$

So our goal is to minimize the parameters in the linear regression $\beta_0, \beta_1$ according to making the error they predict as small as possible.
The most popular way to solve the problem above are with normal equations and the least squares approach. We will instead focus on minimizing the problem above with **Gradient Descent**, another popular approach that gives us certain advantages over the least squares.

## The Idea of Gradient Descent

If you recall the Gradient is simply derivatives in multiple dimensions, we can also think about the gradient as a vector of partial derivatives. Now also recall that the derivative of a 1-dimensional function yields the direction in which the function decreases/increases the quickest (the slope of the tangent line at that point). It doesn't take too much thought to expand this concept to a function in n-dimensions, the gradient plays the same role as the derivative, and simply yields the *directions* in which the function decreases the quickest. In fact gradient descent is simply the partial derivative of a function $f$ in $\mathbb{R}^n$.

<div class="c-article__comments-div-math">
    <p>
        <strong>Definition of Gradient</strong><br>
        The gradient of $f(x,y)$ at a point $P_0(x_0, y_0)$ is the vector

        $$\nabla f  = \left[ \frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}\right] $$

        In the general case we have that the gradient of $f(\mathbf{x})$ for $\mathbf{x}$ in $\mathbb{R}^n$ at a point $P_0(x_1, x_2, ..., x_n)$ is the vector

        $$\nabla f = \left[ \frac{\partial f}{\partial x_1}, \frac{\partial f}{\partial x_2},..., \frac{\partial f}{\partial x_n}\right] $$
    </p>
</div>

Now connecting this to the first example above, since our objective function is convex, means that it has minimum point and we can apply the gradient to this function, each time moving in the direction which decreases the most, we will eventually find ourselves at the minimum, as we desired.

## Gradient Descent on a Simple Function

Lets begin the implementation of gradient descent using a very simple function, whose minimum we know beforehand. We will use a function with a square term so that we can eventually solve the RSS problem above comfortably.

Consider the typical sum of squares function,

$$f(a, b) = a^2 + b^2$$

<br>

<center><img src="/assets/post_assets/imgs/easy-intro-to-gd/sum-of-squares-3d.gif"></center>

<br>

We can express the above using vector notation to impose no restriction on the dimensions.

$$f(\mathbf{x}) = x_1^2 + x_2^2 + ... + x_n^2$$

where $\mathbf{x}$ is a vector in n-dimensions.

At this point recall that the goal of gradient descent is to find those values of $\mathbf{x}$ such that the function $f$ defined as the sum of squares is minimized. Let's also point out that this is exactly when $\mathbf{x} = (0, 0, ..., 0)$. So let's how well we can do.

Let's complete this exercise in Python

To implement gradient descent on this function we need to:

1. Derive the gradient of the function.
2. Define a function that will move us downward given the gradient.
3. Choose first a rate to determing how much to move in the gradient direction, and second a threshold to determine when we are close enough to zero.
4. Create an iteration process that will compute gradients and move downward until we are a threshold away from zero.

### First Implementation

~~~python
# import some required libraries
import math
import random

# use euclidian distance as the distance metric, to test when we are close enough to zero.
def distance(x, y):
    return math.sqrt(sum((i - j)**2 for i,j in zip(x,y)))

# the gradient vector is just the partials of each element
# of the original function, if this confusing I highly recommend reading about list comprehensions in Python
def ss_gradient(vector):
    return [2 * v_i for v_i in vector]

# the step function takes a vector, direction and rate, it then applies
# the rate*direction to each element in v (the original vector)
def step(vector, direction, rate):
    return [v_i + rate * direction_i for v_i, direction_i in zip(v, direction)]

# define a starting point
v = [random.randint(-10,10) for i in range(3)]

# a threhold that determines when we are close enough to a minimum
threshold = 0.0000001

# iteration process
while True:
    gradient = ss_gradient(v)
    next_v = step(v, gradient, -0.01)
    if distance(next_v, v) < threshold:
        break
    v = next_v

print(v)		# print the raw results
~~~

	[-3.0239694121001237e-06, -1.8899808825625798e-06, -3.4019655886126427e-06]

So briefly, above we first established functions needed to implement Gradient Descent. We then create a threshold and a starting point for the algorithm. Next we iterate by first evaluating the gradient at the input, which in the first iteration is `v` we just created with `random.randint`. Next we define a potential minimum vector called `new_min` by subtracting from `v` the `rate` times the gradient direction. Lastly we check if this potential minimum meets our threshold criteria, if it does we are done and return it, otherwise assign this new minimum to the original `v` and iterate again.

A natural question to ask now is, how close can we get 0 and can we ever really get it. This is controlled by the rate. The gradient determines what direction to move, but the rate determines how much in that direction we will move. Being able to pick an appropriate rate is important. Too big, and we might ovserhoot the minimum, and possibly never converge, on the other hand too small and we may be moving too slowly towards a minimum.

Lets implement a new version of the code above to test different step sizes.


### Multiple rates

~~~python
v = [random.randint(-10,10) for i in range(3)]
v_reset = v
print("The initial guess: {}".format(v))
threshold = 0.0000001
rate = [.9, .7, .5, .3, .1, .01]

for i in rate:
    v = v_reset
    total_steps = 0
    while True:
        gradient = ss_gradient(v)
        new_min = step(v, gradient, -i)
        if distance(new_min, v) < threshold:
            break
        v = new_min
        total_steps +=1
    print("-------------")
    print("Rate Value: {}".format(i))
    print("Total steps required: {}".format(total_steps))
    #print("Final Minimum Result: {}".format(v))
~~~

	The initial guess: [2, 1, -4]
	-------------
	Rate Value: 0.9
	Total steps required: 82
	-------------
	Rate Value: 0.7
	Total steps required: 20
	-------------
	Rate Value: 0.5
	Total steps required: 1
	-------------
	Rate Value: 0.3
	Total steps required: 19
	-------------
	Rate Value: 0.1
	Total steps required: 72
	-------------
	Rate Value: 0.01
	Total steps required: 680


We can visuzalize this real quick


~~~python
import matplotlib.pyplot as plt

plt.plot(rate, total_steps_list, color="crimson", marker='o', markerfacecolor="wheat")
plt.xlim(-0.1,1)
plt.xlabel("Step Size")
plt.ylabel("Total Steps")
plt.show()
~~~

<center><a href="/assets/post_assets/imgs/easy-intro-to-gd/step-size-function-ssq.svg" target="_blank"><img src="/assets/post_assets/imgs/easy-intro-to-gd/step-size-function-ssq.svg" width="600" height="600"></a></center>


In the above implementation we picked predefined step sizes and checked how each one worked. We can incorporate this method in the iteration of the algorithm, and each time pick the best one.

~~~python
v = [random.randint(-10,10) for i in range(3)]
threshold = 0.0000001
steps = [.9, .7, .5, .1, .01]
total_steps = 0

while True:
    value = sum_of_squares(v)
    gradient = ss_gradient(v)
    candidate_mins = [step(v, gradient, -s) for s in steps]
    new_min = min(candidate_mins, key=ss_gradient)
    next_v = sum_of_squares(new_min)
    if abs(next_v - value) < threshold:
        break
    v = new_min
    total_steps += 1

print(total_steps)
print(v)
~~~

    394
    [-0.0005701219078744463, 0.00028506095393722317, -0.0014253047696861124]


## Stochastic Gradient Descent

In the above implementation we updated (moved along the surface) only after we saw all the values in the vector, and so we did a simultaneous update (this occurs when we check using the threshold), in fact it is sometimes called **Batch Gradient Descent**. Once again since this is a small toy implementation it does not become a bottleneck. But imagine having to look over a data set of 1,000,000 rows to see if we have found a new minimum. In this case the efficiency of Gradient Descent goes out the window. For this reason we introduce **Stochastic Gradient Descent**. Stochastic Gradient descent will take advantage of the fact that most of the convex functions (error functions) are additive, and so instead of looking at the entire data set to compute the gradient and take a step we will do this each time we see a point.

### SGD Algorithm

The basic algorithm for Stochastic Gradient Descent (SGD) is as follows:

1. Random shuffle of Data
2. Take in the first observation of the randomized training data
3. Update the vector of paramters (in the above example this was the vector v)
4. Read in the second observation of the randomized training data
5. Update the vector of paramters.
6. Repeat until convergence.

### The RSS Example

*This RSS example is based on a Machine Learning Course taught by Andrew Ng at Stanford*

Lets finish the article by implementing SGD on the RSS problem above. We will use the least squares solution to compare our results.

The RSS function is given by,

$$\sum_{i = 1}^n\left(\hat{f}(x_i) - y_i \right)^2$$

where $\hat{f}(x_i) = \beta_0 + \beta_1 x_i$

This is the function we seek to minimize. For mathematical and computational simplicity we will actually conisder minimizing:

$$\frac{1}{2} \sum_{i = 1}^n\left( \hat{f}(x_i) - y_i \right)^2$$

We will call this the **target function**. The data we will implement this on, will be simple and of the form,

$$
\left[
\begin{array}{c|cccc}
y_1 & x_{11} & x_{12} &\dots &x_{1n} \\
y_2 & x_{21} & x_{22} & \dots &x_{2n}\\
\vdots &\vdots &\vdots & \vdots &\vdots\\
y_i &\dots &x_{ij} &\dots &x_{in} \\
\vdots &\vdots &\vdots &\vdots &\vdots  \\
y_m & x_{m1} & x_{m2} &\dots &x_{mn}
\end{array}
\right]
$$


First lets generate some data

~~~python
# generate data
random.seed(5797) # for reproduceability
xs = [random.gauss(2, .08) for _ in range (100)]
ys = [.2*i + random.gauss(0, .01) for i in xs]
~~~

<center><a href="/assets/post_assets/imgs/easy-intro-to-gd/generate-data-sgd.svg" target="_blank"><img src="/assets/post_assets/imgs/easy-intro-to-gd/generate-data-sgd.svg"></a></center>

As we just saw the cost function becomes

$$RSS = \frac{1}{2} \sum_{i = 1}^n\left( \hat{f}(x_i) - y_i \right)^2$$

taking the gradient of this we obtain,

$$\nabla RSS = \left( \beta_0 + \beta_1 x_i - y_i\right)
\begin{bmatrix}
1 \\
x_i \\
\end{bmatrix}$$

and so

$$\nabla RSS = \left[ \left( \beta_0 + \beta_1 x_i - y_i\right) , x_i \left( \beta_0 + \beta_1 x_i - y_i\right)  \right]$$

It is easy to see how we can generalize this to data with $n$ features.

With this we can begin to implement the algorithm in Python

~~~python
import random

def in_random(data):
    indexes = [i for i, _ in enumerate(data)]
    random.shuffle(indexes)
    for i in indexes:
        yield data[i]

# define a target function to evaluate if a new min occurs
# this is the version with .5 at the front, note
# we will carry out the sum of these inside the sgd function
def rss_function(x, y, p):
    return .5 * ((p[0] + p[1]*x) - y)

# gradient function to determine direction of step
def rss_gradient(x, y, p):
    return [(p[0] + p[1]*x) - y, ((p[0] + p[1]*x) - y)*x]

# the function will move in the direction of the gradient
# after observing one data point, if we do not move for 100 iterations
# we break and return that as the min
def stochastic_descent(x, y, p_0, target_fn, gradient_fn, rate_0=0.01):
    p = p_0 # initial guess
    rate = rate_0 # initial rate
    stationary_for = 0
    min_value, min_p = float('inf'), None

    # if using python2 no need to list this zip
    train_data = list(zip(x,y))


    while stationary_for < 100:
        value = sum(target_fn(xi, yi, p) for xi, yi in train_data)

        if abs(value) < abs(min_value):
            print(min_p) # to see the progression of the min
            min_p, min_value = p, value
            rate = rate_0
            stationary_for = 0

        else:
            stationary_for += 1
            rate *= .9

        for xi, yi in in_random(train_data):
            gradient_i = gradient_fn(xi, yi, p)
            step_in_gradient = [rate*i for i in gradient_i]
            p = [i - j for i,j in zip(p, step_in_gradient)]


    return min_p


pr = [0,0]          
print(stochastic_descent(xs, ys, pr, rss_function, rss_gradient))
~~~

    None
    [0, 0]
    [0.07889042172480416, 0.15945685762017703]
    [0.0791549698947851, 0.16038557326496145]
    [0.07791865536963628, 0.16111439706988132]
    [0.07660647127698314, 0.16174957310216193]
    [0.07568182239142766, 0.1622062219202756]
    [0.07474166780974591, 0.16264828283912564]
    [0.07390769861872022, 0.16306573155148854]
    [0.07309305931891151, 0.16347453966631278]



Let's see how well our fitted compares to using the normal equations.

<center><a href="/assets/post_assets/imgs/easy-intro-to-gd/sgd-abline-results.svg" target="_blank"><img src="/assets/post_assets/imgs/easy-intro-to-gd/sgd-abline-results.svg"></a></center>

The above plot depicts in *pink* the normal equations fitted line (produced with `lm` in R). The remaining lines are the approximations of the SGD. The above algorithm run several times on average converged to a minimum after finding 8 minimums. It iterated through the random data around 100 times.

## Conclusion

As we saw both Gradient Descnet and Stochastic Gradient Descent were able to approximate the minimum of convex function. In both cases above we have solutions for the minimum that we could find analytically, but this is not always the case. Cases like this SGD will thrive. Another advantage that GD and specifically SGD gives us is the ability to optimize the learning paramaters as we see new data without computing on the entire data set. This sort of advantage has a lot application in web world or other systems where data is accumulated at a very fast rate.
