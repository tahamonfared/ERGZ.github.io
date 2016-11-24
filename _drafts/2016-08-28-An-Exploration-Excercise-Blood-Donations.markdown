---
layout: post
title: "An Exploration Exercise: Blood Donation Prediction"
---

This post is a verbose documentation of my Exploratory Data Analysis on a Blood
Donations Data set. The goal of the data is to predict whether a donor will
donate blood in the month of March using several predictors. I will make use of
R and some R packages. I will perform no modeling in this post.

## Data Description

The data is not too complex, it consists of an outcome variable that shows whether
an individual returned to make a donation in March 2007, it is labeled as `made_donation_march_2007`. There are four predictors described as follows:

* `months_since_last`: this is the number of months since this donor's most recent donation.
* `number_donations`: this is the total number of donations that the donor has made.
* `total_volume_donated`: this is the total amount of blood that the donor has donated in cubic centimeters.
* `months_since_first`: this is the number of months since the donor's first donation.

You can obtain a copy of the data [here](https://gist.github.com/ERGZ/1419972bc0ba35157370549ad2aee11d)

## The Outcome Variable

The first thing I like to always do is inspect the response. In our case it is
labeled, `made_donation_march_2007`. It takes on values 0/1 to indicate whether
the individual did so or not.

~~~r
library(dplyr)
library(magrittr)

train %>%
  select(made_donation_march_2007) %>%
  group_by(made_donation_march_2007) %>%
  summarise(total = n())
~~~

~~~
# A tibble: 2 x 2
  made_donation_march_2007 total
                    <fctr> <int>
1                        0   351
2                        1   110
~~~

*Note the output here is a `tibble` data frame type. I like to import and use
tibbles, rather the usual `data.frame`, there is no difference here except for
some printing functionality.*  

So as we can see number of people who did not return to donate outnumber those who
did. I like always attach a visualization to aggregate counts to cement the idea.

~~~r
trainViz <- ggplot(data = train) # base graph layer to be used through out
trainViz + geom_bar(aes(made_donation_march_2007,
                        fill = factor(made_donation_march_2007))) +
  guides(fill = FALSE) +
  scale_x_continuous(name = "Made a Donation on March 2007",
                     breaks = c(0, 1),
                     labels = c("No", "Yes"))
~~~

![Made donation barplot](/assets/post_assets/eda-blood-predict/made_donation_march_barplot_color.png)

The visualization above confirms what we observed in the first aggregate. At this
point its important to note the lack of balance of course. When it comes to modeling
and evaluation of the model this will be something we will have to take into
consideration.


## Exploring the Predictors

The bulk of EDA will be how each of the predictors are distributed, and how
they relate to both the response and the rest of the predictors. I like to explore
each one and produce aggregate counts and visualizations.

The questions I am always asking myself when doing EDA are:

* How are the observations spread out?
* Why are they spread out this way?
* Are there any mounds (density plots)?
* Given a certain outcome, what does the predictor look like?
* Is this predictor related to another one?

These are just a few, and of course new questions should and will arise in the
process.

## Months Since Last Donation

In the data this is labeled `months_since_last`, and has integer values depicting
the last time each individual gave a blood donation. There are a number of ways
we can inspect the data, first we will produce some summary statistics, followed
by extensive visuals.

~~~r
summary(train$months_since_last)
sd(train$months_since_last)
~~~

~~~
# summary output
Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
0.00    2.00    8.00    9.26   14.00   39.00

# sd output
[1] 7.348
~~~

As we can see there are people in the data who's current donation was the first,
on the other extreme the longest period between donation is 39 months or just over
three years. The quantile summary tells us that 75% of the individuals have waited
at most 14 months or just over a year. The mean waiting time between donations is
9.25 and the median is at a close 8.00.

A standard deviation of 7 months might seem like a lot but it required (by the
Red Cross) that individuals wait a period 8 weeks between donations, so 2 months.

### Visualizations

Let's turn our attention to visualizations, I will rarely use histograms in favor
of density plots. The idea is the same, and a density plot is essentially a
continuous histogram.

~~~r
trainViz + geom_density(aes(months_since_last)) +
  geom_vline(xintercept = mean(train$months_since_last), linetype = 3) # dot the mean
~~~

![Months Since Last Density ](/assets/post_assets/eda-blood-predict/months_since_last_density_01.png)

Immediately we note the three modes in the data. Clearly there are three distinct
groups represented in the density. We can tell ggplot to draw out more "breaks"
on the x-axis for clarity.

~~~r
trainViz + geom_density(aes(months_since_last)) +
  geom_vline(xintercept = mean(train$months_since_last), linetype = 3) +
  scale_x_continuous(breaks = seq(0, 70, by = 5))
~~~

![Months Since Last Density ](/assets/post_assets/eda-blood-predict/months_since_last_density_02.png)

The new breaks allow us to see the three groups more clearly. We can see groups
cut out by those who made a donation 0-6 months ago, 7-18 months ago, and those
who made a donation 20 or more months ago.

Let's now look at the density above for each of the two possible outcomes

~~~r
trainViz + geom_density(aes(months_since_last, fill = factor(made_donation_march_2007))) +
  guides(fill =FALSE) + # dont need a legend for this plot
  facet_grid(. ~ made_donation_march_2007) + # add facets
  scale_x_continuous("Months Since Last Donation (Color Outcome)", breaks = seq(0, 70, by = 5))
~~~

![Months Since Last Density ](/assets/post_assets/eda-blood-predict/months_since_last_density_03.png)

Once again the mounds are still clearly depicted in each of the two outcome classes.
For those individuals that did donate in March, most of the density of `months_since_last` is abundantly around the 0-6 range.

#### Skewed Distributions

Note here that both of these distributions are very skewed, this in turn does not
allow us to see fine detail in the left edge of the plot. One thing we can do
is instead visualize the log of the density. This is easily done in ggplot,

~~~r
trainViz + geom_density(aes(months_since_last, fill = factor(made_donation_march_2007))) +
  guides(fill = FALSE) +
  scale_x_log10("Months Since Last Donation", breaks = c(1,2,3,4,seq(5, 40, by = 5))) +
  facet_grid(. ~ made_donation_march_2007)
~~~

![Months Since Last Density ](/assets/post_assets/eda-blood-predict/months_since_last_density_04.png)

The above visual displays with much detail how individuals tend to donate 2 Months
after a given donation regardless if they donated again in March. This makes sense
since the waiting time between donations must be at least 2 months. For those
that did donate in March the density here is a global peak. A simple hypothesis
would be, "if an individual donated two months ago, then he or she will donate in
March". This however is a rather noisy hypothesis a simple query on the population
that did not return to donate shows that around 40% of the individuals made a donation
less than 4 months before.

When it comes to modeling it would be a bad idea to treat each of these "groups"
together, but there appears to be an underlying distribution that is affecting these
counts.

## Number of Donations

Like we did before we start with a numeric summary of the data. We would expect to
see a large range, given this was the first donation for some.

~~~r
summary(train$number_donations)

quantile(train$number_donations)
~~~

~~~
# summary output
Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
1.00    2.00    4.00    5.43    7.00   50.00

# quantile output
0%  25%  50%  75% 100%
 1    2    4    7   50
~~~

The outputs here are very similar to that of `months_since_last`. Before producing
a density plot, lets visualize these quantiles in the form of a boxplot.

~~~r
trainViz + geom_boxplot(aes(factor(0), number_donations)) +
  xlab(label = "") + ylab("Number of Donations")
~~~

<img src="/assets/post_assets/eda-blood-predict/number_of_donations_boxplot.png" alt="Number of Donations Boxplot" width="70%">

Just looking at the boxplot we can see that the distribution appears to be skewed.
There are plenty of outliers, something we might have to deal with in the process
of modeling. Let's now take a look at the density plot, split by the outcome variable
using facets, and log scaled (due to the hevy the skewness),

~~~r
trainViz + geom_density(aes(number_donations)) +
  facet_grid(. ~ made_donation_march_2007) +
  scale_x_log10("Number of Donations Made", breaks = c(1, 5, 10))
~~~

![number of donation made](/assets/post_assets/eda-blood-predict/number_of_donations_density_01.png)

The message of this visualization is pretty clear, and loosely stated as:
"Those who did not return for a donation were in the majority people who had just given
their first donation the last time". On the other hand those that did return
to donate blood had already given anywhere between 5 and 10 donations previously.

As stated in the beginning of the article we would like to know how these predictors
relate to each other as well. We tackle this question now.

~~~r
trainViz + geom_jitter(aes(number_donations, months_since_last, color = factor(made_donation_march_2007))) +
  guides(color = guide_legend(title = "March Donation"))
~~~

![number of donation made](/assets/post_assets/eda-blood-predict/number_donations_vs_months_since_last_jitter.png)

This visualization summarizes what we have seen so far, an individual is more likely
to donate in March if he or she has given blood more than 5 times before and the
last time they gave blood was no more than around 5 months ago. This of course is
not a perfect hypothesis, as we see plenty of observations that violate this.

## Total Volume Donated (in lifetime)

Next we look at the total volume donated in an individuals lifetime. This measurement
is given in Cubic Centimeters (cc). Once again first a numeric summary,

~~~
Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
 250     500    1000    1332    1750   12500
~~~

Go right into visualizing with a log density (code left out as it is the same as
previous implementations).

![total volume donated](/assets/post_assets/eda-blood-predict/total_volume_donated_density.png)

Look familiar? This looks to be exactly that of Number of Donations. Which makes sense,
but let's visualize these together,


![total volume donated](/assets/post_assets/eda-blood-predict/total_volume_donated_vs_number_donations.png)

Appears to be a perfect correlation, which we can confirm with,

~~~r
cor(train$number_donations, train$total_volume_donated)
~~~

~~~
[1] 1
~~~

A couple notes here, first a visualization like this can be very helpful in detecting
data entry errors. It would be alarming to see someone with large volume of blood donated with a small number of actual donations. With that in mind we see
no errors here. Second, when it comes to modeling, a perfect correlation such as this will yield unstable solutions and thus proper care must be taken (leave one of these out?).  

Overall we gain no further information with this predictor.

## Months Since First Donation

Let's jump right into a density plot of the predictor,

~~~r
trainViz + geom_density(aes(months_since_first,
                            fill = factor(made_donation_march_2007),
                            alpha = 0.4)) +
  guides(fill = guide_legend(title = "Made donation in March"),alpha = FALSE) # show and hide legends
~~~

![Months Since First Donation](/assets/post_assets/eda-blood-predict/months_since_first_density_01.png)

Once again as it relates to the outcome variable we can see minor differences between
positive and negative responses. Clearly we see that that for those that did not return
to donate the density tails off and thus indicating that a larger proportion of these
individuals have been donating for a longer time. On the other hand a larger proportion
of those individuals that did return had made their first donation more recently (only
slightly). Overall on its own Months Since First donation does not appear to have a
strong predictive power, since the distributions tend to overlap through out.

One thing we might consider doing is looking at the log scale of the domain. The
code just appends `scale_x_log10()` to the end of the ggplot function above, and
yields the following:

![Months Since First Donation](/assets/post_assets/eda-blood-predict/months_since_first_donation_density.png)

This however does not give results we were looking for.

Let's now use this predictor along with the rest to see how together they affect
a positive outcome.

### Scatter Plot Matrices

![Months Since First Donation](/assets/post_assets/eda-blood-predict/ggpairs_01.png)
