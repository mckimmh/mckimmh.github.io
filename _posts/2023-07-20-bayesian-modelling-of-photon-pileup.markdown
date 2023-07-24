---
layout: post
title: "Bayesian Modelling of Photon Pile-up"
date: 2023-07-20 09:58:00 +0100
categories: jekyll update
---
I recently presented a poster on ongoing work on Bayesian modelling of photon pile-up at the conference "Statistical Challenges in Modern Astronomy VIII" at Penn State University in the USA.

The aim of the project is to design a Bayesian spectral model that takes into account a phenomenon called pile-up. That is, we aim to infer the parameters of the spectrum of an astronomical source, whilst allowing for the fact that pile-up may have affected the data. The spectrum of an astronomical source, such as a star, is the distribution of energies of photons emitted by the source. The spectrum may be used to learn about properties of the source, such as its chemical composition.

A detector onboard a telescope records the energy of photons arriving at the detector's surface during short time-intervals, usually of a few seconds in duration. Pile-up occurs when two or more photons arrive during the same time-interval, in which case the sum of their energies will be recorded as the energy of a single photon.

For example, in the figure below, the exact arrival times of photons are represented by crosses and time-intervals are demarcated by lines. In the first time-interval, two photons have arrived: the sum of the energies of these photons will be recorded as the energy of a single photon.

![Piled Photons](/assets/images/exact_arrivals_t_intvls.png)

Failing to account for pile-up causes inaccuracies in the spectral analysis. The figure below shows a spectrum consisting of a continuum term and a single emission line. The continuum term describes the spectrum over the entire energy range. The emission line indicates a large amount of emission of photons in a small energy-range, caused by photons being emitted when an electron falls to a lower energy shell of a specific ion. Emission lines can tell us about the chemical composition of a source. Here, the functional form of the spectrum is: $$ \alpha_1 \exp\{ -\beta e \} + \alpha_2 N(e; \mu, \sigma^2) $$, where $$N(e; \mu, \sigma^2)$$ is the probability density function of a normal distribution. We want to infer the parameters $$\boldsymbol{\theta} = (\alpha_1, \alpha_2, \beta, \mu)$$, ($$\sigma$$ is fixed as its true value).

![Spectrum](/assets/images/spec.png)

Simulating photons from this spectrum and simulating the effect of pileup produces the observed spectrum shown below. We see here that there are three effects of pile-up. Firstly, the spectrum hardens in general, which is to say that more high-energy photons are recorded. Secondly, the emission line widens, which is due to the pile-up of low-energy and emission-line-energy photons. Lastly, the emission line has a ''shadow'', which is due to the pileup of two emission-line-energy photons. That is, the true energy of the emission line has energy 3.0, so there is a small bump in the observed spectrum approximately at energy 6.0, corresponding to events consisting of two piled photons produced by the emission line.

![Observed Spectrum](/assets/images/chnnl_marg_pos.png)

By "event", we refer to one or more photons arriving at the telescope's detector during a single time interval. When a photon hits the detector's surface, it interacts with the detector material, causing electrons to be released. These electrons produce a charge cloud. A grade is a label given to an event, which categories the shape of its charge cloud.

Other than the recorded event energies, the event grades give important information as to how likely it is that the event consists of two or more photons. There are two ways that grades provide information as to whether an event is piled. The first is that a photon's grade depends on its energy. The second has to do with a phenomenon called grade migration. When more than one photon hit the detector at the same time, the charge clouds attributable to the individual photons will combine to form a charge cloud; the grade of this resulting charge cloud depends stochastically on the grades that would have been assigned to the charge clouds of the individual photons, had these individual photons not interacted. In particular, it's possible that two photons with the same grade will combine to be recorded with an entirely different grade; we call this a case of grade migration.

A novelty of our model is that it makes use of data on the photons' grades, yet still has a tractable likelihood. The term "tractable likelihood" is a term used in statistical modelling to refer to a model for which it is possible to compute how likely it was that the observed data was produced by a given set of parameters. Davis (2001) describes a model of pile-up that is tractable, but does not make use of grade information. Tamba et al (2022) conduct a spectral analysis that accounts for pile-up and uses grade information, but they do not use a model with a tractable likelihood and instead use a simulation-based method.

# Preliminary Results

I won't provide any details on the mathematical underpinnings of the model, but skip straight to comparing the model to a traditional spectral model that does not account for pileup. To investigate how well our model performs, we can use simulated data for which we know the true values of the parameters. Here, we simulate data using $$\alpha_1 = 1.28, \alpha_2=0.18, \beta=2.00$$ and $$\mu=3.00$$. We then infer the posterior distribution of the parameters using our pile-up model and compare the results to the posterior produced by a spectral model that does not accout for pile-up.

The figure below shows the inferred parameters, when a traditional model that does not account for pile-up is used. The marginal posteriors for $$\alpha_1, \alpha_2$$ and $$\beta$$ are all well-off!

![Inferred Parameters when Pile-up is not accounted for](/assets/images/kde_4d_pileup_not_accounted.png)

By contrast, the figure below shows the inferred parameters when our model, which accounts for pile-up, is used. The mean of the posterior distribution is a lot closer to the true values of the parameters.

![Inferred Parameters when Pile-up is accounted for](/assets/images/kde_4d_pileup_accounted.png)

# Closing Remarks

The aim of this post has been to provide a non-technical introduction to Bayesian modelling of photon pile-up. I'm sorry to have missed out all of the mathematical details, but I hope to publish that information when the project is complete!

# References

Davis, J. E. (2001). Event Pileup in Charge Coupled Devices. The Astrophysical Journal, 562(1):575.

Tamba, T., Odaka, H., Bamba, A., Murakami, H., Mori, K., Hayashida, K., Terada, Y., Mizuno, T., and Nobukawa, M. (2022). Simulation-based spectral analysis of X-ray CCD data affected by photon pile-up. Publications of the Astronomical Society of Japan, 74(2):364–383.