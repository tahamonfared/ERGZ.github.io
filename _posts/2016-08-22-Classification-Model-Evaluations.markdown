---
layout: post
title: "Evaluating a Classification Model"
comments: true
---

In this short article we will look at several different metrics for evaluating
a classification model.

The running example will make use of the **Adult** dataset from UCI Repo. The
model being implemented is a very simple Logistic Regression model.

~~~r
train <- sample_frac(Adult, 0.7) # 70% for training
test <- setdiff(Adult, train)

fit <- glm(
	income_class ~ age + education_num,
	data = Adult,
	family = binomial
	)
~~~

## Introduction

Unlike regression or scoring models, the evaluation of classification models
are more intuitive and straightforward. What helps even more is the fact
that most of these metrics stem from what is called a confusion matrix. In the
end these metrics of efficiency break down as rations of sums of this matrix.

## Confusion Matrix

Formally a confusion matrix is a cross-tabulation of false-positive, false-negative,
true-positive and true-negative tallies set up in a matrix as follows:

<center>

<table style="width:50%">
  <tr>
    <th></th>
    <th colspan="2" style="background:#FFF7CC">Observed</th>

  </tr>
  <tr>
    <td style="background:#FFF7CC"><strong>Predicted</strong></td>
    <td style="background:#E0DEE0">Pos</td>
    <td style="background:#E0DEE0">Neg</td>
  </tr>
  <tr>
      <td style="background:#E0DEE0">Pos</td>
      <td>True Positive</td>
      <td>False Positive</td>
  </tr>
  <tr>
      <td style="background:#E0DEE0">Neg</td>
      <td>False Negative</td>
      <td>True Negative</td>
  </tr>  
</table>

</center>

As we can see the best model ever will be a diagonal matrix (i.e only non-zero
entries are along the diagonal of the matrix). This is of course rarely the case
and we must a assess a trade-off between these metrics.

**Computation**

In R, there are many ways to create such a matrix, the easiest is with the `table`
base function. Suppose we fit a model `fit`, we first set up a "results" data frame
as follows:

~~~r
# data frame will have as columns the actual observations and the predicted ones
# we use type = "response" to get probabilities of each class
results <- data.frame(
	obs = test$obs,
	preds = predict(fit, test, type = "response")
	)

# personally I prefer to use the tibble type of data frame
# both will work fine in our examples
results <- tibble(
	obs = test$obs,
	preds = predict(fit, test, type = "response")
	)

# be consistent and keep predicted as rows and actual as columns
# formulate the confusion matrix with
confMatrix <- table(predicted = results$obs, observed = results$obs)
print(confMatrix)
~~~

~~~
         Actual
Predicted  <=50K  >50K
    <=50K   6962  1716
    >50K     432   650
~~~

Using this table we move on to put together some metrics.

## Accuracy

The most intuitive of the metrics presented today is the accuracy. Accuracy
is simply the proportion of correct predictions. Using the confusion matrix this
can be calculated with,

$$Accuracy = \frac{True\;Positive + True\;Negative}{Sample\;Size}$$

Using the confusion matrix we computed above:

~~~r
accuracy <- (confMatrix[1,1] + confMatrix[2,2])/sum(confMatrix)
# or
accuracy <- sum(diag(confMatrix))/sum(confMatrix)
~~~

We of course always want our model to have a high accuracy. This however is
not always enough. Unbalanced classes is where accuracy breaks down.

Consider the problem of detecting a fraudulent transaction. One would expect
to have a over 99% of the transactions be fair and not fraud. Thus a naive
(zero-information, predict all observations to be of the large class type)
model will have accuracy of 99%. Yet we will NEVER detect a fraud since the
model is unable to do so.

Thus one should always be careful of using accuracy for model validation.

**Computation**

Let's create the foundation of our `getMetrics` function. The plan is to use a
"list" R data structure (in python these are called dictionaries) to store all
the metrics. Once again the model being fit will make use of the Adult data
from UCI Machine Learning Repo, specifically the model is,

~~~r
income_class ~ age + education_num
~~~

Where,

* income_class -- classification of an individual's income, ">50K" or "<=50K"
* age -- age of an individual
* education_num -- number of school years completed (i.e one year college is 13)


~~~r
getMetrics <- function(confMatrix) {
	res <- list()
	accuracy <- sum(diag(confMatrix))/sum(confMatrix)
	res["accuracy"] <- accuracy
	return (res)
}

getMetrics(confMatrix)
~~~

~~~
$accuracy
[1] 0.7773337
~~~

Initially this seems ok, but upon reexamination of the response we have that,

~~~r
Adult %>%
	select(income_class) %>%
	group_by(income_class) %>%
	summarise(total = n()/dim(Adult)[1])
~~~

~~~
# A tibble: 2 x 2
  income_class     total
        <fctr>     <dbl>
1        <=50K 0.7591904
2         >50K 0.2408096
~~~

And in fact **<=50K** is much more abundant in the data. In fact we can create a **Null Model**
or **Zero-Information Model** that simply classifies all observations as the class of the
most abundant ("<=50K" in this case). Doing so yields an accuracy of obviously $\approx$75%
(this is approx since it depends on the random sample(s) taken for the test and training set).


## Precision and Recall

The next set of metrics we will look at are Precision and Recall.

### Precision

Precision is defined as the ratio of positively predicted observations that
were in fact positive. We can use the confusion matrix to calculate this value,


$$Precision = \frac{True\;Positive}{True\;Positive + False\;Positive}$$

One can think of the precision as a confirmation metric. Once again we will want
our model to always have a fairly high precision, but exact and acceptable precision
is very context related.

For example consider the task of filtering spam from emails (spam is positive). Applying
precision in this context answers the following:

*Of all the email my model marked as spam how many are actually spam?*

*Of all incomes predicted to be under 50K, how many are indeed under 50K?*

In the case of email, and sensitive data, a precision of 90% is horrible, this indicates
that about 10% of the spam email are relevant to the user, but he or she will never see these.
A good spam detector will have precision of 99.99%.


**Computation**

We update our `getMetrics` function to calculate and return a precision.

~~~r
getMetrics <- function(confMatrix) {
	res <- list()
	accuracy <- sum(diag(confMatrix))/sum(confMatrix)
	precision <- confMatrix[2,2]/sum(confMatrix[2,])
	res["accuracy"] <- accuracy
	res["precision"] <- precision
	return (res)
}

getMetrics(confMatrix)
~~~

~~~
$accuracy
[1] 0.779918

$precision
[1] 0.6007394
~~~

As we can see the precision is quite low, and so we have that about 40% of the observations
we predict to have **>50K** income is mislabeled.


### Recall

Recall is defined as the proportion of "truth" the model was able to detect,

$$Recall = \frac{True\;Positive}{True\;Postive + False\;Negative}$$

Whereas precision can be interpreted as a confirmation metric, recall can be
interpreted as tuning metric, or an effectiveness metric.

Returning to the spam example, recall answers:

*How much of the spam can my model actually detect?*

**Computation**

~~~r
getMetrics <- function(confMatrix) {
	res <- list()
	accuracy <- sum(diag(confMatrix))/sum(confMatrix)
	precision <- confMatrix[2,2]/sum(confMatrix[2,])
	recall <- confMatrix[2,2]/sum(confMatrix[,2])
	res["accuracy"] <- accuracy
	res["precision"] <- precision
	res["recall"] <- recall
	return (res)
}

getMetrics(confMatrix)
~~~

~~~
$accuracy
[1] 0.779918

$precision
[1] 0.6007394

$recall
[1] 0.2747253
~~~

Much like precision did, the recall shows how naive our model is. It was only able to find
27% of the incomes over 50K.

### F1 Score

F1 score is the harmonic mean of precision and recall,

$$F1 = 2\times \frac{Precision \times Recall}{Precision + Recall}$$

It allows us to consider both precision and recall in one metric. Note that a drop in
either precision or recall will result in a drop in F1 Score.

**Computation**

~~~r
getMetrics <- function(confMatrix) {
	res <- list()
	accuracy <- sum(diag(confMatrix))/sum(confMatrix)
	precision <- confMatrix[2,2]/sum(confMatrix[2,])
	recall <- confMatrix[2,2]/sum(confMatrix[,2])
	f1 <- 2 * (precision * recall)/sum(precision, recall)
	res["accuracy"] <- accuracy
	res["precision"] <- precision
	res["recall"] <- recall
	res["f1"] <- f1
	return (res)
}

getMetrics(confMatrix)
~~~

~~~
$accuracy
[1] 0.779918

$precision
[1] 0.6007394

$recall
[1] 0.2747253

$f1
[1] 0.3770302
~~~


## Sensitivity and Specificity

Unlike precision, recall and the f1 score, Sensitivity and Specificity are deeply rooted in
Statistical Theory, and heavily used in sciences.

### Sensitivity

Sensitivity is the same as recall, sometimes also called **true positive rate**. From a
probabilistic point of view Sensitivity is defined as,

$$Sensitivity = P\left( Positive \big| Actual\; Positive\right)$$

using the confusion matrix, this is the same as the recall we defined above.

![Low Sensitivity](/assets/post_assets/classification-model-evaluation/low-sensitivity.png)

Note that the above depicts low and high sensitivity but in both cases there exist very low
specificity as well. We learn about this in the next section.

using the sensitivity in the context of counts and proportions it serves the same purpose as
recall. The probabilistic point of view is not useful in all models.

### Specificity

Also known as **true negative rate**, is defined by,

$$Specificity = P\left( Negative \big| Actual\;Negative\right)$$

and using the confusion matrix this is,

$$Specificity = \frac{True\;Negative}{True\;Negative + False\;Positive}$$

In figure above we can see a depiction of low specificity, namely all of the Actual Positive outcomes that
the test did not label as positive. The figure below depicts a situation where we have both high specificity and
sensitivity.

![Low Sensitivity](/assets/post_assets/classification-model-evaluation/high-sensitivity-high-specificity.png)


## Probabilistic Validation Methods

Models like logistic regression do not directly model the response $Y$, but rather a probability
of the response occuring given some attributes (age, education in this case). Thus an
observation with .78  probability of being **>50K** will (most likely) be classified as
**>50K**. The fact that we can extract these probabilities allows us dig deep into the
performance of the model.

**Getting Probabilities**

We have been obtaining probabilities this whole time, however we converted these to classes, by
using a threshold of .5. Thus anything greater 0.5 was classified as **>50K**. Lets set up a data.frame that stores these probabilities as well.

~~~r
testPreds = predict(fit, newdata = test[,-ncol(test)], type = "response")

testResults <- data.frame(
  obs = test$income_class,
  preds = ifelse(testPreds > 0.5, " >50K", " <=50K")
  probs = testPreds
)

head(testResults)
~~~

~~~
     obs  preds     probs
1  <=50K   >50K 0.5324867
2  <=50K  <=50K 0.1286499
3  <=50K  <=50K 0.3051802
4   >50K  <=50K 0.4180843
5  <=50K  <=50K 0.2666534
6  <=50K  <=50K 0.0715819
~~~

### Double Density Plot

Using the probabilities above, as well as known classes, we can build a density plot to
observe the performance,

~~~r
library(ggplot2)
ggplot(data = testResults) +
  geom_density(aes(probs,
                   fill = factor(obs),
                   linetype = factor(obs)),
               alpha = 0.3)
~~~

<center>
<img src = "/assets/post_assets/classification-model-evaluation/double-density-plot.png">
</center>

This is far from ideal. A good model is able to separate the two classes at about the .50 mark.
Moreover the peak of each density should be as far as possible from the other indicating
that the model appropriately assigns probabilities for each of the classes. The worst we can
have is one density on top of the other, in other words the model is unable to find a separation
between the two classes.

One of the good things from the above is that model is peaking at low probabilities for
the **<-50K** class, however it is having a very hard time identifying **>50K** class.


### ROC Curve

Although useless after looking at the density plot above, we can plot the ROC Curve for exercise reasons.

~~~r
library(ROCR)

rocObject <- ROCR::prediction(predictions = testResults$probs,
                              labels = testResults$obs)

modelPerformance <- ROCR::performance(rocObject, "tpr", "fpr")

qplot(x = modelPerformance@x.values[[1]],
      y = modelPerformance@y.values[[1]],
      geom = "line") +
  xlab("False Positive Rate") +
  ylab("True Positive Rate") +
  geom_abline(slope = 1, intercept = 0, color = "black", alpha = 0.7, linetype = 3) +
  coord_fixed(ratio = 1) # note fixed axes to show appropriate depiction of ROC
~~~

<center>
<a href="/assets/post_assets/classification-model-evaluation/roc-curve.png" target="_blank">
<img src="/assets/post_assets/classification-model-evaluation/roc-curve.png">
Click to Enlarge
</a>
</center>

The ROC curve displays the trade-off between true positive rate and false
positive rate for all the values of the threshold.

As we had anticipated, the ROC curve is not a good one. Ideally the curve will
have an almost 90 degree angle hugging the top left corner. The worst possible
curve is shown as dashed line.
