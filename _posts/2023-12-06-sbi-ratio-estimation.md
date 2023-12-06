---
layout: post
title: "The Ratio Estimation Method for Simulation-Based Inference"
date: 2023-12-06 13:40:00 +0000
categories: jekyll update
---

Simulation-Based Inference (SBI) methods are designed to perform Bayesian inference in the setting where evaluating the likelihood is either extremely expensive or impossible, but simulating data from the model is possible. There are several categories of SBI methods, see Lueckmann et al. (2021) for example. This post concerns the Ratio Estimation (RE) method.

The RE method (Hermans et al., 2020; Durkan et al., 2020) learns an estimate of a ratio of densities that is proportional to the likelihood. This estimator may then be used with MCMC to sample an approximation of the posterior distribution.

Suppose that an observation $$z$$ is generated under one of two hypotheses, $$\mathcal{H}_0$$ or $$\mathcal{H}_1$$.
Let $$p(z | \mathcal{H}_i)$$ be the likelihood of observing $$z$$ given that $$\mathcal{H}_i$$ is true, for $$i \in  \{0, 1\}$$. Define the likelihood ratio as

$$
    r(z | \mathcal{H}_0, \mathcal{H}_1) = \frac{p(z | \mathcal{H}_0)}{p(z | \mathcal{H}_1)}.
$$

If $$z$$ is generated under $$\mathcal{H}_0$$, then label it with $$y=1$$. Otherwise, if $$z$$ is generated under $$\mathcal{H}_1$$, then label it with $$y=0$$. Define

$$
    d^*(z) = p(y=1 | z) = \frac{p(z | \mathcal{H}_0)}{p(z | \mathcal{H}_0) + p(z | \mathcal{H}_1)}.
$$

We may write
$$
    r(z | \mathcal{H}_0, \mathcal{H}_1)
$$
as

$$
    r(z | \mathcal{H}_0, \mathcal{H}_1) = \frac{ \left( \frac{p(z | \mathcal{H}_0) }{p(z | \mathcal{H}_0) + p(z | \mathcal{H}_1)} \right) }{ \left( \frac{p(z | \mathcal{H}_1)}{p(z | \mathcal{H}_0) + p(z | \mathcal{H}_1)} \right) }
    = \frac{d^*(z)}{1 - d^*(z)}.
$$

This is called the Likelihood Ratio Trick (LRT). A classifier $$d(z)$$ may be trained on $$\{ (z_i, y_i) \}_{i=1}^n$$, generated from a mixture of $$\mathcal{H}_0$$ and $$\mathcal{H}_1$$, to distinguish samples from $$\mathcal{H}_0$$ and $$\mathcal{H}_1$$. A "good" classifier will recover $$d^*(z)$$. Cranmer et al. (2015) used the LRT for hypothesis testing with the generalized likelihood ratio test statistic. 

The LRT  may be applied to sampling the posterior of a Bayesian model (Hermans et al., 2020; Durkan et al., 2020). Let the observation $$z$$ be $$(x, \theta)$$, for data $$x$$ and model parameters $$\theta$$. Let $$\mathcal{H}_0$$ be that $$(x,\theta)$$ comes from the joint distribution of $$x,\theta$$. Thus, samples $$(x, \theta) \sim p(x, \theta)$$ are labelled $$y=1$$. Let $$\mathcal{H}_1$$ be that $$(x, \theta)$$ comes from the product of the marginal distributions for $$x$$ and $$\theta$$. Thus, samples $$(x, \theta) \sim p(x)p(\theta)$$ are labelled $$y=0$$. Density $$p(\theta)$$ is the prior. We then have

$$
    d^*(x, \theta) = \frac{p(x, \theta)}{p(x, \theta) + p(x)p(\theta)}.
$$

The likelihood ratio is now

$$
    r(x, \theta) = \frac{p(x, \theta)}{p(x)p(\theta)} = \frac{p(\theta | x) p(x)}{p(x)p(\theta)} = \frac{p(\theta | x)}{p(\theta)}.
$$

The posterior can thus be written as:

$$
    p( \theta | x) = r(x, \theta) p(\theta) = \left( \frac{d^*(x,\theta)}{1 - d^*(x, \theta)} \right) p(\theta).
$$

For $$d(x, \theta)$$ a classifier trained to distinguish $$(x,\theta) \sim p(x,\theta)$$ from $$(x,\theta) \sim p(x)p(\theta)$$, the approximation of the posterior is then

$$
    \tilde{p}(\theta | x) = \left( \frac{d(x,\theta)}{1 - d(x, \theta)} \right) p(\theta),
$$

which may be sampled using MCMC.

## References

Cranmer, K., Pavez, J., and Louppe, G. (2015). Approximating Likelihood Ratios with Calibrated Discriminative Classifiers. arXiv

Durkan, C., Murray, I., and Papamakarios, G. (2020). On Contrastive Learning for Likelihood-free Inference. In III, H. D. and Singh, A., editors, Proceedings of the 37th International Conference on Machine Learning, volume 119 of Proceedings of Machine Learning Research, pages 2771–2781. PMLR.

Hermans, J., Begy, V., and Louppe, G. (2020). Likelihood-Free MCMC with Amortized Approximate Ratio Estimators. In Proceedings of the 37th International Conference on Machine Learning, ICML’20. JMLR.org.

Lueckmann, J.-M., Boelts, J., Greenberg, D., Goncalves, P., and Macke, J. (2021). Benchmarking Simulation-Based Inference . In Banerjee, A. and Fukumizu, K., editors, Proceedings of The 24th International Conference on Artificial Intelligence and Statistics, volume 130 of Proceedings of Machine Learning Research, pages 343–351. PMLR.