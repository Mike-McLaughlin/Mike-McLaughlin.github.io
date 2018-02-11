---
title: "HEE-The R Way: Intro to Markov Modeling With heemod"
output: html_document
categories: hee_r
---



## Introduction
This is the first post in what I'll start calling the "Health Economics Evaluation: The R Way" series, in which I'll provide tutorials on how to do common health economics tasks using R. The target audience would be people who have a bit of programming and data analysis skills, and want to learn how to use R for some of the common tasks used in health econmics/outcomes research. In each post I'll try to give a little bit of background to the topic, but the major focus will be on how to do the programming. For better references, I'd recommend Neumann, et al's. book, which is sort of a standard reference--[Amazon link](https://www.amazon.com/Cost-Effectiveness-Health-Medicine-Peter-Neumann/dp/0190492937/ref=sr_1_1?ie=UTF8&qid=1517673774&sr=8-1&keywords=costeffectiveness+in+health+and+medicine)-- or the Handbooks in Health Economics series from Oxford Press--[Amazon link](https://www.amazon.com/Cost-effectiveness-Analysis-Healthcare-Handbooks-Evaluation/dp/0199227284/ref=sr_1_5?ie=UTF8&qid=1517673774&sr=8-5&keywords=costeffectiveness+in+health+and+medicine).

This post covers an introduction to using the `heemod` R package to do Markov modeling.

## Markov modeling in the context of technology evaluation
Let's say we have a new therapy for multiple sclerosis (MS). For those who don't know, MS is a degenerative disorder. This means means that over time the disease causes more and more damage, causing the symptoms that patients experience to worsen. Let's also assume the manufacturer of this new therapy (say it's a pill to be concrete...) wants to price the drug such that therapy would cost \$40,000/yr. A lot of money right? An insurer (or in this case a pharmacy benefits manager since it's a drug) might try to negotiate a lower price on the grounds that \$40,000/yr is too much. There might already be an effective drug that costs \$15,000/yr. The manufacturer could counter, saying that because their new drug slows disease progression much better than the old drug and, as a result, the manufacturer says insurers could actually save money because they have to pay for fewer rehabilitative services. You could imagine this sort of simple negotiating going back and forth. Ultimately, what would help here is a model that could help quanitfy the value of this new drug and compare that to the value of the old drug.

In this case, a Markov model might help. Markov models are a mathetmatical tool that, in their simplest and most common form, treat individuals as belonging to a single state in a given point in time. Individuals are then assumed to transition to new states with a probability that only depends on their current state (this is the Markov property that gives these models their names). 

How can these tools be used to assess the value of drugs? Well, let's say that we can treat the severity and symptoms of MS as a series of 3 states. Call them 'low', 'medium', and 'high' disease severity. We can say that an individual stays in each state for exactly 1 year, then transitions to a new state (or stays in their current state) based on a set of probabilities. Let's say the drug affects these transition probabilities. Also, for each year someone is in a given state we can keep a running count of their expected medical expenditures. This would let us say something about that person's anticipated costs. We could also describe the expected utility, or general quality of life, that someone experiences in each state and keep a running count of that as well.

It's probably easiest to understand if we actually get in and start programming a bit, so let's do that.

## Getting started in R
To do Markov modeling we'll be using the R `heemod` package. You'll first need to see if you have the package installed and, if you don't, you'll need to download it from CRAN. To check if you have a package installed you can run `"heemod" %in% rownames(installed.packages()` which will check for the name 'heedmod' in the list of all packages you have installed. If you don't have it installed (i.e., the command above returns `FALSE`), you can run `install.packages("heemod")` which, if you're connected to the internet, will automatically download the package from CRAN. Once you've downloaded the package you can get started by calling `library(heemod)`. So, your script should simply be:


```r
#To check if you have the package installed you could run: "heemod" %in% rownames(installed.packages())
#Otherwise be sure to install the package->install.packages("heemod")
library(heemod)
```

## Defining transition matrices
Once the package is loaded you can start defining states and transition probabilities. Let's again say we have three states: 'low', 'medium' and 'high'. We can define a set of transition probabilities between the states in a table where the row indicates the starting state, and the column indicates the ending state. These probabilities could be based on results of clinical studies or based on expert opinion or some other source. Also, note that we have to specify probabilities, not percentages. Probabilities have to be numbers between 0.0 and 1.0. 

|        | Low  | Medium | High |
|--------|------|--------|------|
| Low    | 0.70 | 0.20   | 0.10 |
| Medium | 0.0  | 0.70   | 0.30 |
| High   | 0.0  | 0.0    | 1.0  |

We'd read this table by looking for someone's current state in the row, and then reading off the probability of ending up in the next state from the columns. So, for example, our table says that someone who is currently 'low' severity has a probability of 0.70 (or, 70% chance) of staying in 'low' severity next year, a probability of 0.20 of transitioning to 'medium' severity, and a probability of 0.10 of transitioning to 'high' severity. Note that the transition probability for someone in a 'high' state is 1.0, which says that someone who is currently severly impacted cannot reduce their disease severity.

Ok, let's say the table above captures the transition probabilities for the old drug. Let's now specify the transition probabilites for the new drug (the one costing \$40,000/yr). Again, these probabilties could be absed on results of a clinical trial or similar:

|        | Low  | Medium | High |
|--------|------|--------|------|
| Low    | 0.80 | 0.15   | 0.05 |
| Medium | 0.0  | 0.85   | 0.15 |
| High   | 0.0  | 0.0    | 1.0  |

If you compare these tables you'll see that the new drug makes it more likely that someone will stay in lower disease severity. For example, with the old drug, someone who is currently 'low' has a 20% chance of transitioning to 'medium'. With the new drug this chance is only 15%.

We can encode these transition probabilities in R by using the `define_transition` function. Here's an example:

```r
trans_old <- define_transition(
  0.70, 0.20, 0.10,
  0.0, 0.70, 0.30,
  0.0, 0.0, 1.0
)

trans_new <- define_transition(
  0.80, 0.15, 0.05,
  0.0, 0.85, 0.15,
  0.0, 0.0, 1.0
)
```

Let's print the matrices:

```r
trans_old
```

```
## A transition matrix, 3 states.
## 
##   A   B   C  
## A 0.7 0.2 0.1
## B     0.7 0.3
## C         1
```


```r
trans_new
```

```
## A transition matrix, 3 states.
## 
##   A   B    C   
## A 0.8 0.15 0.05
## B     0.85 0.15
## C          1
```

We see that the matrix states are generically named 'A', 'B', and 'C'. We can add names to make things easier to follow. To do this, we'll modify the attributes (kind of like meta-data) of our transition matrix objects. [Here's an article](http://www.dummies.com/programming/r/how-to-play-with-attributes-in-r/) to get better understanding of attributes in R.


```r
states <-  c("low", "medium", "high")
attr(trans_old,'state_names') <- states
attr(trans_new,'state_names') <- states

trans_old
```

```
## A transition matrix, 3 states.
## 
##        low medium high
## low    0.7 0.2    0.1 
## medium     0.7    0.3 
## high              1
```

## Visualizing the transitions
The `heemod` package has some nice functions available that use `ggplot2` and other packages to visualize models and results (note: you may need to install these packages. For exmaple, I initially got an error saying the `diagram` package was required for plotting, so I needed to install it). For example, we can visualize our transitions by calling `plot` on our transition matrices. Here's a plot for the transition matrix with the old drug.


```r
plot(trans_old)
```

```
## Loading required namespace: diagram
```

![plot of chunk plot_trans]({{ site.url }}/images/hee-the-r-way-heemod-plot_trans-1.png)

## Adding values to states
The next thing we'll want to do is add value information to our different states. This value information is how we can specify, for example, an individual's costs and health related quality of life. Since we ultimately want to compare drugs, we'll need to specify values for each drug. To do this we'll use a combination of `define_state` to define values for a particular state and `dispatch_strategy` to tell R that we want the value to depend on which 'strategy' (new or old drug) we are using. Our cost information could again come from clinical studies, observational data, etc.. We'll assume that, exlcuding drug costs, if someone spends a year in a 'low' state they incur \$20,000, if they are 'medium' they incur \$40,000, and in a 'high' state they incur \$100,000. We'll also add the drug costs-\$15,000/yr for the old drug and \$40,000/yr for the new drug. 

Note that, by default, `heemod` models want to do cost-effectiveness analysis (something we won't cover in depth yet). For this reason when you define a state you need to specify both a cost aspect *and* and effect aspect. Here we'll just code effect as `1` although in reality this effect would be something like years of life or quality-adjusted life years (QALYs). 

Here's the code:


```r
state_low <- define_state(
    cost = dispatch_strategy(
      new = 20000 + 40000,
      old = 20000 + 15000),
    effect = 1
)

state_med <- define_state(
    cost = dispatch_strategy(
      new = 40000 + 40000,
      old = 40000 + 15000),
    effect = 1
)

state_high <- define_state(
    cost = dispatch_strategy(
      new = 100000 + 40000,
      old = 100000 + 15000),
    effect = 1
    )
```

## Putting states and transitions together to make a model
Finally, we're ready to combine our states and our transition probabilities together. We'll do this by creating a 'strategy' for each of the drugs using `define_strategy`:


```r
strat_old <- define_strategy(
  transition = trans_new,
  low = state_low,
  medium = state_med,
  high = state_high
)

strat_new <- define_strategy(
  transition = trans_new,
  low = state_low,
  medium = state_med,
  high = state_high
)
```

And we can actually run our model by calling `run_model`. In this instance, we'll run the model for a hypothetical population of 500 people all starting at low severity in year 1, and we will run the model for 5 years (that is, at the end of each year we will transition individuals based on the transition matrix). We specify costs and effects as the 'cost' and 'effect' parts in our states defined above.


```r
mod_rslt <- run_model(
  old = strat_old,
  new = strat_new,
  #Run for five years
  cycles = 5,
  #Hypothetical population of 500 identical people, all starting at low severity
  init = c(500, 0, 0),
  cost = cost,
  effect = effect
)
```


```r
summary(mod_rslt)
```

```
## 2 strategies run for 5 cycles.
## 
## Initial state counts:
## 
## low = 500
## medium = 0
## high = 0
## 
## Counting method: 'life-table'.
## 
## Values:
## 
##          cost effect
## old 130028448   2500
## new 192528448   2500
## 
## Efficiency frontier:
## 
## old
## 
## Differences:
## 
##     Cost Diff. Effect Diff. ICER Ref.
## new     125000            0  Inf  old
```

So, we can read from this model that in this hypothetical population of 500 people the total cost over 5 years if they all took the new drug would be \$192,528,448 versus $130,028,448 for the old drug or \$125,000 more per person over five years. So, this model suggests the manufacturer is wrong--it will cost payers more to switch to the new drug.

## Conclusion
This post should give enough to get someone started with building their own Markov models in R. In future posts I'll show how to extend this functionality so models can become more flexible. For example, showing how you can allow probabilities or costs to vary based on individual demographics. I'll also show how this Markov modeling can be integrated with other R functionality, making it faster and easier to execute very complex analyses.
