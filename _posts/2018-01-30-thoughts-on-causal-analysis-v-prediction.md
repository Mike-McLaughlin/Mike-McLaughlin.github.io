---
title: "Causal Analysis Versus Prediction - Some Thoughts"
layout: post
categories: statistics thoughts
---



In my day to day work I'm typically faced with questions of causality. A typical example might be a program owner who asks what will happen if she can use her program to get people more adherent to their medications. This leads to a type of 'what-if' or causal analysis which, in my mind, is (slightly) different than a question of building great predictions. Here are a couple things I think differentiate them:

## 1. Fewer metrics to assess whether you've drawn good causal conclusions.
Think of the basic workflow in building a predictive model -- assemble data, fit a few models, and assess the model's performance using one or a few of many different metrics (R-squared, RMSE, AUC, etc..). While those metrics are certainly useful for determining whether a model meant for causal analysis is good, they aren't sufficient for determining model quality. Beyond or aside from predictive performance, other things need to be considered like what potential confounders are (or aren't) accounted for, or are the standard errors on certain parameters too generous (or, too stingy!). This leads to point #2:

## 2. A greater emphasis on what types of data are included, rather than how much data is included.
With causation, you really need to be careful about what get's included in the model. Without getting too into the weeds, with causation one of (if not your only) goal is to get to a place where you feel comfortable saying that the variable(s) you're interested in are independent of the outcome conditional on other variables. This sounds simple, but it leads to things like conditioning on post-treatment variables, blocking back-door paths, conditioning on causes of treatment versus causes of the outcome, etc..In practice that means a lot of thought needs to be given to what's going in to your model, and how those variables are helping you to satisfy your assumptions. Contrast that with a predictive approach, which would want more independent sources of variation that are predictive of the outcome, regardless of these considerations.

## 3. Interpretability can take precedence to performance
This somewhat relates to point #1, but a concern with many standard predictive tools like forests or neural networks (or Deep Learning) is that it's hard to derive interpretations of what the model is doing which, in turn, makes it hard to draw conclusions about the cause and effect between variables. I tend to think this point is slightly overexaggerated, but it's certainly valid.

##### Conclusion
This certainly isn't an exhaustive list, nor is all this to say that trying to derive causal inferences is worlds apart from trying to make great predictions. Ultimately, a good understanding of cause and effect should be useful for making good predictions and a good predictive model can give insights into potential cuse/effect relationships. Still, analysts and, most importantly, people who consume analytic insights should recognize that differences exist.
