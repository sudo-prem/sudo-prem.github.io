+++
title = "Linear Regression From Scratch"
description = "Implementing linear regression algorithm from scratch using Python"
date = 2025-12-26
categories = ["__00__Walking Down The Loss Curve"]
tags = ["walking-down-the-loss-curve", "linear-regression", "python", "from-scratch", "ml-algorithms"]
+++

## Introduction

Linear regression is one of the fundamental algorithms in machine learning. In this post, we'll implement linear regression from scratch using Python, understanding the mathematics behind it and building our own gradient descent optimizer.

## What is Linear Regression?

Linear regression is a supervised learning algorithm used to predict continuous values. It models the relationship between dependent variable y and one or more independent variables X by fitting a linear equation to observed data.

## Mathematics Behind Linear Regression

The hypothesis function for linear regression is:

h(x) = θ₀ + θ₁x

Where:
- θ₀ is the y-intercept
- θ₁ is the slope
- x is the input feature

## Cost Function

We use Mean Squared Error (MSE) as our cost function:

J(θ₀, θ₁) = (1/2m) Σ(h(xᵢ) - yᵢ)²

Where m is the number of training examples.

## Implementation

Let's implement linear regression from scratch:

```python
import numpy as np

class LinearRegression:
    def __init__(self, learning_rate=0.01, n_iterations=1000):
        self.learning_rate = learning_rate
        self.n_iterations = n_iterations
        self.weights = None
        self.bias = None
    
    def fit(self, X, y):
        n_samples, n_features = X.shape
        
        # Initialize parameters
        self.weights = np.zeros(n_features)
        self.bias = 0
        
        # Gradient descent
        for _ in range(self.n_iterations):
            y_predicted = np.dot(X, self.weights) + self.bias
            
            # Compute gradients
            dw = (1/n_samples) * np.dot(X.T, (y_predicted - y))
            db = (1/n_samples) * np.sum(y_predicted - y)
            
            # Update parameters
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
    
    def predict(self, X):
        return np.dot(X, self.weights) + self.bias
```

## Example Usage

```python
# Generate sample data
np.random.seed(42)
X = 2 * np.random.rand(100, 1)
y = 4 + 3 * X + np.random.randn(100, 1)

# Create and train the model
model = LinearRegression(learning_rate=0.01, n_iterations=1000)
model.fit(X, y)

# Make predictions
predictions = model.predict(X)
```

## Conclusion

We've successfully implemented linear regression from scratch! This implementation helps us understand the core concepts of gradient descent and how machine learning algorithms optimize their parameters.

In the next post, we'll explore decision trees, another fundamental algorithm in machine learning.