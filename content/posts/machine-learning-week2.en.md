---
title: "Machine Learning Week2 Assignment"
date: 2021-10-30T11:25:00+09:00
draft: false
author: Torres
tags: ["Linear Regression", "Machine Learning"]
categories: ["Machine Learning"]
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true
featuredImage: "/images/posts/machine-learning-week2/cover.jpeg"
featuredImagePreview: "/images/posts/machine-learning-week2/cover.jpeg"

---

This article is for the solution of week 2 assignment of [Machine Learning](https://www.coursera.org/learn/machine-learning/home/welcome) by Andrew Ng.

## Assignment Introduction

In this exercise, you need to implement one variable linear regression. The course has provided several files you would like to use and some of them you need to modify to implement the algorithm. Following list is the files they provide:

- ex1.m - Octave/MATLAB script that steps you through the exercise ex1 
- multi.m - Octave/MATLAB script for the later parts of the exercise 
- ex1data1.txt - Dataset for linear regression with one variable 
- ex1data2.txt - Dataset for linear regression with multiple variables 
- submit.m - Submission script that sends your solutions to our servers
- [(\*) warmUpExercise.m - Simple example function in Octave/MATLAB](#Warm Up Exercise)
- [(\*) plotData.m - Function to display the dataset](#Plot Data)
- [(\*) computeCost.m - Function to compute the cost of linear regression](#Compute\ Cost\ Function)
- [(\*) gradientDescent.m - Function to run gradient descent](#Gradient Descent)
- (†) computeCostMulti.m - Cost function for multiple variables
- (†) gradientDescentMulti.m - Gradient descent for multiple variables
- (†) featureNormalize.m - Function to normalize features
- (†) normalEqn.m - Function to compute the normal equations

> \* indicates files you will need to complete 
> † indicates optional exercises

## Linear Regression with One Variable

### Warm Up Exercise

In `warmUpExercise.m` file, you need to write a program to output an 5x5 identity matrix.

```matlab
function A = warmUpExercise()
%WARMUPEXERCISE Example function in octave
%   A = WARMUPEXERCISE() is an example function that returns the 5x5 identity matrix

A = [];
% ============= YOUR CODE HERE ==============
% Instructions: Return the 5x5 identity matrix 
%               In octave, we return values by defining which variables
%               represent the return values (at the top of the file)
%               and then set them accordingly. 

A = eye(5);
% ===========================================
end

```

### Plot Data

In `plotData.m` file, you need to plot the data using X and y passed as parameters.

```matlab
function plotData(x, y)
%PLOTDATA Plots the data points x and y into a new figure 
%   PLOTDATA(x,y) plots the data points and gives the figure axes labels of
%   population and profit.

figure; % open a new figure window

% ====================== YOUR CODE HERE ======================
% Instructions: Plot the training data into a figure using the 
%               "figure" and "plot" commands. Set the axes labels using
%               the "xlabel" and "ylabel" commands. Assume the 
%               population and revenue data have been passed in
%               as the x and y arguments of this function.
%
% Hint: You can use the 'rx' option with plot to have the markers
%       appear as red crosses. Furthermore, you can make the
%       markers larger by using plot(..., 'rx', 'MarkerSize', 10);

plot(x,y,'rx','MarkerSize',8)
% ============================================================
end

```

### Compute Cost Function

Cost function is one of the important parts in week 2. Following is the formula of cost function:
$$
J(\theta_0, \theta_1,...,\theta_n) = \frac{1}{2m}\sum_{i=1}^{m}(h_{\theta}(x^{(i)}) - y^{(i)})^2
$$
Because we had $h_{\theta}(x) = \theta_0 + \theta_1x$, it equals to $h_{\theta}(x) = \theta_0x_0 + \theta_1x_1$ where $x_0 = 1$. So we can simplify the linear model:

$$
\begin{align} h_{\theta}(x) &= \theta_0 + \theta_1x \\
&= h_{\theta}(x) = \theta_0x_0 + \theta_1x_1 \space \# x_0=1 \\
&= \theta^{T}x \end{align}
$$

Then we can define uppercase X as "designed matrix", which contains all training examples:

$$
X = \left[\\
\begin{matrix} x_0^{1} & x_1^{1} &...& x_n^{1} \\
x_0^{2} & x_1^{2} &...& x_n^{2} \\
x_0^{3} & x_1^{3} &...& x_n^{3} \\
...&...&...&...\\
x_0^{m} & x_1^{m} &...& x_n^{m} \end{matrix}\right],\space y = \left[ \\
\begin{matrix} y^1\\
y^2\\
y^3\\
...\\
y^m \\
\end{matrix}\right]
$$

So above cost function can be simplified as follows:

$$
J(\theta) = \frac{1}{2m}(\theta^{T}X - y)^2
$$

Finally we have the correct code:

```matlab
function J = computeCost(X, y, theta)
%COMPUTECOST Compute cost for linear regression
%   J = COMPUTECOST(X, y, theta) computes the cost of using theta as the
%   parameter for linear regression to fit the data points in X and y

% Initialize some useful values
m = length(y); % number of training examples

% You need to return the following variables correctly 
J = 0;

% ====================== YOUR CODE HERE ======================
% Instructions: Compute the cost of a particular choice of theta
%               You should set J to the cost.
J = (1 / (2 * m)) * sum((X * theta - y) .^ 2);

% =========================================================================

end
```

### Gradient Descent

Andrew Ng has clarified very clearly in the video lectures, so let's see the final answer:

```matlab
function [theta, J_history] = gradientDescent(X, y, theta, alpha, num_iters)
%GRADIENTDESCENT Performs gradient descent to learn theta
%   theta = GRADIENTDESCENT(X, y, theta, alpha, num_iters) updates theta by 
%   taking num_iters gradient steps with learning rate alpha

% Initialize some useful values
m = length(y); % number of training examples
J_history = zeros(num_iters, 1);

for iter = 1:num_iters

    % ====================== YOUR CODE HERE ======================
    % Instructions: Perform a single gradient step on the parameter vector
    %               theta. 
    %
    % Hint: While debugging, it can be useful to print out the values
    %       of the cost function (computeCost) and gradient here.
    %
    
    temp = zeros(length(theta), 1);
    for it = 1:length(temp)
        temp(it) = theta(it) - (alpha / m) * sum((X * theta - y) .* X(:,it));
    end
    theta = temp;

    % ============================================================

    % Save the cost J in every iteration    
    J_history(iter) = computeCost(X, y, theta);

end

end

```

As Andrew mentioned in video lecture we need to **simultaneously** update $\theta_j$, so we created one temp vector to save $\theta_0$ and $\theta_1$.

## Linear Regression with Multiple Variables (Optional)

