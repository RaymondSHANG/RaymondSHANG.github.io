---
layout: post
title: "Sampling Methods"
subtitle: "MCMC and Beyond"
date: 2022-02-22 15:57:08
header-style: text
catalog: true
author: "Yuan"
tags: [Markov Chain, MCMC, Metropolis–Hastings, Bayesian]
---
> 学而时习之,不亦乐乎
# Introduction
Computer sampling provides revolutional tools to Bayesian statistics, where posterior distribution is not always directly available. In Bayesian philosophy, a parameter always appears as a distribution, which could be approached by sampling and simulation. In other senarios, we might want to estimate a function of that parameter, or a comaprison between two parameters. Simulations are the only way to get the results in these flexibilities. How to sample a parameter and how to effectively sample are the key I want to address here.

To make writings and understandings easier, I assume there are 2 parameters to be estimated, namely a1 and a2. Y is your current data.

# Sampling and Simulation approaches
More specificly, I will dicuss the following topics, from the easy to hard
* Direct scanning the sample space with a likelihood function
* Monte Carlo
* Markov Chain Monte Carlo
* Metropolis
* Metropolis-Hastings

## Direct sample space scanning
In this approach, scan the sample space of (a1,a2), calculate l(x|a1,a2). Repeat this for N times, and normalize the l(x|a1,a2) to get pdf(a1,a2). With this pdf, you could further calculate p(a1),p(a2), and use Monte Carlo to sample other p(f(a1,a2)) values.

## Monte Carlo
* 1.sample (a1,a2) from pdf(a1,a2)
* 2.compute f(a1,a2) <span style="background-color: #FFFF00"> <b> Note: for predictive distributions,sample another time x ~ p(x|a1,a2), and let f(a1,a2) = x to fit this frame. In this case, all f(a1,a2) are iid </b> </span>
* 3.repeat 1 and 2 to get f1(a1,a2),f2(a1,a2),...
* 4.compute mean(f),sd(f),g(f) from samples in 3.

## Markov Chain Monte Carlo
In some cases, we could only get p(a1|a2) and p(a2|a1), but not p(a1,a2). Then, we could apply Gibbs sampler.
Some new terms:
>Semiconjugate prior/conditional conjugate
>> p(a1|a2,y) and P(a1|a2) belong to the same distribution family

>Gibbs sampler
>>1. sample a1(s+1) ~ p(a1|a2(s),Y) 
>>2. sample a2(s+1) ~ p(a2|a1(s+1),Y)
>>3. let aa(s+1) = (a1(s+1),a2(s+1)) be your (s+1)th sample

>>Gibbs sampler generates a dependent sequence of [aa(1),aa(2),...]. However, aa(s) is conditionally independent of aa(1),...aa(s-2) given aa(s-1)

>>This is Markov Property, and the sequence is Markov Chain.

* 1.
---
