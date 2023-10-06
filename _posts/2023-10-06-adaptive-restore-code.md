---
layout: post
title: "Guide to adapt_rstr"
date: 2023-10-06 08:55:00 +0100
categories: jekyll update
---

The repository `adapt_rstr` contains the code used for the experiments in McKimm et al. (2022). This post will give overviews of the code and the method for performing the experiments.

## Overview

The directory is structured around four main classes. These are `LogPost`, `RegenDist`, `MCMC` and `BMRestore`. These represent posteriors distribuitons, regeneration distributions, MCMC algorithms and Brownian motion Restore processes. The classes interact with each other. For example, objects of type `MCMC` are constructed using an object of type `LogPost`. The classes all have subclasses. For example, `LogPost` has subclasses `Pump` and `LogReg` to represent the posterior distributions of a hierarchical model of pump failures and a logistic regression model, respectively. We now look at each class in turn.

### Posterior Distributions

Class `LogPost` represents posterior distributions. Its subclasses and the distributions they represent are given in the table below

| Class                | Header file          | Distribution                                      |
| -----                | ------------         | ---------------------                             |
| `BetaTf`             | `beta_dist.h`        | Transformed Beta distribution                     |
| `GaussTarg`          | `gauss_targ.h`       | Gaussian                                          |
| `GaussMixTarg`       | `gm_targ.h`          | Gaussian mixture distribuiton                     |
| `LogGaussCox`        | `log_gauss_cox.h`    | Posterior of log-Gaussian Cox point process model |
| `LogReg`             | `log_reg.h`          | Posterior of logistic regression model            |
| `Pump`               | `pump.h`             | Posterior of pump failure model                   |
| `MVT`                | `t_dist_class.h`     | Multivariate t-distribution                       |

An important member functions for class `LogPost` is `transform_direct`, which makes a linear transformation of the distribuiton. Three more important member functions are: `log_dens`, `update_grad_log_dens` and `laplacian_log_dens`. These may be used to evaluate the log-density, grad-log-density and laplacian-log-density of the distribution.

### Regeneration distributions

Class `RegenDist` (header file `regen_dist.h`) represents regeneration distribuitons ($$\mu$$ and $$\mu_0$$ in the paper). Its important member functions are `log_dens` and `rmu`, which may be used to evaluate its log-density and generate samples from it.

Class `Gaussian` (header file `gaussian.h`) is a subclass of `RegenDist`. An object of this type may be constructed by specifying its mean and covariance matrix.

### MCMC algorithms

Class `MCMC` (header file `mcmc.h`) represents an MCMC algorithm. Class `RWM` (header file `rwm.h`) is a subclass used to represent Markov chains generated using the random walk Metropolis algorithm. Parameters for the algorithm are set upon construction of an object of type `RWM`. Then method `rwm` is used to generate the Markov chain.

### Brownian motion Restore algorithms

Class `BMRestore` (header file `bmrstr.h`) represents a Brownian motion Restore process. Subclass `AdaptRstr` (header file `adapt_rstr.h`) represents and adaptive Restore process. Finally, `ShortTermMemoryRstr` (header file `stm_rstr.h`), a subclass of `AdaptRstr`, represents an adaptive Restore process with short-term memory. Method `gen_t` is used to generate the process for a simulation time $$t$$.

## Examples

The experiments are contained in the subdirectory `examples`. Most experiments are performed using the script `main.cpp`, which is used to sample a target distribution using a given algorithm. The input to `main.out` is data that describes the target distribution and the algorithm used for sampling. Most of the experiments use a shell script to generate an input text file `input.txt`, then pass this file to `main.out`.

## References

McKimm, H., Wang, A. Q., Pollock, M., Robert, C. P., and Roberts, G. O. (2022). [Sampling using Adaptive Regenerative Processes](https://arxiv.org/abs/2210.09901).
