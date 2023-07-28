---
layout: post
title: "Bayesian Modelling of Photon Pile-up"
date: 2023-07-25 10:49:00 +0100
categories: jekyll update
---
I recently presented a poster on ongoing work on Bayesian modelling of photon pile-up at the conference "Statistical Challenges in Modern Astronomy VIII" at Penn State University in the USA. The aim of the project is to design a *Bayesian* model of the *spectrum* of a high-energy astronomical source, which takes into account a phenomenon called *pile-up*. The model is designed for *X-ray* astronomy data gathered by the *Chandra X-Ray Observatory*.

X-rays are high-energy electromagnetic waves (photons). Quoting from van Dyk and Kang (2004): *Roughly speaking, the production of high-energy electromagnetic waves requires temperatures of millions of degrees and signals the release of deep wells of stored energy such as those in very strong magnetic fields, extreme gravity, explosive nuclear forces and shock waves in hot plasmas. Thus, X-ray telescopes can map nearby stars (like our Sun) that have active magnetic fields, the remnants of exploding stars, areas of star formation, regions near the event horizon of a black hole, very distant but very turbulent galaxies or even the glowing gas that embeds a cosmic cluster of galaxies.* X-rays are absorbed by the Earth's atmosphere, so X-ray telescopes (such as Chandra) are space-based. X-ray data is low-count; the telescope is designed to record the energies of individual photons hitting its detector.

The spectrum of an astronomical source is the distribution of the energies of the photons emitted by the source. The spectrum may be used to learn about properties of the source such as its chemical composition, temperature and relative velocity.

A spectral model is a mathematical representation of a spectrum. Such a model specifies the relationship between the energy and the intensity of the spectrum. That is, it states how frequently photons with a particular energy are emitted. The intensity $$\lambda$$ may be written as a function of the energy $$e$$. We aim to infer the parameters $$\theta$$ of the model, using X-ray data. For example, if $$\lambda(e) = \alpha \exp \{ - \beta e \}$$, then we would like to learn about $$\theta = (\alpha, \beta)$$.

A Bayesian model is one for which we use *Bayesian inference* to learn about the unknown parameters. In Bayesian inference, the parameters of a model are more often called *variables*. A *prior* belief about the distribution of the variables is *updated*, based on observed data, to a *posterior* belief, again represented by a probability distribution. For example, before observing any data, we may believe that the variable $$\alpha$$ is somewhere between 0 and 5, but we think that it is equally likely to take any value between these lower and upper bounds. Then having seen the data (and used a method for Bayesian inference), our belief about $$\alpha$$ may be represented by a posterior distribution that, for example, states that in expectation $$\alpha$$ has value 1.2 and that there is a 95% probability that $$\alpha$$ is between 1.0 and 1.4. The main advantage of Bayesian inference is its *quantification of uncertainty*: downstream analyses are able to take into account all possible values of the model's variables, not just the values that are most likely.

Pile-up occurs when two or more photons arrive at the X-ray telescope's detector during the same interval of time. The detector records the energy of photons arriving at the detector's surface during short time-intervals, usually of a few seconds in duration. If two or more photons arrive during the same time-interval, then the sum of their energies is recorded as the energy of a single photon. For example, in the figure below, the exact arrival times of photons are represented by crosses and time-intervals are demarcated by lines. In the first time-interval, two photons have arrived: the sum of the energies of these photons will be recorded as the energy of a single photon.

![Piled Photons](/assets/images/exact_arrivals_t_intvls.png)

# Visualising the Effect of Pile-up

Failing to account for pile-up causes inaccuracies in the spectral analysis. The figure below shows a spectrum consisting of a *continuum* term and a single *emission line*. The continuum term describes the spectrum over the entire energy range. The emission line indicates a large amount of emission of photons in a small energy-range, caused by photons being emitted when an electron falls to a lower energy shell of a specific ion. Here, the intensity of the spectrum is $$\lambda(e) = \alpha_1 \exp\{ -\beta e \} + \alpha_2 N(e; \mu, \sigma^2) $$, where $$N(e; \mu, \sigma^2)$$ is the probability density function of a normal distribution. We want to infer the parameters $$\boldsymbol{\theta} = (\alpha_1, \alpha_2, \beta, \mu)$$, ($$\sigma$$ is fixed as its true value).

![Spectrum](/assets/images/spec.png)

Simulating photons from this spectrum and simulating the effect of pile-up produces the observed spectrum shown below. We see here that there are three effects of pile-up. Firstly, the spectrum hardens, which is to say that more high-energy photons are recorded. Secondly, the emission line widens, which is due to the pile-up of low-energy and emission-line-energy photons. Lastly, the emission line has a ''shadow'', which is due to the pile-up of two emission-line-energy photons. That is, the true energy of the emission line is 3.0, so there is a small bump in the observed spectrum approximately at 6.0, corresponding to the pile-up of two piled photons produced by the emission line.

![Observed Spectrum](/assets/images/chnnl_marg_pos.png)

# A Tractable Model that fully uses Grade information

A novelty of our model is that it takes into account the dependence of a photon's grade on its energy, yet still has a *tractable likelihood* (it is possible to compute how likely it was that the observed data was produced by a given set of parameters).

When a photon hits the detector's surface, it interacts with the detector material, causing electrons to be released. These electrons produce a *charge cloud*. We refer to one or more photons arriving at the telescope's detector during a single time interval as an *event*.  A *grade* is a label given to an event, which categorizes the shape of its charge cloud.

Other than the recorded event energies, the event grades give important information as to how likely it is that the event consists of two or more photons. There are two ways that grades provide information as to whether an event is piled. The first is that a photon's grade depends on its energy. The second has to do with a phenomenon called *grade migration*. When multiple photons hit the detector in the same time-interval, the charge clouds attributable to the individual photons will combine to form a single charge cloud; the grade of this resulting charge cloud depends stochastically on the grades that would have been assigned to the charge clouds of the individual photons, had these individual photons not interacted. In particular, it's possible that two photons with the same grade will combine to be recorded with an entirely different grade; we call this a case of grade migration.

Davis (2001) and Tamba et al (2022) have worked on spectral models that account for pile-up. Davis (2001) describes a model that has a tractable likelihood, but does not allow for the dependence of a photon's grade on its energy. Tamba et al (2022) account for this dependence, but they do not use a model with a tractable likelihood and instead use a *simulation-based* method.

# Preliminary Results

I won't provide any details on the mathematical underpinnings of the model, but skip straight to comparing the model to a traditional spectral model that does not account for pile-up. To investigate how well our model performs, we can use simulated data for which we know the true spectrum. Here, we simulate data from the spectrum $$\lambda(e) = \alpha_1 \exp\{ -\beta e \} + \alpha_2 N(e; \mu, \sigma^2) $$, using $$\alpha_1 = 1.28, \alpha_2=0.18, \beta=2.00$$ and $$\mu=3.00$$. We then infer the posterior distribution of the parameters using our pile-up model and compare the results to the posterior produced by a spectral model that does not account for pile-up.

The figure below visualises, for a particular simulated data-set, the posterior distribution of our model compared to a traditional model that does not account for pileup. For the model that does not account for pile-up, the posterior (shown in orange) is far away from the true value of the parameters, shown by a dashed black vertical line. For our model, which accounts for pile-up, the posterior (shown in blue) is a lot closer to the true value of the parameters.

![Posterior Distribution](/assets/images/piled_unpiled_dens.png)

As an aside, because the data-generating process is noisy and we have a relatively small amount of data, even for a perfect model, the posterior would not necessarily be centred exactly at the true value of the model's variables. So in analysing how well our model performs, it will help to look at the model's *coverage*. For a model with good coverage, the 95% [credible region](https://en.wikipedia.org/wiki/Credible_interval) contains the true value of the unknown parameter 95% of the time. A 95% credible region is the region within which, according to the posterior distribution, the unknown parameter value falls with a 95% probability. Coverage analysis of the model is ongoing... but initial results are promising!

# Closing Remarks

The aim of this post has been to provide a non-technical introduction to Bayesian modelling of photon pile-up. When the project is complete, I aim to publish details on the mathematics of the model, extensive simulation studies and real-data analyses.

# References

Davis, J. E. (2001). Event Pileup in Charge Coupled Devices. The Astrophysical Journal, 562(1):575.

Tamba, T., Odaka, H., Bamba, A., Murakami, H., Mori, K., Hayashida, K., Terada, Y., Mizuno, T., and Nobukawa, M. (2022). Simulation-based spectral analysis of X-ray CCD data affected by photon pile-up. Publications of the Astronomical Society of Japan, 74(2):364–383.

van Dyk, D. A. and Kang, H. (2004). Highly Structured Models for Spectral Analysis in
High-Energy Astrophysics. Statistical Science, 19(2):275 – 293.