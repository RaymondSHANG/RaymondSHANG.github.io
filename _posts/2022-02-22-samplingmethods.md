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

## Markov Chain
P(x(s+1)|x(0),x(1),...x(s)) = P(x(s+1)|x(s)) for any s, then X(0...) are Markov Chain

Some key concepts:
>>1. <b>Irreducible</b> chain:  it is possible to reach any state from any other state (not necessarily in a single time step)
>>2. <b> Aperiodic</b> chain: A state has period k if, when leaving it, any return to that state requires a multiple of k time steps (k is the greatest common divisor of all the possible return path length). If k==1 for all state, then the chain is aperiodic.  For an irreducible Markov chain, if one state is aperiodic then all states are aperiodic [Easy to proof by contradiction].
>>3. <b>recurrent</b>: A state is transient if, when we leave this state, there is a non-zero probability that we will never return to it. Conversely, a state is recurrent if we know that we will return to that state, in the future, with probability 1 after leaving it (if it is not transient). For a <b>recurrent</b> state,  the <b>expected return time</b> is defined as the mean recurrence time when leaving the state: positive recurrent state (finite expected return time) and null recurrent state (infinite expected return time).
>>4. <span style="background-color: #FFFF00"><b>Ergodic Theorem</b></span>: If {x(1), x(2), . . .} is an irreducible, aperiodic and recurrent Markov chain, then there is a unique probability distribution (<b> stationary distribution of the Markov chain</b>) such that s--> \infty:
>>  a. P(x(s))-->pi(x);
>>  b. (sum{g(x(1....s))})/s -->E[g(x(s))]
>>5. A standard empirical method to assess convergence is to run several independent simulated Markov chains and check that the ratio of inter-chain to intra-chain variances for all the parameters sampled is close to 1

The Markov Chain Property above provide theoretical grantee that MCMC could achieve stationary, and we could approach the related parameters using MCMC simulations.
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


* The sampling process is as shown in Gibbs sampler above
  
## Metropolis
If the conjugate or semiconjugate prior is not available, we could apply Metropolis-Hastings algorithm as a general method. Let's start with Metropolis:
Assuming you have aa(s) in the chain, then you want sample aa(s+1):
* 1. Sample aa(*) ~ J(aa|aa(s)), where J(aa|aa(s)) is any symmetric proposal distribution.
* 2. Compute the acceptance ratio: r = p(aa(\*)|y)/p(aa(s)|y) = {p(y|aa(\*)) p(aa(\*))}/{p(y|aa(s)) p(aa(s))}
* 3. Sample u ~ Unif(0,1); aa(s+1) = ifelse(r>u,aa(\*),aa(s))
  
From the above algorithm, aa(s+1) is only depending on aa(s). So the Metropolis algorithm generates a Markov Chain.

If there are multiple parameters, we could:
1. update a1(s+1) based on p(a1(\*),a2(s)|y)/p(a1(s),a2(s)|y)
2. update a2(s+1) based on p(a1(s+1),a2(\*)|y)/p(a1(s+1),a2(s)|y)

...
## Metropolis-Hastings algorithm
Similar to Metropolis algorithm, change J(aa|aa(s)) to any distribution,and modify the acceptance ratio accordingly:
* 1. Sample aa(*) ~ J(aa|aa(s)), where J(aa|aa(s)) is any proposal distribution.
* 2. Compute the acceptance ratio: r = p(aa(\*)|y)/p(aa(s)|y) * J(aa(s)|aa(\*))/J(aa(\*)|aa(s)) 
* 3. Sample u ~ Unif(0,1) and set aa(s+1) = ifelse(r>u,aa(\*),aa(s))


---
