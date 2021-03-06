---
layout: post
title: Counterfactual Estimation and Optimization of Click Metrics for Search Engines
date: 2015-10-17 17:46:57.000000000 -04:00
type: post
published: true
status: publish
categories:
- Research
tags: []
meta:
  _edit_last: '20775'
author:
  login: etosch
  email: etosch@cns.umass.edu
  display_name: Emma Tosch
  first_name: Emma
  last_name: Tosch
---
So <a href="http://eytan.github.io/">Eytan</a> suggested <a href="http://research.microsoft.com/pubs/210171/paper-tr.pdf">this paper</a> as a reference for our work on static analysis for PlanOut, but I was recently thinking about how some of the ideas might be ported to SurveyMan. 
<!--summary-->
Here is a breakdown of the process/logic of this paper:


1. There are two majors problems for people who study user behavior and search engine foo. First of all, the metrics they are trying to optimize cannot be computed ahead of time -- they depend on user behavior, and this behavior is highly contextual. As an example, they cite relevance judgements as inherently flawed, because they have no notion of context. They describe how annotators will rank celebrity's home pages as highly relevant, but search logs reveal different user behavior. Secondly, to compute the true causal effect, researchers need to know "counterfactual" information -- i.e., what the person would have done under different conditions that did not actually occur. This information is needed in order to make decisions about features: people want to know if a particular feature tweak will lead to more promising behavior, such as clicking on ads. Counterfactual estimation is limited in this context because many features interact, the dimension of the factor tuple may be high, and there may not be sufficient replicates in the data to account for covariates. One solution is to take into account a sample of actual user behavior.
2. Of course, this can all be solved by running randomized experiments. However, experiments are expensive! They take time and money! Can something else be done?
3. When attempting offline estimations, we need a proxy, since we cannot actually measure the things we want to measure! They don't say this in the paper, but presumably there is some kind of bound on the difference between the proxy and the true metric of interest (but if there isn't, it's a really bad proxy!)
4. Solution: use the search log to generate counterfactuals using some kind of model, and run A/B tests on the generated data set.
5. Solution: for the model, use contextual bandits. What's a contextual bandit, you ask? It's a vanilla bandit that uses information from the environment to adapt the weights of the arms. I like to think of it as "JITting" the bandit algorithm, but I'm not sure that analogy actually holds.
6. The formalism is a tuple of observed context and a deferred observed reward, plus an action (I say deferred because, as in the usual bandit framework, the reward is not observed until an action is chosen. So, the action causes a state transition.)
7. The goal is to maximize the expectation of the joint distribution of the environment and actions. Note that this is a different goal from [what I understand of] the traditional bandit framework: we might think of the traditional bandit framework as optimizing the conditional distribution of the action, conditioned on the reward. Here, the users actually control the context (or at least elements of it), so it take this context into account when making choices.
8. Their technique relies on randomly selecting the data (i.e. the tuple of context and reward). One thing that is not clear to me is what the authors do in the face of sparsity -- what happens when they don't cover all the possible tuple values? Their comment that this technique only works when all of the propensity scores are greater than zero implies that this leads to some nasty effects.
9. So it sounds like they:
    1. Mine the search logs for all combinations of contexts and *potential* rewards chosen, assuming they are drawn from some unknown distribution (recall: for any individual, they only have one $$(x, r_a)$$, corresponding to the action actually chosen). Note that even though we can't fill in $$K-1$$ cells of the reward vector, this doesn't matter when sampling the tuple.
    2. Compute the propensity scores for actions. Note that we don't know this! The probability of taking an action is not known. It may not be independent from the context, and there may be high variance in distribution of the actions chosen by all respondents across all actions for a particular context. The authors work backwards to infer propensity scores -- they use the logs to check that the expected value of a possible action's score is equal to the score of complement of the action. Some other methods for estimating propensity scores are also suggested. Note that they basically trying to simulate user behavior. In PlanOut, the propensity scores are typically controlled by the system (although for bandits, there is an external process that needs to be reasoned about).
    3. Use these propensity scores to simulate behavior and compute the reward foo.
10. The rest of the paper describes estimating confidence intervals and gives some real-life examples that use domain expertise to help with calculations.

So what I was thinking for SurveyMan was that this issue of power came up when I gave a talk at <a href="http://cs.union.edu/seminar/tosch.html">Union College</a>. Of course with these potentially very large number of randomizations, statistical power becomes an issue. However, there might be a way to bootstrap some simulations using known data and interpolating the unknown data. I'll need to revisit the interpolation literature, since that's basically what I'd be doing. 
