---
layout: post
title: "Bayesian Modelling of Photon Pile-up"
date: 2023-07-20 09:58:00 +0100
categories: jekyll update
---
I recently presented a poster on ongoing work on Bayesian modelling of photon pile-up at Statistical Challenges in Modern Astronomy VIII at Penn State University in the USA.

The aim of the project is to design a Bayesian spectral model that takes into account a phenomenon called pile-up. That is, we aim to infer the parameters of the spectrum of an astronomical source, whilst allowing for the fact that pile-up may have affected the data. A detector onboard a telescope records the energy of photons arriving at the detector's surface during short time-intervals, usually of a few seconds in duration. Pile-up occurs when two or more photons arrive during the same time-interval, in which case the sum of their energies will be recorded as the energy of a single photon.

For example, in the figure below, the exact arrival times of photons are represented by crosses and time-intervals are demarcated by lines. In the first time-interval, two photons have arrived: the sum of the energies of these photons will be recorded as the energy of a single photon.

![Piled Photons](/assets/images/exact_arrivals_t_intvls.png)

Failing to account for pile-up causes inaccuracies in the spectral analysis. The figure below shows a spectrum consisting of a power law and a single line.

![Spectrum](/assets/images/spec.png)

Simulating photons from this spectrum and simulating the effect of pileup produces the observed spectrum shown below. We see here that there are three effects of pile-up. Firstly, the spectrum hardens in general, which is to say that more high-energy photons are recorded. Secondly, the line widens, which is due to the pile-up of low-energy and line-energy photons. Lastly, the line has a ''shadow'', which is due to the pileup of two line-energy photons. That is, the true energy of the line has energy 3.0, so there is a small bump in the observed spectrum approximately at energy 6.0, corresponding to events consisting of two piled photons produced by the line.

![Observed Spectrum](/assets/images/chnnl_marg_pos.png)

To design a spectral model that accounts for pile-up, we assume that there is a maximum number of photons that may arrive in any one time-interval. We observe that as this maximum number of photons per time-interval increases, the posterior of the spectral parameters changes and its coverage improves. A novelty of our model is that it both makes use of data on the photons' grades and has a tractable likelihood. 
