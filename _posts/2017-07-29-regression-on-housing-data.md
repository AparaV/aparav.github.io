---
layout: post
title: Regression on House Prices
subtitle:
description:
published: false
comments: true
---

Linear regression is perhaps the heart of machine learning. At least where it all started.
And predicting the price of houses is the equivalent of the "Hello World" exercise in starting with linear regression.
This article gives an overview of applying linear regression techniques (and neural networks) to predict house prices using the [Ames dataset](https://ww2.amstat.org/publications/jse/v19n3/decock.pdf).
<!--excerpt_ends-->
This is a very simple (and perhaps naive) attempt at one of the beginner level Kaggle competition.
Nevertheless, it is highly effective and demonstrates the power of linear regression.

## Pre-requisites

This article assumes the reader to be fluent in Python to understand the code snippets.
At least a strong background in other programming languages should be necessary.
We will build our models using Tensorflow.
So basic knowledge Tensorflow would be helpful, but is not a necessity.
The tutorial also assumes the reader is familiar with how Kaggle competitions work.

## The Raw Data

First off, we will need the data. The dataset we will be using is the Ames Housing dataset and can be downloaded from [here](https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data).
Opening up the `train.csv`, you will notice nearly 52 features of 1460 houses.
What each of these features represent is described in `data_description.txt`.
The file `test.csv` differs from `train.csv` in that there are fewer houses and the prices for each of the houses is not present.
We will use the `train.csv` file to train and build our model.
Then, using that model, we will predict the prices for each of the houses in `test.csv`.

You might want to spend some time studying this data by graphing charts, etc. to gain a better understanding of the data.
This will definitely be helpful, but we will not do that.

## Cleaning Data

The cleaning of data refers to many operations. Here we will be performing feature engineering (creating new features),
filling in missing values, feature scaling, and feature encoding.

<script src="https://gist.github.com/AparaV/f47e8054f44547f812788a6aa41233aa.js"></script>

52 features is a bit overwhelming.
And if you have spent time studying what each of these features represent,
you'd probably say that many of the features are redundant to some extent i.e., they play a very small role in the price of a house.
So the first thing we will do is remove these features and make life simpler.
The code snippet describes the features we want to get rid off.
But, before we remove them forever, notice that the total porch area and total number of bathrooms is split into 2 columns.
Again, to make life simpler, we will combine them into a single total porch area and a single total number of bathrooms.
Now, we can go ahead and get rid off all these unwanted features.

The next thing we want to do is handle missing values. There are various ways to tackle this problem.
An aggressive approach is to remove that entire training example.
This can be bad if there are lots of missing values because you will lose too much data.
But then, why would you train a model if you think you don't have enough data?
A simple and effective approach is to replace the missing value with mode (the most frequent value taken by that feature).
A more sophisticated (and maybe better) technique is to study the other features and determine the missing value using probability and statistics.
You might have guessed it - we are going to deal with missing values be replacing it with the mode.

The next thing we want to do is scale down the features.
The motivation behind this is that some of our features have a large range of values.
And this makes it difficult for our optimizer to converge. But, more on that later.
We will use the following method for rescaling.

\\[ x'\_i = \frac{x\_i - min(X)}{max(X) - min(X)}\\]

Here, \\(x\_i\\) is the \\(i^{th}\\) example of the feature \\(X\\) and, \\(min(X)\\) and \\(max(X)\\) refer to the minimum and maximum values the feature \\(X\\) takes respectively.
An important thing to note is that you do not want to scale the output i.e., the Sale Price.
This can lead to large errors in output and leave you clueless for a long time.

In machine learning, we almost always deal with numbers.
But many of the features have letters for values where each letter (or sequence of letters) refer to a particular category.
This is true for many datasets. And it also makes life difficult for us. And we do not like it when life becomes difficult.
So, we will encode each of these features i.e., we will map a one-to-one correspondence from each of these categories to a number.
The code snippet demonstrates how we achieve this.

The data we have now is almost ready for training.

## Splitting Dataset

A standard practice is to split the data into 3 parts - training, validation and test datasets.
We will use the training dataset alone to actually train the model.
Then we will use the errors the model gives on the validation dataset to tune our hyperparamters.
But now, the model we trained has "seen" the validation dataset.
This means that if we were to report the error the model produced using either the training or validation datasets, our real error would be biased because this model has been exposed and modified to minimize the error on these datasets.
This is where the test dataset comes into play.
Its purpose is to serve as an unbiased judge and report the error on the model.

Usually, the dataset is divided as 60% training, 20% validation and 20% testing. And we will follow that fashion.
We will also shuffle the dataset to make sure data is equally distributed across the 3 datasets.

So far we have been dealing with `pandas` dataframes. Alas! Tensorflow likes `numpy` arrays better.
So, we will have to fix that by converting the dataframes into matrices.
While doing so, we also need to separate the inputs, \\(X\\), and outputs, \\(y\\).

<script src="https://gist.github.com/AparaV/902692e441c06604703dbc7ffd2d3680.js"></script>

## Linear Regression

### The Algorithm

As I mentioned earlier, linear regression is perhaps the heart of machine learning.
And the algorithm is the equivalent of the "Hello World" exercise.
The algorithm is a very simple linear expression.

\\[Y = WX + b\\]

Here, \\(Y\\) is the output values for \\(X\\), the input values.
\\(W\\) is referred to as the weights and \\(b\\) is referred to as the biases.
Note that \\(Y\\) and \\(b\\) are vectors and \\(W\\) and \\(X\\) are matrices.
This is, in many ways, analogous to the line equation in \\(2\\) dimensions you might be familiar with.

\\[y = mx + c\\]

The only difference is that we are extending and generalizing this relation to \\(n\\) dimensions.
Just like being able to find a line equation between two points i.e., calculation \\(m\\) and \\(c\\),
we are going to find the weights \\(W\\) and biases \\(b\\).

In this way, we are going to map a *linear* relation between the sale prices and the features.
It is important to stress on the fact that this is only a linear relationship.
In reality, very few events are linearly correlated.

Naturally the question we have is figuring out the weights and biases.
To do this we will first randomly initialize the weights, and initialize the biases to \\(0\\).
Then we will calculate the right hand side of the equation and compare it with the left hand side.
We will define the error between them as the \\(Cost\\) or, the more commonly used term in neural networks, \\(loss\\).

\\[loss = \frac{1}{2}\sum\limits_{i = 0}^n{((Y) - (WX + b))^2}\\]

Then, this becomes an optimization problem where we are trying to find \\(W\\) and \\(b\\) to minimize the loss.
There are various methods to optimize this.
As usual we will stick with the simpler one - Gradient Descent Optimizer.
Understanding this optimizer is perhaps beyond the scope of this article.
But imagine optimizing a function in one variable using derivatives and
generalizing that method to a function \\(n\\) variables.
That is the core of gradient descent.

Now, let's jump into the code.

### The Implementation

<script src="https://gist.github.com/AparaV/687220208a52f97ee907cfff091d4eee.js"></script>

In Tensorflow, we first define and implement the algorithm in a structure called `graph`.
The `graph` contains our input, output, weights, biases, and the optimizer.
We will also define the loss function here. Then, we run the `graph` in a `session`.
During each iteration, the optimizer will update the weights and biases based on the loss function.

In our graph, we first define the train dataset values and labels (output), the validation and testing datasets.
Note that we are defining them as `tf.constant`s. This means that these "variables" will not and cannot be modified when the `graph` is running.
Next, we initialize the weights and biases. We treat these as `tf.Variable`s.
Pay attention to the dimensions of these matrices. You will run into compilation errors if you get them wrong.
This means that these "variables" have the capacity to be updated and modified during the course of our `session`.

Now, we predict the \\(Y\\) values using the weights and biases using the `tf.matmul()` function.
This is nothing but matrix multiplication. Then we add that to `biases`.
But if you go back to the definition, `biases` is a single number while `tf.matmul(tf_train_dataset, weights)` is a vector.
This might be confusing because you can only add a vector to another vector.
But Tensorflow is quite clever. It understands that we mean to add the same scalar `biases` to each element of the vector.
Think about this as converting the single number into a vector (or matrix) of same dimensions as the other vector,
and then adding those together. This is called broadcasting.

Then we calculate the `loss` as we defined previously. We can safely ignore `cost` for now.
It's only purpose is to report the error we get.
When using the gradient descent optimizer, we need a parameter (one of the hyperparamters) called learning rate.
The term is self explanatory - it refers to how fast we want to minimize the `loss`.
If it's too big, we will only keep increasing the `loss`. If it's too small, and the algorithm will converge very slowly.
Here, we define `alpha` as the learning rate. After much experimentation, I've decided to use `0.01` as the learning rate.
It might be beneficial to vary this value and test for yourself.
Next, we define the `optimizer`.

 - Algorithm
 - Implementation
 - Results

## Neural Network

 - Algorithm
 - Implementation
 - Results

## Improvements

 - Regularization
 - Less cleaning up
 - Bins to prevent overfitting

## Final Words
