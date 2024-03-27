---
layout: post
title: "Regularization for Deep Learning"
date: 2024-03-27 11:50:00 +0000
categories: jekyll update
---

The following are notes on Chapter 7 of [Deep Learning](https://www.deeplearningbook.org/) by Goodfellow, Bengio and Courville.

*Regularization* refers to any modification of a learning algorithm that is intended to decrease the generalization error but not the training error. There are many strategies for regularization, including:
 - adding extra constraints on the model, such as restrictions on the parameter values.
 - adding extra terms to the objective function; these correspond to a soft constrain on parameter values.
Constrains and penalties can:
 - encode prior knowledge,
 - express a preference for a simpler model,
 - be necessary for making an undetermined problem determined.
  
*Ensemble methods* are a form of regularization. These combine multiple hypotheses that explain the training data. Generally, the best model has a large number of parameters, which are regularized.

## Parameter Norm Penalties

Adding a parameter norm penalty $$\Omega(\theta)$$ to the objective function $$J$$, the regularized objective function is:

$$
    \tilde{J}(\theta; x, y) = J(\theta; x, y) + \alpha \Omega(\theta),
$$

where $$\alpha \in [0, \infty]$$ is the relative weight of the penalty term. Typically, only the weights are penalised, because regularizing the biases can lead to significant underfitting.

# L2 Parameter Regularization

Also known as *ridge regression*, *Tikhonov regularization* or *weight decay*.

$$
    \Omega(\theta) = \frac{1}{2} || w ||_2^2.
$$

Has the effect of driving the weights closer to the origin. Analysing a quadratic approximation of $$J$$, one can show that the effect of weight decay is to rescale $$w^*$$, the minimum of $$J$$, along the axes defined by the eigenvectors of the Hessian matrix of $$J$$ with respect to $$w$$ evaluated at $$w^*$$. This means that directions along which the parameters contributed a lot to reducing $$J$$ are more or less preserved; but directions that contribute only a little to reducing $$J$$ are decayed.

# L1 Parameter Regularization

The penalty term is:

$$
    \Omega(\theta) = \sum_i | w_i |.
$$

Results in a more sparse solution: more weights are set to zero.

## Norm Penalties as Constrained Optimization

Parameter norm penalization may be thought of as imposing a constraint on the weights. For $$L^2$$ normalization, the region is a ball. For $$L^1$$ normalization, it is the region of limited $$L^1$$ norm (a diamond shape for $$w=(w_1, w_2)^T$$. We don't know the size of the constraint region because the constraint region is imposed implicitly.

May alternatively use explicit constraints. For example, if a step of stochastic gradient descent (SGD) results in $$\Omega(\theta) \ge k$$, then we could project $$\theta$$ back to the nearest point satisfying $$\Omega(\theta) < k$$.

## Regularization and Under-Constrained Problems

Some models (for example linear regression and PCA) depend on inverting $$X^T X$$. This is not possible when $$X^T X$$ singular, which may happen when
 - the data-generating distribution has no variance in some direction
 - no variance is observed is some direction, because $$X$$ has more rows (examples) than columns (features).
Regularization then corresponds to inverting $$X^T X + \alpha I$$ instead.

A problem may be *underdetermined*. For example, a logistic regression model for which the classes are linearly separable is underdetermined. Then if $$w$$ achieves perfect classification, so does $$2 w$$. Thus SGD will continually increase $$\mid w \mid$$. Regularization can guarantee convergence of iterative methods applied to underdetermined problems.

## Dataset Augmentation

Can generate data $$(x, y)$$ to add to the dataset by transforming existing data. For example, for images, one can use:
 - translation
 - rotation
 - scaling
 - injecting noise.

## Noise Robustness

For RNNs, one sometimes adds noise to the weights.

## Semi-Supervised Learning

*Semi-supervised learning* (SSL) uses both unlabelled examples from the marginal distribution of $$x$$ and the joint distribution of $$(x, y)$$ to estimate the conditional distribution $$y \mid x$$. For deep learning, SSL usually consists of learning a representation $$h = f(x)$$, with the aim being that examples from the same class have similar representations.

## Multitask Learning

*Multitask learning* aims to improve generalization by pooling the examples of several tasks. Multitask learning applies when the different supervised learning tasks share the same input and some hidden layer. The tasks must be related somehow: it's assumed that the outputs of the tasks all depend on the same hidden layer.

## Early Stopping

*Early stopping* is one of the most frequently used forms of regularization. Instead of running the optimization procedure until convergence, regularly check the validation error and stop training when this error starts to increase. Evaluating the validation error is costly. It's also necessary to maintain a copy of the best parameters, but the cost of doing this is negligible.

For a linear model with quadratic error function and simple gradient descent, early stopping is in fact equivalent to using $$L^2$$-regularization (weight decay). This is because early stopping restricts the optimization procedure to a small volume of parameter space around the starting point.

## Parameter Tying and Parameter Sharing

It's possible to use penalties to force the parameters of different models to be close to each other. However, more popular is to constrain sets of parameters to be equal, which is called *parameter sharing*. This idea can significantly reduce the memory footprint of the network. Parameter sharing is used by *convolutional neural networks*.

## Sparse Representations

Representational sparsity (where many elements of hidden layer are zero) may be encouraged by adding a term to the cost function that penalises the hidden layers.

## Bagging and Other Ensemble Methods

The term *bagging* is short for *bootstrap aggregation*. It refers to combining several models. The idea is to train multiple models separately and then average their predictions. We can show that, if the errors made by the different models are unbiased and have equal variances, then the expected squared error of the prediction of the ensemble of models is smaller than that of a single model.

Each model in the bagged ensemble has a different dataset. This dataset is constructed by sampling the original dataset, with replacement, so that the constructed dataset has the same size as the original. Thus all models in the ensemble may have the same architecture and objective function, as well as use the same training algorithm. Since NNs generally reach a wide variety of solution points, due to the varying initialization points and selection of minibatches etc, they benefit from model averaging. 

## Dropout

Bagging is expensive, because multiple models are trained. As such, usually no more than 10 models are used.

*Dropout* is a method that provides an inexpensive approximation to bagging a number of neural networks that is exponential in the number of units in the network. The method trains the ensemble of subnetworks formed by removing non-output units from the underlying base network. Units are removed by multiplying their output values by zero. Training uses a minibatch-based learning algorithm, such as SGD. At each step, a random subset of data is selected and a different binary mask is sampled. Typically, the probability of including each input unit is set to 0.8 or 0.5.

Writing $$\mu$$ for the masking vector and $$J(\theta, \mu)$$ for the cost function. Dropout minimizes

$$
    \mathbb{E}_\mu[ J(\theta, \mu) ].
$$

If there are $$n$$ hidden units in the network, then there are $$2^n$$ possible values of $$\mu$$ (since each unit can either be included or excluded). Thus the equation above may be expressed as a sum over a number of terms that is exponential in $$n$$ and is thus hugely computationally expensive to evaluate exactly. However, we can attain an unbiased estimate of its gradient by sampling values of $$\mu$$.

Note that dropout is actually very different to bagging. For dropout, a model is sampled and its parameters updated. Then when a new model is sampled, it shares its parameters with the previously sampled model. We can think of this as all the exponentially many models sharing the parameters they have in common with the full model. By contrast, for the few models used in bagging, each is trained to completion.

Once the model has been trained using dropout, we can either make predictions by sampling (for example) 10-20 masks and averaging their predictions. Or as an approximation, we can use the full model with all weights divided by 2.

When training a model with dropout, be aware that dropout reduces the effective capacity of the model (it's ability to represent functions). Thus you must increase the model's size to counteract this effect.