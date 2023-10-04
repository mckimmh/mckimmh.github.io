---
layout: post
title: "Inference for a Bayesian Spectral Model with PyMC"
date: 2023-10-04 12:00:00 +0100
categories: jekyll update
---

## Introduction

The [PyMC homepage](https://www.pymc.io/welcome.html) describes PyMC as *"a probabilistic programming library for Python that allows users to build Bayesian models with a simple Python API and fit them using Markov chain Monte Carlo (MCMC) methods"*. The objective of this post is to demonstrate how PyMC may be used to quite easily fit a Bayesian spectral model. In particular, it will provide:
* An introduction to the Bayesian spectral model.
* How to use PyMC to sample the model.
* How to save samples to a `.nc` file so that sampling may be separated from downstream analyses, such as making kerndel density estimate plots.

## The Bayesian Spectral Model

The spectrum of an astronomical source (such as a star) is the distribution of the energies of the photons emitted by the source. The spectrum may be used to learn about properties of the source such as its chemical composition, temperature and relative velocity. A spectral model specifies the rate $$\lambda$$ at which photons with energy $$e$$ are emitted from the source. Thus $$\lambda$$ is a function of $$e$$. For example, a simple model is $$\lambda(e) = \alpha \exp \{ - \beta e \}$$, in which case inference consists of learning the parameters $$\boldsymbol{\theta} = (\alpha, \beta)^T$$. We write $$\lambda(e, \boldsymbol{\theta})$$ for the spectrum, since it is a function of energy and the parameters $$\boldsymbol{\theta}$$.

The data used for inferring the spectrum is the energies of individual X-ray photons, recorded by a space-based telescope. The difficulty is that the mechanism by which the photons is recorded is quite complicated.

The first modelling assumption is that the energy of photons is discrete. The range of possible energies is divided into a number of *bins* and it is assumed that a photon's energy can only take values equal to the midpoints of the bins. For example, the bins may be $$[0.30, 0.31], [0.31, 0.32], \dots, [10.99, 11.00]$$ (the units are keV), in which it is assumed that a photon may take a value in $$\{ 0.305, 0.315, \dots, 10.995 \}$$. For $$m$$ energy bins, we denote the energy bin mid-points by $$\mathcal{B}_1, \mathcal{B}_2, \dots, \mathcal{B}_m$$.

As a consequence, the spectrum may be represented as a vector of length $$m$$. In particular, the rate at which photons with energy in each bin arrive at the telescope is:

$$
\boldsymbol{\lambda}(\boldsymbol{\theta}) = (\lambda(\mathcal{B}_1, \boldsymbol{\theta}), \dots, \lambda(\mathcal{B}_m, \boldsymbol{\theta}))^T.
$$

An example of this represenation of a telescope is given in the figure below.

{:refdef: style="text-align: center;"}
![Spectrum](/assets/images/pymc_post/true_spec.png)
{: refdef}

Next, not all photons that reach the telescope are necessarily recorded. In fact, many of the photons are deflected and only a portion are actually recorded. The probability that a photon is recorded depends on its energy and may be represented by a vector $$\boldsymbol{\pi}_{\text{ARF}}$$. An example of a vector $$\boldsymbol{\pi}_{\text{ARF}}$$ is given below.

{:refdef: style="text-align: center;"}
![ARF](/assets/images/pymc_post/arf.png)
{: refdef}

Thus the rate at which photons in each energy bin are recorded by the detector is:

$$
\boldsymbol{\xi}(\boldsymbol{\theta}) = \boldsymbol{\lambda}(\boldsymbol{\theta}) \odot \boldsymbol{\pi}_{\text{ARF}}.
$$

Continuing the example, the figure below is a plot of this rate.

{:refdef: style="text-align: center;"}
![arrival_rate](/assets/images/pymc_post/arrival_rate.png)
{: refdef}

We assume for this analysis that the source is a point. That is, it is very far away and is effectively a dot. As a consequence, all the incident photons would hit a single pixel, if it were not for something called the *Point Spread Function* (PSF). The effect of the PSF is that there is some probability that photons are actually recorded in neighbouring pixels. For example, it may be the case that the photons are recorded on a 3 by 3 grid of pixels, with the probability of the photons hitting each pixel given by the following diagram.

{:refdef: style="text-align: center;"}
![psf](/assets/images/pymc_post/psf.png)
{: refdef}

The PSF creates categories of pixels; all pixels in the same category have the same probability that a photon gets deflected into that pixel. Let $$\boldsymbol{\pi}_{\text{PSF}}$$ be a vector, with $$\boldsymbol{\pi}_{\text{PSF}}(k)$$ the probability that a photon is recorded in a *single* pixel of category $$k$$, for $$k=1,\dots,K$$. For the example above, we have $$\boldsymbol{\pi}_{\text{PSF}} = [0.90, 0.015, 0.01]$$. For a single pixel in category $$k$$, the rate at which photons in each energy bin are recorded by the detector is

$$
\boldsymbol{\xi}^{(k)}(\boldsymbol{\theta}) = \boldsymbol{\pi}_{\text{PSF}}(k) \boldsymbol{\xi}(\boldsymbol{\theta}).
$$

Photons are not actually recorded in energy bins. Instead, photons are recorded in energy channels. There is a probabilistic mapping from bins to channels. The channels don't have a unit; for a photon to be recorded in channel $$i$$ is to say that it was recorded in the channel with index $$i$$. The mapping from bins to channels is given by a matrix $$\boldsymbol{M}$$ called the RMF.

{:refdef: style="text-align: center;"}
![rmf](/assets/images/pymc_post/rmf_xray_data_primer.png)
{: refdef}
*Example of a RMF, taken from "An X-ray Data Primer: What I Wish I Knew when Starting X-Ray Astronomy".*

The rate at which photons are recorded in each energy channel, for a single pixel in category $$k$$, is then

$$
\boldsymbol{\mu}^{(k)} = \boldsymbol{M} \boldsymbol{\xi}^{(k)}.
$$

Finally, for $$\Delta_b$$ the width of the energy bins and $$\Delta_t$$ the length of time of the observation, the actual number of photons recorded has a Poisson distribuiton:

$$
\boldsymbol{Y}^{(k)} \sim \text{Poisson}(\Delta_b \Delta_t \boldsymbol{\mu}^{(k)}).
$$

A graphical represenation of the model is given in the figure below.

{:refdef: style="text-align: center;"}
![graphical_model](/assets/images/pymc_post/pileup_not_accounted_graph_model.png)
{: refdef}


## Defining the Model using PyMC

PyMC allows us to express this model with very readable code. First, we import the following libraries:
    
    import math
    import numpy as np
    import pandas as pd
    import pymc as pm

Suppose that we have loaded the channel counts as a two-dimensional `np.array` called `counts`, with a row for each PSF pixel category. Further, suppose that the energy bins have been loaded as `energy_bins`, $$\boldsymbol{\pi}_{\text{ARF}}$$ as `arf` and $$\boldsymbol{M}$$ as `rmf`, all as objects of type `np.ndarray`. In addition, suppose that we have loaded the fixed parameters as a `pd.DataFrame` with column `value` and indices giving the names of the fixed parameters. Parameters $$\Delta_b$$ and $$\Delta_t$$ are stored in variables `bin_width` and `observation_time`. For this example, the functional form of the spectrum is

$$
\lambda(e) = \alpha_1 \exp \{ -\beta e \} + \alpha_2 N(e; \mu, \sigma^2),
$$

where $$N(e; \mu, \sigma^2)$$ is the density of a Normal distribution with mean $$\mu$$ and variance $$\sigma^2$$. We fix the parameters $$\alpha_1, \alpha_2, \mu$$ and $$\sigma$$ at their true values and try to infer $$\beta$$. We can then specify the model using:

    with pm.Model() as mdl:

        # Fixed parameters
        alpha1 = pm.ConstantData("alpha1", params["value"]["alpha1"])
        alpha2 = pm.ConstantData("alpha2", params["value"]["alpha2"])
        sigma = pm.ConstantData("sigma", params["value"]["sigma"])
        mu = pm.ConstantData("mu", params["value"]["mu"])

        energy_bins = pm.ConstantData("energy_bins", energy_bins)
        arf = pm.ConstantData("arf", arf)   
        rmf = pm.ConstantData("rmf", rmf)
        psf = pm.ConstantData("psf", psf) 

        # Define priors
        beta = pm.Uniform("beta", 0.01, 5.0)

        # The true spectrum
        continuum = alpha1 * pm.math.exp(-beta * energy_bins)
        line = (alpha2 / (sigma * pm.math.sqrt(2.0*math.pi))) *  pm.math.exp(-0.5 * (((energy_bins-mu)/sigma)**2.0))
        spectrum = continuum + line

        # Effects of the ARF, PSF and RMF
        arrival_rate = spectrum * arf
    
        psf_rate0 = psf[0] * arrival_rate
        psf_rate1 = psf[1] * arrival_rate
        psf_rate2 = psf[2] * arrival_rate

        chnnl_mean0 = bin_width * observation_time * pm.math.dot(rmf, psf_rate0)
        chnnl_mean1 = bin_width * observation_time * pm.math.dot(rmf, psf_rate1)
        chnnl_mean2 = bin_width * observation_time * pm.math.dot(rmf, psf_rate2)

        y0 = pm.Poisson("y0", mu=chnnl_mean0, observed=counts[0,:])
        y1 = pm.Poisson("y1", mu=chnnl_mean1, observed=counts[1,:])
        y2 = pm.Poisson("y2", mu=chnnl_mean2, observed=counts[2,:]) 

Let's go through some of the key steps in the above. Firstly, the line `with pm.Model() as mdl:` creates a new `Model` object and creates a context manager. The `Model` object is a container for the model's random variables and (because of the context manager) all the statements in the indented block are added to the model.

PyMC uses [PyTensor](https://pytensor.readthedocs.io/en/latest/) to translate models into fast machine code. The PyTensor library uses generalized array data structures called tensors. A feature of PyTensor is efficient symbolic differentiation, which is useful for MCMC simulation, since some algorithms require the derivative of the log posterior density. The function [`pm.ConstantData`](https://www.pymc.io/projects/docs/en/stable/api/generated/pymc.ConstantData.html#pymc.ConstantData) lets PyMC know that the variable is a constant. In particular, the `pm.ConstantData` function registers the `value` as a `TensorConstant`.

The line `beta = pm.Uniform("beta", 0.01, 5.0)` defines a prior distribution for the model's random variable. The random variable is given is a name, which matches the name of the Python variable that they are assigned to.

The vector $$\boldsymbol{\lambda}$$ is given by `spectrum` and $$\boldsymbol{\xi}$$ by `arrival_rate`. The variables `psf_rate0`, `psf_rate1` and `psf_rate2` define the vectors $$\{ \boldsymbol{\xi}^{(k)} \}_{k=1}^K$$, where $$K=3$$. Finally, the variables `chnnl_mean0`, `chnnl_mean1`, `chnnl_mean2` represent the channel means: $$\{ \Delta_b \Delta_t \boldsymbol{\mu}^{(k)} \}_{k=1}^K$$.

Lastly, the observed data is `y0`, `y1`, `y2`, with as `y0 = pm.Poisson("y0", mu=channel_mean0, observed=counts[0,:])` and `y1`, `y2` defined similarly. We call these stochastic variables *observed stochastics*. The `observed` argument passes the data to the variable.

## Sampling the Posterior Distribution

We can then sample from the posterior distribution of the model and save the relevant samples to a file. By default, PyMC generates multiple chains. The number of chains simulated depends on the number of cores on your machine; on my computer 4 chains are simulated in parallel. I had definded `draws` as $$25000$$, so in total $$10^5$$ samples were generated, which took 146 seconds.

    with mdl:
        inf = pm.sample(draws=draws)
        inf.to_netcdf("samples.nc") 

## Plotting the results

You can load the samples into a different file to analyse them (it helps to separate sampling and analysis of samples like this). The figure below shows  kernel density estimates of the posterior and and trace plots of the Markov chains simulated. 

{:refdef: style="text-align: center;"}
![results](/assets/images/pymc_post/kde_and_trace_pileup_not_accounted.png)
{: refdef}

## Conclusion

Bayesian spectral models may be used to infer the parameters of the spectrum of an astronomical source, such as a star. PyMC is very helpful for quickly defining a model and sampling its posterior distribution. It's a good idea to have separate files for sampling the posterior distribution and analysing the samples.
