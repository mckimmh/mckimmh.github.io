---
layout: post
title: "Inference for a Bayesian Spectral Model with PyMC"
date: 2023-08-16 09:00:00 +0100
categories: jekyll update
---

## Introduction

The [PyMC homepage](https://www.pymc.io/welcome.html) describes PyMC as *"a probabilistic programming library for Python that allows users to build Bayesian models with a simple Python API and fit them using Markov chain Monte Carlo (MCMC) methods."*

Post objectives:
* Introduction to a Bayesian Spectral Model
* Introduction to using PyMC via an example of a Bayesian Spectral Model
* When to use `pm.Deterministic`
* How to save samples to a `.nc` file and plot using a different script

## The Bayesian Spectral Model

Let the vector of counts in each energy channel be $$\boldsymbol{Y} \in \mathbb{R}^n$$. Typically, $$n=1024$$. The parameters for the spectrum are given by a vector $$\boldsymbol{\theta} \in \mathbb{R}^d$$. The mid-point of the energy bins are $$\mathcal{B}_1, \mathcal{B}_2, \dots, \mathcal{B}_m$$. For example, there might be $$m=1070$$ energy bins, with mid-points $$0.305, 0.315, \dots, 10.995$$. The width of the energy bins is denoted by $$\delta$$, which in the above example is $$0.01$$. A function $$\lambda : \mathbb{R}^+ \times \mathbb{R}^d \rightarrow \mathbb{R}^+$$ maps pairs of energy-bins and spectral parameter vectors to the rate at which photons with energy in that bin are emitted from the source. The expected number of photons that reach the telescope, with energy in each bin, is thus given by a vector

$$ \boldsymbol{\lambda}(\boldsymbol{\theta}) = \delta \cdot (\lambda(\mathcal{B}_1, \boldsymbol{\theta}), \dots, \lambda(\mathcal{B}_m, \boldsymbol{\theta}))^T. $$

However, not all photons that reach the telescope are necessarily recorded. In fact, many of the photons are deflected and only a portion are actually recorded. The probability that a photon is recorded depends on its energy and may be represented by a vector $$\boldsymbol{d}$$. Thus the expected number of photons in each energy bin that are absorbed by the detector is:

$$ \boldsymbol{\eta}(\boldsymbol{\theta}) = \boldsymbol{\lambda}(\boldsymbol{\theta}) \odot \boldsymbol{d}. $$

The detector records photons in energy channels. There is a probabilistic mapping from energy bins to channels, which may be presented as a matrix $$\boldsymbol{M}$$. The expected number of photons in each energy channel is therefore

$$ \boldsymbol{\xi}(\boldsymbol{\theta}) = \boldsymbol{M} \boldsymbol{\eta}(\boldsymbol{\theta}).$$

The actual number of photons recorded in each channel is Poisson distributed, with mean $$ \boldsymbol{\xi}(\boldsymbol{\theta}) $$. In full, the Bayesian Spectral model may be expressed as:

$$ \boldsymbol{Y} \sim \text{Poisson}(\boldsymbol{\xi}(\boldsymbol{\theta})), $$

$$ \boldsymbol{\xi}(\boldsymbol{\theta}) = \boldsymbol{M} \boldsymbol{\eta}(\boldsymbol{\theta}),$$

$$ \boldsymbol{\eta}(\boldsymbol{\theta}) = \boldsymbol{\lambda}(\boldsymbol{\theta}) \odot \boldsymbol{d}, $$

$$ \boldsymbol{\lambda}(\boldsymbol{\theta}) = \delta \cdot (\lambda(\mathcal{B}_1, \boldsymbol{\theta}), \dots, \lambda(\mathcal{B}_m, \boldsymbol{\theta}))^T. $$

## Defining the Model using PyMC

PyMC allows us to express this model with a few lines of code. First, we import the following libraries:
    
    import math
    import numpy as np
    import pandas as pd
    import pymc as pm

Suppose that we have loaded the channel counts as `counts`, the energy bins as `energy_bins_array`, the detection probability vector as `detection_prob_array` and the blurring matrix as `blurring_mat_array`, all as objects of type `np.ndarray`. In addition, suppose that we have loaded the fixed parameters as a `pd.DataFrame` with column `value` and indices giving the names of the fixed parameters. We can then specify the model using:

    with pm.Model() as mdl:

        # Fixed variables
        alpha1 = pm.ConstantData("alpha1", params["value"]["alpha1"])
        alpha2 = pm.ConstantData("alpha2", params["value"]["alpha2"])
        sigma = pm.ConstantData("sigma", params["value"]["sigma"])
        energy_bins = pm.ConstantData("energy_bins", energy_bins_array)
        detection_prob = pm.ConstantData("detection_prob", detection_prob_array)
        blurring_mat = pm.ConstantData("blurring_mat", blurring_mat_array.T)

        # Define priors
        mu = pm.Uniform("mu", 3.0, 7.0)
        beta = pm.Uniform("beta", 0.01, 5.0)

        continuum = alpha1 * pm.math.exp(-beta * energy_bins)
        line = (alpha2 / (sigma * pm.math.sqrt(2.0*math.pi))) *  pm.math.exp(-0.5 * (((energy_bins-mu)/sigma)**2.0))
        spectrum = continuum + line
        detected_spectrum = spectrum * detection_prob
        channel_mean = pm.math.dot(blurring_mat,detected_spectrum)

        y = pm.Poisson("y", mu=channel_mean, observed=counts)  

Let's go through some of the key steps in the above. Firstly, the line `with pm.Model() as mdl:` creates a new `Model` object and creates a context manager. The `Model` object is a container for the model's random variables and (because of the context manager) all the statements in the indented block are added to the model.

PyMC uses [PyTensor](https://pytensor.readthedocs.io/en/latest/) to translate models into fast machine code. The PyTensor library uses generalized array data structures called tensors. A feature of PyTensor is efficient symbolic differentiation, which is useful for MCMC simulation, since some algorithms require the derivative of the log posterior density. The function [`pm.ConstantData`](https://www.pymc.io/projects/docs/en/stable/api/generated/pymc.ConstantData.html#pymc.ConstantData) lets PyMC know that the variable is a constant. In particular, the `pm.ConstantData` function registers the `value` as a `TensorConstant`.

The lines `mu = pm.Uniform("mu", 3.0, 7.0)` and `beta = pm.Uniform("beta", 0.01, 5.0)` define prior distributions for the two random variables in the model. The variables are given names, which match the names of the Python variables that they are assigned to.

The lines defining `continuum`, `line`, `spectrum`, `detected_spectrum` and `channel_mean` all describe determinstic transformations of the registered variables.

Lastly, the observed data is modelled as `y = pm.Poisson("y", mu=channel_mean, observed=counts)`. We call this stochsatic variable an *observed stochastic*. The `observed` argument passes the data to the variable.

## Sampling the Posterior Distribution

We can then sample from the posterior distribution of the model and save the relevant samples to a file using:

    with mdl:
        inf = pm.sample(chains=1, draws=n_samples)

        samples = inf["posterior"][["mu", "beta"]].to_dataframe()
        samples = samples.reset_index(drop=True)

        samples.to_csv("samples.csv")

## Conclusion

## DELETE LATER

Another key function is [`pm.Deterministic`](https://www.pymc.io/projects/docs/en/stable/api/generated/pymc.Deterministic.html), which we use to define `continuum`, `line`, `spectrum`, `detected_spectrum` and `channel_mean`. This function creates a named determinstic variable. They are deterministic in the sense that they are a transformation of other variables and don't actually add any randomness themselves.
