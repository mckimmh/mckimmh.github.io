---
layout: post
title: "Statistical Challenges in Modern Astronomy VIII"
date: 2023-07-25 11:28:00 +0100
categories: jekyll update
---
The conference "Statistical Challenges in Modern Astronomy VIII" (SCMA) at Penn State University, 12-16 June 2023, was a great experience!

## What *are* the statistical challenges in modern astronomy?

### Astronomers have A LOT of data

Astronomers are preparing for a number of astronomical surveys that are launching in the next few years. One of these is the [Legacy Survey of Space and Time](https://rubinobservatory.org/explore/lsst), conducted by the [Rubin Observatory](https://rubinobservatory.org/about). This survey will produce about 20 terabytes of data every night for ten years! Whilst it's great to have such large quantities of data, it poses computational problems.

1. You may want to repeat the same analysis on each image. Then the more images you have, the greater the cost of doing all the computation.

2. In a Bayesian hierarchical model, the number of parameters in the model scales with the size of the data. Thus a fully Bayesian analysis of the model becomes more difficult as the data size increases.

3. MCMC methods evaluate the full likelihood at each step. Thus as the data size increases, the cost of evaluating the likelihood increases. By comparison, optimization methods such as stochastic gradient descent subsample the data, so that a noisy approximation of the gradient of the loss function is computed at each step, which depends on only a small portion of the data. 


### Astronomers have Complex Models
