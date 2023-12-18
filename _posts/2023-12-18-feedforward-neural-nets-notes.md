---
layout: post
title: "Notes on Deep Feedforward Networks"
date: 2023-12-18 11:35:00 +0000
categories: jekyll update
---

The following are notes on Chapter 6 of [Deep Learning](https://www.deeplearningbook.org/) by Goodfellow, Bengio and Courville.

The goal of a Feedforward Neural Network (FNN), also known as a Multilayer Perceptron (MLP) is to approximate some function $$f^*$$. A Neural Network (NN) defines a mapping

$$
    \boldsymbol{y} = f(\boldsymbol{x}, \boldsymbol{\theta}).
$$

This function in turn may be represented by composing many different functions. For example, dropping the dependence on $$\boldsymbol{\theta}$$:
$$
    f(\boldsymbol{x}) = f^{(3)}( f^{(2)}( f^{(1)}(\boldsymbol{x}) ) ).
$$
The length of the chain of functions is the depth of the NN. The final layer is called the *output* layer; other layers are called *hidden*. The function $$f$$ is trained to approximate $$f^*$$. An alternative representation of the layers is of many units acting in parallel, each representing a vector-to-scalar function. The NN may also be represented as a directed acyclic graph.

## Example: learning XOR

The XOR function is an operator on binary variables $$x_1$$ and $$x_2$$, which returns 1 if exactly one variable is 1 and otherwise returns 0. There are four possible inputs. Suppose that we want to use a model to perfectly learn the relationship between input and output. We can show that a linear model has weights 0 and bias $$0.5$$. Thus, a linear model predicts $$0.5$$ for all inputs, which is not very useful!

A single-layer NN is able to perfectly learn the function. Crucial to doing this it to break free of linearity. A network can do this by using Rectified Linear Units (ReLUs). This means that a linear transformation of the inputs is passed through the ReLU function

$$
    g(z) = \max (0,z).
$$

## Gradient-Based Learning

The nonlinearity of the NN means that most loss functions are *nonconvex*. Training uses iterative gradient-based optimizers (with no convergence guarantees), which are sensitive to the initial parameters.

# Cost Functions

NNs are trained using the principle of maximum likelihood. This means that the cost function is the negative log-likelihood, or equivalently, the cross-entropy between the training data and the model distribution. Letting $$\hat{p}_{\text{data}}$$ be the true distribution of the data and $$p_{\text{model}}$$ be the density defined by the NN, then the cost function is:

$$
    J(\boldsymbol{\theta}) = - \mathbb{E}_{\boldsymbol{x}, \boldsymbol{y} \sim \hat{p}_{\text{data}}}\left[ \log p_{\text{model}}( \boldsymbol{y} | \boldsymbol{x} ) \right].
$$

Since the NN is trained using gradient-based optimizers, we want the gradient $$\nabla_{\boldsymbol{\theta}}J(\boldsymbol{\theta})$$ to be "nice": to be large and predictable. Functions that *saturate* (become very flat) undermine this, since their gradient becomes very close to zero. Activation functions are key to getting nicely behaved cost function gradients.

Note that we may view a NN as a functional: a mapping from functions to real numbers.

# Output Units

Let $$\boldsymbol{h}$$ be hidden features. The *output layer* provides an additional transformation.
* Linear units are often used to produce the mean of a conditional Gaussian distribution. The advantage of linear units is that they don't saturate.
* For binary classification tasks, we may define a Bernoulli distributon over $$y \mid \boldsymbol{x}$$; the NN predicts $$\mathbb{P}(y=1 \mid \boldsymbol{x})$$. The output must be restricted to $$[0,1]$$. We use the logistic sigmoid function, 

$$ \hat{y} = \sigma(z), \quad z = \boldsymbol{w}^T \boldsymbol{h} + \boldsymbol{b}, \quad \sigma(x) = (1 + e^{-x})^{-1}$$

* For classification tasks with $$ n $$ possible values, we can define a Multinoulli distribution over $$ y \mid \boldsymbol{x} $$. We use the *softmax* function to predict the values $$ \mathbb{P}(y=i \mid \boldsymbol{x}) $$

## Hidden Units

The default choice for the hidden units is ReLU. The design process relies on trial and error.

# Rectified Linear Units and their Generalizations

ReLUs use the activation function $$g(z) = \max \{ 0 , z \}$$. They are easy to optimize because they are similar to linear units, however learning is not possible on examples for which their activation is zero. Generalizations are designed to have non-zero gradient everywhere and have form:

$$
    h_i = g(z_i, \alpha) = \max \{ 0 , z_i \} + \alpha_i \min \{0, z_i \}.
$$

# Logistic Sigmoid and Hyperbolic Tangent

These activation functions saturate across most of their domain, which makes gradient-based learning difficult. Thus their use within FNNs is discouraged. However, sigmoidal units have uses within RNNs, probabilistic models and some autoencoders.

## Architecture Design

*Architecture* refers to the overall structure of the network, such as the number of hidden units and how these are connected. Most NNs are organised into layers, with each layer being a function of the previous layer. The main architectural considerations are then the depth of the network and the width of the layers. Design choices are usually made by a process of trial and error. The validation error serves as the guide during this process.

## Back-Propagation and Other Differentiation Algorithms

Forward propagation refers to the flow of information from the data to the cost function. Backpropagation is then the flow of information backwards, from the cost function to the gradient. Backpropagation is a form of automatic differentiation, which enables us to compute the gradient of the network.
